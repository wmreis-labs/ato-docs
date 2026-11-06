# ato-docs

Portal público de documentação do produto **ATO** (product discovery, identidade, decisões de MVP e mapa do console).

O site é gerado com [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/), no mesmo padrão dos demais `*-docs` da org (`hostto-docs`, `airtestto-docs`, etc.).

## Por que um repositório dedicado?

`airtestto-docs` documenta o **backend airtestto** e os produtos que o consomem. ATO é um produto novo (plataforma de agentes); a org já publica um portal por produto (`hostto-docs`, `admin-docs`, `zesalgados-docs`). Por isso o discovery ATO vive em **`wmreis-labs/ato-docs`**, não como seção do airtestto-docs.

## Pré-visualizar

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```

Abra `http://127.0.0.1:8000`.

## Publicação (GitHub Pages)

1. Em **Settings → Pages**, fonte: **GitHub Actions**.
2. Cada push na branch `main` roda `.github/workflows/pages.yml` (build MkDocs + `deploy-pages`).
3. URL esperada: <https://wmreis-labs.github.io/ato-docs/>

## Conteúdo

| Página | Origem |
| --- | --- |
| Contexto do projeto | `project-context.md` do discovery |
| Identidade visual | Product discovery ATO |
| Decisões MVP | Decisões fechadas em 6 nov 2026 |
| Arquitetura MVP | `arquitetura-mvp.md` completo (6 nov 2026) |
| 14 telas | Operador (6) + Admin (8); Canvas + Figma de referência |
| Front | Notas do console (`ato-console` sugerido) e Amplify |
| Anexos | Fit da stack Fase 3 e Evolution API |

O código do console **não** fica neste repositório — apenas documentação.
