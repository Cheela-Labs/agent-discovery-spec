# ADS Core — 05. Extensibility

Normative. This is the mechanism that lets ADS stay small: instead of
anticipating every field every implementer will ever want, Core defines a
narrow set of required fields and a disciplined way to add more without
touching Core itself.

## The one rule everything else follows

> A Core-conformant client **MUST NOT** fail to process a manifest solely
> because it contains a field, capability, `transport`, `auth` value, or
> `extensions` entry it doesn't recognize. It ignores what it doesn't
> understand and uses what it does.

Every extensibility mechanism below exists to make that rule safe to follow
— i.e., to guarantee that "ignore what you don't recognize" never means
"silently misinterpret something important as something harmless."

## Namespacing

Every `capabilities[].name` and every key under `manifest.extensions` **MUST**
be namespaced: at least two dot-separated segments, reverse-DNS style
(`com.example.thing`), matching the `namespacedName` pattern in
[schema/manifest.schema.json](schema/manifest.schema.json).

- The namespace portion (all segments except the last) **MUST** be compared
  case-insensitively, matching DNS semantics — `Com.Example.foo` and
  `com.example.foo` are the same namespace.
- The leaf segment MAY use camelCase for readability
  (`com.example.lookupOrder`) — it does not carry collision-avoidance
  weight the way the namespace prefix does.
- `core.*` is reserved. It's for capability categories that ADS itself
  standardizes in a future proposal (none exist yet in Core) — implementers
  MUST NOT use `core.*` for vendor-specific capabilities.
- Anyone can mint a namespace they control (if you own `example.com`,
  `com.example.*` is yours by construction — same trust model as Java
  package names or reverse-DNS app IDs). No registration is required to
  *use* a namespace this way.

### Namespace registry (optional, for short prefixes)

Reverse-DNS namespacing is collision-safe but verbose. For projects that
want a short, memorable prefix instead of spelling out a domain every time,
[registry/namespaces.json](../registry/namespaces.json) is a first-come,
first-served registry of short prefixes, maintained by lightweight PR (no
full proposal process needed — see
[registry/README.md](../registry/README.md)).

Registering a short prefix is optional. Using your own reverse-DNS domain
requires no registration at all and is equally valid.

## The `extensions` block

`manifest.extensions` is an open object. Keys **SHOULD** be
`{namespace}/{extensionVersion}` (e.g. `"com.example.billing/1.0"`) so a
single system can expose multiple versions of the same extension
simultaneously during a migration, and so a client can tell at a glance
which shape to expect without parsing the contents first.

An extension's internal shape is defined by whoever owns that namespace —
not by ADS Core. If an extension becomes widely used enough that it's worth
standardizing, propose it as an **Extension-track** ADS proposal (see
[proposals/0000-process.md](../proposals/0000-process.md)); a `Final`
Extension proposal gets documented alongside Core (in
`spec/extensions/`, once any exist) but still does not become part of Core
itself — extensions never gain required-field status.

## Well-known open enums

`capabilities[].endpoint.transport` and `.auth` are open strings, not closed
enums, precisely so a new transport or auth scheme never requires a Core
version bump. New well-known values get documented (not "approved" — there's
no gate) via a short PR to the well-known-values tables in
[02-manifest-schema.md](02-manifest-schema.md), purely so tooling authors
have one place to look them up. A value that never gets documented there is
still perfectly legal to use; it just means fewer clients will recognize it
out of the box.

Custom auth schemes not on the well-known list **MUST** use a `custom:`
prefix (e.g. `custom:hmac-sig`) so clients can distinguish "an auth scheme I
don't recognize" from "a typo of one I do."

## What extensibility does not cover

Changing the *meaning* of an existing Core field, or making a currently
optional Core field required, is not an extension — it's a Core change and
goes through the full proposal process with a MAJOR version bump, same as
any other breaking change (see
[04-versioning-strategy.md](04-versioning-strategy.md)). Extensibility adds;
it does not let you route around the versioning rules for changing what's
already there.
