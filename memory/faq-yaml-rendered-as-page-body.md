---
name: faq-yaml-rendered-as-page-body
description: In x-cmd's public document system, faq.yml is rendered as the last section of the page body — not as a sidebar or collapsed FAQ widget.
metadata:
  type: project
---

In the public x-cmd document system, every article's
`faq.yml` is **rendered as the last section of the page body**
— not as a sidebar or collapsed FAQ widget.

**Why:** This is how the x-cmd public site builds articles.
The site renderer concatenates `*.en.md` (or `*.cn.md`) body
content with `faq.yml` Q&A pairs in the same page, so the
FAQ becomes part of the readable article flow.

**How to apply:**

- **faq.yml is part of the article's main content.** It runs
  inline after the `.en.md` / `.cn.md` body and before
  related-resources links.
- **Each FAQ entry should be self-contained.** The reader
  may scroll to the FAQ section without reading the article
  body. Don't assume prior context — link back to specific
  sections in the body when needed.
- **FAQ entries can be longer than typical FAQ.** A short
  one-liner doesn't help if the section is rendered as
  body. Each entry can be 2-4 sentences with code blocks,
  tables, or commands when useful.
- **Structure**: group entries by topic under `data[].name`
  (e.g., `data[].name: { en: 'headers', cn: '响应头' }`), not
  by FAQ-style alone. Each entry has `id`, `question`,
  `answer`, `confidence` (1-9), and `reference` (list of
  article files that back the answer).
- **Use FAQ for content that doesn't fit the article's
  linear flow** — deep dives, edge cases, side
  comparisons, troubleshooting recipes, configuration
  examples. The article body carries the narrative; the
  FAQ carries the lookup-style fragments.

This is different from typical web FAQs (where the FAQ is a
sidebar) and from collapsed accordion widgets. Treat
faq.yml as a continuation of the article.

Related: [[x-cmd-public-repo-discipline]] (faq.yml must be
content-only, no strategic / commercial discussion).