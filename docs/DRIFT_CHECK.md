# Checking the Figma file for drift

`docs/figma-snapshot.json` is a structural baseline of every component and component-set in the Portal 2.0 Figma file (names, variant lists, IDs) as of the date in its `generatedAt` field. It does **not** capture token bindings, colors, or spacing — only "what components/variants exist." That's a deliberate scope limit: it's cheap to regenerate and diff, but it can't tell you a color got rebound to the wrong token. For that, someone still needs to ask for a real audit on the specific area that changed.

## How to run a check (with Claude + the Figma MCP connection)

Ask: **"Check the Figma file for drift against docs/figma-snapshot.json"**

What happens:
1. Claude re-runs the same discovery script against the live Figma file (one page at a time — `figma.getNodeByIdAsync` + `setCurrentPageAsync` + `findAllWithCriteria({ types: ['COMPONENT', 'COMPONENT_SET'] })`) and collects fresh names/variants/IDs.
2. Diffs the fresh results against `figma-snapshot.json`: new component IDs = added, missing IDs = removed, same ID with a different name = renamed, same COMPONENT_SET with a different variant list = variants added/removed.
3. Reports what changed, mapped to which `docs/components/*.md` or `docs/patterns/*.md` file needs updating.
4. If you confirm, overwrites `figma-snapshot.json` with the fresh data so it becomes the new baseline, and updates the relevant `.md` docs.

This requires whoever asks to have their own Figma access to the file and their own Claude session connected to the Figma MCP — the repo carries the baseline, not the live connection.

## Known limitations

- Structural only — doesn't catch a component keeping its name/variants but changing its token bindings, colors, or spacing internally.
- `variantCount`/`variants` on very large sets (Button, Cells, Avatars) are summarized rather than enumerated per-ID to keep the file readable — a real audit of those still needs a fresh `findAllWithCriteria` pass.
- Pages made of variables/styles only (Color, Typography, Foundation) aren't covered by this file at all — they'd need a separate variable-collection diff if that ever becomes a problem.
