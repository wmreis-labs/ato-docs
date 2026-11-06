# Mapa das 14 telas — console ATO

Estrutura e fluxos eleitos no discovery. A pele visual é **ATO** (não Atende). Mock de referência: Figma das 14 telas; implementação de exploração no Canvas.

## Artefatos

| Artefato | Link |
| --- | --- |
| Canvas ATO — console MVP | [Abrir Canvas](https://cursor.com/canvas/store-e933c4ea-dcae-43c5-be9e-dc7378ed0996/7dbf40e4-d47d-40f0-9808-e7cd5c309e1e) |
| Figma (mock eleito, rebrand Atende → ATO) | [Abrir Figma](https://www.figma.com/design/ZnAnHc69565j5cx033A9ZK) |

## Shell

- Sidebar grafite + área de conteúdo clara (tokens ATO)
- Empresa ativa e usuário no shell
- Densidade compacta; tipografia **Plus Jakarta Sans**
- Papéis Cognito alinhados às áreas: Operador vs Admin (e Gestor / Auditoria conforme decisão MVP)

## Operador (6)

| # | Tela | Função |
| --- | --- | --- |
| 1 | Visão geral | Resumo operacional do dia / empresa ativa |
| 2 | Atendimento | Conversas WhatsApp: histórico, status, handoff |
| 3 | Agenda | Grade / slots (staff × horário) |
| 4 | Pendências | Itens que exigem ação humana |
| 5 | Conhecimento | Base consultável pelo time / agente |
| 6 | Implantação | Checklist e status de go-live do tenant |

## Admin (8)

| # | Tela | Função |
| --- | --- | --- |
| 7 | Empresas | Lista multi-tenant: nome, status, plano |
| 8 | Agentes | Lista de agentes da empresa ativa |
| 9 | Agente (detalhe) | Instruções, modelo, ferramentas, publicar versão |
| 10 | Canal WhatsApp | WABA / número do cliente, status da conexão |
| 11 | Integrações | Conexões externas (agenda, ERP, etc.) |
| 12 | Políticas | Limites, confirmação antes de mutar, modos OFF/SHADOW/ON |
| 13 | Pessoas | Usuários e papéis Cognito |
| 14 | Uso e auditoria | Consumo e trilha de auditoria |

## Fluxos-chave

1. **Publicar agente** — Empresas → empresa → Agentes → detalhe → ferramentas → publicar (versão publicada fica só leitura).
2. **Acompanhar conversa** — Atendimento → abrir → histórico + execuções de ferramenta.
3. **Transferir para humano** — Na conversa, ação Transferir; o agente para de responder aquele fio.
4. **Trocar empresa** — Shell troca a empresa ativa; listas passam a filtrar por ela.

## Fora desta leva

Construtor visual de agentes, marketplace de skills, app do cliente final, WhatsApp embutido no console, billing multi-tenant, relatórios avançados.
