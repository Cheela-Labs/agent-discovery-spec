# Security Policy

This repository contains a specification, schemas, and documentation — not a
running service. "Security" here mostly means: does the spec itself have a
design flaw that would make conformant implementations unsafe by default.

## Reporting a specification-level security issue

Examples of what belongs here:

- The discovery flow is vulnerable to spoofing, cache poisoning, or
  downgrade attacks (e.g. tricking a client into trusting an unsigned or
  stale manifest).
- The extensibility mechanism allows a malicious extension block to be
  misinterpreted as a core field by a conformant client.
- The manifest schema encourages implementers to leak credentials or
  internal topology by design (not just "an implementation could choose to
  do this carelessly" — that's not a spec issue).

Please **do not** open a public issue for these. Instead, open a private
[GitHub Security Advisory](../../security/advisories/new) on this
repository, or contact a project editor directly. Include:

- Which document/section of the spec is affected
- The concrete attack scenario (what a conformant-but-naive implementation
  would do wrong)
- Suggested fix, if you have one

We'll acknowledge within a few days and work with you on a coordinated fix —
typically a clarifying PATCH to the affected spec document, or in more
serious cases an expedited proposal if the fix changes normative behavior.

## Reporting a vulnerability in a specific implementation

This repository doesn't own or ship any implementation (including Cheela's).
If you found a bug in how a specific product implements ADS, report it to
that product's own security contact, not here — unless the root cause is
that the spec itself required or encouraged the unsafe behavior, in which
case both reports are useful.

## Scope note

Tooling in this repo (schema validators, example scripts, if any get added
later) is covered by normal vulnerability reporting too — same process as
above.
