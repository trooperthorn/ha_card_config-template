# Decisions

## 2026-09-08: start from upstream `beta`, not `master` or `dev`

Upstream carries three lines that do not contain each other. `master` is the 2021 code that imports from the deprecated `lit-element` entrypoint and is what every 1.3.x release ships; that import is the source of the browser warning "The main 'lit-element' module entrypoint is deprecated". `dev` holds 2025 feature work (1.3.7-beta.1). `beta` is a Lit 3 rebuild made from `master`, so it dropped the `dev` features, which is why 2.0.0-b1 regressed on embedded templates and multi-statement blocks.

`beta` was chosen because it carries the modern toolchain (Lit 3, TypeScript 5.9, ESLint 9, vitest) and typed configuration; the lost behaviour was smaller to re-add than the toolchain was to rebuild on `dev`.

## 2026-09-08: which upstream pull requests were taken

| Upstream item | Taken | Reason |
| --- | --- | --- |
| PR 202 (`_helpers` in `shouldUpdate`) | yes | Fixes issue 201, the most-reported defect; four lines. |
| PR 199 (`eval` in function scope, templated `entities` string) | yes | Restores multi-statement blocks; supersedes PR 200. |
| PR 198 (template literal for embedded templates) | yes | Restores strings with embedded `${}`; applied after PR 199 so it uses the same evaluator. |
| PR 158 (`height: 100%`) | yes | One line, visible improvement in stacks and grids. |
| PR 178 (dashboard variables before card variables) | already present | `beta` already evaluates dashboard variables first. Its console.error wrapper was not taken; the existing error log carries more context. |
| PR 171 (view-level variables and entities) | no | Reads `hui-root.___curView`, a private frontend field that Home Assistant states may change in any release. Revisit if a public path appears. |
| PR 159 (static variables, `config` and `output`, async templates, `<$ $>` delimiters) | no | Draft abandoned by its author in 2025-03. The delimiter change breaks every existing configuration; the async scheduler adds complexity to a card that already recreates its child on each update. `staticVariables` and `config`/`output` are candidates for separate small changes, listed in backlog.md. |
| Dependabot PRs 188, 189, 193, 194, 195 | no | Target `master`'s Yarn 1 lockfile; moot on `beta`'s Yarn 4 lockfile. |

Upstream's tag 2.0.0-b2 points at the Lit 3 rebuild alone and does not contain the fixes for issues 190, 191, or 192, although its release note says it does; a contributor confirmed that on upstream PR 198.

## 2026-09-08: Home Assistant floor 2026.9.0

Upstream's README claimed 2026.2.3 for 2.0.0, but the beta source has no version check and the release notes do not state a reason. The fork declares the release it is built and tested against. The card uses only `window.loadCardHelpers`, so it may run on older releases, and that is stated as unverified in the README rather than promised.

## 2026-09-08: CalVer with a bare tag

Versions are `YYYY.MM.DD.N`. Tags carry no `v` prefix because upstream's existing tags (`1.3.6`, `2.0.0-b2`) are bare, and the baseline rule is never to change an existing prefix. HACS compares versions with awesomeversion, which orders `2026.09.08.1` above `2.0.0-b2`, so a user switching from upstream to the fork sees an update rather than a downgrade.

`package.json` stays at `0.0.0` because Yarn requires SemVer there; the shipped version lives in `VERSION` and `src/const.ts`, both listed in `.release.json`.

## 2026-09-08: `dist/` is committed

HACS looks for the card in `dist/` before it looks at release assets. Committing `dist/config-template-card.js` lets HACS install from the tree as well as from a release, and CI fails when the committed file differs from a fresh build so the two cannot drift.

## 2026-09-08: upstream's Claude and Copilot files

`master` carried two workflows that call the Claude Code action with an OAuth secret this repository does not have; they were not carried over. `beta`'s `.github/copilot-instructions.md`, with `AGENTS.md` and `CLAUDE.md` pointing at it, is kept as the repository's agent contract because its rules (Lit 3 patterns, strict typing, configuration shape in `types.ts`) match the house style.
