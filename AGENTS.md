# Repository Guide

## Site structure

- This is a Ruby/Jekyll site using the `jekyll-theme-chirpy` gem; theme layouts and includes are not vendored here. Repository overrides are `_config.yml`, `_plugins/`, `_tabs/`, `index.html`, `_posts/`, and `_data/`.
- Add dated posts under `_posts/`; pages in `_tabs/` are the `tabs` collection and are emitted at `/:title/`. Post URLs default to `/posts/:title/` unless a post sets `permalink`.
- `_plugins/posts-lastmod-hook.rb` derives post modification dates from Git history. Do not build from a source tree without `.git` when last-modified metadata matters.
- `assets/lib` is the `chirpy-static-assets` Git submodule. Treat its minified third-party content as external; update it through the submodule rather than editing vendored files.

## Commands

- Install Ruby dependencies with `bundle install` (CI uses Ruby 3.3).
- Run the live-reloading local server with `bash tools/run.sh`; pass `-H 0.0.0.0` only when the server must be externally reachable, or `-p` for production mode.
- Run the production build and local link/content checks with `bash tools/test.sh`. It recreates `_site`, builds with `JEKYLL_ENV=production`, and runs `htmlproofer` with external URLs disabled.
- To test a configuration overlay, use `bash tools/test.sh -c "_config.yml,other-config.yml"`; the script derives the output path from the final non-empty `baseurl`.

## Delivery and formatting

- GitHub Pages deploys only after pushes to `main` or `master`; CI builds and tests before deployment.
- Follow `.editorconfig`: two spaces, LF endings, final newlines; do not trim trailing whitespace in Markdown.
