# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Stack

BigCommerce Stencil theme (Cornerstone base) — Handlebars templates, vanilla JS (ES6 modules), SCSS, Webpack 5, Grunt, Jest, ESLint (Airbnb config), Stylelint. Node 20 / npm 10 required.

## Commands

```bash
# Local dev server (via Stencil CLI, not npm scripts)
stencil start

# Bundle for production
npm run build

# Bundle for dev (unminified)
npm run buildDev

# Lint JS + generate SVG sprite
grunt

# Lint SCSS
npm run stylelint
npm run stylelint:fix

# Run tests
npm test
npm run test:watch

# Regenerate SVG sprite only
grunt svgstore
```

> `package-lock.json` must be generated with Node 20 + npm 10. Delete it before installing if using a different version.

## Architecture

### JS — Page-per-module pattern

`assets/js/app.js` is the Webpack entry point. It maps BC page types (e.g. `product`, `cart`, `category`) to dynamic `import()` calls so each page module is code-split into its own chunk.

Each page module lives in `assets/js/theme/<page>.js` and extends `PageManager` (`assets/js/theme/page-manager.js`). Override `onReady()` to run page-specific logic. Access Stencil template context via `this.context` (injected via `{{inject}}` / `{{jsContext}}` helpers in templates).

Global code shared across all pages goes in `assets/js/theme/global/`.

Webpack output lands in `assets/dist/` as `theme-bundle.[name].js` chunks. jQuery is globally provided (no explicit import needed). `$` / `jQuery` / `window.jQuery` are all available.

### Templates — Handlebars via Stencil

- `templates/layout/` — base layout (`base.html` inlines the SVG sprite)
- `templates/pages/` — one `.html` per page type, maps 1:1 to JS page modules
- `templates/components/` — reusable partials

Pass data to JS with `{{inject "key" value}}` then read `JSON.parse({{jsContext}})` in the bundle.

### SCSS

Entry: `assets/scss/theme.scss`. Additional entries for `checkout.scss`, `optimized-checkout.scss`, `invoice.scss`, `maintenance.scss`.

Structure:
- `settings/` — variables (Foundation + custom overrides)
- `components/` — component partials
- `layouts/` — layout-level styles
- `tools/` / `utilities/` — mixins and helpers
- `vendor/` — third-party overrides

### SVG Icons

Individual SVGs in `assets/icons/` → bundled into a single sprite via `grunt svgstore` → inlined in `templates/layout/base.html` via svg-injector. Reference icons as:
```html
<svg><use xlink:href="#icon-<filename>" /></svg>
```

### Config / Schema

- `config.json` — theme settings values (keyed by setting ID)
- `schema.json` — Page Builder UI schema (defines controls merchants see in the BC admin)
- `schemaTranslations.json` — i18n keys for schema labels
- `lang/` — storefront i18n strings

### Polyfills

`assets/js/polyfills.js` loads only when `templates/components/common/polyfill-script.html` detects a missing feature. Main bundle assumes modern browsers.

### Tests

Unit tests in `assets/js/test-unit/`. Jest with jsdom environment. Run a single test file:
```bash
npx jest assets/js/test-unit/<file>.spec.js
```
