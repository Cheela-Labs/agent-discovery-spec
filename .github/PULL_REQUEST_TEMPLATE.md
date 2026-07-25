<!--
  Which kind of PR is this? Delete the sections that don't apply — see
  CONTRIBUTING.md if you're not sure which one you're making.
-->

## Editorial fix (typo, broken link, clarification with no behavior change)

- [ ] This does not change what a conformant implementation would do
- [ ] If it touches `spec/schema/`, the examples in `spec/schema/examples/`
      still validate

## Namespace registry addition

- [ ] One entry added to `registry/namespaces.json`, alphabetically sorted
- [ ] Prefix isn't already taken and isn't `core`
- [ ] `contact` is a real, reachable email or URL

## New or updated proposal

- [ ] Uses `proposals/TEMPLATE.md` with all required sections filled in
- [ ] Filed as `proposals/0000-short-slug.md` (editor assigns the real number on merge)
- [ ] Added a row to `proposals/README.md`'s index table
- [ ] Includes a backward-compatibility statement, even if "no impact"

## Folding a `Final` proposal into `spec/`

- [ ] References the ADS number(s) being folded in
- [ ] `spec/schema/manifest.schema.json` updated to match
- [ ] `spec/schema/examples/` updated/added and re-validated
- [ ] `CHANGELOG.md` entry added
- [ ] Version bump level matches `spec/04-versioning-strategy.md`
