# Changelog

## [Unreleased]

### Changed

- Brighten prose links in dark mode for higher contrast.

## [0.2.18] - 2026-05-02

### Added

- Add a dark theme with a navbar toggle that persists in localStorage, paired with the `tokyonight-night` Chroma syntax-highlighting theme ([#93]).

### Changed

- Hide the tags sidebar on mobile, show tags as chips on index/tag pages, highlight the current tag on tag pages, and tighten mobile list spacing while preserving list markers on wider screens ([#90]).

## [0.2.17] - 2026-05-02

### Added

- Add a `generator` meta tag to rendered HTML identifying the npub version used for the build.
- Add `npub clear` to remove only the managed `<cache_path>/build` output directory, guarded by an npub marker file and dangerous-path checks ([#86]).

### Changed

- Rename the cache lock file from `.deploy.lock` to `.npub-cache.lock` and make `npub clear --dry-run` validate marker guardrails ([#87]).
- Remove the `npub build --out` override so builds always target the managed `<cache_path>/build` directory ([#86]).
- Adopt an Unreleased-first changelog and release-PR versioning workflow so multiple PRs can be bundled into one release ([#83]).

### Fixed

- Make `npub build` atomic by rendering into a temporary directory before replacing `<cache_path>/build`, leaving the previous successful build intact if rendering fails ([#88]).
- Fail fast when another `npub deploy` is already using the same cache directory, avoiding concurrent git index mutations ([#84]).

[#93]: https://github.com/dreikanter/npub/pull/93
[#90]: https://github.com/dreikanter/npub/pull/90
[#88]: https://github.com/dreikanter/npub/pull/88
[#87]: https://github.com/dreikanter/npub/pull/87
[#86]: https://github.com/dreikanter/npub/pull/86
[#84]: https://github.com/dreikanter/npub/pull/84
[#83]: https://github.com/dreikanter/npub/pull/83

## [0.2.16]

### Changed

- Pin `github.com/dreikanter/notes` to the released `v0.3.30` tag, replacing the pseudo-version introduced in #81 ([#82]).

[#82]: https://github.com/dreikanter/npub/pull/82

## [0.2.15]

### Changed

- Re-point the `notes` library dependency from `github.com/dreikanter/notesctl` to `github.com/dreikanter/notes` to follow the upstream rename. No behavior change ([#81]).

[#81]: https://github.com/dreikanter/npub/pull/81

## [0.2.14] - 2026-05-01

### Added

- `npub deploy` publishes the site to a git remote configured via the new `deploy_repo` YAML key. `npub build` writes the rendered site to `<cache_path>/build` (offline; no git or network involvement); `npub deploy` keeps a bare clone of `deploy_repo` at `<cache_path>/git` and uses `build/` as a temporary work-tree via `--git-dir` + `--work-tree`, so a single `git add -A` reconciles changed, added, and deleted files against origin's last published state. No second copy of the site is held on disk. Pass `--dry-run` to commit locally without pushing. Errors at every step (missing `git`, malformed `deploy_repo`, clone failure, mismatched origin URL, missing build output, push rejection) surface git's own message rather than a bare exit code. ([#78])
- New `cache_path` YAML key overrides the per-site cache directory. Defaults to `~/.cache/npub/<repo>`. Use it to keep cache state on a different volume or to give multiple `npub.yml` configs distinct cache locations. Honors `~/` and `$VAR` expansion like the other path settings. ([#78])

### Changed

- `build_path` is removed. `npub build` writes to `--out <dir>` if given, otherwise to `<cache_path>/build` (with `cache_path` defaulting to `~/.cache/npub/<repo>`). `npub serve` resolves its target the same way, substituting `--dir` for `--out`. There is no implicit `./dist` fallback any more — one of `deploy_repo` or the per-command flag must be set. `npub config` no longer prints `build_path`; the resolved path appears in `npub build`'s log line. ([#78])

[#78]: https://github.com/dreikanter/npub/pull/78

## [0.2.13] - 2026-04-30

### Changed

- Trim help text across the CLI for brevity. `npub --help` is a one-line `Long` that points at `npub config` for resolution details. `npub build --help` drops the prose paragraph and the three usage examples in favor of a one-sentence note that flags override YAML. `npub config --help` keeps the resolution details (now the only place they appear) but compacts the discovery list and folds the `NOTES_PATH` double-duty paragraph into a single sentence. `npub serve --help` drops its three examples; the `--dir` fallback note is preserved. ([#74])

[#74]: https://github.com/dreikanter/npub/pull/74

## [0.2.12] - 2026-04-30

### Added

- `npub config` prints the absolute path of the resolved config file along with the final value of every option after merging YAML, CLI flags, environment variables, and built-in defaults. Accepts the same overrides as `build` so you can preview how a build would see its configuration. When required fields are missing, the partial configuration is still printed and the command exits with an error. ([#73])

[#73]: https://github.com/dreikanter/npub/pull/73

## [0.2.11] - 2026-04-30

### Changed

- `npub --help`, `npub build --help`, and `npub serve --help` now expose `Long` descriptions covering the flag-over-YAML precedence rule, config discovery order, and the two roles of `NOTES_PATH` (default for `notes_path` and hint location for `npub.yml`). `build` and `serve` also include usage examples. README's "Config file discovery order" section now calls out the discovery role of `NOTES_PATH`. ([#71])

[#71]: https://github.com/dreikanter/npub/pull/71

## [0.2.10] - 2026-04-30

### Fixed

- `npub serve --port` is now declared as an `Int` flag, so cobra rejects non-numeric values up front (e.g. `--port abc`) instead of letting `net.Listen` fall back to `/etc/services` lookup with the opaque `lookup tcp/abc: unknown port`. A `validatePort` range check in `RunE` rejects values outside `1..65535` with a clear pre-bind error. ([#70])

[#70]: https://github.com/dreikanter/npub/pull/70

## [0.2.9] - 2026-04-30

### Changed

- Share config-discovery between `build` and `serve`. `build` calls `loadConfig` (strict); `serve` calls `loadConfigOpt`, a thin wrapper that falls back to `BuildPath: "./dist"` when the config is missing/invalid and `--config` wasn't set explicitly. Guard the internal `--path` lookup with `Flags().Lookup("path") != nil` so future subcommands without that flag don't silently get an empty notes path. ([#69])

[#69]: https://github.com/dreikanter/npub/pull/69

## [0.2.8] - 2026-04-30

### Changed

- Move `--config` from `build` and `serve` to a `rootCmd` persistent flag. The two identical declarations are now a single one, so future subcommands inherit it for free and `--config` shows up under `npub --help`. `cmd.Flags().GetString("config")` already resolves persistent flags, so `RunE` handlers are unchanged. ([#68])

[#68]: https://github.com/dreikanter/npub/pull/68

## [0.2.7] - 2026-04-30

### Fixed

- Path inputs now expand `$VAR` and `${VAR}` uniformly alongside `~/`, regardless of source. Previously env-var expansion was only applied to `--config`, `--dir`, and the `init [dir]` positional, while `--path` and the YAML path fields (`notes_path`, `assets_path`, `static_path`, `build_path`) silently passed `$VAR` through. Factored a single `config.ExpandPath` helper and call it at every boundary. ([#67])

[#67]: https://github.com/dreikanter/npub/pull/67

## [0.2.6] - 2026-04-30

### Fixed

- Drop dead `cfg.BuildPath != ""` guard in `serveCmd`'s config-resolution switch. `config.Load` always defaults `cfg.BuildPath` to `./dist` when the YAML omits it, so the guard never failed. ([#66])

[#66]: https://github.com/dreikanter/npub/pull/66

## [0.2.5] - 2026-04-30

### Added

- `npub serve --host` flag (default `localhost`) to control the bind interface. Previously `serve` always bound on all interfaces; the safer default now only listens on loopback. Pass `--host 0.0.0.0` to expose on the LAN.

### Changed

- `npub init`'s positional argument is now `[dir]` (was `[path]`) for clearer naming and to avoid confusion with `--path` (notes path).

## [0.2.4] - 2026-04-30

### Changed

- `npub serve` once again defaults to the configured `build_path` rather than a notes path. The flag is `--dir` (override the directory to serve), and `--config` selects the config file. Falls back to `./dist` when no config is found; surfaces config errors when a config was explicitly requested.

### Removed

- Drop the `NPUB_CONFIG` environment variable. Use `--config` or rely on the standard discovery order (`npub.yml` inside `$NOTES_PATH`, then in the current directory).

## [0.2.3] - 2026-04-30

### Changed

- Rename `--dir`/`--notes` to `--path` on `npub serve` and `npub build`. Both flags now share the help text `notes path (default: NOTES_PATH)` and resolve from `--path` then `$NOTES_PATH`.
- `npub serve` no longer reads the config or accepts `--config`/`--notes`.
- Suppress cobra's usage dump on command errors so error messages stand alone.

### Fixed

- `npub serve` and `npub build` now fail fast with explicit messages when the notes path is unset, missing, inaccessible, or not a directory, instead of failing later with an opaque error.

## [0.2.2] - 2026-04-30

### Fixed

- `npub serve` now defaults to the configured `build_path` instead of always falling back to `./dist`, so it serves the same directory `npub build` writes to. Pass `--dir` to override.

## [0.2.1] - 2026-04-26

### Added

- Add `npub init [path]` to generate a sample `npub.yml` configuration.
- Add GitHub Actions workflows for tests, linting, and vulnerability scanning.
- Add the embedded nview favicon asset to generated sites.

### Changed

- Switch the notes dependency to `github.com/dreikanter/notesctl`.
- Refactor tests to use `testify` assertions and shared helpers.
- Update Go and Tailwind dependencies to the latest stable versions.
- Update the auto-tag workflow to `actions/checkout@v6`.
- Pin the local lint command to `golangci-lint` v2.11.4.

### Fixed

- Bump the Go patch version to `1.25.9` to resolve standard-library vulnerability findings.
- Address lint findings reported by `golangci-lint`.

## [0.2.0] - 2026-04-25

### Changed

- Rename project, module, and CLI from `notespub` / `notes-pub` to `npub`.

## [0.1.14] - 2026-04-24

### Changed

- Bump CHANGELOG heading to `0.1.14` to resync with existing git tags. The prior auto-patch workflow had advanced tags to `v0.1.13` while `CHANGELOG.md` was seeded at `0.1.7`, so the first run of the CHANGELOG-driven workflow skipped with "Tag v0.1.7 already exists". Picking up one past the highest existing tag restores the invariant that `CHANGELOG.md` leads the tag.

## [0.1.7] - 2026-04-24

### Changed

- `.github/workflows/tag.yml` now tags merged PRs using the topmost `## [X.Y.Z]` heading from `CHANGELOG.md` instead of auto-incrementing the patch off the latest git tag. Bump major/minor/patch by writing the desired heading in the PR.
