# ATO — matriz papel × tela × API

7 nov 2026. Fonte: Cognito pool ATO (`AtoAuthStack`), `ato-console` (`AtoGroup` / rotas / menus), [arquitetura MVP](arquitetura-mvp.md), stubs Atendimento e contrato BFF interno. **IDs Cognito sem espaço**; labels PT-BR só na UI.

Ver também o [padrão de auth (convite → aprovação)](auth-approval.md).

## IDs canônicos

| Cognito `groupName` | Label UI (`AtoGroupLabel`) | Escopo |
| --- | --- | --- |
| `AdminPlataforma` | Admin da plataforma | Cross-tenant (lista empresas; troca tenant ativo) |
| `Operador` | Operador | Tenant da sessão (`custom:tenant`) |
| `GestorCliente` | Gestor do cliente | Tenant da sessão |
| `Auditoria` | Auditoria | Tenant da sessão; **read-mostly** |
| `PendingApproval` *(adotar)* | Aguardando aprovação | Autenticado, **sem** acesso de negócio até aprovação |

Claim obrigatório: `custom:tenant` (ex. `miralis`). Header `x-tenant` deve coincidir com o claim (Admin plataforma pode enviar o tenant ativo após switch de empresa).

Alinhamento de strings: front em [ato-console#27](https://github.com/wmreis-labs/ato-console/pull/27); CDK em `airtestto` `ATO_COGNITO_GROUPS` (`AdminPlataforma`, `Operador`, `GestorCliente`, `Auditoria`). **Não** usar labels com espaço como `groupName`.

---

## 1. Papel → menus / rotas (`ato-console`)

Legenda: **✓** acesso de shell · **R** leitura · **W** escrita (quando a tela deixar de ser stub) · **—** sem menu / redirect · **GAP** comportamento atual incompleto.

| Rota / menu | `AdminPlataforma` | `Operador` | `GestorCliente` | `Auditoria` | `PendingApproval` |
| --- | --- | --- | --- | --- | --- |
| `/login` | público | público | público | público | público |
| Home (`/`) | → `/admin/empresas` | → `/operador` | → `/operador` | → `/auditoria/uso` | → tela “aguardando aprovação” *(GAP: inexistente)* |
| **Shell Operador** | | | | | |
| `/operador` Visão geral | ✓ | ✓ | ✓ | ✓ (R) | — |
| `/operador/atendimento` | ✓ W | ✓ W | ✓ W | ✓ R | — |
| `/operador/agenda` | ✓ | ✓ | ✓ | ✓ R | — |
| `/operador/pendencias` | ✓ | ✓ | ✓ | ✓ R | — |
| `/operador/conhecimento` | ✓ | ✓ R | ✓ W *(política)* | ✓ R | — |
| `/operador/implantacao` | ✓ | ✓ | ✓ | ✓ R | — |
| **Shell Admin** (`RequireAdminAccess` → só `AdminPlataforma` hoje) | | | | | |
| `/admin/empresas` | ✓ W | — | — | — | — |
| `/admin/agentes` (+ `/:agentId`) | ✓ W | — | ✓ W *(GAP: UI não expõe)* | — | — |
| `/admin/canal-whatsapp` | ✓ W | — | — *(secrets: só Admin)* | — | — |
| `/admin/integracoes` | ✓ W | — | ✓ R/W tenant *(GAP)* | — | — |
| `/admin/politicas` | ✓ W | — | ✓ W *(GAP)* | — | — |
| `/admin/pessoas` | ✓ W | — | ✓ W tenant *(GAP)* | — | — |
| `/admin/uso` | ✓ R | — | — | ✓ R *(GAP: `AuditLayout` aponta aqui, mas `/admin/*` exige Admin)* | — |
| **Shell Auditoria** | | | | | |
| `/auditoria/uso` | ✓ | — | — | ✓ | — |

Guards atuais (`RequireOperatorAccess` / `RequireAdminAccess` / `RequireAuditAccess`):

- Operador shell: `AdminPlataforma` \| `Operador` \| `GestorCliente` \| `Auditoria`
- Admin shell: **somente** `AdminPlataforma`
- Auditoria shell: `AdminPlataforma` \| `Auditoria`

### GAPs de UI (não confundir com authz de API)

1. **GestorCliente** no front é tratado como Operador (mesmo shell). Política de produto (abaixo) prevê mutações de agentes/políticas/pessoas **no tenant**; falta menu/rotas dedicadas.
2. **`AuditLayout`** navega para `/admin/uso`, bloqueado por `RequireAdminAccess`. Corrigir nav para `/auditoria/uso`.
3. **`PendingApproval`**: grupo ainda não provisionado no `AtoAuthStack`; sem rota “aguardando aprovação”.
4. Telas Admin/Operador (exceto Atendimento) ainda são stubs — menu existe, API ainda não.

---

## 2. Papel → BFF / API (allow / deny)

Base path alvo: `…/ato-console` (contrato Atendimento). Autenticação: `Authorization: Bearer <access token>`. Tenant: claim `custom:tenant` + `x-tenant` coerente. **UI hide ≠ autorização** — toda rota abaixo deve checar JWT groups + tenant no BFF.

Legenda: **A** allow · **D** deny (403) · **R** só GET · **—** fora do papel · **\*** stub/contrato inventado (BFF ainda não publicado) · **GAP** inconsistência conhecida.

### 2.1 Atendimento

| Método | Path | `AdminPlataforma` | `Operador` | `GestorCliente` | `Auditoria` | `PendingApproval` |
| --- | --- | --- | --- | --- | --- | --- |
| GET | `/conversations` | A | A | A | R | D |
| GET | `/conversations/:id` | A | A | A | R | D |
| GET | `/conversations/:id/messages` | A | A | A | R | D |
| POST | `/conversations/:id/handoff` | A | A | A | D | D |
| POST | `/conversations/:id/messages` | A | A | A | D | D |

Escrita só com conversa em `waiting_human` (após handoff), como no mock do console.

**GAP contrato:** rascunhos internos ainda citam grupos legados (`Admin`, `AtoAdmin`, `Gestor`). Canonicalizar para a tabela desta seção.

### 2.2 Console restante (política a partir da arquitetura + menus)

Rotas ainda não implementadas no package `ato-console-api`; usar estes paths como contrato alvo.

| Método | Path | `AdminPlataforma` | `Operador` | `GestorCliente` | `Auditoria` | Notas |
| --- | --- | --- | --- | --- | --- | --- |
| GET | `/tenants` (empresas) | A | D | D | D | Lista cross-tenant |
| GET/PATCH | `/tenants/:id` | A | D | R próprio tenant * | D | \* Gestor: só o próprio `custom:tenant` |
| GET | `/agents` | A | R | A | R | Filtrado por tenant ativo |
| POST/PATCH | `/agents`, `/agents/:id` | A | D | A | D | Publish versão = mutação sensível |
| POST | `/agents/:id/publish` | A | D | A | D | |
| GET | `/channels/whatsapp` | A | D | R | D | Sem token em claro |
| PUT | `/channels/whatsapp` | A | D | D | D | Secrets / WABA: só Admin plataforma |
| GET/PATCH | `/integrations` | A | D | A | R | Escopo tenant |
| GET | `/policies` | A | R | A | R | |
| PUT | `/policies/:agentId` | A | D | A | D | |
| GET | `/knowledge` | A | R | A | R | |
| POST/PATCH/DELETE | `/knowledge…` | A | D | A | D | |
| GET | `/people` | A | R | A | R | Usuários do tenant / convites |
| POST/PATCH | `/people…` | A | D | A | D | Inclui aprovação (sair de `PendingApproval`) |
| GET | `/usage` | A | D | R | A | Contadores por tenant |
| GET | `/audit/events` | A | D | D | A | Read-mostly; sem PII completo |

`AdminPlataforma` em rotas tenant-scoped usa o **tenant ativo** (header) após escolher empresa; sem header válido → 403 (sem fallback `system`).

### 2.3 Webhook Meta / runtime

**Não** fazem parte do console BFF. Channel Gateway autentica HMAC Meta, não JWT Cognito de operador. Operadores não chamam `/ato/webhook`.

---

## 3. Regra: UI hide ≠ authz

1. Esconder item de menu ou bloquear rota React (`Require*Access`) é **UX**, não segurança.
2. O BFF **deve** validar em toda request:
   - JWT válido (authorizer Cognito / session);
   - presença de pelo menos um grupo de negócio **ou** tratamento explícito de `PendingApproval` → 403 em APIs de domínio;
   - `custom:tenant` válido e coerente com `x-tenant`;
   - allowlist de grupos por método/rota (tabelas §2).
3. `Auditoria` = deny em POST/PATCH/PUT/DELETE das rotas de domínio, mesmo que a UI mostre o shell Operador.
4. Tokens WABA e mutações de canal nunca confiam no front “esconder o campo”.

---

## 4. Forma sugerida de middleware compartilhado (airtestto BFF)

Compor sobre `withTenant({ tenantSource: 'session' })` (já rígido) + checagem de grupos. Authorizer deve repassar `cognito:groups` (ou lista normalizada) no context.

```ts
// packages/shared/libs/withAtoAuthz.ts (sugestão)
type AtoGroup =
  | 'AdminPlataforma'
  | 'Operador'
  | 'GestorCliente'
  | 'Auditoria'
  | 'PendingApproval';

type AtoAuthzOptions = {
  /** Grupos que podem chamar o handler (OR). */
  allow: readonly AtoGroup[];
  /** Se true, PendingApproval → 403 mesmo autenticado. Default true. */
  denyPending?: boolean;
};

function groupsFromEvent(event: APIGatewayProxyEvent): string[] {
  const raw =
    event.requestContext.authorizer?.claims?.['cognito:groups'] ??
    event.requestContext.authorizer?.groups ??
    '';
  return Array.isArray(raw) ? raw : String(raw).split(',').map((g) => g.trim()).filter(Boolean);
}

export function withAtoAuthz(handler: TenantHandler, opts: AtoAuthzOptions) {
  return withTenant(async (event, context, tenant, createdBy) => {
    const groups = groupsFromEvent(event);
    if (opts.denyPending !== false && groups.includes('PendingApproval') && groups.length === 1) {
      throw new ForbiddenError('Conta aguardando aprovação');
    }
    const ok = opts.allow.some((g) => groups.includes(g));
    if (!ok) throw new ForbiddenError('Grupo sem permissão para esta rota');
    return handler(event, context, tenant, createdBy);
  }, { tenantSource: 'session' });
}

// Exemplo Atendimento
export const listConversations = withAtoAuthz(listHandler, {
  allow: ['AdminPlataforma', 'Operador', 'GestorCliente', 'Auditoria'],
});
export const handoff = withAtoAuthz(handoffHandler, {
  allow: ['AdminPlataforma', 'Operador', 'GestorCliente'], // Auditoria deny
});
```

Checklist de implementação:

| Item | Status |
| --- | --- |
| `withTenant` session-only no caminho ATO | padrão existente; reusar |
| Propagar `cognito:groups` no session authorizer | **GAP** (hoje context tem tenant/email/sub) |
| `withAtoAuthz` / allowlists por rota | **GAP** (package `ato-console-api`) |
| Constantes compartilhadas = `ATO_COGNITO_GROUPS` / `AtoGroup` | manter sync front ↔ CDK ↔ BFF |
| Testes mismatch tenant + deny Auditoria em POST | backlog authz por grupo |

---

## 5. `PendingApproval` no fluxo compartilhado

Fluxo plataforma ([auth-approval](auth-approval.md)): **convite → pending → aprovação humana → grupo + `custom:tenant`**.

| Fase | Cognito | Console | API |
| --- | --- | --- | --- |
| Convite aceito / 1º login | Usuário existe; grupo só `PendingApproval` (ou nenhum grupo de negócio); `custom:tenant` pode já estar setado pelo convite | Após login, **não** entra em `/operador` nem `/admin`; tela “Aguardando aprovação” | **Deny** em todas as rotas de domínio (403). Opcional: `GET /me` / `GET /people/me` só para status |
| Aprovação (Admin plataforma ou Gestor do tenant) | Remove `PendingApproval`; adiciona `Operador` \| `GestorCliente` \| `Auditoria` (+ tenant se faltava) | Próximo login / refresh de sessão → home do papel | Allow conforme §2 |
| Sem aprovação | Permanece pending | Sem menus de negócio | Sem API de negócio |

Notas:

- Pool ATO hoje: `selfSignUpEnabled: false` — onboarding via admin/convite, alinhado ao fluxo.
- `PendingApproval` **ainda não** está em `ATO_COGNITO_GROUPS`; incluir no CDK ao adotar o fluxo compartilhado Hostto/Zesalgados/ATO.
- Front: filtrar `PendingApproval` em `parseGroups` **ou** tratá-lo explicitamente (hoje `ALL_ATO_GROUPS` só tem os 4 papéis — usuário só-pending parece “sem grupo” e cai em `/login` via `defaultHomePath`). **GAP** a fechar junto com a tela de espera.

---

## Referências

- [Arquitetura MVP](arquitetura-mvp.md), [Decisões MVP](decisoes-mvp.md), [Padrão auth-approval](auth-approval.md)
- Front: `ato-console` — `src/auth/groups.ts`, rotas, layouts, cliente Atendimento
- CDK: `airtestto/packages/auth/cdk/ato-auth-stack.ts` (`ATO_COGNITO_GROUPS`)
- PR alinhamento IDs: [ato-console#27](https://github.com/wmreis-labs/ato-console/pull/27)
