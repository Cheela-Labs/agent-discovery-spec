# Governance

This describes who decides what, modeled on how Ethereum's EIP process and
IETF's RFC process both separate "who can edit the document" from "who
decides what's true."

## Roles

**Editors** — maintain the mechanics of the repo: assign ADS numbers, check
that proposals follow the template and don't conflict with an already-`Final`
proposal, merge editorial/PATCH-level fixes, and manage the release of new
spec versions once a proposal reaches `Final`. Editors do **not** unilaterally
decide whether a proposal is a good idea. That distinction matters — an
editor can reject a proposal for being incomplete or malformed, never for
being one they personally disagree with.

Editors are added by rough consensus of existing editors, same as most
open-spec projects at this stage. As the project grows past a single-digit
number of active implementers, this document gets revisited — a spec used by
one implementer doesn't need the same governance weight as one used by
twenty, and we'd rather under-specify governance early than pretend to a
process we don't have the community to support yet.

**Implementers** — anyone shipping something that reads or writes an ADS
manifest. Implementers don't need permission to implement (it's MIT
licensed), but proposals that would break existing implementations need
explicit sign-off from at least one implementer affected by the change
before moving past `Review`.

**Contributors** — everyone else. No gate to contribute; see
[CONTRIBUTING.md](CONTRIBUTING.md).

## How a proposal becomes part of the spec

1. `Draft` — merged into `proposals/` as a discussion document. No behavior
   changes yet.
2. `Review` — open for implementer feedback. An editor tags it `Review` once
   the proposal is complete enough to evaluate (has a full manifest schema
   diff, a rationale section, and a backward-compatibility statement).
3. `Last Call` — a minimum 14-day window with no unresolved objections from
   implementers. Objections must be specific ("this breaks X because Y"),
   not procedural stalling.
4. `Final` — the proposal is normative. An editor folds the change into
   `spec/` and the JSON Schema, and cuts the next spec version per
   [spec/04-versioning-strategy.md](spec/04-versioning-strategy.md).

A proposal can also land as `Withdrawn` (author pulls it) or `Stagnant` (no
activity for 6+ months) — both are archival states, not failures; anyone can
revive a stagnant proposal by picking up discussion again.

## Disputes

If editors disagree on whether `Last Call` objections are resolved, the
proposal stays in `Last Call` — silence is not consensus, and an
unresolved dispute defaults to *not* shipping the change rather than
overriding it. There is no single BDFL tiebreaker by design; a spec meant to
outlive any one company shouldn't have a single point of failure in its
decision process either.

## Trademark and implementation conformance

"AI Discovery Specification" and "ADS" describe the specification text
itself. There is no certification program, no conformance mark, and no
trademark gate on calling an implementation "ADS-conformant" — if it
satisfies the MUST requirements in `spec/`, it's conformant, full stop. If a
formal conformance test suite gets built later, that's itself a proposal,
not something editors add unilaterally.

## Relationship to any single implementer

No implementer, including Cheela, gets special standing in this process
beyond what any other implementer gets: a vote in `Last Call` on proposals
that affect them. Editors who are also employed by an implementing company
recuse their editor judgment (though not their implementer feedback) on
proposals where their employer has a direct commercial interest in the
outcome.
