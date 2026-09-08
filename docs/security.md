# Security

## Trust boundary

The card evaluates JavaScript that appears in dashboard configuration. That is the feature. The boundary is the dashboard editor role: anyone who can save a dashboard can already ship arbitrary JavaScript to every viewer's browser through any custom resource, so the card grants nothing new to an editor and nothing at all to a viewer.

Templates run with the viewing user's session. `hass` is the same object every card receives, so a template can call any service or WebSocket command that user is authorized for, and nothing more. The card holds no credentials and has no server-side component.

## What is enforced and what is not

| Control | Status |
| --- | --- |
| Templates cannot reach beyond the viewer's own Home Assistant permissions | enforced by Home Assistant, not by the card |
| A malformed template fails loudly (console error with the template, expression, and variables) instead of rendering a wrong value | enforced |
| Templates from one card cannot read another card's variables | enforced: variables are rebuilt per evaluation |
| The wrapped card receives only the evaluated configuration | enforced |
| Sandboxing of template code | not present; templates run in page scope with access to `window` |

## Release integrity

Every release is cut by the Release workflow from a `main` commit, after CI rebuilds `dist/config-template-card.js` and confirms it matches the committed file byte for byte. The release carries the file and its SHA-256. To verify a download:

```bash
gh release download <tag> --repo trooperthorn/ha_card_config-template --pattern 'config-template-card.js*'
sha256sum --check config-template-card.js.sha256
```

Workflows run with `contents: read` except the release job, which needs `contents: write` to create the tag and release. Every action is pinned to a commit SHA with its resolution date in a comment.
