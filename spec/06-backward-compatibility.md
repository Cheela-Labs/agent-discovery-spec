# ADS Core — 06. Backward Compatibility Strategy

Normative. Versioning rules (previous document) say *when* something counts
as breaking. This document says what implementations **MUST** and **SHOULD**
do so that a breaking change never arrives as a surprise to a client that
was behaving correctly.

## The golden rule

> A client that only reads the fields it already understood from a previous
> MINOR/PATCH version of a manifest **MUST** continue to work correctly
> against every later MINOR/PATCH version within the same MAJOR.

This is the whole strategy, restated as a test: if shipping a change would
make a well-behaved, already-deployed client start behaving incorrectly
without that client changing anything, the change is MAJOR — full stop,
no matter how small it looks in a diff.

## Deprecating a capability

A capability is deprecated in place, not silently removed:

```json
{
  "name": "com.example.legacyCancelOrder",
  "version": "1.0.0",
  "deprecated": true,
  "deprecatedSince": "1.0.0",
  "removalNotBefore": "2.0.0",
  "endpoint": { "...": "..." }
}
```

Rules:

- `deprecated: true` **MUST** be accompanied by `deprecatedSince` (see
  [02-manifest-schema.md](02-manifest-schema.md#capabilities)).
- A deprecated capability **MUST** remain fully functional — deprecation is
  a signal, not a degradation. Silently returning errors from a
  "deprecated" capability is a spec violation; removing it entirely is what
  `removalNotBefore` governs.
- The gap between `deprecatedSince` and `removalNotBefore` **MUST** be at
  least one full MAJOR version of that capability. Deprecating in `1.3.0`
  means the earliest legal removal is in a `2.x` release of that
  capability, never a later `1.x`.
- Systems **SHOULD** additionally give a minimum wall-clock deprecation
  window (six months is a reasonable default absent a stronger constraint)
  in addition to the version-based rule, since capability version bumps and
  calendar time don't always move together.

The same pattern applies to deprecating an entire manifest field in a
future Core version, via the corresponding proposal's own migration
section — see [proposals/TEMPLATE.md](../proposals/TEMPLATE.md).

## Serving multiple spec versions concurrently

A system that needs to support clients on different ADS Core MAJOR versions
through a migration window MAY serve version-pinned manifests alongside the
default:

```text
/.well-known/ai-discovery.json        → latest supported specVersion
/.well-known/ai-discovery.v1.json     → pinned to the latest 1.x
/.well-known/ai-discovery.v2.json     → pinned to the latest 2.x
```

This is optional — a system MAY instead just run two MAJOR versions in
parallel until old clients age out, then drop the old one. Both are
conformant; which to pick is an operational decision, not a spec one.

## Version negotiation (optional)

A system MAY support content negotiation for clients that want to state
their supported range up front, via an `Accept` header parameter:

```text
GET /.well-known/ai-discovery.json
Accept: application/json; ads-version=^1.2
```

A system that doesn't implement negotiation **MUST** ignore the parameter
rather than reject the request — negotiation is additive, not a gate.
Absent negotiation, a client falls back to fetching the default manifest
and checking `specVersion` itself per
[03-discovery-flow.md](03-discovery-flow.md).

## What's never allowed, even across a MAJOR bump

Two things are treated as compatibility violations regardless of version
number, because they break the *trust* model rather than just the schema:

1. **Reusing `id` for a materially different system.** `id` is how a client
   recognizes "same system, new manifest" — repurposing it breaks every
   client's identity assumptions, not just their schema assumptions.
2. **Silently changing what a capability does without bumping its
   `version`.** A capability's behavior and its declared `version` are a
   contract; changing one without the other is not a compatibility
   question, it's simply non-conformant.

## Summary checklist for implementers shipping a breaking change

- [ ] Confirmed it's actually MAJOR per [04-versioning-strategy.md](04-versioning-strategy.md), not something that could ship additively instead
- [ ] Existing capabilities marked `deprecated` before removal, not removed directly
- [ ] `removalNotBefore` gives at least one full MAJOR version of runway
- [ ] Considered whether version-pinned manifest paths are worth offering during the transition
- [ ] Changelog entry written *before* the release, not after
