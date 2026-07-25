---
ads: <to be assigned by an editor on merge — leave blank in your PR>
title: <concise, descriptive title>
status: Draft
type: <Core | Extension | Meta | Informational>
author: <your name/handle, and anyone else co-authoring>
created: <YYYY-MM-DD>
requires: <ADS numbers this depends on, if any — omit if none>
---

<!--
  Delete this comment block before submitting.

  Before filling this out, read proposals/0000-process.md — especially
  the "what a proposal must contain" section. Proposals missing any of
  the required sections below will be sent back for completion before
  an editor assigns a number.

  File this as proposals/NNNN-short-slug.md where NNNN is left as
  0000 in your PR — an editor renames it on merge once the real number
  is assigned.
-->

## Summary

One paragraph. What changes, in plain language, for someone who hasn't
read the rest of this document.

## Motivation

The problem this solves, with a concrete example of a manifest or discovery
scenario the current spec can't correctly express today. Not "this would be
nice to have" — show the gap.

## Specification

The actual normative text and/or schema diff being proposed. Write this as
you'd want it to read inside `spec/` if accepted — this section is what
gets folded in on `Final`, not paraphrased from it.

If this is a **Core** or **Extension** proposal, include:

- The exact JSON Schema fragment being added/changed
- At least one full example manifest exercising the new behavior, which
  actually validates against your proposed schema change

## Rationale

Alternative shapes you considered and why this one won. If you looked at
how another spec (OpenAPI, MCP, OIDC discovery, etc.) solves a similar
problem and deliberately diverged, say so and why.

## Backward compatibility

State explicitly, per
[spec/04-versioning-strategy.md](../spec/04-versioning-strategy.md):

- What version bump this requires (MAJOR/MINOR/PATCH), and why
- What existing manifests need to do, if anything, to remain valid
- What existing clients need to do, if anything, to keep working
- If this deprecates or removes something: the deprecation window per
  [spec/06-backward-compatibility.md](../spec/06-backward-compatibility.md)

If this proposal has **no** backward compatibility impact, say that
explicitly rather than omitting the section.

## Reference implementation

Optional but strongly encouraged — a link to a system (even experimental)
that implements this behavior today. Proposals with a working reference
move through `Review` measurably faster.

## Security considerations

Anything a client or system implementing this should be careful about.
"None" is an acceptable answer, but say it explicitly rather than skipping
the section — an omitted section reads as "not considered," not as "none."

## Copyright

By submitting this proposal you agree it's contributed under this
repository's [MIT License](../LICENSE), same as every other document here —
no separate copyright assignment or CLA required.
