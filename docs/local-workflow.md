# Local Hugo workflow

This guide covers the complete local workflow for Erlidev: cloning the repository, writing and previewing content, validating production output, publishing through Git and Cloudflare Pages, and updating PaperMod.

## 1. Clone the repository

On a new computer, clone the repository and its PaperMod submodule:

```sh
git clone --recurse-submodules git@github.com:USERNAME/REPOSITORY.git
cd REPOSITORY
```

PaperMod is a Git submodule. The Erlidev repository records a particular PaperMod commit rather than copying the theme into its own history. The `--recurse-submodules` option checks out that recorded theme commit.

If the repository was cloned without its submodules, initialize them afterward:

```sh
git submodule update --init --recursive
```

Confirm that Hugo and PaperMod are available:

```sh
hugo version
git submodule status
```

The project currently targets Hugo Extended 0.162.1. The Extended edition includes asset-processing features that themes may require.

## 2. Start the development server

From the repository root, run:

```sh
hugo server --buildDrafts
```

The shorthand form is:

```sh
hugo server -D
```

Hugo prints a local address, normally `http://localhost:1313/`. Open it in a browser.

The development server:

- Reads `hugo.yaml`.
- Loads PaperMod from `themes/PaperMod`.
- Converts Markdown under `content/` into HTML.
- Watches content, configuration, layouts, and assets for changes.
- Rebuilds the site and refreshes the browser after most changes.
- Includes draft pages because `--buildDrafts` was supplied.

Stop the server with `Ctrl+C`.

The development server normally builds the site in memory. It is for local previewing, not production hosting.

### Test from another local device

To test on a phone or another computer on the same network:

```sh
hugo server --buildDrafts --bind 0.0.0.0 --baseURL http://YOUR-LAN-IP:1313
```

This exposes the development server to the local network. Do not expose it directly to the public internet.

## 3. Create a post

Create posts as page bundles:

```sh
hugo new content posts/my-first-post/index.md
```

This creates:

```text
content/
└── posts/
    └── my-first-post/
        └── index.md
```

A page bundle gives each post its own directory. Images, downloads, benchmark data, and other post-specific resources can live beside `index.md`:

```text
content/posts/my-first-post/
├── index.md
├── benchmark-results.png
├── architecture.svg
└── results.csv
```

Hugo creates the new file from `archetypes/default.md`.

## 4. Configure the front matter

A new post begins with YAML metadata:

```yaml
---
title: "My First Post"
date: 2026-08-04T12:00:00-06:00
draft: true
description: ""
summary: ""
categories: []
tags: []
series: []
showToc: true
TocOpen: false
cover:
  image: ""
  alt: ""
  caption: ""
  relative: true
---
```

The fields have the following purposes:

- `title`: Browser title, page heading, feed title, and social metadata.
- `date`: Publication date as an ISO 8601 timestamp with a timezone.
- `draft`: Excludes the page from normal production builds when `true`.
- `description`: Search-engine and social metadata. Use a specific one- or two-sentence description.
- `summary`: Short text shown in post listings.
- `categories`: Broad editorial classifications.
- `tags`: Specific subjects, models, tools, or organizations.
- `series`: Groups related multi-part posts.
- `showToc`: Enables the table of contents for the post.
- `TocOpen`: Controls whether the table of contents starts expanded.
- `cover`: Configures the PaperMod cover image.
- `cover.relative`: Resolves the cover image relative to the post bundle.

Example front matter for an LLM benchmark:

```yaml
---
title: "Benchmarking Model A Against Model B"
date: 2026-08-04T14:30:00-06:00
draft: true
description: "A reproducible comparison of Model A and Model B across coding, reasoning, and retrieval tasks."
summary: "Model A leads on coding accuracy, while Model B performs better on retrieval-heavy tasks."
categories:
  - Benchmarks
tags:
  - LLM
  - Model A
  - Model B
  - Evaluation
series:
  - LLM Benchmarks
showToc: true
TocOpen: true
cover:
  image: "benchmark-results.png"
  alt: "Bar chart comparing Model A and Model B benchmark scores"
  caption: "Aggregate scores across the evaluation suite."
  relative: true
---
```

Use a small, consistent set of broad categories. A practical starting set is `Benchmarks`, `AI News`, `Engineering`, `Projects`, and `Essays`. Tags can be more specific.

## 5. Write the Markdown body

The article follows the closing `---` delimiter:

```markdown
This benchmark compares Model A and Model B under identical conditions.

## Methodology

The evaluation used three task groups:

1. Code generation
2. Multi-step reasoning
3. Document retrieval

## Results

![Benchmark results](benchmark-results.png)

The complete measurements are available in
[CSV format](results.csv).
```

Because the assets reside inside the page bundle, relative references work locally and in production.

### Code blocks

Use fenced code blocks with a language identifier:

````markdown
```python
def calculate_score(correct: int, total: int) -> float:
    return correct / total
```
````

Hugo applies syntax highlighting and line numbers. PaperMod adds a copy button.

### Headings

Start body sections at heading level two:

```markdown
## Main section

### Subsection
```

The post title is already rendered as the level-one heading. A second level-one heading creates an incorrect document hierarchy.

### Raw HTML

Raw HTML in Markdown is disabled in the current configuration. Use Markdown or a supported Hugo shortcode where possible.

## 6. Preview drafts

Keep the development server running while writing:

```sh
hugo server --buildDrafts
```

Review the following before publishing:

- Desktop and narrow mobile layouts
- Light and dark themes
- Table of contents
- Code highlighting
- Internal and external links
- Images and alternative text
- Search results
- Category, tag, and series pages
- Description and summary quality

Draft pages appear locally because of `--buildDrafts`. Cloudflare's normal production build excludes them.

To preview using the production environment without drafts:

```sh
hugo server --environment production
```

Add `--buildDrafts` if a production-mode preview must also include unfinished pages.

## 7. Validate the production build

Run:

```sh
hugo --gc --minify
```

The options mean:

- `--gc`: Removes unused generated resources from Hugo's resource cache.
- `--minify`: Compresses generated HTML, CSS, XML, JSON, and related output.

Hugo writes the completed site to `public/`. Do not edit anything in that directory because Hugo regenerates it. The directory is excluded from Git.

Inspect repository state and formatting errors:

```sh
git status --short
git diff --check
git diff
```

Generated `public/`, `.hugo_build.lock`, and cache files should not appear in `git status`.

The current build may report PaperMod deprecation warnings related to renamed Hugo language properties. These originate in the theme and do not prevent the build.

## 8. Publish a post

Set the post to publishable:

```yaml
draft: false
```

Build again:

```sh
hugo --gc --minify
```

Stage only the source content:

```sh
git add content/posts/my-first-post
```

Commit and push it:

```sh
git commit -m "publish benchmark of Model A and Model B"
git push
```

Cloudflare Pages detects a push to `main`, checks out the repository and PaperMod submodule, runs Hugo, and publishes the generated `public/` directory.

## 9. Manage unfinished work

There are two normal approaches.

### Keep a draft on `main`

Leave `draft: true`. The source can be committed and pushed, but Hugo excludes it from the production site. If the repository is public, the unpublished Markdown remains publicly visible on GitHub.

### Work on a branch

Create a branch:

```sh
git switch -c post/model-comparison
```

Write and commit the draft:

```sh
git add content/posts/model-comparison
git commit -m "draft model comparison"
git push -u origin post/model-comparison
```

Cloudflare Pages can create a preview deployment for the branch or pull request. Merge the branch into `main` when the post is ready. This approach provides review history and a hosted preview before publication.

## 10. Update an existing post

Edit its `index.md`, preview it, and rebuild:

```sh
hugo server --buildDrafts
hugo --gc --minify
```

Then commit the source change:

```sh
git add content/posts/my-first-post
git commit -m "fix methodology in model comparison"
git push
```

Hugo regenerates the page, RSS feed, taxonomy pages, search index, and sitemap automatically.

Avoid changing a published post's directory name without preserving the old URL. Otherwise, existing external links will break. Add an alias to the post's front matter when a URL must change:

```yaml
aliases:
  - /posts/old-post-name/
```

## 11. Create a project page

Project pages belong under `content/projects/`:

```sh
hugo new content projects/project-name/index.md
```

Example metadata:

```yaml
---
title: "Project Name"
date: 2026-08-04T14:30:00-06:00
draft: true
description: "A concise explanation of what the project does."
summary: "A short project-listing description."
categories:
  - Projects
tags:
  - Python
  - Open Source
---
```

Use the page body for the problem statement, screenshots, features, installation instructions, repository and demo links, current status, and roadmap.

## 12. Edit permanent pages

The permanent pages are:

- `content/about.md`: Biography and contact information.
- `content/archives.md`: PaperMod archive layout configuration.
- `content/search.md`: PaperMod search layout configuration.
- `content/posts/_index.md`: Description of the posts section.
- `content/projects/_index.md`: Description of the projects section.

Files named `_index.md` describe an entire section rather than an individual content page. The archive and search pages use PaperMod-specific layouts and generally do not require body content.

## 13. Change site-wide settings

Edit `hugo.yaml`. Important settings include:

- `title`: Site name.
- `baseURL`: Canonical production address.
- `params.description`: Default search and social description.
- `params.homeInfoParams`: Home-page introduction.
- `params.socialIcons`: Social profile and RSS links.
- `menu.main`: Navigation entries.
- `params.ShowToc`: Default table-of-contents behavior.
- `params.ShowShareButtons`: Sharing controls.
- `params.defaultTheme`: `auto`, `light`, or `dark`.

Restart `hugo server` after configuration changes if it does not reload automatically.

## 14. Update PaperMod

PaperMod is pinned to a specific Git commit and does not update automatically. Fetch and update it explicitly:

```sh
git submodule update --remote --merge themes/PaperMod
```

Review the new theme revision:

```sh
git diff --submodule
```

Test both the development site and production build:

```sh
hugo server --buildDrafts
hugo --gc --minify
```

After testing, commit the new submodule pointer:

```sh
git add themes/PaperMod
git commit -m "chore: update PaperMod"
git push
```

Theme updates can change markup, styling, or configuration behavior. Test and commit them separately from content changes so regressions are easy to identify and revert.
