---
name: topic-library-directory-style
description: In x-cmd's public topic-library repos (gpg, seo, cve, ratelimit, uptime, ...), each article lives in its own folder under docs/<slug>/ as cn.md, en.md, llms.md, faq.yml. The folder name carries the slug; filenames have no prefix.
metadata:
  type: project
---

In the x-cmd public topic-library repos, every article lives in
its own folder as `cn.md` / `en.md` / `llms.md` / `faq.yml`. The
folder name carries the slug; the filenames have no `<slug>.`
prefix.

**Deployment (where each repo's docs land):**

The folder name IS the URL slug. Each public topic repo is
published at `x-cmd.com/<repo-name>/<slug>/` (the VitePress
consumer in an external repo reads the docs/ tree and serves
it under that path).

| Repo | Source | Served at | Notes |
| --- | --- | --- | --- |
| `x-cmd/gpg` | `gpg/docs/<slug>/{cn,en,llms}.md + faq.yml` | `x-cmd.com/gpg/<slug>/` | |
| `x-cmd/seo` | `seo/docs/<slug>/...` | `x-cmd.com/seo/<slug>/` | |
| `x-cmd/cve` | `cve/docs/<slug>/...` | `x-cmd.com/cve/<slug>/` | daily-refreshed TSV data lives in `cve/data/`, separate from docs |
| `x-cmd/ratelimit` | `ratelimit/docs/<slug>/...` | `x-cmd.com/ratelimit/<slug>/` | |
| `x-cmd/uptime` | `uptime/docs/<slug>/...` | `x-cmd.com/uptime/<slug>/` | reference implementation |
| `x-cmd/dns` | `dns/docs/<slug>/...` | `x-cmd.com/dns/<slug>/` | dev happens on `feat/*` branches |
| `x-cmd/doc-2026` | `data/{install,mod,pkg,start,ai,root,theme,action}/...` | `x-cmd.com/{install,mod,pkg,start,ai,root,theme,action}/...` | multiple sub-trees, see [[x-cmd-content-tree-layout]] if it exists |
| `x-cmd/mneme` | `memory/<topic>.md` | **not deployed** | org map + agent memory; AI agents read directly from `main` / `agent-notes-dnrs` |

**Why:**

- One slug = one folder. Browser-friendly URLs land on the
  folder directly (`/docs/<slug>/` resolves through
  `cn.md` / `en.md` per language).
- LLMs (which read `*.md` directly) see a uniform shape across
  every topic repo (`cn.md` / `en.md` / `llms.md` / `faq.yml`)
  with no per-repo variation to memorize.
- Matches the `x-cmd/uptime` reference implementation
  ([docs/1-uptime-content/cn.md](https://github.com/x-cmd/uptime/blob/main/docs/1-uptime-content/cn.md)
  and friends).

**How to apply:**

- The full pattern is:

  ```
  docs/<slug>/
    cn.md          # Chinese body
    en.md          # English body
    llms.md        # LLM-readable summary (frontmatter + body)
    faq.yml        # rendered as the last section of the page body — see [[faq-yaml-rendered-as-page-body]]
  ```

- A lang-less summary file (e.g. `<slug>.md`) maps to
  `<slug>/index.md` during migration.
- Files like `_sidebar.yml`, `README.md`, `LICENSE`, `CHANGELOG.md`,
  and any `assets/` directory at `docs/` root are NOT migrated —
  meta stays flat.
- In-content links use **site-relative paths** (`/docs/<slug>/`
  or whatever the rendered URL is) — never filesystem paths.
  A file move does not break in-content references because
  URLs are unaffected.
- FAQ `reference:` arrays in `<slug>/faq.yml` may legitimately
  point at the **old** filesystem paths after migration; this
  is accepted as a known trade-off (the FAQ schema doesn't
  validate them at runtime, and rewriting them all is more
  disruption than value).

**Migration tool:** `.x-cmd/migrate-flat-docs.sh` (in
`x-cmd/doc-2026`) takes a `<docs-dir>` argument and converts
flat-with-prefix into directory-without-prefix. `DRY=1` for
plan, `DRY=0` (or `--apply`) to execute `git mv`. Idempotent
on re-run via `[ -e src ]` check.

**Coverage as of 2026-10-16:**

| Repo | Articles | Status |
| --- | --- | --- |
| `x-cmd/gpg` | 6 | migrated (`417fce3..2dc7c53`) |
| `x-cmd/seo` | 10 | migrated (`b0a6849..74bc124`) |
| `x-cmd/cve` | 7 | migrated (`3583935..534f415`) |
| `x-cmd/ratelimit` | 8 | migrated (`9ae8086..488af44`) |
| `x-cmd/uptime` | 7 | reference (already directory-style) |
| `x-cmd/doc-2026/data/install/<cat>/<slug>/` | ~2000 | migrated (`1084eea8`) |
| `x-cmd/doc-2026/data/root/<slug>/` | 8 | migrated (`1084eea8`) |
| `x-cmd/doc-2026/data/ai/0-overview/<file>/` | 4 | migrated (`839059fd`) |
| `x-cmd/doc-2026/data/mod/<module>/<n>/` | 175 | migrated (`947543a9`) |
| `x-cmd/doc-2026/data/start/<cat>/<n>-<name>/` | ~14 | migrated (`947543a9`) |
| `x-cmd/doc-2026/data/theme/0-overview/<n>-<name>/` | 7 | migrated (`947543a9`) |
| `x-cmd/doc-2026/data/action/0-overview/<n>-<name>/` | 6 | migrated (`947543a9`) |

**Exceptions to the pattern:**

- `x-cmd/doc-2026/data/pkg/<cat>/<slug>/<n>-<lang}.md` keeps
  the `${number}-` prefix because each module has multiple
  numbered articles; the number is sidebar-sort ordering, not
  a slug. Dropping it would break the sort.
- `x-cmd/doc-2026/data/lctt/0-overview/` was not migrated (1
  file with no en/llms siblings, judged not worth the
  surgery).

Related: [[faq-yaml-rendered-as-page-body]] (the rendering
rule that makes the FAQ `reference:` trade-off tolerable).