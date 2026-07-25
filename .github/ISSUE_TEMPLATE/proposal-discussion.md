---
name: Proposal discussion
about: Start the conversation before writing a full ADS proposal
title: "[Proposal] "
labels: proposal-discussion
---

<!--
  This is step 1 of proposing a spec change — see CONTRIBUTING.md.
  Use this to describe the PROBLEM before anyone (possibly you) writes
  the actual proposal document. It's much cheaper to catch "this was
  already tried" or "this doesn't need a spec change at all" here than
  after a full proposal PR is up.
-->

## What's the problem?

Describe a concrete manifest or discovery scenario the current spec can't
express, or expresses ambiguously. Link the relevant `spec/` section(s).

## What have you tried within the current spec?

Sometimes the answer is "use the `extensions` block" or "register a
namespace prefix" rather than a Core/Extension change — walk through why
those don't cover your case, if you've already considered them.

## Rough shape of a fix (optional)

You don't need a full schema diff here — that's what the proposal document
is for once this issue confirms there's appetite for it.
