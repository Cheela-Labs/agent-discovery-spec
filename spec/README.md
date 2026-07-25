# ADS Core Specification

Current version: **0.1.0** (`Draft`)

This directory is the normative specification text. Read in order if you're
new:

1. [01-overview.md](01-overview.md) — terminology, scope, conformance levels
2. [02-manifest-schema.md](02-manifest-schema.md) — the manifest, field by field
3. [03-discovery-flow.md](03-discovery-flow.md) — how clients find and validate a manifest
4. [04-versioning-strategy.md](04-versioning-strategy.md) — the three version numbers and what bumps each
5. [05-extensibility.md](05-extensibility.md) — namespacing, `extensions`, the namespace registry
6. [06-backward-compatibility.md](06-backward-compatibility.md) — deprecation policy, the additive-evolution rule

Machine-readable schema and examples: [schema/](schema/).

This is **ADS Core** — the required baseline. There are no ADS Extension
documents yet; once an Extension-track proposal reaches `Final`, it'll be
added here under `spec/extensions/` without changing anything above.

Changes to any of this go through [../proposals/](../proposals/), never a
direct PR — see [../CONTRIBUTING.md](../CONTRIBUTING.md).
