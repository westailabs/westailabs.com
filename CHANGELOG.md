# CHANGELOG — westailabs.com

## [Unreleased] — 2026-11-23

### Added
- Per-page SEO and share cards via `jekyll-seo-tag` (`{% seo %}` in the default layout): canonical, `og:url`, `og:type` (article on research pages), and a single Twitter card. Research `excerpt` is the page description.
- `jekyll-sitemap` (`/sitemap.xml`), `robots.txt` (points at the sitemap), and `jekyll-feed` (`/feed.xml` for the research collection).
- Root `favicon.ico` (16×32, from the existing PNG).
- Homepage research section with three cards, Poisoned Orchestrator first, generated from the research collection.
- Draft outline and test plan for "Grounding is not safety: retrieval poisoning in private RAG" in `_drafts/` (not built, not linked, not published).

### Changed
- Removed hardcoded generic Open Graph / Twitter tags that pinned every page to the homepage URL and site-wide description.
- Research library "Last updated" is derived from the newest `last_updated` in the collection.

## [2026-02-27] — Jekyll migration (historical)

### Added
- Jekyll site structure: `_config.yml`, `_layouts/default.html`
- Nav bar with links: Services, Open Source, About, Research, Blog, Community, Contact
- "Join the Community" section with Discord CTA (https://discord.gg/vtJykN3t)
- Footer with full link bar
- `ARCHITECT-REVIEW-2026-02-27.md` — architecture decisions documented
- `READY-TO-REVIEW.md` — pre-publish checklist

### Changed
- `index.html` converted to Jekyll page with YAML front matter
- Layout extracted to `_layouts/default.html`
- About section moved after Community section in page order
- Contact section now includes Discord Community link

### Preserved (unchanged)
- Charcoal/cyan/violet color scheme (#1e2024 / #22d3ee / #a78bfa)
- All hero, services, open source, about, contact copy verbatim
- GA tag G-FY6CCE6E37
- CNAME file (westailabs.com)
- All assets in /assets/
- Fade-in scroll animation JS
- Mobile hamburger menu
- All Nebulus Stack repo cards
