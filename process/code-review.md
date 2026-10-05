# Code Review

Code review exists to catch defects, spread knowledge, and keep the codebase coherent. It is not
a gate for proving seniority or a place to relitigate settled decisions.

## Rules at a glance

| Rule | Default |
| --- | --- |
| PR size | Under 400 changed lines, excluding generated files and lockfiles |
| Approvals | 1 approval; 2 for money paths, auth, migrations on hot tables, and infra |
| First response | Within 1 business day of the request (in the reviewer's time zone) |
| Small PRs (< 50 lines) | Same business day when possible |
| Merge | Author merges after approval and green CI |
| Merge strategy | Squash merge to `main`, Conventional Commit title |
| Stale PRs | Open more than 3 business days gets raised in standup |

## For authors

### Before requesting review

- [ ] CI is green: lint, types, unit and integration tests.
- [ ] The PR does one thing. Refactors and behavior changes go in separate PRs.
- [ ] The description uses the [PR template](../templates/pull_request_template.md): why, what,
      how to test, risk, rollback.
- [ ] You reviewed your own diff in the GitHub UI first. Most nits are caught here.
- [ ] New behavior has tests; bug fixes have a test that failed before the fix.
- [ ] Migrations follow the [Postgres guidelines](../backend/postgres-guidelines.md).
- [ ] If an AI assistant wrote part of the change, you have read and understood every line (see
      [AI-assisted development](ai-assisted-development.md)).

### Keeping PRs small

Over 400 lines, review quality drops sharply. Ways to split:

1. **Preparatory refactor first.** Rename, extract, move. No behavior change, fast review.
2. **Schema before code.** The expand migration ships on its own.
3. **Behind a flag.** Merge the skeleton dark, fill it in over several PRs.
4. **Stacked PRs.** Each one reviewable alone, merged in order.

If a PR must be large (a generated client, a mechanical rename), say so in the first line of the
description and point reviewers at the files that actually need attention.

### Responding to feedback

- Reply to every `blocking` comment, either with a fix or a reasoned disagreement.
- Push fixes as new commits during review; PRs are squashed on merge anyway.
- If a thread goes past two rounds, move to a call for 10 minutes, then write the outcome in the
  PR.

## For reviewers

### Priorities, in order

1. **Correctness.** Does it do what the description says? Edge cases, error paths, concurrency.
2. **Safety.** Security, data integrity, money handling, migrations, backward compatibility.
3. **Operability.** Logs, metrics, alerts, feature flag, rollback path.
4. **Design.** Right module? Right abstraction? Will the next person understand it?
5. **Tests.** Do they test behavior, not implementation? Would they catch a regression?
6. **Style.** Only what the linter does not already enforce. Usually nothing.

### What to look for in this stack

| Area | Look for |
| --- | --- |
| Money | Idempotency key honored, ledger entries balanced, integer minor units, no float math |
| Postgres | Missing index for new query, `CREATE INDEX` without `CONCURRENTLY`, long transaction, N+1 |
| NestJS | DTO validation on every input, no business logic in controllers, no cross-module repository access |
| API | Problem Details errors, pagination on list endpoints, no breaking change without a version |
| Async | Unhandled promise, missing `await`, retries without backoff, non-idempotent consumers |
| Security | Secrets in code or logs, PII in logs, missing authorization check (not just authentication) |
| Ops | New failure mode without a metric or alert, log level noise |
| AI-assisted | Invented APIs or config keys, tests that mirror the implementation, edits outside the stated scope, weakened or deleted assertions |

### Comment prefixes

Every comment starts with a prefix so the author knows what is required.

| Prefix | Meaning | Blocks merge? |
| --- | --- | --- |
| `blocking:` | Must be addressed before merge: bug, security, data risk | Yes |
| `question:` | The reviewer does not understand; please explain (maybe in a code comment) | Until answered |
| `suggestion:` | The reviewer thinks this is better; author's call | No |
| `nit:` | Cosmetic, take it or leave it | No |
| `praise:` | This is good, keep doing it | No |
| `followup:` | Valid, but out of scope; create a ticket | No |

Examples:

```text
blocking: This credits the wallet before the bet settlement commits. If the transaction rolls
back, the credit stays. Move the ledger write inside the same transaction.

question: Why is the retry limit 10 here when the client default is 3?

nit: `getUsr` -> `getUser`.

followup: The referral service has the same pattern. Worth a ticket to fix both.
```

### Approving

- **Approve** when remaining comments are non-blocking. Trust the author to handle nits.
- **Request changes** only for `blocking:` issues. Say what would unblock you.
- **Comment** without approving when you reviewed part of it; say which part.

Do not hold a PR hostage to your preferred style. If it is a team-wide preference, propose a lint
rule.

## AI in review

### Automated first pass

An AI reviewer runs on every PR in CI with the team's `review-pr` checklist (see
[AI-assisted development](ai-assisted-development.md)).

- It comments; it never approves or blocks. Approvals come from people.
- Its comments use the same prefixes as human comments. The author resolves each one, either with
  a fix or a one-line reason. Silently resolving bot comments is not allowed.
- It runs before human review is requested, so humans spend their time on design and
  correctness, not on what a machine already caught.
- If it is mostly noise in a repo, tune the skill or turn it off there. A bot that everyone
  ignores teaches people to ignore comments.

### Reviewing agent-written changes

Agents make large diffs cheap to write, but reviewing them still costs the same. The limits in
this document apply in full:

- The 400-line limit applies to agent PRs. "The agent generated it" is not a reason for a large
  PR; ask the agent for smaller, stacked PRs.
- The PR description includes the approved plan. Reviewers check the diff against the plan, and
  anything outside it needs a reason.
- The author reviews the full diff before requesting review. A PR the author has not read is
  sent back without comments.
- Unattended agent PRs (dependency bumps, lint fixes) are opened by a bot identity and need an
  owner who requests the review and answers questions.

## SLAs and flow

- Reviewers are assigned automatically via `CODEOWNERS`, plus anyone the author adds.
- Check the review queue at the start of your day and after lunch. Reviews beat new work: an
  unreviewed PR is inventory, not progress.
- If you cannot review within 1 business day, say so on the PR and reassign.
- For time zone gaps, authors should request review before their end of day so the other side
  picks it up in their morning.

## What review is not

- Not a design review. If the approach is wrong at PR time, the design step was skipped. Large
  designs go through an [RFC](rfc-process.md) first.
- Not QA. Reviewers read; tests verify.
- Not a substitute for pairing. If a change is hard to review, pairing on it may be faster.

## Metrics to watch

The team reviews these monthly, together, never per person:

- Median time to first review
- Median PR size
- Median time from open to merge
- Share of PRs over 400 lines
- The same numbers for `ai-assisted` PRs, plus how many are reverted or rewritten within 30 days

If agent-written PRs push size up or review time out, the fix is smaller plans, not lighter
review.

If time to first review creeps past one day, discuss it in [retro](../people/team-rituals.md).
