# Definition of Done

A ticket is done when it is running in production, observable, and someone other than the author
could support it. Merged is not done. "Works on my machine" is not done.

## Checklist

Use this for every ticket. Items that truly do not apply can be skipped with a one-line reason in
the PR.

### Code

- [ ] Merged to `main` through a reviewed PR (see [code review](code-review.md)).
- [ ] Acceptance criteria from the ticket are met and demonstrated in the PR or a demo.
- [ ] No new lint errors, type errors, or `@ts-ignore` without a comment explaining why.
- [ ] No TODOs without a linked ticket.

### Tests

- [ ] New behavior has tests at the right level (see [testing strategy](testing-strategy.md)).
- [ ] Bug fixes include a regression test that failed before the fix.
- [ ] Money-moving logic has tests for retries, duplicates, and partial failures.

### Data

- [ ] Migrations are backward compatible with the currently deployed release.
- [ ] Long-running migrations or backfills have been timed against production-sized data.
- [ ] Contract-phase cleanups have their own follow-up ticket.

### Operability

- [ ] Logs at the right level with correlation IDs; no secrets or PII.
- [ ] Metrics for the new path: rate, errors, duration at minimum.
- [ ] Alerts added or updated if the feature can fail in a way users notice.
- [ ] Runbook updated for any new failure mode or manual procedure.
- [ ] Feature flag in place if rollout is gradual; cleanup ticket created.

### Release

- [ ] Deployed to production.
- [ ] Verified in production: dashboard checked, smoke test or real traffic observed.
- [ ] Flag rollout plan executed or scheduled.

### Documentation

- [ ] API changes reflected in the OpenAPI spec.
- [ ] ADR written if the change made an architectural decision.
- [ ] README or docs updated if setup, configuration, or behavior changed.
- [ ] Support or product notified of user-visible changes.

## Levels of done

Different work has different "done". Keep the base checklist and add to it.

| Work type | Additional requirements |
| --- | --- |
| Feature | Product owner accepts it in staging or production |
| Bug fix | Root cause noted in the ticket, not only the symptom |
| Spike | Written summary with a recommendation, linked from the ticket |
| Tech debt | Before/after measure if one exists (build time, latency, error rate) |
| Incident action item | Linked from the postmortem; postmortem updated |
| Infrastructure | Rollback tested; runbook updated; cost impact noted |

## Why this matters

- **Done means done for the team, not for the author.** The real cost of a feature includes
  whoever supports it at 3 a.m.
- **It makes estimates honest.** If alerts and docs are part of done, they are part of the
  estimate, and they stop getting cut.
- **It keeps the board truthful.** A "Done" column full of things not yet in production hides
  risk.

## Common shortcuts to avoid

| Shortcut | Why not |
| --- | --- |
| "I'll add tests in a follow-up" | The follow-up rarely happens; the regression does |
| "Monitoring can come later" | Later is when the first incident happens, without data |
| "It's behind a flag, so it's done" | Dark code is inventory; done is released and verified |
| "Docs are obvious from the code" | Not for the person on call who did not write it |

## When to bend it

Incidents. During an active incident, the goal is to restore service. Ship the minimal fix,
then open tickets to bring it up to this bar. Those tickets are action items in the
[postmortem](../templates/postmortem-template.md) and get prioritized, not parked.
