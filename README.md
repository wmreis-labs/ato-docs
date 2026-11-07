# ato-docs

Portal público de documentação do produto **ATO** (product discovery, identidade, decisões de MVP e mapa do console).

## Publicação (GitHub Pages)

1. Em **Settings → Pages**, fonte: **GitHub Actions** (`build_type: workflow`).
2. Cada push na branch `main` roda `.github/workflows/pages.yml` (build MkDocs + `actions/deploy-pages`).
3. URL: <https://wmreis-labs.github.io/ato-docs/>

Pré-visualizar: `pip install -r requirements.txt && mkdocs serve`

