# CI/CD, Dependencies & Build

## requirements.txt

```
mkdocs>=1.6
mkdocs-material>=9.5
mkdocs-glightbox>=0.4
```

Pin minimum versions — avoid pinning exact versions unless reproducibility is critical.

## Makefile

```makefile
.PHONY: start build help

start: ## Start mkdocs dev server
	python -m mkdocs serve

build: ## Build mkdocs site
	python -m mkdocs build --strict

help: ## Show help
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-15s\033[0m %s\n", $$1, $$2}'
```

## GitHub Actions Workflow

```yaml
name: Deploy MkDocs to GitHub Pages

on:
  push:
    branches:
      - master

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.x'

      - name: Cache pip dependencies
        uses: actions/cache@v4
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}
          restore-keys: |
            ${{ runner.os }}-pip-

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Build site
        run: mkdocs build --strict

      - name: Copy CNAME to build output
        run: cp CNAME site/CNAME

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./site

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

- Always use `--strict` in CI — fail on broken links and warnings
- Cache pip dependencies by `requirements.txt` hash
- Remove the CNAME step if not using a custom domain
- Enable GitHub Pages in repo settings: Source = "GitHub Actions"
