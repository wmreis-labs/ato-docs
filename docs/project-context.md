# Contexto do projeto — ATO

Plataforma de agentes inteligentes (Testto / Milton Reis) para micro e pequenas empresas: automatizar processos, executar tarefas e acompanhar resultados via WhatsApp e console interno.

## Objetivos estáveis

1. **Canal oficial:** WhatsApp Business Cloud API (Meta); número/WABA do cliente; sem WhatsApp Web / Evolution no caminho crítico.
2. **Piloto:** Miralis primeiro (único no MVP).
3. **Arquitetura:** packages novos em `wmreis-labs/airtestto` (TypeScript/Lambda/Dynamo), reuso de shared/auth/message/audit/ai; front em **repo GitHub novo** (React + MUI 5 + Plus Jakarta Sans + marca ATO), Amplify com subdomain e ambientes dev/prod separados.
4. **Runtime:** conversação (Bedrock Nova Lite) separada de execução determinística (tools com permissão e idempotência).
5. **Auth:** Cognito com Admin plataforma, Operador, Gestor cliente, Auditoria; tenant via `custom:tenant`.
6. **Fora do MVP:** billing multi-tenant, marketplace de skills, app do cliente final.

## Documento de arquitetura

Detalhamento de componentes, dados, segurança e riscos: [`arquitetura-mvp.md`](arquitetura-mvp.md) (6 nov 2026).
