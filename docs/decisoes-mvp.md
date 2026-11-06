# Decisões MVP — ATO (6 nov 2026)

| # | Tema | Decisão |
| --- | --- | --- |
| 1 | Canal | WhatsApp Business Cloud API (Meta) |
| 2 | WhatsApp Web | Não implementar |
| 3 | Número | Do cliente (WABA/número do tenant) |
| 4 | Piloto | Miralis primeiro (único) |
| 5 | LLM | Bedrock Nova Lite (padrão MVP; subir só se eval falhar) |
| 6 | Auth | Cognito: Admin plataforma, Operador, Gestor cliente, Auditoria |
| 7 | Repo | Packages novos em **airtestto**, reuso do compartilhado |
| 8 | Fora do MVP | Billing multi-tenant, marketplace de skills, app do cliente final |

## Nota LLM

Nova Lite é o melhor custo/benefício inicial para intent + tool calling em atendimento/agenda com volume de piloto. Critério de troca: fixtures de conversa Miralis com qualidade insuficiente em PT-BR ou falha sistemática de tools.

## Contexto

Ver também [mapa das 14 telas](telas.md), [front do console](front.md) e o anexo [fit da stack Fase 3](anexos/stack-fit-fase3.md).
