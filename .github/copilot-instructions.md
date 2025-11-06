## Edge theme – Copilot instructions

This repo is a Ghost CMS theme using Handlebars templates, PostCSS, and a small JS bundle. These notes capture the specifics you’ll need to work productively here.

### Architecture at a glance
- Templates: `default.hbs` (layout shell), `index.hbs` (grid feed), `post.hbs`, `page.hbs`.
- Partials: `partials/loop.hbs` (feed card), `partials/feature-image.hbs`, `partials/related-posts.hbs`, `partials/pswp.hbs`, plus `partials/icons/*`.
- Assets:
  - CSS entry: `assets/css/screen.css` imports shared Ghost styles and theme CSS; output goes to `assets/built/screen.css`.
  - JS entry: `assets/js/main.js`; vendor libs are pulled from `@tryghost/shared-theme-assets` (v1) and optional `assets/js/lib/*.js`; bundle is `assets/built/main.min.js`.
- Feed grid uses Masonry with a sizer: `.post-feed` contains `.grid-item` and a single `.grid-sizer` element. Don’t remove `.grid-sizer`.
- Lightbox uses PhotoSwipe: markup is in `partials/pswp.hbs`; wiring is in `assets/js/main.js` via helpers from shared assets.

### Custom settings and body classes
- Theme settings live in `package.json > config.custom` and are exposed as `@custom.*` in templates.
  - `navigation_layout` toggles body classes in `default.hbs` (`left-logo`, `middle-logo`, `stacked`).
  - `title_font` / `body_font` toggle serif/sans classes (`has-serif-title`, `has-serif-body`).
  - `show_related_posts` and `related_posts_title` control `partials/related-posts.hbs` rendering.
- Image sizes used in templates (`s/m/l/xl`) are defined in `package.json > config.image_sizes`; keep these in sync with `feature-image.hbs` srcset.

### Build, dev, and packaging
- Requirements: Node, Yarn, Gulp. Commands (run at repo root):
  - `yarn` – install
  - `yarn dev` – build CSS/JS, start LiveReload, and watch `*.hbs`, `assets/css/**`, `assets/js/**`
  - `yarn test` – run Ghost Theme Scanner (gscan) against the theme
  - `yarn zip` – build production assets and package `dist/<name>.zip` (zip filename comes from `package.json:name`)
- CSS pipeline (`gulpfile.js`): PostCSS with `postcss-easy-import`, `autoprefixer`, `cssnano`. Don’t edit `assets/built/*` directly; edit files under `assets/css/**`.
- JS pipeline: concatenates shared assets JS (v1), optional `assets/js/lib/*.js`, then `assets/js/main.js`; minifies into `assets/built/main.min.js` with sourcemaps.

### Integration points and helpers
- Shared Ghost assets: `@tryghost/shared-theme-assets` v1 supplies PhotoSwipe, Masonry, imagesLoaded, pagination helpers that `main.js` expects (see `getJsFiles('v1')`). Avoid re-adding these in `assets/js/lib`.
- Search: buttons use `data-ghost-search` (see `default.hbs`); Ghost injects the search behavior—no extra JS required.
- Members (Portal): account/signin/signup links use `data-portal` and are gated by `@site.members_enabled` and `@member` in `default.hbs`.
- Related content: `partials/related-posts.hbs` uses Ghost’s `get` helper to fetch related posts by tag and exclude the current post id.

### Patterns and examples
- Add vendor JS: drop files into `assets/js/lib/` and they will be bundled before `main.js` (ensure no duplicates with shared v1 libs).
- Feed card pattern: `partials/loop.hbs` expects `feature_image` and renders a caption block. Maintain `.grid-item` wrapper for Masonry.
- Responsive images: use `{{img_url feature_image size="m"}}` etc. with sizes that exist in `package.json`.

### Gotchas
- Keep `.grid-sizer` and `.related-title` (used as a Masonry stamp) in feeds—`main.js` relies on them.
- If you change `package.json > name`, the zip output name changes too.
- LiveReload is enabled via `gulp-livereload`; it expects a LiveReload-capable browser or extension.

Key files to start from: `gulpfile.js`, `package.json`, `default.hbs`, `partials/loop.hbs`, `assets/css/screen.css`, `assets/js/main.js`.
