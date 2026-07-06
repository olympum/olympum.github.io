# Bruno's Journal

Personal blog of Bruno Fernández-Ruiz — *CTO by week, composer by weekend,
triathlete by obsession.* Built with [Hugo](https://gohugo.io/) and deployed to
GitHub Pages at [www.olympum.com](https://www.olympum.com/).

Migrated from Jekyll to Hugo with a custom editorial design; the site uses its
own layouts in `layouts/` rather than an external theme.

## Prerequisites

[Hugo (extended)](https://gohugo.io/installation/), v0.163.3 or newer to match
the CI build:

```
brew install hugo
```

## Local development

```
hugo server -D        # serve at http://localhost:1313 (-D includes drafts)
```

Build the production site into `public/`:

```
hugo --gc --minify
```

## Writing

Posts live in `content/posts/` as Markdown with front matter. Create a new
draft:

```
hugo new content posts/my-post-title.md
```

The archetype (`archetypes/default.md`) sets `draft = true`; set it to `false`
(or remove it) to publish.

## Project layout

```
content/          Markdown content
  posts/          Blog posts
  about.md        About page
layouts/          Custom HTML templates (Editorial Noir design; no theme)
static/           Images and static assets (img/, robots.txt)
archetypes/       Front-matter templates for `hugo new`
hugo.toml         Site configuration
public/           Generated output (git-ignored)
```

## Deployment

Every push to `main`/`master` runs `.github/workflows/hugo.yml`, which builds
with Hugo and publishes to GitHub Pages. The custom domain is configured via the
`CNAME` file (www.olympum.com).
