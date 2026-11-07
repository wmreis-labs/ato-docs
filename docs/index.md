# ATO — documentação de product discovery

**ATO** é a plataforma de agentes inteligentes que automatizam processos, executam tarefas e acompanham resultados.

> Inteligência que faz acontecer.

Domínio de referência: `ato.testto.com.br`

## Neste portal

| Documento | O que cobre |
| --- | --- |
| [Contexto do projeto](project-context.md) | Objetivos estáveis do ATO |
| [Identidade visual](identidade.md) | Marca, paleta, tipografia, tokens de UI |
| [Decisões MVP](decisoes-mvp.md) | Canal, piloto, LLM, auth, escopo |
| [Arquitetura MVP](arquitetura-mvp.md) | Componentes, dados, segurança, pacotes |
| [Matriz de permissões](matriz-permissoes.md) | Papel × tela × API; IDs Cognito; UI ≠ authz |
| [Auth (aprovação)](auth-approval.md) | Convite → pending → aprovação; construct compartilhado |
| [Mapa das 14 telas](telas.md) | Operador (6) + Admin (8) |
| [Front (console)](front.md) | Repo sugerido, stack React/MUI, Amplify |
| [Anexos](anexos/index.md) | Fit da stack Fase 3 e Evolution API |

## Artefatos de design

- **Canvas (14 telas ATO):** [ATO — console MVP](https://cursor.com/canvas/store-e933c4ea-dcae-43c5-be9e-dc7378ed0996/7dbf40e4-d47d-40f0-9808-e7cd5c309e1e)
- **Figma de referência:** mock eleito das 14 telas (rebrand Atende → ATO)

## Escolha de repositório

Este site vive em **`wmreis-labs/ato-docs`**, alinhado ao padrão da org (um `*-docs` por produto).
