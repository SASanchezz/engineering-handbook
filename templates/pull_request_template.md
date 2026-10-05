## Why

<!-- The problem or goal. Link the ticket: PAY-123. The diff shows what; explain why. -->

## What

<!-- Short list of the changes. Point reviewers at the files that matter most. -->

-

## How to test

<!-- Steps, commands, or requests a reviewer can run. Note what the tests cover. -->

## Risk and rollback

- **Risk:** low / medium / high, and why
- **Feature flag:** name, or none
- **Migration:** none / expand / contract (backward compatible with current release?)
- **Rollback:** redeploy previous release / turn off flag / other

## Checklist

- [ ] Under 400 changed lines, or the reason is explained above
- [ ] Tests added or updated; bug fixes include a regression test
- [ ] Logs, metrics, and alerts considered for new failure modes
- [ ] No secrets, PII, or debug code
- [ ] Money paths: idempotency, ledger balance, integer minor units
- [ ] API changes are non-breaking or versioned; OpenAPI updated
- [ ] Docs, runbooks, or ADR updated if needed
- [ ] AI-assisted: the author has reviewed and understands every line (label `ai-assisted` if used)
