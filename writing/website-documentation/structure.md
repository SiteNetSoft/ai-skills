# Project Structure

## Directory Layout

```
project-root/
├── docs/
│   ├── assets/           # Logo, favicon, extra.css
│   ├── overrides/        # Jinja2 template overrides (extend base.html)
│   ├── index.md          # Homepage
│   └── <section>/        # One directory per nav section
│       ├── overview.md
│       └── <topic>.md
├── .github/workflows/
│   └── deploy.yml        # CI/CD for GitHub Pages
├── mkdocs.yml            # Main configuration
├── requirements.txt      # Python dependencies
├── Makefile              # Dev commands (start, build)
├── CNAME                 # Custom domain (if applicable)
└── README.md             # Quick setup for contributors
```

## Key Rules

- Keep all source content in `docs/` — never edit `site/` (generated output)
- One Markdown file per topic — no mega-files
- Group related pages into subdirectories matching nav sections
- Put static assets (images, CSS, SVGs) in `docs/assets/`
- Add `site/` to `.gitignore`

## .gitignore

```gitignore
site/
.venv/
__pycache__/
*.pyc
```

## File Naming

- Lowercase, hyphen-separated: `api-gateway.md`, `getting-started.md`
- Match the conceptual hierarchy: `application-cache-redis.md` for nested topics
- Keep names short but descriptive
