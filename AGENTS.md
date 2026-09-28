---
title: Repository Guide
description: Project-specific guidance for AI coding agents working on this Jekyll site
---

## Site structure

* This is a Ruby/Jekyll site using `jekyll-theme-chirpy` `~> 7.6`; theme layouts and includes are supplied by the gem, not vendored here. Repository override surfaces are `_config.yml`, `_plugins/`, `_tabs/`, `_data/`, `assets/css/jekyll-theme-chirpy.scss`, `index.html`, and site content.
* Add dated posts under `_posts/YYYY-MM-DD-slug.md`. Post URLs default to `/posts/:title/`; legacy posts can have public URL-preserving `permalink` values. Do not change existing permalinks casually.
* New posts normally use `author: Joe`, `description`, lowercase inline `tags`, and inline or block `categories`. The default configuration supplies `layout: post`, and Jekyll can derive the date from the filename. Optional established fields include `excerpt`, site-root-relative `image`, and `pin: true`.
* Quote post front matter titles containing `:` so YAML parses them as strings.
* Use `{% post_url 2026-08-26-hdhomerun-web %}` for new links to posts. Preserve existing root-relative media and tab links unless intentionally making the site base-URL compatible.
* Pages in `_tabs/` are the sorted `tabs` collection and are emitted at `/:title/`; authored pages normally set `icon`, `order`, `author`, and `title`. Archive, category, and tag tabs use dedicated layouts.
* `_data/authors.yml`, `_data/contact.yml`, and `_data/share.yml` are theme-consumed configuration. Media commonly lives in `assets/post-content/` or `assets/wp-content/uploads/`.
* `_plugins/posts-lastmod-hook.rb` derives post modification dates from Git history. Do not build from a source tree without `.git` when last-modified metadata matters.
* `assets/lib` is the `chirpy-static-assets` Git submodule. Treat its minified third-party content as external; update it through the submodule rather than editing vendored files.
* `_site/` is generated output, ignored by Git, and deleted and recreated by the test script. Never edit it directly.

## Commands

* Install Ruby dependencies with `bundle install`; production builds use Ruby 3.3.
* Run the live-reloading local server with `bash tools/run.sh`; pass `-H 0.0.0.0` only when the server must be externally reachable, or `-p` for production mode.
* Run the production build and local link/content checks with `bash tools/test.sh`. It recreates `_site`, builds with `JEKYLL_ENV=production`, and runs `htmlproofer` with external URLs disabled.
* To test a configuration overlay, use `bash tools/test.sh -c "_config.yml,other-config.yml"`; the script derives the output path from the final non-empty `baseurl`.

## Delivery and formatting

* Cloudflare Pages builds and hosts the site. Set `RUBY_VERSION=3.3`, `JEKYLL_ENV=production`, and `BUNDLE_WITHOUT=development:test`; use `git fetch --unshallow && git submodule update --init --recursive && bundle exec jekyll build` with output directory `_site`. The Git history is required for post modification dates; `--unshallow` assumes Cloudflare's shallow checkout.
* The former GitHub Pages workflow is preserved in `.github/workflows-archive/pages-deploy.yml`, outside the active `.github/workflows/` directory. It no longer deploys the site; local link/content checks remain available through `bash tools/test.sh`.
* Production enables local static assets and PWA caching. Keep the `assets/lib` submodule initialized in the Cloudflare build.
* Follow `.editorconfig`: UTF-8, two spaces, LF endings, and final newlines. Do not trim trailing whitespace in Markdown. Prefer double quotes in YAML and single quotes in JavaScript, CSS, and SCSS.
* VS Code formats Markdown, Liquid/HTML, and shell files with repository-configured extensions; `*.html` is associated with Liquid.

## Documentation

* `README.md` is the upstream Chirpy Starter overview, not repository-specific operational documentation. Use this guide and the scripts in `tools/` for repository workflows.
