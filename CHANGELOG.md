# Changelog

All notable changes to the AI Discovery Specification are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/);
versioning follows the rules in
[spec/04-versioning-strategy.md](spec/04-versioning-strategy.md).

## [0.1.0] — 2026-03-01

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
- HTTP well-known discovery (`/.well-known/ai-discovery.json`), DNS TXT
  record discovery, and inline/local discovery mechanisms
- Namespace registry (`registry/`) for short capability/extension prefixes
- ADS proposal process (`proposals/0000-process.md`) and template
- Project governance, contribution guide, code of conduct, and security
  policy
