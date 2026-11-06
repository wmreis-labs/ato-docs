# Front — console ATO

Notas de produto para o **repositório do console**. Este portal (`ato-docs`) documenta; **não** contém a aplicação.

## Nome do repositório

Sugestão preferencial: **`wmreis-labs/ato-console`**

Alternativa aceitável: `ato-web`

Criar como repo GitHub **novo** na org, separado de Hostto / Admin / Zé Salgados.

## Stack

| Item | Escolha |
| --- | --- |
| Framework | React |
| UI | MUI 5 |
| Fonte | Plus Jakarta Sans |
| Tema | Tokens ATO ([identidade](identidade.md)) — blue `#2563EB` / `#3B82F6`, ciano `#06B6D4`, grafite `#0F172A`, cinza `#64748B` |
| Auth | Cognito (mesmos grupos do MVP) |
| Deploy | AWS Amplify — subdomínio em `ato.testto.com.br` (ou equivalente), ambientes **dev** e **prod** |

## Relação com Hostto

- Usa a stack Hostto apenas como **inspiração** (React + MUI, padrões de shell empresarial).
- **Não** herda dívida técnica dos fronts antigos (tenant no build, shells acoplados, etc.).
- Telas e fluxos: as [14 telas](telas.md) do discovery (rebrand Atende → ATO).

## Escopo deste documento

Planejamento e restrições. Implementação do app fica fora de `ato-docs`.
