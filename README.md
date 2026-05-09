# npub

A static site builder for Markdown notes. Reads notes from a local directory, renders them to HTML with syntax highlighting, and outputs a complete static site with tag pages and an Atom feed.

## Prerequisites

- Go 1.25+
- Node.js (for Tailwind CSS)

## Install

```sh
go install github.com/dreikanter/npub/cmd/npub@latest
```

## Build

Install dependencies and build:

```sh
npm install
make build
```

`npm install` is only needed once (or when dependencies change). `make build` compiles the Tailwind CSS stylesheet and builds the `npub` binary.

## Configuration

Run `npub init [dir]` to drop a commented `npub.yml` sample (defaults to the current directory). The sample lists every option with its default and a short description — edit it to suit your site. See [`npub.yml.sample`](./npub.yml.sample) for the full list.

The recommended location is your notes directory itself:

```sh
npub init "$NOTES_PATH"
```

Keeping `npub.yml` next to the notes lets discovery find it automatically (step 2 below) and versions the config alongside the content it describes.

Point npub at a non-default config file with `--config <path>`. CLI flags override YAML values; use `npub config` to preview the merged result.

Config file discovery order:

1. `--config` flag
2. `npub.yml` inside `$NOTES_PATH` (or `--path` value) if it exists
3. `npub.yml` in the current directory

`NOTES_PATH` plays two roles: it is the default source for `notes_path` when neither `--path` nor the YAML sets it, and it acts as a hint location for config discovery (step 2).

### Image assets

Notes can embed images by URL. The first time a build encounters a new URL, npub downloads it into the assets directory under a per-note subfolder (`<assets_path>/<note_uid>/<filename>`) and records the URL→file mapping in `index.json`. Subsequent builds reuse the cached file, so each external host is hit only once per image. The cache survives across builds and is checked into your notes store, which keeps deploys offline-friendly and avoids broken images if the original URL disappears.

The assets directory defaults to `<notes_path>/images`. Override with `assets_path` in the config or `--assets`.

### Static files

Files in the `static` subdirectory of `notes_path` are copied as-is to the build output root. Use this for `CNAME`, `robots.txt`, `favicon.ico`, etc. Override with `static_path` in the config or `--static` flag.

## Usage

### Initialize a project

```sh
npub init "$NOTES_PATH"   # recommended: place npub.yml inside your notes dir
# or create a new project directory
npub init ./my-notes-site
```

### Build the site

```sh
npub build
```

`npub build` writes the site to `<cache_path>/build` (where `cache_path` defaults to `~/.cache/npub/<repo>`). Either `deploy_repo` or `cache_path` must be set; there is no implicit `./dist`. `build` never contacts the deploy remote — all git operations happen in `deploy`.

### Inspect resolved configuration

```sh
npub config
```

The `config` command prints the absolute path of the config file npub will use along with every option after merging YAML, CLI flags, environment variables, and defaults. It accepts the same overrides as `build`, so `npub config --path ~/notes --url https://example.com` previews how a build would see its configuration.

### Serve locally

```sh
npub serve
```

`serve` starts a local HTTP server on `localhost:4000` (override with `--host` and `--port`). It serves `<cache_path>/build`. Pass `--dir <path>` to point it at a different directory; either `deploy_repo` or `--dir` must be set.

### Deploy

```sh
npub build
npub deploy
```

`npub deploy` keeps a bare clone of `deploy_repo` at `~/.cache/npub/<repo>/git` and uses `~/.cache/npub/<repo>/build` as a temporary work-tree (via git's `--git-dir` and `--work-tree` options) when committing. There is no second copy of the site on disk: deploy fetches, points git at the build directory, and runs `add -A` + commit + push. Stale files are removed and changed files updated by the same `add -A` pass. Use `--dry-run` to commit locally without pushing.

### Clear the build output

```sh
npub clear
```

`npub clear` removes only the managed `<cache_path>/build` directory. It does not accept arbitrary paths. Non-empty build output must contain npub's `.npub-build` marker, which `npub build` writes as a deletion guardrail. Use `--dry-run` to validate the target and marker without removing anything.

## Notes format

Notes are Markdown files with YAML frontmatter, managed by [notes](https://github.com/dreikanter/notes). A note is published only when its frontmatter includes `public: true`.

Recognized frontmatter fields:

- `title` — page heading; also the fallback for `slug`.
- `slug` — URL slug. If omitted, npub slugifies `title` and falls back to the numeric note ID.
- `public` — boolean; only `true` notes are published.
- `created_at` — RFC 3339 timestamp; controls feed and index ordering (newest first).
- `tags` — list of strings; each tag gets its own index page.
- `description` — optional meta description for SEO.

Example:

```markdown
---
title: Hello, world
public: true
created_at: 2026-01-15T09:00:00Z
tags:
  - intro
  - golang
---

This is the body of the note in **Markdown**.

![A diagram](https://example.com/diagram.png)
```

Image URLs in the body are downloaded into the assets cache on first build (see [Image assets](#image-assets)).

## Versioning

`CHANGELOG.md` is the source of truth for the version. On PR merge, GitHub
Actions (`.github/workflows/tag.yml`) reads the topmost `## [X.Y.Z]` heading
from `CHANGELOG.md` and pushes `vX.Y.Z` as a git tag. Bump major/minor/patch
by writing the desired heading in the PR.
