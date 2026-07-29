---
ads: 1
title: Transport security, redirect handling, and Security Considerations
status: Draft
type: Core
author: Viren Tanti
created: 2026-07-29
---

## Summary

Closes three gaps in ADS Core that make the spec unsafe to implement
literally and unready to register as a well-known URI. Adds a normative
`https` requirement for manifest fetches, defines what a client does with a
`3xx` on the well-known path (the spec currently says nothing), constrains
the DNS `ads=` pointer the same way, and adds
`spec/07-security-considerations.md`.

## Motivation

Three concrete gaps, each reachable today by a conformant implementation.

**1. No transport requirement.** `03-discovery-flow.md` writes the path as
`https://{host}/.well-known/agent-discovery.json` but never states that
`https` is required. A client that fetches over `http` is conformant. Since
a manifest names the endpoint an agent will call and the auth scheme it will
present, a network attacker who can rewrite that response chooses where the
agent sends its credential. The spec should not leave that to inference from
an example.

**2. Redirects are undefined.** The word "redirect" does not appear in
`spec/`. So this is currently conformant:

```
GET https://example.com/.well-known/agent-discovery.json
→ 302 Location: https://attacker.example/manifest.json
→ 200 {"capabilities":[{"name":"billing.charge","endpoint":{...}}]}
```

A client following that redirect attributes `attacker.example`'s
capabilities to `example.com`. Discovery exists to answer "what can *this*
origin do." Silent cross-origin redirect makes that question unanswerable,
which is a failure of the spec's core purpose, not merely a security bug.

**3. No Security Considerations.**
[RFC 8615 §4](https://www.rfc-editor.org/rfc/rfc8615.html#section-4) makes
this a review criterion for the well-known URI registry, and the Designated
Expert checks it. `SECURITY.md` is a vulnerability-reporting policy for this
repository — a different artifact answering a different question. Nothing in
`spec/` currently states ADS's threat model.

The DNS TXT mechanism carries gap 2 in a sharper form: `ads=` can point
anywhere, and unauthenticated DNS is spoofable by anyone who can answer the
query.

## Specification

Normative text as it should read in `spec/`. Applied in this branch.

**In `03-discovery-flow.md`, HTTP well-known requirements** — add: request
**MUST** use `https`; client **MUST NOT** fetch over plaintext `http` or
downgrade on TLS failure; single exception for loopback hosts (`localhost`,
`127.0.0.0/8`, `[::1]`) in local development.

**In `03-discovery-flow.md`, new "Redirects" subsection** — same-origin
`3xx` (per [RFC 6454](https://www.rfc-editor.org/rfc/rfc6454.html)) **MUST**
be followed; cross-origin `3xx` **MUST NOT** be followed silently (fail, or
surface the target origin for an explicit trust decision); chains **MUST**
cap at 5 hops; the attributed origin is always the one originally
requested, never the final one in the chain.

**In `03-discovery-flow.md`, DNS TXT mechanism** — `ads=` **MUST** be
`https`; cross-origin targets get the redirect rule; malformed records and
multi-`ads=` records **MUST** be ignored rather than arbitrarily resolved;
clients **SHOULD** prefer the well-known path when both exist.

**In `03-discovery-flow.md`, error-handling table** — six new rows covering
plaintext URLs, same-origin and cross-origin `3xx`, hop-limit exhaustion,
and cross-origin DNS pointers.

**New `spec/07-security-considerations.md`** — eight sections: manifests as
untrusted input; transport as the primary integrity control; origin
attribution; server-side disclosure; automated discovery as an injection
channel; signature verification; privacy; explicit out-of-scope.

No JSON Schema fragment accompanies this proposal: every change constrains
client and server *behavior*, and none adds, removes, or retypes a manifest
field. `manifest.schema.json` is untouched, and the existing examples under
`spec/schema/examples/` validate unchanged.

## Rationale

**Why MUST NOT follow cross-origin rather than MUST fail.** A hard fail
would break legitimate multi-CDN and vanity-domain setups. Requiring an
explicit trust decision preserves those while removing the silent case,
which is the actual vulnerability. This mirrors how OAuth 2.0 treats
`redirect_uri`: not forbidden, but never implicit.

**Why 5 hops.** Matches `fetch()`'s limit in the WHATWG Fetch Standard, so
browser-based clients inherit the correct behavior without special-casing.

**Why loopback is exempted from `https`.** Requiring TLS on `localhost`
means every local development setup provisions a certificate, and the usual
outcome is developers disabling verification globally — a worse end state
than the exemption. RFC 8252 §8.3 reaches the same conclusion for native
OAuth clients.

**Why a separate document rather than a section in 01-overview.** RFC 8615
review looks for a locatable Security Considerations section, and one
citable path is easier to reference from a registration template than a
subsection. It also matches how the numbered Core documents are already
organized.

**Divergence from A2A.** A2A's Agent Card leaves redirect handling to the
HTTP client. ADS diverges because ADS clients are frequently autonomous
agents rather than a browser enforcing its own policy, so leaving it
unstated means it goes unimplemented.

## Backward compatibility

Requires a **MINOR** bump: `0.1.0` → `0.2.0`. Per
[04-versioning-strategy.md](../spec/04-versioning-strategy.md#pre-10-note),
`0.x` MINOR bumps may carry small breaking changes where no known
implementation depends on the changed behavior. **This change is breaking
and is called out as such**, per that same rule.

- **Existing manifests:** no action. No schema change; every currently valid
  manifest stays valid.
- **Existing servers:** a server publishing over plaintext `http` on a
  non-loopback host becomes non-conformant and must move to `https`. A
  server relying on a cross-origin redirect to serve its manifest must serve
  it from its own origin instead.
- **Existing clients:** must add the `https` check, the redirect rules, and
  the 5-hop cap. A client that previously followed redirects transparently
  will now refuse some fetches it used to accept — deliberate, and the point
  of the change.
- **Deprecation window:** none applies. No feature is removed;
  under-specified behavior is being constrained. No known implementation
  depends on it.

## Reference implementation

None yet. This is the gap that most limits the proposal — and, separately,
the main obstacle to well-known URI registration, which weighs demonstrated
use. A reference client enforcing these rules should land before `Final`.

## Security considerations

This proposal is entirely security considerations; see
[spec/07-security-considerations.md](../spec/07-security-considerations.md).

The one residual risk worth stating: requiring an "explicit trust decision"
on cross-origin targets pushes a judgment onto the client, and a client that
implements that as an auto-accepted prompt has complied with the letter and
gained nothing. Core cannot prevent this — it can only decline to make the
unsafe path the default one.

## Copyright

Contributed under this repository's [MIT License](../LICENSE).
