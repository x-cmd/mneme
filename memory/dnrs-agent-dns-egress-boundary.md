---
name: dnrs-agent-dns-egress-boundary
description: dnrs is positioned as the Agent-sandbox DNS egress boundary (auditable, deny-by-default, honeypot-capable) — explicitly NOT a CoreDNS substitute. CoreDNS answers "what can this DNS server do"; dnrs answers "does this Agent deserve to send this query".
metadata:
  type: project
---

dnrs (`github.com/dnrs-dev/dnrs` or `github.com/dnrs/dnrs`,
both initial homes TBD; the `dnrs-dev` org already exists;
possible future move to `x-cmd/dnrs`) is a Rust DNS
resolver. Its market is **not** "secure recursive resolver" or
"Kubernetes DNS replacement" — those are crowded with unbound,
Knot, BIND, CoreDNS, Pi-hole.

dnrs's category is **DNS egress boundary for AI Agent
sandboxes**:

- deny-by-default
- per-query audit log (tamper-evident, forensically
  replayable)
- Agent-aware (binds `agent_id` / `run_id` / `sandbox_id`)
- built-in honeypot responses (`echo` / `ai` / poison
  wildcard)
- policy is Rust code, not a runtime rule engine
- extreme perf target (p99 < 1μs, zero-heap-alloc) because it
  sits in the hot path of every sandboxed tool call

It is the **opposite** of CoreDNS:

| | CoreDNS | dnrs |
|---|---|---|
| core philosophy | plugin chain / Corefile | hard-coded pipeline / compile-time policy |
| target user | platform / K8s ops | Agent platform / security |
| audit | optional plugin | first-class |
| config surface | grows with plugins | minimal, policy = code |
| Agent awareness | none | native |

**Why:** today's real exfiltration risk in Agent sandboxes is
DNS. HTTP proxy and shell lockdown are routine; DNS is the
unguarded channel. CoreDNS can be configured to gate this, but
it was not built for it. dnrs is. This positioning gives the
project a sharp, defensible category instead of competing in
the generic DNS-server pool.

**How to apply:** when asked "what is dnrs?" or "is dnrs like
CoreDNS / Pi-hole / Unbound?", answer from this file — not from
generic DNS-resolver framings. The comparison table above is
safe to surface in README/sales copy; the commercial pitch
(kept in private session memory) is internal-only.

Related: [[dnrs-vs-coredns]] (full positioning archive,
private — includes the commercial pitch and "why this thesis
holds"), [[dnrs-r-roadmap]] (brand evolution aligned with
version milestones), [[dnrs-license-cla]] (Apache-2.0 + CLA,
re-license option reserved).

Lives in `x-cmd/mneme/memory/` per the org convention — agent
notes, not user-facing content.