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

The `main` branch is protected by a GitHub ruleset named `Protect main`. Production deploys happen from `main`, so publish through pull requests rather than direct pushes.

1. Create a branch.
2. Add or edit Markdown content.
3. Preview locally with `hugo server --buildDrafts`.
4. Open a pull request.
5. Wait for the required `Cloudflare Pages` status check to pass.
6. Review the Cloudflare Pages preview deployment.
7. Merge to `main`.
8. Cloudflare Pages deploys production to `blog.hezebonica.ca`.

Current `main` branch guardrails:

- Pull request required before merging.
- `Cloudflare Pages` status check required.
- Branch deletion blocked.
- Force pushes blocked.
- Branches do not need to be up to date before merging.

## Cloudflare Pages settings

Project settings:

- Production branch: `main`
- Build command: `hugo --gc --minify`
- Build output directory: `public`
- Environment variable: `HUGO_VERSION=0.169.0`
- Custom domain: `blog.hezebonica.ca`

Cloudflare's default Hugo version may be older than PaperMod requires, so keep `HUGO_VERSION` pinned in both production and preview environments.

## Licensing

- Site code/configuration/scripts: MIT License. See `LICENSE`.
- Original content/posts/media: CC BY-NC-ND 4.0. See `CONTENT_LICENSE.md`.
