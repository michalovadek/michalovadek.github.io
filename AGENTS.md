# AGENTS.md — michalovadek.github.io

Working notes for any agent (human or AI) touching this repo. This file is the
single source of truth for conventions; if the code disagrees with this file,
update one or the other so they match.

## 1. What this repo is

A **Quarto** website served at `https://michalovadek.github.io/`. It consolidates
what used to live as four standalone R Markdown sites (see §9), plus the existing
static landing/publications/tools pages. Most pages re-render **nightly via
GitHub Actions** from live data sources.

Design principles (set by owner, do not silently violate):

1. **Sustainable, not wasteful** — one nightly render covers all pages; we don't
   duplicate fetch/render logic per page.
2. **Graceful degradation** — if an upstream API fails, the page must still
   render using the last good cached snapshot, with a visible "data as of <date>"
   stamp. Never show silently stale content.
3. **Prefer simple code** — small composable R scripts in `R/`, qmd pages call
   them. Avoid clever metaprogramming.
4. **One unified look** — shared SCSS theme + shared ggplot theme + shared table
   style. Per-page custom CSS is a smell.
5. **Respect GitHub limits** — see §8.

## 2. Toolchain (verified on this laptop, 2026-05-23)

| Tool    | Version           | Notes                                                  |
|---------|-------------------|--------------------------------------------------------|
| R       | 4.4.3 (2025-02-28)| `C:\Program Files\R\R-4.4.3\bin\Rscript.exe` (not on PATH) |
| Quarto  | 1.7.17            | `quarto` on PATH                                       |
| uv      | 0.10.7            | Python is **only** used through uv in this repo        |
| git     | 2.40.0.windows.1  |                                                        |

R is the **primary** language. Use Python only when there's no good R option,
and always via uv (`uv run python ...`, `uv add <pkg>`). Never `pip install`
into a system Python.

R packages are pinned via **renv**. After adding a `library()` call, run
`renv::snapshot()` so the CI install matches local.

## 3. Repo layout (target)

```
michalovadek.github.io/
├── _quarto.yml              # site config (nav, theme, output, freeze)
├── index.qmd                # landing (replaces index.html)
├── publications.qmd         # (replaces publications.html)
├── tools.qmd                # link hub (kept; trackers move into this site)
├── trackers/
│   ├── eu-vetoes.qmd        # from eu-veto-tracker
│   ├── eu-court.qmd         # from eucourt
│   ├── eu-law.qmd           # from eulaw
│   └── eu-finance.qmd       # from eufinance
├── R/                       # shared R helpers (theme, table style, fetchers, cache)
│   ├── theme.R              # ggplot theme + ggiraph defaults + colour palette
│   ├── table.R              # shared DT / gt wrapper
│   ├── fetch_*.R            # one per upstream source; idempotent, cache-aware
│   └── freshness.R          # "as of <date>" badge helpers
├── data-manual/             # owner-curated CSVs (committed)
├── data-apis/               # nightly snapshots of API pulls (committed if small, else gitignored + LFS-considered)
├── data-external/           # data from sister projects (mirrored, see §4)
├── _site/                   # rendered HTML (GITIGNORED)
├── _freeze/                 # Quarto freeze (GITIGNORED)
├── .quarto/                 # Quarto cache (GITIGNORED)
├── assets/
│   ├── style.scss           # site-wide style
│   ├── pic.png              # profile photo
│   └── fonts/               # if we self-host fonts
├── pyproject.toml           # uv-managed (created by `uv init`)
├── uv.lock                  # committed
├── renv.lock                # committed
├── renv/                    # only `activate.R` and `settings.json` committed
└── .github/workflows/
    └── render.yml           # nightly cron + manual dispatch
```

The existing `index.html`, `publications.html`, `tools.html`, `style.css` will be
deleted once their qmd equivalents render correctly. Until then they coexist.

## 4. Data conventions

Three folders, no overlap:

- **`data-manual/`** — entered/edited by hand by the owner. Treat as authoritative.
  Read-only from R scripts. Example: `veto_data.csv`.
- **`data-apis/`** — written by `R/fetch_*.R` scripts during the nightly run.
  Each fetcher writes a *timestamped snapshot* (e.g. `eurlex_acts_2026-05-23.parquet`)
  plus updates a `latest.parquet` symlink-equivalent. If the API errors, the
  fetcher must exit non-zero **without** touching `latest`, so the next render
  falls back to the previous snapshot.
- **`data-external/`** — copies/mirrors of data from other repos under
  `C:\Users\uctqova\Documents\github\`. Each file in here must have a sibling
  `<file>.source.txt` recording the source repo, commit hash, and date pulled.

Every fetcher writes a metadata sidecar `<file>.meta.json` containing at minimum
`{ "fetched_at": ISO8601, "source": "...", "rows": N }`. The freshness badge
on each page reads this.

**Storage format default: parquet** (via `arrow`). Falls back to CSV only for
files we expect humans to edit. Parquet is ~5× smaller than CSV for typical
tracker data and avoids the >50 MB warnings.

## 5. Code conventions

- R style: tidyverse-ish, no `magrittr` (use base `|>`). Snake_case names. No
  side effects at the top of source files.
- Every R helper in `R/` must be **idempotent and pure** (input args → return
  value or file path); printing, plotting, and table rendering happens in the
  .qmd file.
- No `setwd()`. Paths resolve from project root via `here::here()`.
- Python (when needed): uv project; scripts live in `py/`; called from R via
  `processx::run("uv", c("run", "python", "py/foo.py", ...))`.
- Comments are rare and explain *why*, never *what*. Names do the explaining.

## 6. Plotting & theming

- **All charts: `ggplot2` + `ggiraph`** (interactive on hover/click). No plotly,
  highcharter, or echarts4r unless we have a documented reason.
- **All tables: `DT`** for long sortable/filterable; **`gt`** for short
  presentation tables. Wrap both in `R/table.R` so styling stays unified.
- Shared theme function `theme_mo()` in `R/theme.R`. It returns a `theme()`
  object (Lato/Open Sans, white bg, subtle gridlines) and registers ggiraph
  defaults (hover colour, tooltip CSS) at package load.
- Colour palette: limited categorical (≤8 colours) + a sequential and a
  diverging scale. Defined once in `R/theme.R` as named vectors; never inline.
- Site theme inherits from a bootswatch base (TBD — pick one and stick with it,
  do **not** repeat the previous mistake of `sandstone` + `cosmo` + `united`
  across pages) and overrides via `assets/style.scss`.

## 7. Build & CI

- **Local:** `quarto preview` for live reload during development.
- **CI:** `.github/workflows/render.yml`. Read the file for the canonical
  definition; this section only summarises and explains the choices.
  - Triggers: nightly cron `17 3 * * *` (03:17 UTC, chosen to land after the
    overnight EU API refresh windows — do **not** use `59 23 * * *` like the old
    standalone trackers did, that races with Eurostat's update and yields stale
    numbers); plus `push` to `main` for layout/code changes (docs-only paths
    skipped); plus `workflow_dispatch` for manual reruns.
  - Pipeline: `setup-r@v2 (4.4.3)` → `setup-renv@v2` (cached on hash of
    renv.lock) → `setup-uv@v5 (0.10.7, cached)` → `uv sync --frozen
    --no-install-project` → `setup-quarto@v2 (1.7.17)` → run every
    `R/fetch_*.R` with graceful failure → `quarto render` →
    `upload-pages-artifact@v3` → `deploy-pages@v4` (currently inert, see below).
  - **Graceful degradation is enforced in the workflow itself** (not in R):
    each fetcher runs in its own `Rscript` invocation; a non-zero exit emits a
    GitHub Actions `::warning::` annotation but does NOT fail the job. The
    .qmd pages read `latest` snapshots from `data-apis/` and display a "data
    as of <date>" badge sourced from the `.meta.json` sidecar.
  - When we add R packages needing system libs (cairo, poppler, freetype, …),
    insert `r-lib/actions/setup-r-dependencies@v2` BEFORE `setup-renv` and it
    will install them from DESCRIPTION/renv.lock.

### Cut-over from static HTML to Quarto

The deploy job in `render.yml` is gated `if: false`. The build/render half runs
nightly and validates pipeline health; the deploy half is held until cut-over.
To cut over (one commit, can be done any time):

1. GitHub repo Settings → **Pages → Source → "GitHub Actions"** (currently
   "Deploy from a branch / main / root").
2. Rename `home.qmd` → `index.qmd` AND delete `index.html`,
   `publications.html`, `tools.html`, `style.css` from the repo root in the
   same commit.
3. Change `if: false` → `if: true` (or remove the line) in
   `.github/workflows/render.yml`.
4. Verify by visiting https://michalovadek.github.io after the workflow runs.

Until step 1 happens, even `if: true` deploy attempts will fail (no Pages
deployment target wired) — so step 1 is the actual irreversible action; the
rest is reversible by reverting commits.

## 8. Git, gitignore, and GitHub size limits

GitHub enforces:

- **100 MB** hard limit per file (push rejected).
- **50 MB** soft warning per file.
- **~1 GB** recommended max repo size; **5 GB** absolute.
- Pages-served sites: any single file < 100 MB, total site < 1 GB.

Therefore:

- **Always gitignored:** `_site/`, `_freeze/`, `.quarto/`, `renv/library/`,
  `renv/python/`, `renv/staging/`, `.Rproj.user/`, `.Rhistory`, `.RData`,
  `.Ruserdata`, `__pycache__/`, `.venv/`, `*.pyc`, `node_modules/`.
- **Committed when small** (<5 MB): everything in `data-manual/`, parquet
  snapshots in `data-apis/`.
- **Compress or partition** when a single snapshot approaches 10 MB. Strategies,
  in order of preference: parquet > parquet with row-group filtering > split by
  year > drop columns we don't actually plot.
- **Git LFS** is allowed but is a last resort (GH free tier = 1 GB LFS storage,
  1 GB/month bandwidth; easy to blow through with a nightly cron).
- **Never commit** rendered HTML to `main`. (The old repos did this and
  `eucourt/docs/index.html` is now 5.4 MB checked in.)

`.gitignore` ships with the repo and covers the above.

## 9. Migration map (old → new)

| Old repo                                           | Old build         | New location              | Data path                |
|----------------------------------------------------|-------------------|---------------------------|--------------------------|
| `../eu-veto-tracker/tracker.qmd`                   | Quarto + Action   | `trackers/eu-vetoes.qmd`  | `data-manual/vetoes.csv` |
| `../eucourt/eucourtstats.Rmd`                      | Rmd + nightly cron| `trackers/eu-court.qmd`   | `data-apis/eurlex_*`     |
| `../eulaw/eulawstats.Rmd`                          | Rmd + nightly cron| `trackers/eu-law.qmd`     | `data-apis/eurlex_*`     |
| `../eufinance/eufinancestats.Rmd`                  | Rmd + nightly cron| `trackers/eu-finance.qmd` | `data-apis/ecb_*, eurostat_*` |
| `index.html` / `publications.html` / `tools.html`  | Static HTML       | `index.qmd` / `publications.qmd` / `tools.qmd` | n/a |

Old repos stay live (their `gh-pages` branches keep serving) until the new pages
are verified equivalent. Then archive them on GitHub (don't delete — incoming
links from publications shouldn't 404).

Shared R-package dependencies seen across old trackers (consolidate in single
`renv.lock`): `eurlex`, `eurostat`, `ggplot2`, `ggiraph`, `dplyr`, `tidyr`,
`purrr`, `stringr`, `forcats`, `lubridate`, `DT`, `gt`, `gdtools`, `gfonts`,
`httr2`, `rvest`, `xml2`, `arrow`, `here`, `countrycode`, `ISOweek`,
`modelsummary`, `ggforce`.

## 10. Open questions / TODOs

- [ ] Pick bootswatch base + final colour palette (currently undecided).
- [ ] Decide whether `data-apis/` parquet snapshots are committed (history of
      data) or only `latest.parquet` (smaller repo). Default for now: keep last
      30 daily snapshots + a monthly snapshot retained indefinitely.
- [ ] Long-term: extract `R/fetch_eurlex.R` into the `eurlex` package itself
      (owner already maintains it) so other projects benefit.
- [ ] CNAME? (owner has no custom domain at time of writing.)
- [ ] Decide which old `origin/devel` / `origin/master` branches in this repo
      can be deleted — they look stale.
