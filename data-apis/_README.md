# data-apis/

**Latest snapshots only.** Written nightly by `R/fetch_<src>.R` scripts via the
CI workflow.

Naming:
- `<src>_latest.parquet` — the data (parquet, zstd-compressed).
- `<src>_latest.meta.json` — sidecar: `{source, fetched_at, rows, schema_hash}`.

Historical snapshots are NOT kept here. If a fetcher needs older data it
re-queries the live source. (We opted against accumulating monthly archives
because the API can always reconstruct history; see
docs/specs/2026-05-23-shared-infra-design.md §2.)

Writes must be atomic (tempfile + rename) so a reader never sees a
half-written snapshot. The `R/_fetch_template.R` template enforces this.

See AGENTS.md §4.
