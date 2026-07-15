# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is Ishrak Hayet's personal academic website, built on the [al-folio](https://github.com/alshedivat/al-folio) Jekyll theme. It is a static site (no application code/backend) — almost all work here is editing Markdown/YAML content, Jekyll front matter, Liquid templates, and SCSS, not writing logic.

## Commands

### Local development (Ruby/Jekyll, native)

```bash
bundle install               # install gem dependencies
bundle exec jekyll serve     # serve locally at http://localhost:4000 with live reload
bundle exec jekyll build     # build static site into _site/
```

### Local development (Docker, matches CI environment more closely)

```bash
docker compose pull && docker compose up      # serve at http://localhost:8080
docker compose -f docker-compose-slim.yml up  # smaller image, same functionality
```

A `.devcontainer/devcontainer.json` is also available for VS Code Dev Containers; on attach it runs `./bin/entry_point.sh`, which starts `jekyll serve` and auto-restarts it whenever `_config.yml` changes.

### Formatting

```bash
npx prettier . --check   # check formatting (matches .github/workflows/prettier.yml CI check)
npx prettier . --write   # fix formatting
```

Formatting is governed by `.prettierrc` (uses `@shopify/prettier-plugin-liquid` for `.liquid` files, printWidth 150) and `.prettierignore` (excludes minified assets, generated search JS, and one legacy post).

### CI build check

`bin/cibuild` just runs `bundle exec jekyll build` — this is what GitHub Actions uses to verify the site builds.

There is no test suite (no unit/integration tests) — correctness is verified by building the site and checking rendered output, plus the Prettier/link-checker/Lighthouse/Axe CI workflows described below.

## Deployment

Two independent mechanisms exist; **this repo uses GitHub Actions on push to `main`** (see `.github/workflows/jekyll.yml` / `deploy.yml`), which builds with `bundle exec jekyll build --baseurl "..."` and deploys via the native GitHub Pages Actions flow (no `gh-pages` branch involved).

`bin/deploy` is the older manual alternative: it builds the site, runs `purgecss`, and force-pushes the built output to a `gh-pages` branch. Do not run this unless explicitly asked — it force-pushes and rewrites branch history.

## Architecture / content structure

This is a data-and-content-driven Jekyll site. To make a change, first figure out which of these it belongs in rather than hunting through templates:

- `_config.yml` — the single source of truth for site-wide settings: identity (`first_name`/`last_name`), theme, analytics, Jekyll Scholar (bibliography) behavior, collections definitions, etc. Changes here require a rebuild/restart of `jekyll serve` to take effect (unlike content files).
- `_data/*.yml` — structured content consumed by templates:
  - `socials.yml` — social links/handles (email, GitHub, LinkedIn, Google Scholar ID, etc.)
  - `cv.yml` — CV content, used only as a **fallback** when `assets/json/resume.json` (jsonresume.org format) is absent
  - `repositories.yml` — GitHub users/repos shown in the repositories section
  - `coauthors.yml`, `venues.yml` — used by the bibliography/publication rendering
- `_bibliography/papers.bib` — publications in BibTeX, rendered via `jekyll-scholar`. Author highlighting matches on `scholar.last_name`/`scholar.first_name` in `_config.yml`. Extra per-paper fields (`pdf`, `arxiv`, `code`, `website`, `abstract`, etc.) are supported directly in the `.bib` entry.
- `_pages/` — top-level site pages (front matter `layout` + `permalink` control routing/rendering). `_pages/about.md` is the home page (`permalink: /`).
- `_posts/` — blog posts, filename format `YYYY-MM-DD-title.md` (standard Jekyll).
- `_news/` — short announcements shown on the about/home page; front matter `inline: true` renders in place, otherwise it links to a full page.
- `_projects/` — portfolio/project entries rendered on the projects page grid (`importance`/`category` front matter control ordering/grouping).
- `_layouts/` and `_includes/` — Liquid templates; `_layouts` are chosen via a page's `layout:` front matter, `_includes` are reusable partials referenced with `{% include %}`.
- `_sass/` — theme SCSS, notably `_variables.scss` (available theme colors) and `_themes.scss` (`--global-theme-color` and other active theme variables).
- `_plugins/` — custom Ruby Jekyll plugins (e.g. `google-scholar-citations.rb`, `inspirehep-citations.rb`, `external-posts.rb`) that extend build-time behavior beyond stock Jekyll/Liquid.
- `assets/` — static assets (images, JS, CSS, PDFs, bibliography-linked files, etc.).

`news` and `projects` are Jekyll *collections* (declared in `_config.yml`); adding a new collection means declaring it there, creating the matching folder, and adding a landing page similar to `_pages/projects.md`.

## Conventions specific to this fork

- All edits should be made on `main`; nothing should be hand-edited on `gh-pages` (GitHub Actions owns that output).
- `url` in `_config.yml` is set to `http://localhost:4000` for local dev — the production value (`https://ihayet.github.io`) is commented out alongside it; don't "fix" this without checking whether it's intentional for local serving.
