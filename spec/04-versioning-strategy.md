# ADS Core — 04. Versioning Strategy

Normative. There are **three independent version numbers** in this
specification, each answering a different question. Conflating them is the
single most common source of confusion in specs like this one, so each gets
its own subsection and its own explicit "what bumps this" table.

## 1. Spec version (`specVersion`)

**Question it answers:** "Which version of ADS Core does this manifest
follow?"

Follows [SemVer 2.0.0](https://semver.org/): `MAJOR.MINOR.PATCH`.

| Change | Bump | Examples |
|---|---|---|
| Remove or rename a required top-level or nested field | MAJOR | Renaming `provider.name` to `provider.displayName` |
| Change the meaning of an existing field | MAJOR | Redefining `discovery.cacheTtlSeconds` to mean milliseconds instead of seconds |
| Change the discovery flow in a way old clients can't follow | MAJOR | Requiring a new mandatory header on the well-known request |
| Add a new optional field | MINOR | Adding `capabilities[].costHint` |
| Add a new well-known `transport` or `auth` value | MINOR | Adding `websocket` as a well-known transport |
| Clarify ambiguous prose without changing behavior | PATCH | Rewording a MUST for clarity |
| Fix a schema bug that rejected previously-intended-valid manifests | PATCH | Loosening an overly strict regex |
| Fix a schema bug that accepted previously-unintended-invalid manifests | MINOR (see below) | Tightening a regex that was too permissive |

That last row is the one edge case worth calling out explicitly: tightening
validation is technically a breaking change for anyone who was (even
accidentally) relying on the old looseness, but it's needed to fix a genuine
defect. These ship as MINOR with a clear changelog entry and, if the impact
looks nontrivial, a deprecation window per
[06-backward-compatibility.md](06-backward-compatibility.md) rather than an
immediate hard rejection.

A manifest's `specVersion` MUST reflect the highest ADS Core version whose
MUST requirements it fully satisfies. A client that only understands
`specVersion` `0.x` MUST treat an unrecognized future MAJOR (`1.x`, `2.x`,
...) as "I cannot process this manifest," not as "close enough, try anyway."
Two manifests differing only in MINOR/PATCH MUST be mutually
processable by any client that understands the older one, per the additive
evolution rule in [06-backward-compatibility.md](06-backward-compatibility.md).

## 2. Capability version (`capabilities[].version`)

**Question it answers:** "Has *this specific capability's* contract
changed?"

Independent of `specVersion` — bumping the spec version doesn't require
touching every capability's version, and vice versa. Also SemVer:

| Change | Bump |
|---|---|
| Remove a required input field, change output shape incompatibly, change what the capability does | MAJOR |
| Add an optional input field, add new output fields, widen accepted input | MINOR |
| Fix a description typo, clarify `inputSchema` without changing what validates | PATCH |

A client SHOULD pin to a capability version range the same way it would pin
a package dependency, and treat a capability's MAJOR bump as "this needs
re-integration," not "this will probably still work."

## 3. Proposal number (`ADS-N`)

**Question it answers:** "Which change proposal am I referencing?"

Not a version — a sequential, permanent identifier assigned once to a
proposal document, the way an EIP number or an RFC number never gets
reused or incremented. See [proposals/0000-process.md](../proposals/0000-process.md).
A single spec version bump may correspond to zero, one, or several `Final`
proposals landing together.

## How a spec version release actually happens

1. One or more proposals reach `Final` (see
   [GOVERNANCE.md](../GOVERNANCE.md)).
2. An editor determines the resulting bump level using the tables above —
   the highest bump level among the included proposals wins (one MAJOR
   proposal in a batch means the whole release is MAJOR).
3. `spec/`, `spec/schema/manifest.schema.json`, and `CHANGELOG.md` are
   updated together in one release PR that references every included
   proposal by number.
4. The release is tagged (`v0.1.0`, `v0.2.0`, ...) and the new
   `specVersion` becomes valid for manifests to declare.

## Pre-1.0 note

While `specVersion` is `0.x`, MINOR bumps MAY include small breaking changes
if — and only if — no known implementation depends on the changed behavior
yet, consistent with standard SemVer's treatment of `0.x` as "still
stabilizing." Every such change MUST still be called out explicitly in
`CHANGELOG.md` as breaking. Once `specVersion` reaches `1.0.0`, this
exception ends and the tables above apply without exception.
