# AGENTS.md — `x-cmd/mneme` (public org map)

This repo lists the public repos in the `x-cmd` GitHub org and
what each one is for. Use it to find the right repo before
editing or asking.

## Org layout

### `x-cmd` (public org)

| Repo | What it is |
| --- | --- |
| `x-cmd/x-cmd` | Main shell monorepo. Holds `mod/*` (every `x <subcmd>`), `X`, `adv/`, etc. The thing `x` is. |
| `x-cmd/install` | Canonical install database. One YAML per software under `src/<category>/<name>.yml`. Drives `x install <name>`. |
| `x-cmd/cve` | Daily CVE / CWE TSVs consumed by `x cve` and `x cwe`. Articles under `docs/` are published to `x-cmd.com/cve`. |
| `x-cmd/gpg` | The team's GPG public keyring + `index.tsv` manifest. Articles under `docs/` are published to `x-cmd.com/gpg`. |
| `x-cmd/mneme` *(this repo)* | The public org map. |
| `x-cmd/<other>` | Various site, doc, lab, promo, blog repos. See each repo's own README / CONTRIBUTING. |

### `x-cmd-install` (mostly-private org)

| Repo | What it is |
| --- | --- |
| `x-cmd-install/x-cmd-install` | Long-form docs, draft entries, vendored sources, operational notes. |
| `x-cmd-install/<software>` | One **collector** per piece of software. Each is an empty repo (no source code) running `x-cmd-install/x-cmd-install-action@main` on a cron; writes `data/card/<YYMMDD>.yml` + README files. Auto-updated; manual edits to `data/` are overwritten. |
| `x-cmd-install/x-cmd-install-action` | The reusable Action invoked by every collector. |
| `x-cmd-install/x-cmd-install-stat` | Aggregator. Walks every collector, writes `stat/<software>/latest.{card,report}.{yml,json}`. The website reads from here. |
| `x-cmd-install/mneme` | **Private** internal notes. Visibility = `private`. Public agents must not write here. |

## Repo conventions — the README / CONTRIBUTING / SKILL triad

Every public repo in the `x-cmd` org uses the same three-file
contract. `x-cmd/cve` and `x-cmd/gpg` are the canonical examples.

| File | Audience | Content |
| --- | --- | --- |
| `README.md` (+ `README.cn.md`) | End users, scanners | Front-of-page tables, FAQ, "how do I…", sister-repo links. |
| `CONTRIBUTING.md` | Developers, maintainers | Repo layout, schema, scripts, CI. |
| `SKILL.md` | AI agents | The 4-section "two consumption paths → schema → common queries → decision". |
| `LICENSE` | Lawyers, license reviewers | SPDX + per-repo rationale. |
| `AGENTS.md` *(this repo)* | Cross-repo agents | Org-level — the boundary rules and conventions that don't fit in any single sibling repo. |

Concrete touch-points:

- `x-cmd/cve` — pure producer; data flow is one-way
  (cvelistV5 → repo → release assets → `x cve`).
- `x-cmd/gpg` — pure trust-anchor; data flow is HTTPS pull
  → `gpg --import` → consumer.
- `x-cmd/install` — pure metadata; data flow is YAML → `x install`.
- `x-cmd/mneme` *(this repo)* — pure map.

## Article convention in topic repos (`docs/`)

The topic repos (`x-cmd/cve`, `x-cmd/gpg`) hold their articles
under `docs/`. Each article slot is four files, kept in sync:

```
docs/
├── 0-<slug>.en.md        # English article
├── 0-<slug>.cn.md        # Chinese version
├── 0-<slug>.llms.md      # LLM-friendly summary (YAML frontmatter + flat prose)
└── 0-<slug>.faq.yml      # Structured Q&A for SEO + JSON-LD
```

Canonical examples:

- `x-cmd/cve/docs/0-cve-for-ai-builders.{en,cn}.md` — overview.
- `x-cmd/cve/docs/0-cve-for-ai-builders.llms.md` — LLM summary.
- `x-cmd/cve/docs/0-cve-for-ai-builders.faq.yml` — structured FAQ.
- `x-cmd/gpg/docs/0-x-cmd-gpg-overview.{en,cn}.md` — overview.
- `x-cmd/gpg/docs/3-annual-key-strategy-explained.{en,cn}.md` — deep dive.

The leading integer in the filename is the reading order.

## Boundary rules

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

## Public-repo discipline

Public repos in the `x-cmd` org — `x-cmd/mneme` included — have a
**strict content-only** rule for commits and issues:

- ✅ **Allowed:** correcting facts, fixing typos, adding or
  updating technical content, fixing broken links, fixing front-
  matter, regenerating tables, fixing CI.
- ❌ **Not allowed:** discussion of intent, strategic direction,
  commercialization, internal org context, or anything that
  should live in the private `x-cmd-install/mneme`.

If you're unsure whether a change is content-only, it isn't —
default to opening an issue first or moving the discussion to
the private notes.

## What the agent does

1. **Read this file first** to pick the right sibling repo.
2. **Read that repo's `README.md` + `AGENTS.md` / `CONTRIBUTING.md`
   before editing** — every repo has its own conventions.
3. **Prefer the lightest-weight repo that satisfies the request.**
   If the user wants to look up a CVE, `x cve` is the answer;
   cloning `x-cmd/cve` is overkill. If they want to install
   software, `x install <name>` is the answer; editing
   `x-cmd/install`'s YAML is the wrong move unless the user is
   asking to *add* software to the database.
4. **Quote from sibling repos with attribution.** Copy their
   prose only when the audience matches, and link back.

## Cross-repo reference — `cve` and `gpg`

```
x-cmd/cve
├── .x-cmd/                 # Python 3.8+ stdlib-only scripts (tsv.py, cwe.py, …)
├── data/                   # regenerated on every CI run — NOT in git
├── report/                 # regenerated on every CI run — committed to main
├── docs/                   # articles published to x-cmd.com/cve
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
├── docs/                   # articles published to x-cmd.com/gpg
├── README.md               # English — front-of-page key catalog
├── README.cn.md            # Chinese version
├── SKILL.md                # AI-agent recipe (fingerprint pin patterns)
├── CONTRIBUTING.md         # maintainer pipeline
└── LICENSE                 # strict-rights-reserved variant
```

Both repos use the same **`BEGIN/END … .md` inline markers**
pattern in `README.md` so regenerated tables round-trip without
merge conflicts. New repos should copy that marker pattern.

## See also

- [`x-cmd/install/AGENTS.md`](https://github.com/x-cmd/install/blob/main/AGENTS.md) — install-database agent rules.
- [`x-cmd/cve/SKILL.md`](https://github.com/x-cmd/cve/blob/main/SKILL.md) — CVE / CWE consumption patterns.
- [`x-cmd/gpg/SKILL.md`](https://github.com/x-cmd/gpg/blob/main/SKILL.md) — GPG keyring consumption patterns.
- [`x-cmd/x-cmd`](https://github.com/x-cmd/x-cmd) — module source (`mod/`).