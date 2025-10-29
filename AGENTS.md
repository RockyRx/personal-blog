# Operational guide for agents working on this blog

## Goals
- Keep the site always deployable on GitHub Pages under `gh-pages` branch.
- Never break homepage listings or taxonomy pages.

## Source vs Deploy
- Source lives in: `/Users/oshadhagunawardena/Projects/personal/personal_blog`
- Deployed site lives in: `/Users/oshadhagunawardena/Projects/personal/personal-blog` on branch `gh-pages`

## Build & deploy (manual)
1. Build:
   - `cd personal/personal_blog`
   - `zola build --output-dir public --force`
2. Publish to gh-pages:
   - `cd ../personal/personal-blog`
   - Ensure repo is initialized: `git init && git remote add origin git@github.com:RockyRx/personal-blog.git || true`
   - `git checkout -B gh-pages`
   - Sync artifacts: `rsync -av --delete ../personal_blog/public/ .`
   - `git add -A && git commit -m "Publish" && git push -f origin gh-pages`

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
  - Filters out pages with `extra.hide_from_list == true`.
- Section template: `themes/radion/templates/section.html` (overrides are allowed under `templates/` if needed).

## Do not do
- Do not edit files directly in `personal-blog` (gh-pages) except during the rsync-based publish.
- Do not delete `.git` in `personal-blog`.
- Do not add standalone HTML pages under `public/` by hand.

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
