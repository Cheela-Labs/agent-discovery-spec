# Namespace Registry

This is a first-come, first-served registry of short capability/extension
namespace prefixes, as an alternative to spelling out a full reverse-DNS
domain (`com.example.*`) every time. See
[spec/05-extensibility.md](../spec/05-extensibility.md#namespacing) for the
full rules — registering here is optional, not required, to use a
namespace.

## How to register a prefix

1. Add one entry to [namespaces.json](namespaces.json), keeping the array
   sorted alphabetically by `prefix`.
2. Open a PR. This is **not** a full ADS proposal — no `Draft`/`Review`/
   `Last Call` cycle, no proposal number. An editor merges it once they've
   checked:
   - the prefix isn't already taken
   - the prefix isn't `core` (reserved) or confusingly similar to an
     existing entry
   - the `contact` field is a real, reachable email or URL
3. That's it. The prefix is yours to use in `capabilities[].name` and
   `manifest.extensions` keys from the moment it's merged.

## What registering does *not* do

- It does not review or approve what you build under that prefix — that's
  entirely yours to define.
- It does not grant trademark rights to the prefix string itself.
- It is not a namespace *reservation* against future use by others under a
  different, unrelated meaning of the same short string in a different
  context — it only governs collision avoidance within ADS manifests.

## Releasing a prefix

If you no longer need a registered prefix, open a PR removing your entry.
Released prefixes go on an "available again" basis immediately — there's no
cooldown period at this stage of the project.
