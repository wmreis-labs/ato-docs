# Padrão de auth — convite → pending → aprovação

Padrão compartilhado da plataforma WMReis (Hostto, Zé Salgados, ATO): ninguém recebe papel operacional só por se cadastrar. Um admin convida ou o self-signup fica pendente; só depois da aprovação humana o usuário ganha grupo + `custom:tenant`.

Implementação: construct CDK `AuthApproval` em `airtestto` (`packages/auth/cdk/auth-approval.ts`).

Para o mapa papel × tela × API do ATO, ver a [matriz de permissões](matriz-permissoes.md).

## Fluxo

```text
Admin ──invite──► Cognito user (disabled) + grupo PendingApproval
                         │
                         ▼
              e-mail de marca (CustomMessage / SES)
                         │
Admin ──approve──► EnableUser + remove PendingApproval
                   + AdminAddUserToGroup(papel)
                   + custom:tenant
                         │
                         ▼
              usuário pode entrar; API autoriza por grupo+tenant
```

Self-signup (quando o pool permite): `PostConfirmation` desabilita o usuário e coloca em `PendingApproval` até o approve.

## Regras

| Regra | Detalhe |
| --- | --- |
| Autorização | Sempre na **API** (grupo + tenant). UI só reflete. |
| Pending | Grupo Cognito `PendingApproval` (precedence alta). Usuário **disabled** até approve. |
| Approve | `AdminEnableUser` + assign role group + set `custom:tenant` (exceto grupos platform-admin configurados como opcionais). |
| Marca | `appId`: `hostto` \| `zesalgados` \| `ato` — logo/cores/copy via config, sem hard-code de um app. |
| Convite | `AdminCreateUser` (MessageAction SUPPRESS) + e-mail brandado; token custom com TTL fica como extensão futura. |

## Construct API (`AuthApproval`)

```ts
new AuthApproval(this, 'AuthApproval', {
  userPool,                    // pool existente (aditivo)
  appId: 'ato',                // brand
  brandOverrides?: { logoUrl, primaryColor, ... },
  approvableGroups: ['Operador', ...],
  adminGroups: ['AdminPlataforma'],
  tenantOptionalGroups?: ['AdminPlataforma'],
  pendingGroupName?: 'PendingApproval',
  functionNamePrefix: 'AtoAuth-dev',
  stageName: 'dev',
  enableSelfSignupPending?: false,
  sesFromEmail?: 'noreply@...',
  consoleUrl?: 'https://...',
  inviteRequireTenant?: false,
});
```

**Exports:** `inviteUserFn`, `approveUserFn`, `customMessageFn`, `postConfirmationFn?`, `pendingGroup`, `brand`.

### Handlers (payload)

- **Invite** `POST` body: `{ email, tenant?, name? }` → 201, user disabled + `PendingApproval`.
- **Approve** `POST` body (ou `username` no path): `{ username?, groupName, tenant?, sendPasswordReset? }` → enable + role + tenant.

Caller deve estar em `adminGroups` (claim `cognito:groups`).

## Por app

| App | Pool | Self-signup pending | Admin group | Papéis |
| --- | --- | --- | --- | --- |
| ATO | `AtoAuthStack` (já wired) | off (`selfSignUpEnabled=false`) | `AdminPlataforma` | `AdminPlataforma`, `Operador`, `GestorCliente`, `Auditoria` |
| Hostto | `AuthStack` | on (adapter stub) | `Admin` | `Admin`, `Operacional`, `Financeiro`, `Leitor` |
| Zesalgados | a wiring | on (adapter stub) | app-specific | ver `adapters/zesalgados.ts` |

## Migração (Hostto / Zesalgados)

1. Adicionar `AuthApproval` no stack do pool **sem** recriar o pool.
2. Expor Lambdas no API Gateway admin (protegidas).
3. Front: tela “Convites / Pendentes / Aprovar” — fora do escopo deste documento; UIs atuais não precisam ser reescritas agora.
4. SES: opcional. Sem `sesFromEmail`, Cognito segue com delivery default; `CustomMessage` ainda brandiza subject/body quando Cognito envia e-mail.

## Deploy

- Synth/diff OK em qualquer env.
- **Dev only** até validar (aditivo: grupo + Lambdas + trigger CustomMessage).
- **Prod:** não deploy automático — aprovação explícita.

Ver também `packages/auth/README.md` no monorepo `airtestto`.
