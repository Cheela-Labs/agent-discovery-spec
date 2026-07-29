# ADS Core — 02. Manifest Schema

Normative. The authoritative machine-readable form of everything in this
document is [schema/manifest.schema.json](schema/manifest.schema.json) — if
they disagree on structure, the JSON Schema wins.

## Top-level fields

| Field | Type | Required | Description |
|---|---|---|---|
| `specVersion` | string (SemVer) | **Yes** | The ADS Core version this manifest conforms to, e.g. `"0.1.0"`. See [04-versioning-strategy.md](04-versioning-strategy.md). |
| `id` | string | **Yes** | Stable identifier for the system, unique within whatever scope the client discovered it in. SHOULD be reverse-DNS style (`com.example.support-agent`) or a URN. MUST NOT change across manifest revisions — it's how a client recognizes "same system, new manifest" versus "different system." |
| `name` | string | **Yes** | Human-readable display name. |
| `description` | string | No | Human-readable summary of what the system does. |
| `provider` | object | **Yes** | Who operates this system. See below. Not the LLM/model provider — see [01-overview.md](01-overview.md#terminology). |
| `capabilities` | array | **Yes** | The list of things this system can do. MAY be empty (e.g. a system temporarily offering nothing), but SHOULD be non-empty in normal operation. |
| `discovery` | object | No | Metadata about the manifest itself — caching, integrity. See below. |
| `extensions` | object | No | Namespaced, additive data outside ADS Core. See [05-extensibility.md](05-extensibility.md). |
| `lastUpdated` | string (RFC 3339 date-time) | No | When this manifest content was last changed. Purely advisory — clients MUST NOT treat manifests as stale based on this alone; use `discovery.cacheTtlSeconds` and standard HTTP caching instead. |

### `provider`

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | **Yes** | Operator's display name. |
| `url` | string (URI) | No | Operator's homepage or documentation root. |
| `contact` | string | No | Email address or URL for reports/questions about this system. |

### `capabilities[]`

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | **Yes** | Namespaced capability name (e.g. `com.example.lookupOrder`). MUST follow the namespacing rules in [05-extensibility.md](05-extensibility.md). |
| `invocationName` | string | No | Identifier to use where `name` cannot be, such as an LLM tool-calling API. Presentation only — `name` remains the identity. See [below](#invocationname). |
| `version` | string (SemVer) | **Yes** | Version of this specific capability's contract — independent of `specVersion`. Bump this when `inputSchema`/`outputSchema`/behavior changes, not when the manifest wrapper changes. |
| `description` | string | No | Human-readable summary. |
| `inputSchema` | object (JSON Schema) | No | JSON Schema describing valid input to this capability. Absence means "consult the transport/endpoint documentation" — not "no input." |
| `outputSchema` | object (JSON Schema) | No | JSON Schema describing this capability's output shape. |
| `endpoint` | object | **Yes** | Where and how to invoke this capability. See below. |
| `deprecated` | boolean | No | Defaults to `false`. See [06-backward-compatibility.md](06-backward-compatibility.md) for the full deprecation lifecycle. |
| `deprecatedSince` | string (SemVer) | Required if `deprecated: true` | The capability `version` at which deprecation was announced. |
| `removalNotBefore` | string (SemVer) | No | The earliest capability `version` at which this MAY be removed. |

#### `invocationName`

`name` is required to contain at least one dot. Most LLM tool-calling APIs
constrain function names to `^[a-zA-Z0-9_-]{1,64}$` and reject a dot outright,
so a conformant `name` can never be passed to one directly. `invocationName`
carries the identifier to use in those places.

When present it MUST match `^[A-Za-z][A-Za-z0-9_-]{0,63}$` and MUST be unique
across all capabilities in the manifest.

When absent, a client that needs a constrained identifier MUST derive one by
replacing every `.` in `name` with `-`, and MUST NOT derive one by truncating
`name` to a subset of its segments. Truncation is prohibited because two
capabilities in one manifest may share a leaf segment — `com.example.orders.get`
and `com.example.refunds.get` both truncate to `get` — and a client that
truncates maps them to the same identifier without detecting the collision.

If the derived identifier exceeds 64 characters, a client MUST treat the
capability as not invocable through that interface and SHOULD surface the
reason, rather than truncating to fit.

`invocationName` carries no identity and MUST NOT be used for authorization,
routing, or capability lookup; a client resolving an incoming tool call back to
a capability MUST do so via `name`. Two manifests describing the same capability
MUST agree on `name`; they need not agree on `invocationName`.

#### `endpoint`

| Field | Type | Required | Description |
|---|---|---|---|
| `transport` | string | **Yes** | How to invoke this capability. Not a closed enum — see [Well-known transport values](#well-known-transport-values) below. A client that doesn't recognize a `transport` value MUST skip that capability, not fail the whole manifest. |
| `address` | string | **Yes** | Transport-specific locator — a URL for `http`/`grpc`, a server identifier for `mcp`, etc. |
| `auth` | string | **Yes** | The auth scheme required, at the level of "which kind," not "here are your credentials." One of the [well-known auth values](#well-known-auth-values), or a namespaced custom value. |
| `authDetails` | object | No | Free-form, auth-scheme-specific hints (e.g. an OAuth2 `authorizationUrl`/`tokenUrl`/`scopes` block). Not standardized further at Core level — see extension mechanism if you need a normative shape for a specific auth scheme. |

##### Well-known `transport` values

`http`, `grpc`, `mcp`, `queue`. This list is intentionally short and
intentionally open — new transports don't need a spec change, they just need
clients to recognize the string. Document new well-known values via a
lightweight proposal so tooling has somewhere to look them up; unrecognized
values are always legal, just unusable by clients that don't know them.

##### Well-known `auth` values

`none`, `apiKey`, `oauth2`, `mtls`. Same openness rule as `transport` —
`custom:` prefixed values (e.g. `custom:hmac-sig`) are legal for anything
that doesn't fit, at the cost of fewer clients being able to use it
out of the box.

### `discovery`

| Field | Type | Required | Description |
|---|---|---|---|
| `cacheTtlSeconds` | integer | No | How long a client MAY cache this manifest before revalidating. Absent means "use standard HTTP caching headers if the transport has them, otherwise don't assume it's safe to cache at all." |
| `signature` | object | No | Optional integrity block — see [03-discovery-flow.md](03-discovery-flow.md#manifest-integrity) for how clients use it. |

## What clients MUST tolerate

Per [05-extensibility.md](05-extensibility.md), a Core-conformant client
**MUST NOT** fail to parse a manifest solely because it contains:

- an unrecognized top-level field
- an unrecognized `capabilities[].endpoint.transport` or `.auth` value (it
  just can't use that particular capability)
- a populated `extensions` block it doesn't understand

Failing on any of the above is a conformance bug, not strictness.

## Minimal valid manifest

```json
{
  "specVersion": "0.1.0",
  "id": "com.example.minimal",
  "name": "Minimal Example",
  "provider": { "name": "Example Inc." },
  "capabilities": []
}
```

See [schema/examples/](schema/examples/) for a minimal and a fully-populated
example, both of which validate against
[schema/manifest.schema.json](schema/manifest.schema.json).
