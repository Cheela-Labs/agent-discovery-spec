# IANA Well-Known URI Registration — `agent-discovery.json`

**Status: DRAFT — NOT SUBMITTED.** See [Blockers](#blockers) before filing.

| | |
|---|---|
| Registry | <https://www.iana.org/assignments/well-known-uris/> |
| Governing RFC | [RFC 8615](https://www.rfc-editor.org/rfc/rfc8615.html) §3 |
| Procedure | Specification Required (Designated Expert review) |
| Current expert | [@mnot](https://github.com/mnot) |
| Cost | None. IANA does not charge for protocol parameter registration. |
| Submit via | GitHub issue on [protocol-registries/well-known-uris](https://github.com/protocol-registries/well-known-uris) (preferred), or `wellknown-uri-review@ietf.org` |
| Turnaround | Approximately 14 days |

---

## Registration template

Per [RFC 8615 §3.1](https://www.rfc-editor.org/rfc/rfc8615.html#section-3.1).
All five fields are required.

**URI suffix:**
`agent-discovery.json`

**Change controller:**
Cheela Labs — *TODO: a role-based address (e.g. `spec@cheelalabs.com`), not a
personal one. Change controllers outlive individuals.*

**Specification document(s):**
Agent Discovery Specification (ADS) Core, version *TODO: tagged release*.

*TODO: a permanent versioned URL — a git tag or release asset. Not a `main`
branch link; `main` is not a stable reference and the expert will say so.*

Normative discovery behaviour: `spec/03-discovery-flow.md`.
Manifest format and media type: `spec/02-manifest-schema.md` and
`spec/schema/manifest.schema.json`.
Security considerations: `spec/07-security-considerations.md`.

**Status:**
`provisional`

> Deliberate. Permanent requires demonstrated use — A2A obtained it for
> `agent-card.json` on 2025-08-01 backed by Linux Foundation stewardship and
> production deployment. Provisional is the honest ask for a young spec, has
> a materially lower bar, and promotes later once the expert finds it in
> use. Requesting permanent now converts a soft "not yet" into a decline on
> the merits.

**Related information:**
The manifest is a JSON document served with media type `application/json`
(or a `+json` structured syntax suffix). Applies to the `https` scheme;
plaintext `http` is non-conformant except on loopback hosts. A `404` is
defined as a valid negative signal meaning the origin does not implement
ADS, not an error condition.

---

## Blockers

The registry's stated criteria:

> Single-purpose websites and single-owner GitHub repositories lack
> sufficient credibility unless they demonstrate significant deployment.

> Registration should occur when your specification is mature for wide
> review. [The guidance] discourages anticipatory requests from
> non-standards bodies.

Repository state measured 2026-07-29: created 2026-07-25, 0 stars, 0 forks,
1 push. That is the profile the first criterion excludes. Filing now
produces a public decline that becomes the first thing anyone finds when
searching ADS.

### Resolved in 0.2.0

- [x] **Security Considerations** — added at
      `spec/07-security-considerations.md`. RFC 8615 §4 makes this a review
      criterion. `SECURITY.md` did not satisfy it; that is a
      vulnerability-reporting policy for the repository, a different
      artifact answering a different question.
- [x] **TLS mandated normatively** — the path was written `https://` but
      never required it.
- [x] **Redirect handling defined** — previously unaddressed, which made
      cross-origin capability spoofing conformant.
- [x] **License** — MIT, already present and valid. (An earlier assessment
      claimed this was missing based on the GitHub API reporting
      `NOASSERTION`. Reading the file showed a clean MIT license. The API
      metadata was wrong, or stale.)

### Outstanding — must clear before filing

- [ ] **Cut a tagged, immutable release** and cite the tag. "Stable
      reference" is a hard requirement.
- [ ] **Demonstrate deployment.** Even a handful of origins serving live
      manifests changes the conversation. Zero is the weakest position
      available.
- [ ] **Ship a reference client.** A spec nobody can consume reads as a
      proposal, not a standard. This is also the gap flagged in the ADS-1
      proposal.
- [ ] **Widen ownership past one org.** A second independent implementer is
      the strongest single answer to "single-owner repository."
- [ ] **Fix the CHANGELOG date inconsistency.** 0.1.0 is dated 2026-03-01;
      the repository was created 2026-07-25. A reviewer who checks will
      read that as backdating.

### Consider — strengthens but not required

- [ ] **Register a media type.** ADS currently rides generic
      `application/json`. `application/agent-discovery+json` is a separate
      IANA process ([RFC 6838](https://www.rfc-editor.org/rfc/rfc6838.html)),
      also free. Attempt only after the above are done.
- [ ] **Reconsider MIT for the prose.** MIT is a software license carrying
      no patent grant, which matters for a specification. The common split
      is CC-BY-4.0 for spec text plus Apache-2.0 for schema and reference
      code. Not a blocker, and a decision for the project owner rather than
      a defect.

---

## Sequencing

1. Close the outstanding items. They are spec and project gaps regardless of
   IANA.
2. Get ADS onto real origins and into one client someone else runs.
3. File for **provisional** status using the template above.
4. Request promotion to permanent once deployment is demonstrable.

Do not compress steps 1–2. The registry is not the bottleneck on ADS
adoption and never was — a registered path that nothing implements is still
a path that nothing implements. Registration ratifies traction; it does not
create it.
