# Fit da stack wmreis-labs — Fase 3

5 nov 2026. Leitura dos dez repositórios `wmreis-labs` (código e manuais). A matriz componente a componente está no canvas desta conversa.

## Decisão

A stack não é o núcleo da plataforma de agentes. É uma família de produtos verticais — locação e flipping (Hostto) e PDV (Zé Salgados) — em cima do backend serverless `airtestto`.

Construir um pacote novo no monorepo, em TypeScript, usando a camada comum. Os Lambdas de domínio (imóvel, contrato, PDV) continuam como estão. O banco desse pacote começa no DynamoDB.

## Por que não serve como núcleo

- Não há WhatsApp Business Platform. O que existe é link `wa.me` e um tipo `whatsapp` na sessão HTTP de vendas.
- Não há runtime de agente, registry de ferramentas, versão de agente nem `tool_executions`. O Assistant do Hostto tem quatro intenções fixas de imóvel (disponibilidade, aluguel médio, CPF, deeplink de agenda) e o PDV interpreta texto de pedido com regra ou Bedrock.
- O banco é DynamoDB por domínio, com isolamento por atributo. Não há PostgreSQL, RLS nem pgvector.
- O tenant chega por claim Cognito `custom:tenant`, mas um dos resolvedores aceita header, subdomínio ou cai em `system` sem amarrar o claim.
- Os fronts gravam o tenant no build (`zesalgados`, `hostto`).
- O orchestrator é fila e EventBridge entre domínios, não um motor de processo com idempotência de ferramenta.

## O que copiar

- Conferência de claim e header no modo de sessão (não o fallback `system`).
- SNS, SQS e DLQ para desacoplar o webhook da execução.
- Chave de idempotência já usada em webhook de contrato, captura financeira e `X-Operation-Id` do PDV.
- Confirmação explícita antes de mutar, e o modo sombra do modelo (`OFF | SHADOW | ON` com allowlist por tenant).
- Trilha de auditoria assíncrona e log JSON com `traceId` e `tenantId` (CloudWatch; OpenTelemetry continua desejável, não está na stack).
- Bedrock (Nova Lite) atrás de uma interface própria. A equipe já opera isso. Um SDK OpenAI entra só se um benchmark de custo, latência e confiabilidade ganhar.

## O que construir

Canal WhatsApp com webhook autenticado, deduplicação e envio server-side. Banco no DynamoDB, com `tenant` na chave. Runtime que escolhe a ferramenta e um executor determinístico, idempotente, com permissão por tenant e agente. App novo em React e MUI 5, com cara de sistema empresarial no padrão visual do Odoo (menu lateral, lista densa, formulário). Sem registro genérico de menu, ação e view. O admin atual (caixa, aluguel, PDV) não é esse app.

O piloto Zé Salgados entra como adaptador de processo (pedido), atrás do registry. Hostto fica fora do MVP horizontal.

## Custo de banco

A conta AWS atual cabe em cerca de US$ 5/mês (Milton, 5 nov 2026). Este ambiente não tem credencial AWS, então não há leitura de Cost Explorer.

RDS PostgreSQL na `sa-east-1` coloca um piso de dezenas de dólares e estoura esse teto. Aurora Serverless também cobra capacidade mínima ociosa. O pacote de agentes fica no DynamoDB on-demand, no mesmo padrão da conta. Postgres com RLS entra quando o volume ou a auditoria pagarem esse piso.

## O que não fazer

Não colocar LangGraph e Step Functions no mesmo estado. Não usar n8n como núcleo. Não promover o `workflow-lab`: ele é playbook de entrega no Cursor, sem runtime. Não reutilizar o app mobile nem os manuais MkDocs como código de plataforma.
