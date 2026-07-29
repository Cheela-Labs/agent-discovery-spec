---
ads: 0
title: ADS Proposal Process
status: Living
type: Meta
author: Agent Discovery Specification Contributors
created: 2026-07-25
---

## Summary

Defines how the specification itself changes: proposal types, numbering,
status lifecycle, and what's required before a proposal can affect
`spec/`. Modeled closely on Ethereum's EIP-1, scaled down for a much
smaller specification.

## Proposal types

- **Core** — changes to ADS Core: the manifest schema, the discovery flow,
  versioning rules, or anything else in `spec/`. Requires a MAJOR or MINOR
  spec version bump per
  [spec/04-versioning-strategy.md](../spec/04-versioning-strategy.md).
- **Extension** — a new, optional, namespaced capability category or field
  block that doesn't change Core itself. Once `Final`, documented under
  `spec/extensions/` (created on first Extension proposal) as a reference,
  but never folded into required Core fields.
- **Meta** — process changes: this document, `GOVERNANCE.md`,
  `CONTRIBUTING.md`. Affects how the project works, not what a conformant
  manifest looks like.
- **Informational** — non-normative guidance: best practices, design notes,
  reference-implementation write-ups. Carries no conformance weight; a
  system can ignore every Informational proposal and still be fully
  ADS-conformant.

## Numbering

Every proposal gets a permanent, sequential integer (`ADS-N`) the moment
it's merged into `proposals/` as a `Draft`. Numbers are never reused, even
if the proposal is later `Withdrawn`. `ADS-0` (this document) is reserved
for the process definition itself, matching EIP-1's role for EIPs.

## Status lifecycle

```text
Draft → Review → Last Call → Final
  ↓         ↓
Withdrawn  Stagnant (revivable)
```

| Status | Meaning |
|---|---|
| `Draft` | Merged as a discussion document. Not normative yet. |
| `Review` | Complete enough to evaluate; open for implementer feedback. An editor promotes to this once the proposal has a full schema diff (for Core/Extension), a rationale, and a backward-compatibility statement. |
| `Last Call` | Minimum 14-day window. Moves to `Final` if no unresolved, specific implementer objections remain when the window closes. |
| `Final` | Normative. For Core/Extension proposals, folded into `spec/` (or `spec/extensions/`) as part of the next version release. |
| `Withdrawn` | Author pulled it. Terminal. |
| `Stagnant` | No activity for 6+ months in `Draft` or `Review`. Not terminal — anyone can revive it by resuming discussion, which moves it back to `Draft`. |

Full authority detail (who can move a proposal between statuses, how
disputes in `Last Call` get resolved) lives in
[../GOVERNANCE.md](../GOVERNANCE.md) rather than duplicated here.

## What a proposal must contain

Use [TEMPLATE.md](TEMPLATE.md). At minimum, before `Review`:

- **Summary** — one paragraph, what changes and why.
- **Motivation** — the problem, with a concrete example of where the
  current spec falls short. "This would be nice" is not motivation; "here's
  a manifest I can't correctly express today, and here's why that matters"
  is.
- **Specification** — the actual normative text/schema diff being proposed.
- **Rationale** — alternatives considered and why this shape won.
- **Backward compatibility** — explicit statement of the version bump this
  requires (per [spec/04-versioning-strategy.md](../spec/04-versioning-strategy.md))
  and what existing manifests/clients need to do, if anything.
- **Reference implementation** (optional but strongly encouraged) — a link
  to at least one system that has actually implemented the proposed
  behavior, even experimentally. Proposals with a working reference move
  through `Review` faster because there's something concrete to argue
  about.

## Why this process exists

A specification only has value if implementers can trust that it won't
change under them without warning. The overhead here — proposal numbers,
a mandatory `Last Call` window, an explicit backward-compatibility
statement — exists entirely to make that trust earned rather than assumed.
If a change is small enough that this feels like overkill, it's very
likely small enough to ship as a documentation PATCH instead (see
[../CONTRIBUTING.md](../CONTRIBUTING.md#1-something-is-unclear-wrong-or-typod))
and doesn't need a proposal at all.
