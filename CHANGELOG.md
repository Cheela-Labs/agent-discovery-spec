# Changelog

All notable changes to the Agent Discovery Specification are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/);
versioning follows the rules in
[spec/04-versioning-strategy.md](spec/04-versioning-strategy.md).

## [0.3.0] — 2026-07-29

Adds `invocationName`, resolving a conflict between capability names and every
major LLM tool-calling API. No breaking changes — every manifest valid under
0.2.0 remains valid under 0.3.0.

### Added

- `capabilities[].invocationName` — the identifier to use where `name` cannot
  be. `name` is required to contain a dot; OpenAI, Anthropic, Google and every
  OpenAI-compatible endpoint constrain tool function names to
  `^[a-zA-Z0-9_-]{1,64}$` and reject a dot outright, so no conformant `name`
  could ever be passed to one. Optional, presentation only — `name` remains the
  sole identity. See [ADS-2](proposals/0002-tool-invocable-capability-names.md).

- A normative derivation rule for clients that need a constrained identifier and
  find no `invocationName`: replace dots with hyphens, never truncate to a
  subset of segments. Clients were already deriving one and disagreeing —
  last segment, dot replacement, last two segments — so the same manifest
  produced different tool names in different clients. Truncation is now
  prohibited rather than discouraged, because two capabilities in one manifest
  may share a leaf segment and a truncating client maps them to the same
  identifier silently.

### Changed

- `spec/schema/manifest.schema.json` `$id` now carries the current version. It
  had read `0.1.0` since that release; 0.2.0 changed no schema so it went
  unnoticed.

## [0.2.0] — 2026-07-29

Hardens the discovery flow and adds the Security Considerations document.
**Contains breaking changes**, permitted pre-1.0 per the
[pre-1.0 note](spec/04-versioning-strategy.md#pre-10-note) and called out
explicitly here as that rule requires. No schema change — every manifest
valid under 0.1.0 remains valid under 0.2.0.

### Added

- [spec/07-security-considerations.md](spec/07-security-considerations.md) —
  threat model, manifests as untrusted input, transport as the primary
  integrity control, origin attribution, server-side disclosure, automated
  discovery as an injection channel, signature verification, privacy, and
  explicit out-of-scope boundaries. Required for well-known URI registration
  under [RFC 8615 §4](https://www.rfc-editor.org/rfc/rfc8615.html#section-4).
- Normative redirect handling on the well-known path: same-origin followed,
  cross-origin never followed silently, 5-hop cap, manifest always attributed
  to the origin originally requested.
- Six error-handling rows covering plaintext URLs, redirect outcomes, hop
  exhaustion, and cross-origin DNS pointers.
- `ietf/well-known-registration.md` — draft IANA registration template for
  `agent-discovery.json`. Not submitted; see the blockers in that file.

### Changed

- **Breaking.** Manifest fetches now **MUST** use `https`. Plaintext `http`
  on a non-loopback host is no longer conformant, and clients **MUST NOT**
  downgrade on TLS failure. Loopback hosts are exempt for local development.
- **Breaking.** The DNS `ads=` pointer **MUST** be `https` and is subject to
  the cross-origin rule. Malformed and multi-`ads=` records **MUST** be
  ignored rather than arbitrarily resolved. Clients **SHOULD** prefer the
  well-known path where both exist.

### Migration

- Manifests: no action.
- Servers: move any non-loopback plaintext manifest to `https`; serve the
  manifest from your own origin rather than via cross-origin redirect.
- Clients: add the `https` check, the redirect rules, and the 5-hop cap.

## [0.1.0] — 2026-07-25

Initial draft release. Not yet `1.0.0` — expect breaking changes without a
deprecation window while the spec stabilizes, per the pre-1.0 note in
[spec/04-versioning-strategy.md](spec/04-versioning-strategy.md#pre-10-note).

### Added

- ADS Core specification: [01-overview](spec/01-overview.md),
  [02-manifest-schema](spec/02-manifest-schema.md),
  [03-discovery-flow](spec/03-discovery-flow.md),
  [04-versioning-strategy](spec/04-versioning-strategy.md),
  [05-extensibility](spec/05-extensibility.md),
  [06-backward-compatibility](spec/06-backward-compatibility.md)
- Normative JSON Schema at `spec/schema/manifest.schema.json`, with a
  minimal and a full-featured example under `spec/schema/examples/`
- HTTP well-known discovery (`/.well-known/agent-discovery.json`), DNS TXT
  record discovery, and inline/local discovery mechanisms
- Namespace registry (`registry/`) for short capability/extension prefixes
- ADS proposal process (`proposals/0000-process.md`) and template
- Project governance, contribution guide, code of conduct, and security
  policy
