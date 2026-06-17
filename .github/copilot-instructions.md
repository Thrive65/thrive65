# Agent Instructions — thrive65

This is a personal website built on [Eleventy Excellent](https://github.com/madrilene/eleventy-excellent) (v4.6.1), an opinionated Eleventy starter using CUBE CSS methodology, design tokens, and Tailwind for utility classes.

---

## Stack

| Layer | Tool |
|---|---|
| Static site generator | [Eleventy](https://www.11ty.dev/) v3 (`@11ty/eleventy`) |
| Template languages | Nunjucks (`.njk`), Markdown, WebC (`.webc`) |
| CSS pipeline | PostCSS → Tailwind CSS v3 → autoprefixer → cssnano |
| JS bundler | esbuild (targets ES2020) |
| CSS methodology | [CUBE CSS](https://cube.fyi/) — Composition, Utility, Block, Exception |
| Design tokens | JSON files in `src/_data/designTokens/` |
| Deployment | Netlify (build output: `dist/`) |
| Node requirement | `>=20.x.x` |

---

## Project structure

```
src/
  _config/           # Eleventy config modules (collections, filters, plugins, shortcodes, events)
    events/          # build-css.js, build-js.js, svg-to-jpeg.js — run before/after Eleventy
    filters/         # Custom Nunjucks filters
    plugins/         # Plugin wrappers (markdown, drafts, html-config)
    setup/           # One-off scripts: favicons, screenshots, color palette generation
    utils/           # clamp-generator.js, tokens-to-tailwind.js
  _data/
    designTokens/    # colors.json, fonts.json, spacing.json, textSizes.json, etc.
    meta.js          # Site metadata, author, blog settings, theme colors
    navigation.js    # Top/bottom nav link arrays
    personal.yaml    # Contact details and platform links (YAML)
    helpers.js       # Global JS data helpers
    github.js        # Fetched GitHub data
    builtwith.json   # Sites built with this starter
  _includes/
    head/            # Nunjucks partials for <head> (CSS inline, JS inline/defer, meta, schema)
    partials/        # Header, footer, etc.
    webc/            # WebC components (custom-card, custom-masonry, YouTube/PeerTube embeds, etc.)
    schemas/         # JSON-LD schema partials
    css/             # Compiled CSS output (generated, do not edit)
    scripts/         # Compiled JS output (generated, do not edit)
  _layouts/          # base.njk, page.njk, post.njk, tags.njk
  assets/
    css/
      global/        # CUBE blocks, compositions, utilities, base styles, global.css entry point
      local/         # Page/component-scoped CSS (forms, pagination, post, styleguide, etc.)
      components/    # Standalone component CSS (output goes to dist/assets/css/components/)
    scripts/
      bundle/        # JS inlined into pages (details, dialog, nav-drawer, theme-toggle, etc.)
      components/    # Standalone component JS (custom-easteregg, custom-masonry)
    fonts/           # Self-hosted web fonts
    images/          # Template images, favicons, OG images
    svg/             # SVG assets (logo at src/assets/svg/misc/logo.svg)
  pages/             # Top-level pages (about, blog, index, styleguide, etc.)
  posts/             # Blog posts organised by year (2022–2025/)
  docs/              # Documentation pages for the starter itself
  common/            # Meta pages: sitemap, RSS feeds, robots.txt, redirects, OG image generator

dist/                # Build output (git-ignored)
eleventy.config.js   # Main Eleventy config — imports from src/_config/*
tailwind.config.js   # Tailwind config — imports design tokens; generates CSS custom properties
```

---

## Key configuration files to know

| File | Purpose |
|---|---|
| `src/_data/meta.js` | Site name, description, author, blog RSS settings, theme colors, pa11y config |
| `src/_data/navigation.js` | Top and bottom nav arrays (`{ text, url }`) |
| `src/_data/personal.yaml` | Email, address, social platform URLs |
| `src/_data/designTokens/*.json` | Single source of truth for colors, spacing, type scale, etc. |
| `tailwind.config.js` | Converts tokens to Tailwind theme; emits CSS custom properties on `:root` |
| `.env` (copy from `.env-sample`) | `URL=http://localhost:8080` — required for local builds |
| `netlify.toml` | Build command, publish dir (`dist`), security headers, cache plugin |
| `vercel.json` | Vercel deployment config (alternate deploy target) |

---

## Development commands

```bash
# Install dependencies
npm install

# Copy env file and set URL
cp .env-sample .env

# Start dev server with hot reload
npm start          # alias for: cross-env ELEVENTY_ENV=development eleventy --serve

# Production build (clean + compile everything)
npm run build      # rimraf dist → eleventy build

# Clean compiled artifacts only
npm run clean

# One-off setup utilities (run manually when needed)
npm run favicons       # Generate favicon set from src/assets/svg/misc/logo.svg
npm run colors         # Regenerate color tokens from colorsBase.json
npm run screenshots    # Generate OG image screenshots

# Accessibility tests
npm run test:a11y      # pa11y-ci against pages defined in meta.js tests.pa11y
```

---

## Build pipeline

CSS and JS are **not** processed by Eleventy itself — they are compiled before the Eleventy build runs via `eleventy.before` event hooks:

1. **CSS** (`src/_config/events/build-css.js`):
   - Entry: `src/assets/css/global/global.css` → `src/_includes/css/global.css`
   - Local CSS: `src/assets/css/local/**/*.css` → `src/_includes/css/<name>.css`
   - Component CSS: `src/assets/css/components/**/*.css` → `dist/assets/css/components/<name>.css`
   - Pipeline: `postcss-import-ext-glob` → `postcss-import` → `tailwindcss` → `autoprefixer` → `cssnano`

2. **JS** (`src/_config/events/build-js.js`):
   - Bundle scripts: `src/assets/scripts/bundle/**/*.js` → `src/_includes/scripts/<name>.js` (inlined into pages)
   - Component scripts: `src/assets/scripts/components/**/*.js` → `dist/assets/scripts/components/<name>.js`
   - Bundler: esbuild (minified, ES2020)

CSS and JS in `src/_includes/css/` and `src/_includes/scripts/` are **generated files** — edit only the source files in `src/assets/`.

---

## Design tokens

All design tokens live in `src/_data/designTokens/*.json`. The `tailwind.config.js` imports them, converts them to Tailwind theme values, and also emits them as CSS custom properties on `:root` (e.g., `--color-pink`, `--space-m`, `--size-step-1`).

Token files:
- `colors.json` — palette (grays, pink, blue, gold) with `prefix: "color"`
- `colorsBase.json` — base color values used by `npm run colors`
- `fonts.json` — font family stacks
- `spacing.json` — fluid spacing scale (clamped)
- `textSizes.json` — fluid type scale (clamped)
- `textLeading.json` — line-height values
- `textWeights.json` — font weights
- `borderRadius.json` — border radius values
- `viewports.json` — breakpoints (`sm`, `md`, `navigation`)

Semantic color custom properties (e.g., `--color-bg`, `--color-text`, `--color-primary`) are defined in `src/assets/css/global/base/variables.css` and switch between light/dark themes.

---

## CUBE CSS layer structure

CSS files follow CUBE methodology:

| Layer | Location | Naming |
|---|---|---|
| **C**omposition | `src/assets/css/global/compositions/` | Layout primitives: `cluster`, `flow`, `grid`, `repel`, `sidebar`, `wrapper` |
| **U**tility | `src/assets/css/global/utilities/` | Single-purpose helpers + Tailwind utilities |
| **B**lock | `src/assets/css/global/blocks/` | Reusable UI components (nav, button, footer, etc.) |
| Local/scoped | `src/assets/css/local/` | Page-specific styles |

Do not add utility classes to Tailwind's `preflight` reset — it is disabled. Instead, the global reset is at `src/assets/css/global/base/reset.css`.

---

## Templates and layouts

- **Nunjucks** is the primary template language. Markdown files also process through Nunjucks (`markdownTemplateEngine: 'njk'`).
- **Layouts** are in `src/_layouts/` with aliases: `base`, `page`, `post`, `tags`.
- **WebC components** live in `src/_includes/webc/` — registered globally.
- **Shortcodes** available in templates:
  - `{% svg "icon-name" %}` — inlines SVG from `src/assets/svg/`
  - `{% image "path", "alt text" %}` — optimized image via eleventy-img (outputs WebP + JPEG)
  - `{% imageKeys %}` — lists available image keys
  - `{% year %}` — current year string
- **Filters** available: `toIsoString`, `formatDate`, `markdownFormat`, `splitlines`, `striptags`, `shuffle`, `alphabetic`, `slugify`

---

## Content authoring

### Blog posts
- Location: `src/posts/<year>/<slug>.md`
- Inherit defaults from `src/posts/posts.json`
- Front matter fields: `title`, `description`, `date`, `tags`, `image` (optional), `draft: true` (hides from build)

### Pages
- Location: `src/pages/<name>.md` or `src/pages/<name>.njk`
- Use layout aliases in front matter: `layout: page`

### Navigation
- Edit `src/_data/navigation.js` to add/remove top (`top[]`) or bottom footer (`bottom[]`) links.

### Platform links
- Edit `src/_data/personal.yaml` for social/platform URLs shown in the footer.

---

## Collections

Defined in `src/_config/collections.js`:
- `allPosts` — all blog posts, sorted by date
- `showInSitemap` — all pages eligible for the XML sitemap
- `tagList` — deduplicated list of all tags used in posts

---

## Image optimization

Handled automatically by `eleventyImageTransformPlugin` — any `<img>` in templates is processed to output WebP + JPEG at original width with `loading="lazy" decoding="async"`.

Manual usage via the `{% image %}` shortcode (`src/_config/shortcodes/image.js`).

OG images are auto-generated for blog posts via `src/common/og-images.njk` and converted to JPEG by the `eleventy.after` event hook (`src/_config/events/svg-to-jpeg.js`).

---

## Code style

- **Prettier** is configured (`.prettierrc`): single quotes, no trailing commas, 110-char print width, `prettier-plugin-jinja-template` for `.njk` files.
- **ES modules** throughout — all `.js` files use `import`/`export` (`"type": "module"` in `package.json`).
- Config is split across `src/_config/` modules; `eleventy.config.js` only wires them together.

---

## Deployment

- **Netlify** (primary): `npm run build` → publishes `dist/`. Cache plugin caches `.cache/` between builds. Security headers set in `netlify.toml`.
- **Vercel** (alternate): config in `vercel.json`.
- Environment variable `URL` must be set in the host to the production domain for correct absolute URL generation (OG images, sitemap, feeds).
