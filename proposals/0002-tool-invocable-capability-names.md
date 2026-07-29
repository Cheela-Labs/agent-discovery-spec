---
ads: 2
title: Tool-invocable capability names
status: Final
type: Core
author: Cheela Labs
created: 2026-07-29
---

## Summary

A conformant ADS capability name can never be passed to an LLM tool-calling API.
The specification requires `namespacedName` to contain at least one dot; every
major tool-calling API rejects a function name containing a dot. Since the
overwhelmingly common use of a discovered capability is to hand it to a model as
a callable tool, every client faces the same undocumented mapping problem, and
each will solve it differently — which means the same manifest produces different
tool names in different clients.

This proposal adds an optional `invocationName` to the capability object, and
normative guidance for deriving one when it is absent.

## Motivation

`$defs.namespacedName` is:

```
^[A-Za-z][A-Za-z0-9-]{0,63}(\.[A-Za-z][A-Za-z0-9-]{0,63})+$
```

The trailing `+` makes at least one dot mandatory. So `com.example.lookupOrder`
is conformant and `lookupOrder` is not.

OpenAI's Chat Completions API — and every OpenAI-compatible endpoint, which in
practice includes OpenRouter, Together, Groq, and most self-hosted gateways —
constrains `tools[].function.name` to:

```
^[a-zA-Z0-9_-]{1,64}$
```

No dot. This is enforced, not advisory. A request carrying the manifest's own
capability name is rejected before the model is invoked:

```
HTTP 400
{"error":{"code":"invalid_request_error",
          "message":"tool function names must match ^[a-zA-Z0-9_-]{1,64}$",
          "param":"tools[0].function.name"}}
```

Anthropic's `tools[].name` and Google's `functionDeclarations[].name` impose
equivalent restrictions.

The two rules are mutually exclusive, not merely awkward: ADS *requires* the
character that tool APIs *forbid*. **No conformant capability name is usable as
a tool name.** This is not a property of one vendor, and it cannot be waited out.

The gap this leaves is not that clients cannot cope — any client can strip the
namespace — but that the specification does not say how, so clients that
interoperate on the manifest stop interoperating the moment they build tools
from it:

- A client that takes the last segment maps `com.example.orders.get` to `get`.
- A client that replaces dots maps it to `com-example-orders-get`.
- A client that takes the last two segments maps it to `orders-get`.

A manifest advertising both `com.example.orders.get` and `com.example.refunds.get`
collapses to one tool name under the first strategy, silently. The model then
sees two identical names, or one — and which capability actually runs depends on
the client.

There is also a length trap. `namespacedName` permits segments of 64 characters
each with no limit on segment count, so a conformant name can exceed the 64-character
tool-name ceiling on its own, before any transformation.

## Specification

The following text is proposed for `spec/02-manifest-schema.md`, in the
capability object section.

> ### `invocationName`
>
> Type: string. Optional.
>
> The name a client SHOULD use when exposing this capability to a system that
> constrains identifiers more tightly than `namespacedName` does — in particular
> an LLM tool-calling API, whose function names typically must match
> `^[a-zA-Z0-9_-]{1,64}$` and therefore cannot contain the dot that
> `namespacedName` requires.
>
> When present, `invocationName` MUST match `^[A-Za-z][A-Za-z0-9_-]{0,63}$` and
> MUST be unique across all capabilities in the manifest.
>
> When absent, a client that needs a constrained identifier MUST derive one by
> replacing every `.` in `name` with `-`, and MUST NOT derive one by truncating
> the name to a subset of its segments. Truncation is prohibited because two
> capabilities in the same manifest may share a leaf segment, and a client that
> truncates would map them to the same identifier without detecting the
> collision.
>
> If the derived identifier exceeds 64 characters, the client MUST treat the
> capability as not invocable through that interface and SHOULD surface the
> reason, rather than truncating to fit.
>
> `invocationName` is a presentation concern and carries no identity. `name`
> remains the sole identifier for a capability. Two manifests describing the same
> capability MUST agree on `name`; they need not agree on `invocationName`.

Schema fragment, added to `$defs.capability.properties`:

```json
{
  "invocationName": {
    "type": "string",
    "pattern": "^[A-Za-z][A-Za-z0-9_-]{0,63}$",
    "description": "Identifier to use when exposing this capability to a system with a constrained identifier syntax, such as an LLM tool-calling API. Presentation only; `name` remains the identity. See spec/02-manifest-schema.md."
  }
}
```

Example manifest exercising it — note the two capabilities that share a leaf
segment, which is exactly the case truncation gets wrong:

```json
{
  "specVersion": "0.2.0",
  "id": "com.example.shop",
  "name": "Example Shop",
  "provider": { "name": "Example Shop", "url": "https://shop.example" },
  "capabilities": [
    {
      "name": "com.example.shop.orders.get",
      "invocationName": "orders-get",
      "version": "1.0.0",
      "endpoint": { "transport": "http", "address": "https://shop.example/ads/orders-get", "auth": "none" }
    },
    {
      "name": "com.example.shop.refunds.get",
      "invocationName": "refunds-get",
      "version": "1.0.0",
      "endpoint": { "transport": "http", "address": "https://shop.example/ads/refunds-get", "auth": "none" }
    }
  ]
}
```

## Rationale

**Why not relax `namespacedName` to permit dotless names?** It would remove the
guarantee that names are globally unambiguous, which is the reason the reverse-DNS
shape was chosen. Two systems could then both publish `search` and a client
holding manifests from both would have no way to distinguish them. The dot is
load-bearing for identity; the problem is only that identity and invocation are
being served by one field.

**Why not require clients to derive silently, with no new field?** Deriving is
what clients do today, and the divergence above is the result. A derivation rule
alone would fix interoperability but takes the choice away from the publisher,
who may have a preferred short name, and offers no escape when the derived form
collides or overruns 64 characters.

**Why replacement rather than truncation as the default derivation?** Truncation
is lossy in a way that fails silently and unevenly — it only collides for some
manifests, so an implementation tests clean against its own examples and breaks
against someone else's. Replacement is total and collision-free by construction,
because it preserves every segment.

**Why not adopt MCP's approach?** MCP tool names live in a flat per-server
namespace and never carry a dotted prefix, so it does not have this problem to
solve — the server *is* the namespace. ADS deliberately makes names portable
across manifests, which is what creates the tension. OpenAPI's `operationId` is
the closer analogue: a separate, optional, presentation-level identifier
alongside the structural path, which is the shape adopted here.

**Why `MUST NOT` truncate rather than `SHOULD NOT`?** A truncating client and a
replacing client disagree about which capability a given tool call refers to.
That is a correctness failure across implementations, not a quality-of-implementation
matter, so it warrants a `MUST NOT`.

## Backward compatibility

**MINOR.** The change is purely additive: a new optional property on the
capability object.

- **Existing manifests:** remain valid unchanged. No republication required.
- **Existing clients:** keep working. A client unaware of `invocationName`
  ignores it, as `spec/06-backward-compatibility.md` already requires for
  unrecognised fields.
- **Deprecations/removals:** none. No deprecation window applies.

The normative derivation rule constrains behaviour that was previously
unspecified rather than changing behaviour that was specified, so a client
already deriving by dot-replacement is conformant on adoption. A client deriving
by truncation becomes non-conformant, which is the intended effect — but it is
worth naming plainly, because that is a real behaviour change for anyone who
chose truncation, even though it was never sanctioned.

## Reference implementation

Cheela's runtime rejects capability names that cannot be expressed as a tool
name at registration time, and its control plane publishes manifests whose leaf
segments are always derivable without collision:

- Name validation: `packages/sdk/src/utils/capability-name.ts` in
  [Cheela-Labs/platform](https://github.com/Cheela-Labs/platform)
- Manifest generation: `packages/adp/src/map.ts`

This is a stricter posture than this proposal requires — Cheela forbids at
authoring time what the proposal makes derivable at read time — and it exists
because the failure was found in production: capabilities named `catalog.search`
deployed cleanly, published a conformant manifest, and then failed with the 400
above on the first real execution. Nothing in the toolchain caught it, because
nothing between authoring and the provider call had any reason to.

## Security considerations

`invocationName` carries no identity and MUST NOT be used for authorization,
routing, or capability lookup — a client that resolves an incoming tool call
back to a capability MUST do so via `name`. Treating a presentation-level,
publisher-controlled, non-unique-across-manifests field as an identifier would
let a manifest choose an `invocationName` matching an unrelated capability in a
different manifest a client had also loaded.

The uniqueness requirement is per-manifest and enforceable by a validator. Cross-manifest
collisions are expected and harmless as long as the rule above holds.

## Copyright

By submitting this proposal you agree it's contributed under this repository's
[MIT License](../LICENSE), same as every other document here — no separate
copyright assignment or CLA required.
