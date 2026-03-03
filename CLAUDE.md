# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository

GitHub Pages site for `hubreb.github.io`. Hosted at `https://hubreb.github.io/`.
Uses the Beautiful Jekyll remote theme (`daattali/beautiful-jekyll@6.0.1`).

## Remote

- Origin: `git@github.com:HubReb/hubreb.github.io.git`
- Default branch: `main`
- Deploys via GitHub Pages from the `main` branch

## Local Development

```bash
bundle install
bundle exec jekyll serve
```

Then visit `http://localhost:4000`.

## Adding a New Blog Post

Create a file in `_posts/` named `YYYY-MM-DD-title.md` with front matter:

```yaml
---
layout: post
title: Post Title
subtitle: Optional subtitle
tags: [tag1, tag2]
---
```

## Site Structure

- `_config.yml` — Site configuration and theme settings
- `index.html` — Homepage (blog feed)
- `aboutme.md` — About page
- `projects.md` — Projects showcase
- `_posts/` — Blog posts
- `404.html` — Custom error page
