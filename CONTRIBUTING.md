# Contributing

Thanks for reading this before opening a PR — it saves everyone a round trip.

There are three different kinds of contribution here, and they go through
different doors. Picking the right one matters more than the content of what
you're proposing.

## 1. Something is unclear, wrong, or typo'd

Open an issue, or just send a PR. This includes:

- Broken links, typos, formatting
- A section that's ambiguous enough that two implementers would read it
  differently
- An example that doesn't actually validate against the schema

These get reviewed like any normal documentation PR — no process overhead.
If the fix is purely editorial (doesn't change what a conformant
implementation would do), it merges as a **PATCH** to the spec version.

## 2. You want to change or extend the specification

This is the important path, and it does *not* start with a PR to `spec/`.

The specification is a standard other people build against. A drive-by PR
that changes the manifest schema is the same failure mode as a drive-by PR
to a wire protocol — it might be a great idea, but it needs to be proposed,
discussed, and given a chance for implementers to object *before* it lands,
not after.

Start here instead: [proposals/0000-process.md](proposals/0000-process.md)
and [proposals/TEMPLATE.md](proposals/TEMPLATE.md). In short:

1. Open an issue describing the problem (not the solution yet) — this is
   where "has someone already tried this" gets caught early.
2. If there's appetite for it, write a proposal from the template and open
   it as a PR into `proposals/`. It gets a number (`ADS-N`) and a `Draft`
   status the moment it's merged as a discussion document — merging the
   proposal document is not the same as accepting the change.
3. The proposal moves through `Draft → Review → Last Call → Final` as
   consensus builds. Only a `Final` proposal that changes core behavior gets
   folded into `spec/` as part of the next version bump.

This sounds heavier than a normal PR because it is, deliberately. Anyone
who's had a dependency's "standard" change under them without warning will
recognize why.

## 3. You want to register a namespace prefix

Smaller and faster than a full proposal. See
[spec/05-extensibility.md](spec/05-extensibility.md#namespace-registry) —
it's a one-line addition to a registry file and doesn't need consensus
beyond "is this prefix already taken."

## Style notes for spec documents

- Normative requirements use **MUST** / **MUST NOT** / **SHOULD** / **MAY**
  in the RFC 2119 sense — say so explicitly the first time a document uses
  them.
- Prefer a worked example over an abstract description. If a rule can't be
  shown in a 10-line JSON snippet, the rule is probably too complicated.
- Non-normative commentary (rationale, "why we didn't do X instead") goes in
  a clearly marked `> **Rationale:**` blockquote, never mixed into the
  normative text itself.
- Every schema change needs a matching example under
  `spec/schema/examples/` that actually validates — don't hand-wave it.

## Governance

Who can merge what, and how disputes get resolved, is in
[GOVERNANCE.md](GOVERNANCE.md). The short version: editors merge process and
editorial changes on their own judgment; anything that changes normative
behavior needs a `Final` proposal first, full stop, no exceptions for
"obviously good ideas."

## Code of conduct

Covered by [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md). Be someone people want
to collaborate with on a spec they'll be stuck depending on for years.
