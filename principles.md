# Engineering Principles

These are the values used to break ties. When two reasonable engineers disagree, one of these
should usually settle it. If none does, write an [RFC](process/rfc-process.md).

Each principle comes with what it costs. A principle that costs nothing is a slogan.

## 1. Ship small, ship often

A change that touches 80 lines gets a real review. A change that touches 2,000 gets a rubber stamp.

**In practice**

- PRs stay under 400 changed lines (see [code review](process/code-review.md)).
- Work merges to `main` at least daily, behind a feature flag if it is not ready for users.
- Deploy to production several times a week as a floor, not a ceiling.

**What it costs:** more up-front slicing, feature flags to maintain and clean up, and the
discipline to merge half-built work safely.

**Why:** small batches shorten feedback loops, make rollbacks boring, and keep incidents small.

## 2. You build it, you run it

The team that writes a service carries its pager, owns its dashboards, and reads its error logs.

**In practice**

- Every service has an owning team in its README and in the service catalog.
- Engineers join [on-call](operations/on-call.md) after onboarding, including seniors and leads.
- "Done" includes alerts, dashboards, and a runbook entry (see
  [definition of done](process/definition-of-done.md)).

**What it costs:** on-call load on the people who would rather be building.

**Why:** nobody writes better alerts or fewer bugs than the person who gets woken up by them.

## 3. Write it down

If a decision only exists in a call, it did not happen. If a procedure only exists in someone's
memory, it will be wrong at 3 a.m.

**In practice**

- Significant decisions get an [ADR](templates/adr-template.md) in the repo they affect.
- Proposals go through an [RFC](process/rfc-process.md) before a meeting, not after.
- Runbooks live next to the code and are linked from alerts.
- Meetings end with written notes and owners.

**What it costs:** writing time, and the occasional feeling that a doc is overkill.

**Why:** the team is remote and across time zones. Writing is how context scales without scaling
meetings. It is also the only onboarding material that stays up to date.

## 4. Boring technology by default

Pick tools with large communities, known failure modes, and good operational tooling.
PostgreSQL before a new database. A queue already in production before a new broker.

**In practice**

- Each team gets a small number of "innovation tokens" per year. Spending one requires an RFC.
- New infrastructure needs an owner, a runbook, and a backup story before it reaches production.

**What it costs:** sometimes missing a tool that would have been slightly better.

**Why:** the hard part of a platform is operating it for years, not building it for months.

## 5. Correctness first where money moves

When the system holds other people's money, a bug is not a bad UX; it is a financial and
regulatory event.

**In practice**

- Balances change only through double-entry ledger entries, never by `UPDATE balance = ...`.
- Every money-moving endpoint accepts an idempotency key.
- Amounts are integers in minor units or `numeric`, never floating point.
- Every state change is in an append-only audit log.

Details: [security baseline](operations/security-baseline.md),
[Postgres guidelines](backend/postgres-guidelines.md), [API design](backend/api-design.md).

**What it costs:** more code and more tests for features that look simple ("just add a bonus").

**Why:** bonus, wagering, and referral logic is where edge cases turn into real losses. It has
happened before; design for it.

## 6. Make the right thing the easy thing

Rules that rely on memory fail. Rules enforced by tooling hold.

**In practice**

- Lint, format, type checks, and tests run in CI and block merge.
- Templates ([PR](templates/pull_request_template.md), [ADR](templates/adr-template.md),
  [postmortem](templates/postmortem-template.md)) live in the repo.
- AI assistants get the same rules via committed `CLAUDE.md` files and shared skills, and the
  critical ones are enforced by hooks and permission rules (see
  [AI agent guardrails](process/ai-agent-guardrails.md)).

**Why:** a team scales its habits through defaults, not reminders.

## 7. Automate the second time

Do it by hand once to understand it. Script it the second time. Put it in CI the third time.

**What it costs:** some upfront time on scripts that may get used only a few times.

**Why:** manual steps are where releases go wrong and where knowledge silos form.

## 8. Blameless, not accountable-less

Do not ask "who did this?" Ask "why did the system allow it, and what stops it next time?"
The owner is still expected to drive the fix.

**In practice**

- Postmortems use the [blameless template](templates/postmortem-template.md).
- Action items have one owner and a due date, and are reviewed weekly until closed.

**Why:** people hide mistakes in blameful cultures, and hidden mistakes repeat.

## 9. Disagree, then commit

Argue hard in the RFC or the PR. Once a decision is made and written down, everyone executes it fully,
including the people who argued against it. Revisiting is allowed when new information appears,
via a new ADR that supersedes the old one.

## 10. Optimize for the reader

Code is read far more often than it is written. So are docs, PR descriptions, and commit messages.

- Name things for what they do, not how they do it.
- Prefer explicit over clever.
- A PR description explains why; the diff already shows what.

## How these trade off

| When these conflict | Usually choose | Because |
| --- | --- | --- |
| Speed vs correctness on money paths | Correctness | Rework on a ledger costs more than delay |
| Speed vs correctness elsewhere | Speed, with a flag and a rollback plan | Reversible decisions should be fast |
| New tool vs existing tool | Existing tool | Operational cost dominates |
| Docs vs shipping | Ship, then write within the same sprint | An undocumented feature is not done |
| Consensus vs deadline | Decision owner decides, dissent recorded in the ADR | Stalemates cost more than imperfect calls |
