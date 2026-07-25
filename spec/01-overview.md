# ADS Core — 01. Overview

## Status of this document

This document is part of **ADS Core**, the required baseline every
conformant implementation must satisfy. It is normative unless marked
otherwise.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD
NOT**, and **MAY** in this document are to be interpreted as described in
[RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

## What this specification is

The Agent Discovery Specification (ADS) defines two things:

1. A **capability manifest** format — a small, self-describing, versioned
   JSON document that says what an agent-addressable system can do.
2. A **discovery flow** — how a client finds that manifest, without prior
   out-of-band knowledge beyond an entry point (a URL, a domain name, or a
   local path).

Everything else — how you authenticate to a capability, what wire format you
use to invoke it, what the model behind it does — is explicitly **out of
scope**. ADS answers "what exists and how do I start," not "how do I finish."

## What this specification is not

- **Not an invocation protocol.** ADS doesn't define request/response shapes
  for calling a capability — it points at an `endpoint`, and the transport
  named there (HTTP+REST, MCP, gRPC, etc.) owns the invocation contract.
- **Not an auth protocol.** A manifest declares *that* a capability requires
  `apiKey` or `oauth2`, not how to complete that flow.
- **Not a model or agent framework.** ADS doesn't care if a capability is
  backed by an LLM, a deterministic function, a human in the loop, or
  another agent.
- **Not a registry or marketplace.** Nothing here defines how manifests get
  listed, ranked, or discovered in aggregate — only how a client resolves
  one specific system's manifest.

## Terminology

**Manifest** — the JSON document defined in
[02-manifest-schema.md](02-manifest-schema.md) that describes a system's
capabilities.

**System** — the thing being described: an agent, a runtime, a tool server,
a single function. ADS is deliberately agnostic about what's actually behind
the manifest.

**Capability** — one discrete, named thing a system can do, as declared in
the manifest's `capabilities` array.

**Provider** (in `manifest.provider`) — whoever operates the system
being described. **This is not the same thing as an LLM/model provider**
(OpenAI, Anthropic, etc.) — a manifest for a system built on top of an LLM
describes the *system's* operator here, and may separately note model
details inside a capability's own metadata or an extension block if that's
relevant to the client.

**Client** — anything performing discovery: a human developer's tooling, an
orchestrator, another agent, a registry crawler.

**Extension** — additive, namespaced data attached under
`manifest.extensions`, defined outside ADS Core (see
[05-extensibility.md](05-extensibility.md)).

**Namespace** — the dotted prefix (e.g. `com.example.*`) that scopes a
capability or extension name to avoid collisions.

## Two separate numbering systems — don't confuse them

This trips people up coming from EIPs, so it's called out explicitly:

- **Spec version** (`0.1.0`, `1.0.0`, ...) — the SemVer version of the ADS
  Core document set and schema itself. This is what a manifest's
  `specVersion` field references. See
  [04-versioning-strategy.md](04-versioning-strategy.md).
- **Proposal number** (`ADS-0`, `ADS-1`, ...) — the sequential ID assigned to
  a *change proposal* document under `proposals/`, the same way an EIP
  number identifies a proposal, not a version. A single spec version bump
  (e.g. `0.1.0 → 0.2.0`) may fold in several `Final` proposals at once.

A manifest never references an ADS proposal number directly — only the spec
version.

## Document structure

| Doc | Normative? | Covers |
|---|---|---|
| 01-overview.md | Partially (terminology + scope are normative; framing is not) | This document |
| 02-manifest-schema.md | Yes | The manifest format, field by field |
| 03-discovery-flow.md | Yes | How clients find and validate a manifest |
| 04-versioning-strategy.md | Yes | SemVer rules for spec, manifest, and capability versions |
| 05-extensibility.md | Yes | Namespacing rules, the `extensions` block, namespace registry |
| 06-backward-compatibility.md | Yes | Deprecation policy, multi-version serving |
| schema/manifest.schema.json | Yes — this is the authoritative source | Machine-readable JSON Schema |

If prose and the JSON Schema ever disagree, the JSON Schema in
`spec/schema/manifest.schema.json` is authoritative for structural
validation; the prose is authoritative for behavioral (MUST/SHOULD)
requirements the schema can't express (like caching semantics).

## Conformance levels

- **Core-conformant** — implements everything in this document set with no
  extension blocks. This is the minimum bar; a client that only understands
  ADS Core MUST be able to fully parse and act on a Core-conformant
  manifest.
- **Extended** — Core-conformant, plus one or more namespaced extension
  blocks under `manifest.extensions`. A client that doesn't understand a
  given extension MUST still be able to use every Core field normally (see
  [05-extensibility.md](05-extensibility.md) for the exact tolerance rule).

There is no "partial Core conformance" — an implementation either satisfies
every MUST in ADS Core or it isn't ADS-conformant. This is a deliberately
small, deliberately strict baseline so the one thing ADS promises (you can
always at least discover *something* usable) never gets watered down by
optional-required fields.
