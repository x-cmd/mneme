# AGENTS.md — `x-cmd/mneme` and the orgs around it

This repo (`x-cmd/mneme`) is the **public** index of x-cmd's
multi-repo architecture: which repo lives where, what each one
is for, what conventions it follows, and what the agent's read /
write boundary is. It is the org-map counterpart to the per-repo
`README.md` / `CONTRIBUTING.md` / `SKILL.md` triad.

> **Not** a fork, **not** a mirror of upstream source code, **not**
> an install database, **not** a keyring. The role is *map + guide*:
> a single page an agent (human or AI) reads before deciding which
> of the sibling repos to touch.

## How to read this file

1. **Org layout** — which org is which, where each repo lives.
2. **Per-repo conventions** — the README / CONTRIBUTING / SKILL
   pattern, illustrated with `x-cmd/cve` and `x-cmd/gpg` as the
   canonical examples.
3. **Read path / write path** — where data flows, and where the
   agent is allowed to edit.
4. **Boundary rules** — what stays in this repo, what gets pushed
   to a sibling, what stays private.

## Org layout

x-cmd is split across **two GitHub orgs** plus this index repo:

### `x-cmd` (public org)

| Repo | Role | Editing model |
| --- | --- | --- |
| `x-cmd/x-cmd` | The main shell monorepo. Holds `mod/*` (every `x <subcmd>`), `X`, `adv/`, `extmeta.tar`, etc. The thing `x` is. | Internal team. PRs from outside are reviewed, rarely merged for `mod/` core. |
| `x-cmd/install` | Canonical install database. One YAML per software under `src/<category>/<name>.yml`. Drives `x install <name>`. | **Public PRs welcome.** |
| `x-cmd/cve` | Producer of the daily CVE / CWE TSVs that `x cve` and `x cwe` consume. | Issues welcome. `data/` and `report/` regenerate on CI. |
| `x-cmd/gpg` | The team's GPG public keyring + `index.tsv` manifest. | **Team-only** for `keyring/` and `index.tsv`. Doc fixes via issue, not PR. |
| `x-cmd/mneme` *(this repo)* | Public org-map. Documents the architecture for external agents. | **Public PRs welcome** — docs only. |
| `x-cmd/<other>` | Various site, doc, lab, promo, blog repos. | See each repo's own `CONTRIBUTING.md`. |

### `x-cmd-install` (mostly-private org)

This org holds the **data-collection substrate** behind
`x-cmd/install`. It is **not** a mirror of upstream source code
and **not** a fork of `x-cmd/install`.

| Repo | Role | Editing model |
| --- | --- | --- |
| `x-cmd-install/x-cmd-install` | Internal "truth" for the install database — long-form docs, draft YAMLs, vendored upstream sources, operational notes. **Not** what `x install` reads at runtime. | Team-only. |
| `x-cmd-install/<software>` | One **collector** per piece of software. Each is an empty repo (no source code) running `x-cmd-install/x-cmd-install-action@main` on a cron; writes `data/card/<YYMMDD>.yml` + README files. | Auto-updated; manual edits to `data/` are overwritten. |
| `x-cmd-install/x-cmd-install-action` | The reusable Action invoked by every collector. Single source of truth for what metadata gets fetched. | Team-only. |
| `x-cmd-install/x-cmd-install-stat` | Aggregator. Walks every collector, writes `stat/<software>/latest.{card,report}.{yml,json}`. The website reads from here. | Team-only for `stat/` (auto-regenerated). |
| `x-cmd-install/mneme` | **Private** internal notes — organizational quirks, past mistakes, operational context that should not leak. Visibility = `private`. | **Agents contributing publicly must not write here.** |

### The pattern: producer → collector → aggregator → consumer

```
upstream GitHub repo          (real source code, e.g. jqlang/jq)
        ↓  queried by collector Action — NOT forked
x-cmd-install/<software>      (collector writes data/card/...yml + README.{md,cn.md})
        ↓  read by x-cmd-install-stat's sync.sh
x-cmd-install-stat            (aggregator, stat/<software>/latest.card.yml)
        ↓  consumed by
cn.x-cmd.com/install/<name>   (the public website view)

x-cmd/install                 (the canonical YAML database)
        ↓  read by
x install <name>              (the runtime command)
```

The two arms (`x-cmd/install` for metadata, `x-cmd-install-stat`
for rich data) intentionally do not share a storage layer — they
serve different audiences:

- `x install` needs a single YAML it can ship in the binary;
  schema is rigid, entries are small, updates ship with the
  release.
- The website wants aggregated OpenSSF scorecard, repology
  status, contributor activity, license recency — much richer
  than what the runtime needs.

## Conventions — the README / CONTRIBUTING / SKILL triad

Every public repo in the `x-cmd` org follows the same three-file
contract. `x-cmd/cve` and `x-cmd/gpg` are the canonical examples;
new repos should copy the shape.

| File | Audience | Tone | Content |
| --- | --- | --- | --- |
| `README.md` (+ `README.cn.md`) | End users, scanners | Marketing-grade overview | Front-of-page tables, FAQ, "how do I…", sister-repo links. The file the GitHub social preview shows. |
| `CONTRIBUTING.md` | Developers, maintainers | Technical | Repo layout, schema, scripts, CI. The file you read *after* you've decided to change something. |
| `SKILL.md` | AI agents | Recipe | The 4-section "two consumption paths → schema → common queries → decision". Frontmatter `name` + `description` so the agent's skill loader can pick it up. |
| `LICENSE` | Lawyers, license reviewers | Legal | SPDX + per-repo rationale (see `x-cmd/gpg/LICENSE` for a strict-rights-reserved variant; most others are Apache-2.0). |
| `AGENTS.md` *(this repo)* | Cross-repo agents | Org-level | What is **not** in any single sibling repo: the org boundary, the producer→aggregator read path, the public/private split. |

Concrete touch-points:

- `x-cmd/cve` — pure producer; data flow is one-way
  (cvelistV5 → repo → release assets → `x cve`).
- `x-cmd/gpg` — pure trust-anchor; data flow is HTTPS pull
  → `gpg --import` → consumer.
- `x-cmd/install` — pure metadata; data flow is YAML → `x install`
  lookup.
- `x-cmd/mneme` *(this repo)* — pure map.

### What goes where, with examples

| Need to… | Read | Edit |
| --- | --- | --- |
| Look up a CVE id | `x cve info CVE-2024-0001` (consumes `x-cmd/cve`'s release assets) | (none — read-only) |
| Add a brand-new software to `x install` | `x-cmd/install/AGENTS.md` | Add `src/<category>/<name>.yml` in `x-cmd/install`. **Not** in `x-cmd-install/x-cmd-install/`. |
| Get the website showing fresh data for `<software>` | `x-cmd-install/<software>/README.md` (auto) | Trigger the `card.yml` workflow in that collector. |
| Audit a GPG key fingerprint | `x-cmd/gpg/index.tsv` + `keyring/<handle>.asc` | **Don't.** File an issue; team regenerates. |
| Understand why a collector exists | `x-cmd-install/mneme/AGENT.md` (private) | **Don't** — that repo is internal-only. |
| Understand which repo to touch | **this file** | **this file** — open a PR. |

## What lives in `x-cmd/mneme` (this repo)

This repo is **the index** of the architecture above. It is not a
frequently-updated source-of-truth repo — when you change one of
the sibling repos' roles, you come back here and update the map.

Authoritative content kept here:

- Org layout (the table above).
- The README / CONTRIBUTING / SKILL / AGENTS pattern.
- The producer → collector → aggregator → consumer read path.
- The public-vs-private boundary between `x-cmd` (public) and
  `x-cmd-install` (mostly private).
- Cross-repo pointers that don't fit in any single sibling repo.

Everything else belongs in a sibling repo. In particular:

- Install YAML → `x-cmd/install`.
- CVE data → `x-cmd/cve`.
- GPG keys → `x-cmd/gpg`.
- Module source → `x-cmd/x-cmd`.
- Internal org notes → `x-cmd-install/mneme` (private).

## What the agent does (and doesn't) do

When the user asks an AI agent to act inside the x-cmd ecosystem,
the agent should:

1. **Read this file first** to pick the right sibling repo.
2. **Read that repo's `README.md` + `AGENTS.md` / `CONTRIBUTING.md`
   before editing** — every repo has its own conventions, and the
   conventions in `x-cmd/install` differ from those in `x-cmd/cve`
   or `x-cmd/gpg`.
3. **Stay inside the public boundary.** `x-cmd-install/mneme` is
   private; even if it appears in a clone list, the agent must
   not propose PRs to it or quote its content into a public
   context. The privacy split is deliberate.
4. **Prefer the lightest-weight repo that satisfies the request.**
   If the user just wants to look up a CVE, `x cve` is the answer;
   cloning `x-cmd/cve` is overkill. If the user wants to install
   software, `x install <name>` is the answer; editing
   `x-cmd/install`'s YAML is the wrong move unless the user is
   asking to *add* software to the database.
5. **Quote from sibling repos with attribution.** The `x-cmd/cve`
   FAQ and `x-cmd/gpg` README are written in a specific voice for
   a specific audience; copy their prose only when the audience
   matches, and link back.

## Boundaries the agent must respect

- **Do not** commit to `x-cmd/x-cmd`'s `mod/` directly without
  checking `x scotty mod install` works locally first (per
  `CLAUDE.md`).
- **Do not** edit `data/` in any `x-cmd-install/<software>`
  collector — the next scheduled run will overwrite your change.
  Fix the action, not the output.
- **Do not** edit `keyring/` or `index.tsv` in `x-cmd/gpg` — file
  an issue; the team regenerates.
- **Do not** edit `x-cmd-install/mneme/` from a public workflow.
- **Do not** treat `x-cmd-install/<software>` as a fork of the
  upstream source. It contains **metadata**, not source code.
- **Do not** put install YAML, GPG keys, or CVE data into this
  repo. Wrong place; open a PR in the right sibling repo instead.

## Cross-repo reference — `cve` and `gpg`

Two public repos define the patterns this org-map is built on.
Their layouts:

```
x-cmd/cve
├── .x-cmd/                 # Python 3.8+ stdlib-only scripts (tsv.py, cwe.py, …)
├── data/                   # regenerated on every CI run — NOT in git
├── report/                 # regenerated on every CI run — committed to main
├── README.md               # English — front-of-page tables + FAQ
├── README.cn.md            # Chinese version (auto-updated)
├── SKILL.md                # AI-agent recipe
├── CONTRIBUTING.md         # developer pipeline
└── LICENSE                 # Apache-2.0

x-cmd/gpg
├── keyring/
│   ├── <handle>.asc        # ASCII-armored public keys
│   └── keyring.asc         # concatenated keyring (regenerated)
├── index.tsv               # 5-col manifest: handle, uid, fingerprint, created, purpose
├── README.md               # English — front-of-page key catalog
├── README.cn.md            # Chinese version
├── SKILL.md                # AI-agent recipe (fingerprint pin patterns)
├── CONTRIBUTING.md         # maintainer pipeline
└── LICENSE                 # strict-rights-reserved variant
```

Their CI models differ — `cve` regenerates TSVs every 4h from
upstream, `gpg` runs `verify.yml` on every push to enforce
fingerprint invariants. Both use the same **`BEGIN/END … .md`
inline markers** pattern in `README.md` so regenerated tables
round-trip without merge conflicts. New repos should copy that
marker pattern.

## See also

- [`x-cmd/install/AGENTS.md`](https://github.com/x-cmd/install/blob/main/AGENTS.md) — install-database agent rules.
- [`x-cmd/cve/SKILL.md`](https://github.com/x-cmd/cve/blob/main/SKILL.md) — CVE / CWE consumption patterns.
- [`x-cmd/gpg/SKILL.md`](https://github.com/x-cmd/gpg/blob/main/SKILL.md) — GPG keyring consumption patterns.
- [`x-cmd/x-cmd`](https://github.com/x-cmd/x-cmd) — module source (`mod/`).
- [`x-cmd-install/mneme/AGENT.md`](https://github.com/x-cmd-install/mneme) — private internal notes (read access only, no edits from public agents).