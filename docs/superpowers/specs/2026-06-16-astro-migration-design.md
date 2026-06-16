# Astro Migration Design

**Date:** 2026-06-16
**Status:** Approved

## Overview

Migrate `sher.github.io` from old compiled Octopress HTML output to a new Astro blog using the Cactus theme. Deploy via GitHub Actions to GitHub Pages.

## Repository Structure

Scaffold the Cactus template into a temp directory, then copy files into the existing `sher.github.io/` repo to preserve git history.

```
sher.github.io/
  src/
    content/
      blog/
        core-data-automate-master-data-preloading.md
        xcode-development-using-cocoapods.md
        zig.md
        fdb.md
    pages/
    layouts/
  public/
    images/
    favicon.png
    zig.html        <- preserved for embedding
    fdb_menu.html   <- preserved for embedding
  astro.config.mjs
  package.json
  tsconfig.json
  .github/
    workflows/
      deploy.yml
```

Old directories removed: `blog/`, `stylesheets/`, `javascripts/`, `assets/`.

## Content Migration

### 2 old Octopress posts

Source: `blog/2013/05/06/*/index.html`

- Convert HTML body to Markdown using `pandoc`
- Restore frontmatter (`title`, `date`, `tags`) from HTML metadata
- Copy images from `/images/` to `public/images/`, keep same paths

Posts:
- `core-data-automate-master-data-preloading.md` — date: 2013-05-06, tags: [core-data, objective-c, xcode]
- `xcode-development-using-cocoapods.md` — date: 2013-05-06, tags: [xcode, cocoapods, objective-c]

### zig.html and fdb.html

These are generated/hand-crafted HTML with custom syntax highlighting that cannot be cleanly converted to Markdown.

Approach:
- Copy `zig.html` and `fdb_menu.html` to `public/` as-is
- Create `zig.md` and `fdb.md` as blog posts with a short Markdown intro
- Embed the raw HTML via an `<iframe>` or inline `<Fragment>` in MDX pointing to the `public/` copy

## Deployment

- **Source branch**: `master` (Astro source)
- **Deploy branch**: `gh-pages` (compiled `dist/` output)
- **Trigger**: push to `master`
- **Tool**: `withastro/action` GitHub Action
- **Astro config**: `site: 'https://sher.github.io'`, `base: '/'`

## Theme Configuration

Theme: `astro-cactus`

- Site title: "The Cookbook"
- Author: Sher
- RSS: enabled (default)
- Search (pagefind): disabled
- Comments: disabled
- Social links: strip defaults, add manually as needed

No custom CSS or extra components in the initial migration.
