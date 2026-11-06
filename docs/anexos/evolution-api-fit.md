# Evolution API — fit para ATO (Channel Gateway)

6 nov 2026. Discovery técnico do repositório [evolution-foundation/evolution-api](https://github.com/evolution-foundation/evolution-api) (v2.3.7 no `package.json`; ~10k stars; TypeScript). Fontes: README, `AGENTS.md`, LICENSE, `docker-compose.yaml`, `.env.example`, guards/controllers de canal Meta/Baileys/webhook, schema Prisma, docs oficiais (`docs.evolutionfoundation.com.br`). Contexto ATO: `stack-fit-fase3.md`, `discovery-fase3-brief.md`, identidade ATO.

## Correção (6 nov 2026, noite)

Havia um gap no entendimento do **caminho que o Hub/piloto usa na prática**.

| Caminho | Como entra no WhatsApp | Meta Business / WABA / token Cloud API |
| --- | --- | --- |
| **Evolution Go + whatsmeow** (desenho atual do Hub citado) | WhatsApp Web / multi-device: parear número comum via **QR** (Portal ou API) | **Não usa** — não chama a Cloud API oficial |
| **Evolution API Node “clássica”** | Pode Baileys (Web/QR) **ou** Cloud API (`WHATSAPP-BUSINESS`) | Cloud API só se configurar essa integração |

Ou seja: no fluxo Go + whatsmeow, a Evolution **não** faz o handshake oficial com `developers.facebook.com`. Precisa: API no ar (`.env` com URL/chave/webhook), canal WhatsApp no Hub, pareamento QR, webhook da conta para eventos. É integração **não oficial**: sobe sem Meta Business, mas sujeita a bloqueio/limitação do WhatsApp.

Isso **não muda** o veredito de não colocar Evolution no caminho crítico de produção do ATO; **muda** o porquê: o atrativo real do Hub não é “atalho para Cloud API”, e sim “WhatsApp sem WABA via sessão Web”.

## Veredito

**Take ideas only / avoid as production dependency.** Sidecar só se o piloto **explicitamente** aceitar WhatsApp Web (QR) e risco de ban — nunca como canal oficial Depilk/Miralis/Zé em produção.

Não reutilizar como Channel Gateway do ATO no desenho alvo (Cloud API + Lambda + Dynamo). A Evolution (Node ou Go) continua processo **sempre ligado** + store + multi-instância própria; conflita com tenant Cognito, envio só após workflow e teto ~US$ 5/mês serverless.

## O que a Evolution é

| Aspecto | Fato no repo/docs |
| --- | --- |
| Produto | REST API open-source para WhatsApp e mensageria multi-canal (motor do ecossistema Evolution Foundation) |
| Stack | Node.js 20+, TypeScript 5+, Express, Prisma, Redis, Docker |
| Deploy | Processo long-running / imagem `evoapicloud/evolution-api`; compose com API + Redis + Postgres 15 (+ Manager UI) |
| Provedores WhatsApp | **Primário no Hub citado:** Evolution Go + **whatsmeow** (WhatsApp Web / QR). No Node clássico: **Baileys** (Web/QR) e opcionalmente **Meta Cloud API** (`WHATSAPP-BUSINESS`) — dois mundos distintos |
| Multi-tenant | Isolamento por **instância** (`instanceName` / `instanceId`), não por tenant Cognito |
| Auth | Header `apikey` (chave global ou token da instância) |
| Persistência | PostgreSQL ou MySQL (Prisma); mensagens/contatos/chats/status opcionalmente gravados |
| Eventos out | Webhook HTTP (retry/backoff), WebSocket, RabbitMQ, SQS, Kafka, NATS, Pusher |
| Licença | Apache 2.0 + condições extras (logo/copyright no console; notificação de uso; marca Evolution) |
| Testes | Sem suite formal (`AGENTS.md`) |

Arquitetura (README/`AGENTS.md`): Client → Evolution API → canais (Baileys / Cloud API / Evolution) + chatbots (Typebot, Chatwoot, OpenAI, Dify, …) + eventos + storage S3/MinIO.

## Fit vs necessidades de canal ATO

| Necessidade ATO | Evolution | Fit |
| --- | --- | --- |
| Webhook gateway WhatsApp | Sim: `GET/POST …/webhook/meta`; roteia por `phone_number_id` → instância | Parcial |
| Validação de assinatura Meta (`X-Hub-Signature-256`) | Só `hub.verify_token` no GET; POST em `meta.router.ts` / `meta.controller.ts` **não** valida assinatura HMAC | Fraco |
| Idempotência / dedup por message id | Mensagens persistidas com `key.id`; status update busca por id. Não há contrato claro de dedup at-least-once no edge (depende de DB + handlers) | Fraco / não prova |
| Enviar só após workflow determinístico | Envio REST imediato (`sendMessage`); chatbots internos disparam respostas | Conflito |
| Multi-tenant (WABA/número por tenant, ex. Depilk) | Uma instância por número (`number` = Number ID; `businessId`; token BM). Mapeável a tenant, mas o modelo é “instância Evolution”, não `tenantId` ATO | Adaptável com cola |
| Templates | Sim (`template.router`, `SendTemplateDto`, status de template via webhook Meta) | Bom |
| Status events | Sim (`MESSAGES_UPDATE`, deletes, etc. → webhook/SQS) | Bom |
| Baixo custo AWS / Dynamo serverless | Exige Postgres + Redis + container 24/7; SQS é só fan-out de eventos | Conflito forte |
| TypeScript monorepo / Lambda | TypeScript sim; runtime Express long-lived — **não** Lambda-native | Conflito |

## O que dá para reaproveitar (ideias, não o produto)

1. **Contrato de eventos** — lista de eventos (`MESSAGES_UPSERT`, `MESSAGES_UPDATE`, `CONNECTION_UPDATE`, …) e fan-out para SQS/webhook com retry/backoff (bom modelo mental para o Channel Gateway ATO → SNS/SQS).
2. **Mapeamento Cloud API** — criar “conexão” com Number ID + Business ID + token permanente; webhook único Meta e dispatch por `phone_number_id` (padrão útil para Depilk / multi-WABA).
3. **Superfície de envio** — texto, mídia, template, botões/listas; status de entrega separados do conteúdo.
4. **Isolamento por conexão** — toda operação escopada a uma instância/conexão (análogo a amarrar WABA ao `tenantId`).

Não copiar o monólito, o schema Prisma completo, nem as integrações Chatwoot/Typebot/Dify/OpenAI — isso compete com o Agent Runtime / Conversation Manager do ATO.

## Conflitos e riscos

1. **Custo e ops** — `stack-fit-fase3.md` rejeita piso de RDS/Postgres para o pacote de agentes. Evolution **depende** de Postgres/MySQL + Redis + processo sempre ligado. Estoura o teto ~US$ 5 e foge do padrão airtestto (Dynamo + Lambda).
2. **WhatsApp Web (whatsmeow / Baileys)** — caminho real do Hub Go: QR, número comum, sem WABA. Não-oficial; risco de ban, queda de sessão, limite de multi-device. Para produto B2B (ATO / Depilk) o **alvo de produção** continua **Cloud API oficial**. Aceitar Web só como experimento consciente de piloto.
3. **Assinatura de webhook** — no caminho Cloud API do Node, gap de `X-Hub-Signature-256` nos entrypoints lidos. No caminho Web, a “assinatura Meta” nem se aplica: a confiança fica na Evolution + apikey + webhook do Hub. ATO em Cloud API precisa HMAC no edge.
4. **Controle de envio** — Evolution envia na hora e ainda embute bots. ATO exige: receber → dedup → orquestrar → executar ferramenta com idempotência → **só então** enviar.
5. **Licença / marca / telemetria** — uso comercial Apache 2.0 ok, mas: notificação de uso; logo no console; docs de licensing (ativação/heartbeat a partir de linhas 2.4+; 2.3.x atual do repo ainda sem isso). Fork/remoção de telemetria implica não usar a marca Evolution.
6. **Superfície demais** — manager UI, Chatwoot, N8N, Dify, etc. Aumentam superfície de ataque e drift vs o console ATO (React + MUI 5).
7. **Qualidade** — sem testes formais; serviço Meta/Baileys monolíticos (dezenas/centenas de KB num arquivo).

## O que construir nós mesmos

Manter o plano de `stack-fit-fase3.md` / discovery:

- **Channel Gateway** (Lambda): verify Meta + assinatura HMAC; dedup por `wamid`/message id no Dynamo; normalizar; publicar SNS/SQS; mapear `phone_number_id` → `tenantId`.
- **Send adapter** Cloud API: texto/template/mídia **somente** após Tool/Workflow Executor (idempotency key).
- **Status consumer**: delivery/read/failed → estado da conversa/auditoria.
- **Tenant Manager**: WABA/número/token por tenant (Depilk, …), sem “instância Evolution”.

Não adotar Evolution como sidecar no MVP: o sidecar ainda puxa Postgres/Redis/EC2-Fargate e duplica o gateway. Reavaliar só se um piloto **exigir** Baileys (não recomendado) ou se o custo always-on deixar de ser restrição.

## Matriz de decisão

| Opção | Quando faria sentido | Para ATO agora |
| --- | --- | --- |
| Reuse as-is | Precisar de Baileys + Chatwoot self-hosted rápido, aceitar VPS | Não |
| Adapt as sidecar | Protótipo throwaway; Cloud API-only atrás de Evolution; SQS out → ATO | Não no MVP (custo/ops) |
| Take ideas only | Desenhar gateway próprio alinhado a Meta + AWS | **Sim** |
| Avoid | Dependência de runtime Evolution no caminho crítico | **Sim como dependência** |

## Referências

- Repo: https://github.com/evolution-foundation/evolution-api  
- Docs: https://docs.evolutionfoundation.com.br/evolution-api/  
- Cloud API setup: https://docs.evolutionfoundation.com.br/evolution-api/integrations/cloudapi.md  
- Contexto interno: [`stack-fit-fase3.md`](./stack-fit-fase3.md) (brief interno permanece no Project store)
