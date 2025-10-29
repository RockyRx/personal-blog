# Operational guide for agents working on this blog

## Goals
- Keep the site always deployable on GitHub Pages under `gh-pages` branch.
- Never break homepage listings or taxonomy pages.
- **CRITICAL**: ALWAYS use `gh-pages` branch for deployment. Never use GitHub Actions.

## Repository Structure
- **Single repository**: `/Users/oshadhagunawardena/Projects/personal/personal-blog`
- **Two branches**:
  - `main`: Source code (config.toml, content/, templates/, themes/, etc.)
  - `gh-pages`: Built site (HTML, CSS, JS - what GitHub Pages serves)

## Development Workflow
1. **Always work on `main` branch**:
   - `cd /Users/oshadhagunawardena/Projects/personal/personal-blog`
   - `git checkout main`
   - Make changes to content, templates, config, etc.
   - Test locally: `zola serve` (available at http://127.0.0.1:1111)

2. **Commit changes to `main`**:
   - `git add .`
   - `git commit -m "Your commit message"`
   - `git push origin main`

## Build & Deploy (Manual - ALWAYS USE THIS)
**Never use GitHub Actions. Always deploy manually to `gh-pages` branch.**

1. **Build the site** (from `main` branch):
   ```bash
   cd /Users/oshadhagunawardena/Projects/personal/personal-blog
   git checkout main
   zola build --output-dir public --force
   ```

2. **Deploy to gh-pages**:
   ```bash
   git checkout -B gh-pages
   rsync -av --delete --exclude='.git/' public/ .
   git add -A
   git commit -m "Publish: $(date +%Y-%m-%d)"
   git push -f origin gh-pages
   git checkout main
   ```

3. **One-liner for quick deploy**:
   ```bash
   cd /Users/oshadhagunawardena/Projects/personal/personal-blog && \
   git checkout main && \
   zola build --output-dir public --force && \
   git checkout -B gh-pages && \
   rsync -av --delete --exclude='.git/' public/ . && \
   git add -A && \
   git commit -m "Publish: $(date +%Y-%m-%d)" && \
   git push -f origin gh-pages && \
   git checkout main
   ```

## Content rules
- All posts go under `content/blog/` as Markdown files with TOML front matter.
- Required front matter keys: `title`, `date`, optional: `tags`, `categories`, `draft=false`.
- To hide non-post utility pages (like Tags/Categories) from homepage lists, set:
  - `extra.hide_from_list = true` in their front matter.

## Taxonomies
- Configured in `config.toml`:
  - `taxonomies = [{name = "categories", feed = true}, {name = "tags", feed = true}]`
- Styled list pages are driven by:
  - Template: `templates/taxonomy_list.html`
  - Pages: `content/tags.md` and `content/categories.md`
- Do NOT place raw HTML into `public/` manually; always rebuild with Zola.

## Templates
- Homepage template: `templates/index.html`
  - Filters out pages with `extra.hide_from_list == true` and paginates blog posts.
- Section template: `themes/radion/templates/section.html` (overrides are allowed under `templates/` if needed).

## Do not do
- **NEVER use GitHub Actions** - always deploy manually to `gh-pages` branch.
- Do not edit files directly on `gh-pages` except during the publish step (syncing `public/`).
- Do not delete `.git` in the repository.
- Do not add standalone HTML pages under `public/` by hand.
- Do not keep `themes/radion` as a git submodule; ensure it's regular files (no nested `.git`).
- Do not create or enable `.github/workflows/` - we use manual deployment only.

## Quick checks after changes
- Run `zola build` and confirm it prints `Creating N pages and M sections` without errors.
- Verify locally:
  - Homepage lists recent posts; does not list Tags/Categories.
  - `public/categories/` and `public/tags/` exist and are styled.
  - Random post renders and links work.

## Rollback
- If deployment breaks, force-push the previous working commit in `gh-pages`:
  - `git log --oneline`
  - `git reset --hard <good_sha> && git push -f origin gh-pages`
