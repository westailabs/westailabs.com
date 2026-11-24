# AGENTS.md — westailabs.com

## Project overview

- Source for the public West AI Labs website, [westailabs.com](https://westailabs.com): the landing page plus long-form research articles.
- Jekyll site deployed to GitHub Pages. Public repo: [westailabs/westailabs.com](https://github.com/westailabs/westailabs.com). Code is MIT (`LICENSE`); articles, page copy, and images are CC BY 4.0 (`LICENSE-CONTENT`). Logos and the West AI Labs name are trademarks, not licensed. Keep the footer license line in `_layouts/default.html` in sync.

## Setup and commands

Requires Ruby 3.2 and Bundler (CI uses `ruby/setup-ruby` with Ruby 3.2).

```bash
bundle install                                # install
bundle exec jekyll serve --livereload         # dev server on http://127.0.0.1:4000
JEKYLL_ENV=production bundle exec jekyll build  # production build into _site/
```

## Build, test, and lint

- The site must build cleanly with `bundle exec jekyll build` before work is called done.
- CI: `.github/workflows/deploy.yml` builds on pushes to `develop` and `main` and on PRs. Only `main` deploys to GitHub Pages.
- Check new pages in a browser at desktop and mobile widths. Keep internal links relative.

## Layout

| Path | Purpose |
|------|---------|
| `index.html` | Landing page (front matter + content) |
| `_layouts/` | `default`, `page`, and `research` layouts |
| `_research/` | Research articles (collection, output at `/research/:title/`) |
| `research/index.html` | Research listing page |
| `assets/` | CSS, images, JS |
| `_config.yml` | Site config, collections, build excludes |
| `CNAME` | Custom domain. Do not change |

## Content style

- Plain English. Cut jargon, keep claims sourced, and link sources inline.
- New research articles go in `_research/` with front matter (`title`, `date`, `description`), and get a `CHANGELOG.md` entry.
- Preserve the color scheme (`#1e2024` / `#22d3ee` / `#a78bfa`), the analytics tag in `_config.yml`, and the mobile menu.
- GitHub Pages plugin whitelist only (currently `jekyll-seo-tag`, `jekyll-sitemap`, and `jekyll-feed`). Default layout uses `{% seo %}` — do not hard-code `og:url` or duplicate Twitter/OG tags. New research pages need `title`, `date`, `last_updated`, and `excerpt` (excerpt is the share-card description).

## Git workflow

- `develop` is integration; `main` is what is live. Merging to `main` deploys the site.
- Branch off `develop` (`feat/`, `fix/`, `docs/`, `chore/`), conventional commits, merge back with `--no-ff` or squash.
- Never commit straight to `main`. Keep branches local until Jason approves a push. Do not push or deploy unless asked.

## Security and privacy

- Never commit secrets, tokens, or `.env` files. This repo is public.
- No proprietary employer data or internal identifiers in articles. Pattern-level discussion only.
- Do not publish personal or family details.
