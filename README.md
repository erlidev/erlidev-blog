# Erlidev

Hugo site using the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme. The production URL is configured as `https://erli.xyz/`.

## Requirements

- Git
- Hugo Extended 0.146.0 or newer

Clone the repository with its theme:

```sh
git clone --recurse-submodules <repository-url>
```

If it was cloned without submodules:

```sh
git submodule update --init --recursive
```

## Local authoring

See the [complete local workflow](docs/local-workflow.md) for detailed setup, writing, preview, validation, publishing, and theme-update instructions.

Start the local server, including drafts:

```sh
hugo server --buildDrafts
```

Create a post as a page bundle so its images can live beside it:

```sh
hugo new content posts/my-post/index.md
```

Place images in `content/posts/my-post/` and reference them with a relative path such as `diagram.png`. Set `draft: false` when the post is ready to publish.

Build the production site:

```sh
hugo --gc --minify
```

Generated files are written to `public/` and must not be committed.

## Content organization

- `content/posts/`: articles, benchmarks, news analysis, and essays
- `content/projects/`: project landing pages and announcements
- `categories`: broad grouping, such as `Benchmarks`, `AI News`, `Engineering`, or `Essays`
- `tags`: specific technologies, models, organizations, and topics
- `series`: multi-part article sequences

Every post should have a unique `description`, a concise `summary`, and useful taxonomy values. The archetype at `archetypes/default.md` supplies the standard front matter.

## Cloudflare Pages

Connect the GitHub repository in **Workers & Pages > Create application > Pages > Import an existing Git repository** and use:

| Setting | Value |
|---|---|
| Production branch | `main` |
| Build command | `hugo --gc --minify --cacheDir "$PWD/.cache"` |
| Build output directory | `public` |
| Root directory | `/` |
| Environment variable | `HUGO_VERSION=0.162.1` |

Set `HUGO_VERSION` for both Production and Preview. Preview deployments use their temporary Pages URL; production uses the canonical `baseURL` from `hugo.yaml`.

After the first successful deployment, open the Pages project, choose **Custom domains > Set up a custom domain**, and enter `erli.xyz`. Cloudflare will create or update the required DNS record when the domain is already in the same account.

## Theme updates

PaperMod is a Git submodule. Update it explicitly, test, and commit the new submodule pointer:

```sh
git submodule update --remote --merge themes/PaperMod
hugo --gc --minify
git add themes/PaperMod
git commit -m "chore: update PaperMod"
```
