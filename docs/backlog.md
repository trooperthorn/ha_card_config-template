# Backlog

Dated open items. Upstream issue numbers refer to iantrich/config-template-card.

- 2026-09-08: In-place `hass` updates. `render` recreates the wrapped element on every watched change (upstream 112, 144, 174, 45). Cache the created element, compare the evaluated configuration, push `hass` when unchanged, recreate only when it differs. This is the one change that would make the card cheap enough to leave wrapped around cards that update often.
- 2026-09-08: `staticVariables`, evaluated once at `setConfig`, from upstream PR 159. Small, self-contained, and useful for expensive lookups.
- 2026-09-08: `config` and `output` template variables (the unevaluated card config and the partially evaluated output), from upstream PR 159 and issue 160.
- 2026-09-08: View-level variables (upstream PR 171) if a public way to read the current view appears; the private `___curView` field is not acceptable.
- 2026-09-08: Wildcard `entities` entries (upstream issue 95) and attribute or state translation helpers (issue 102) have no design yet.
- 2026-09-08: The GitHub App variable and secret for zero-touch release bumps are not yet set on this repository.
