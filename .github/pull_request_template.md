## Change

Describe the behavior and security impact.

## Verification

- [ ] `yarn lint`, `yarn typecheck`, and `yarn test` pass
- [ ] `yarn rollup` produces no drift in committed `dist/`
- [ ] No secret, credential, token, or private installation detail was committed
- [ ] Template evaluation changes are covered by a test in `src/config-template-card.test.ts`

## Risk and rollback

State affected trust boundaries and a rollback method.
