# AI Discovery Specification (ADS)

**Status:** Draft · **Spec version:** 0.1.0 · **License:** MIT

A small, open, provider-agnostic specification for discovering what an
AI-addressable system can do — its capabilities, how to invoke them, what
they require, and how to keep asking that question safely as the system
changes over time.

If you've ever wanted `/.well-known/openid-configuration`, but for "what can
this agent, runtime, or tool server actually do" — this is that document.

---

## Why this exists

Every AI provider, runtime, and agent framework has grown its own private
answer to "what can you do and how do I call it": OpenAI has tool schemas,
Anthropic has tool definitions, MCP servers list tools over their own
protocol, and every agent framework invents its own registration format.
None of them agree, and none of them are meant to be discovered *before* you've
already committed to that provider's SDK.

The AI Discovery Specification defines one thing, and only one thing: a
**capability manifest** — a small, versioned, self-describing document — and
the **discovery flow** for finding it. It does not define how you invoke a
capability once you've found it (that's a transport concern — HTTP, MCP,
gRPC, a message queue, whatever the manifest points at). It defines how a
client learns that the capability exists, what it needs, and whether the
client and the system still agree on what any of that means.

## Design goals

- **Provider agnostic** — says nothing about which model, vendor, or API
  shape sits behind a capability.
- **Runtime agnostic** — works for a single serverless function, a
  long-running agent process, or a fleet behind a load balancer.
- **Framework agnostic** — doesn't assume LangChain, MCP, OpenAI's tool
  format, or anything else. Any of them can be *described by* a manifest.
- **Open source** — MIT licensed, no CLA, no trademark gate on
  implementations.
- **Community driven** — changes go through the proposal process in
  [proposals/](proposals/), not a single vendor's roadmap.
- **Extensible** — new capability categories and vendor-specific data attach
  without touching the core schema or bumping a major version.
- **Versioned** — both the specification itself and every manifest instance
  carry an explicit version, so implementations can evolve without breaking
  each other.

## The shape of it, in one example

A conformant system serves a manifest, typically at
`/.well-known/ai-discovery.json`:

```jsonc
{
  "specVersion": "0.1.0",
  "id": "com.example.support-agent",
  "name": "Example Support Agent",
  "description": "Handles order lookup and refund requests.",
  "provider": { "name": "Example Inc.", "url": "https://example.com" },
  "capabilities": [
    {
      "name": "com.example.lookupOrder",
      "version": "1.0.0",
      "description": "Look up an order by id.",
      "inputSchema": { "type": "object", "properties": { "orderId": { "type": "string" } }, "required": ["orderId"] },
      "endpoint": { "transport": "http", "address": "https://api.example.com/v1/orders/lookup", "auth": "apiKey" }
    }
  ],
  "discovery": { "cacheTtlSeconds": 3600 }
}
```

A client fetches that document, checks `specVersion` against what it
understands, picks a capability by name and version, and calls whatever
`endpoint` says to call. That's the whole flow — see
[spec/03-discovery-flow.md](spec/03-discovery-flow.md) for the full version,
including non-HTTP discovery and caching rules.

## Reading the spec

| Document | What it covers |
|---|---|
| [spec/01-overview.md](spec/01-overview.md) | Terminology, conformance levels, document structure |
| [spec/02-manifest-schema.md](spec/02-manifest-schema.md) | The capability manifest, field by field |
| [spec/03-discovery-flow.md](spec/03-discovery-flow.md) | How a client finds and validates a manifest |
| [spec/04-versioning-strategy.md](spec/04-versioning-strategy.md) | SemVer rules for the spec, manifests, and individual capabilities |
| [spec/05-extensibility.md](spec/05-extensibility.md) | Namespacing, the `extensions` block, the namespace registry |
| [spec/06-backward-compatibility.md](spec/06-backward-compatibility.md) | Deprecation policy, multi-version serving, the additive-evolution rule |
| [spec/schema/manifest.schema.json](spec/schema/manifest.schema.json) | The normative JSON Schema |
| [proposals/](proposals/) | The ADS proposal process — how the spec itself changes |

## Relationship to Cheela

This specification does not depend on [Cheela](https://cheelalabs.com) or
any other product. It's designed to be adopted by any platform that wants a
standard discovery story for the AI systems it exposes.

Cheela is the first production implementer — its runtime registry publishes
ADS-conformant manifests for every registered runtime — but that's a
statement about who built something on top of this spec first, not a
statement about ownership. The spec is versioned, governed, and evolved
independently, the way TC39 doesn't belong to V8 and OpenAPI doesn't belong
to Swagger.

## Versioning

The specification follows semantic versioning (see
[spec/04-versioning-strategy.md](spec/04-versioning-strategy.md) for the full
rules):

- **MAJOR** — breaking changes to required manifest fields or the discovery
  flow
- **MINOR** — new optional, additive capabilities
- **PATCH** — clarifications and non-normative fixes

Every manifest declares the spec version it conforms to (`specVersion`), so
old and new manifests can coexist while clients migrate.

## Contributing

Proposals to change or extend the spec go through the process in
[CONTRIBUTING.md](CONTRIBUTING.md) and
[proposals/0000-process.md](proposals/0000-process.md) — modeled closely on
how Ethereum's EIPs work, adapted for a much smaller spec.

## License

[MIT](LICENSE). See [GOVERNANCE.md](GOVERNANCE.md) for how decisions get
made and who has merge authority.
