# hezebonica-blog

Git-backed Hugo blog for `blog.hezebonica.ca`.

## Stack

- Static site generator: [Hugo](https://gohugo.io/)
- Theme: [PaperMod](https://github.com/adityatelange/hugo-PaperMod)
- Git host: GitHub
- Hosting: Cloudflare Pages
- Production branch: `main`

## Local development

Clone with submodules:

```bash
git clone --recurse-submodules https://github.com/ezebunandu/hezebonica-blog.git
cd hezebonica-blog
```

If you already cloned without submodules:

```bash
git submodule update --init --recursive
```

Install Hugo, then run a local preview:

```bash
hugo server --buildDrafts
```

Open the local URL Hugo prints, usually:

```text
http://localhost:1313/
```

Create a new post:

```bash
hugo new content posts/my-post-title.md
```

Set `draft = false` when the post is ready to publish.

## Publishing workflow

1. Create a branch.
2. Add or edit Markdown content.
3. Preview locally with `hugo server --buildDrafts`.
4. Open a pull request.
5. Review the Cloudflare Pages preview deployment.
6. Merge to `main`.
7. Cloudflare Pages deploys production to `blog.hezebonica.ca`.

## Cloudflare Pages settings

Intended project settings:

- Production branch: `main`
- Build command: `hugo --gc --minify`
- Build output directory: `public`
- Hugo version: set `HUGO_VERSION` in Cloudflare Pages if the default is too old

## Licensing

- Site code/configuration/scripts: MIT License. See `LICENSE`.
- Original content/posts/media: CC BY-NC-ND 4.0. See `CONTENT_LICENSE.md`.
