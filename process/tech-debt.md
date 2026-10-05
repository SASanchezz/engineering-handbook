# Tech Debt

Tech debt is any part of the system that makes future change slower or riskier than it should be.
Some of it is taken on deliberately to ship. The problem is never having debt; it is not knowing
how much exists or never paying it down.

## Stance

- **Deliberate debt is fine** when it is written down with a reason and a payback trigger.
- **Accidental debt is normal.** Find it, record it, and treat it like any other work.
- **Debt competes for priority on cost, not on taste.** "I don't like this code" is not a ticket.
  "This module causes a third of all incidents" is.

## Budget

| Bucket | Share of sprint capacity |
| --- | --- |
| Product work | about 70% |
| Tech debt and platform health | about 20% |
| Unplanned (incidents, support, bugs) | about 10% |

The 20% is a floor to protect, not a target to ignore. If unplanned work eats it two sprints in a
row, that is a signal the debt is already charging interest; raise it with product and adjust
the plan, not quietly drop the budget.

Larger items (more than a sprint) do not come out of the 20%. They go through an
[RFC](rfc-process.md) and are planned as projects, with a business case.

## How to track it

All debt lives in Jira with the label `tech-debt`. Not in TODO comments, not in a wiki page that
nobody opens, not in someone's head.

Each ticket includes:

```text
Title:    Wallet service: settlement and bonus logic share one 1,800-line service class
Area:     wallet
Kind:     design
Cost now: changes to bonus rules take ~3x longer to review; 2 of last 5 wallet bugs here
Risk:     high (money path)
Fix:      split into SettlementService and BonusWageringService behind existing interfaces
Size:     M (about 1 sprint)
Trigger:  before next bonus type is added
```

### Kinds of debt

| Kind | Example |
| --- | --- |
| Design | Wrong module boundaries, god services, leaky abstractions |
| Code | Duplication, dead code, missing types, `any` everywhere |
| Test | Flaky tests, missing integration coverage on a critical path |
| Infrastructure | Manual deploy steps, unpinned base images, slow pipeline |
| Dependency | Outdated framework, library with known CVEs, unmaintained package |
| Data | Missing constraints, denormalized fields drifting, no index on a hot query |
| Documentation | Runbooks out of date, undocumented env vars |
| Operability | Alerts that page for nothing, missing metrics on a critical path |

## How to prioritize

Score each item on two axes and look at the quadrant, not a precise number.

| | Low effort | High effort |
| --- | --- | --- |
| **High cost / risk** | Do next sprint | Plan as a project (RFC) |
| **Low cost / risk** | Fix when you are in the area | Leave it; revisit quarterly |

Cost and risk go up when the debt:

- sits on a money path, auth path, or data integrity path,
- shows up in postmortems,
- slows down the work on the roadmap for the next quarter,
- blocks a dependency upgrade with a security fix.

The tech lead keeps the top 10 visible and ordered. Engineers pick from the top when the 20%
bucket has room.

## The boy scout rule, with limits

Leave code a little better than you found it, inside the scope of the PR you are already making:
rename a confusing variable, add a missing type, extract an obvious function. Anything that
doubles the diff becomes its own PR (see [code review](code-review.md)).

## Taking on debt deliberately

When choosing a shortcut to hit a date, write it down in the PR:

```text
Debt: referral payout uses a nightly batch instead of an event consumer.
Why:  launch date; event consumer needs the outbox pattern, which does not exist yet.
Payback trigger: before referral volume exceeds what the batch processes in 30 minutes,
or Q3 planning, whichever comes first.
Ticket: PAY-501
```

A shortcut without a ticket and a trigger is not deliberate debt. It is a bug nobody has found
yet.

## Reporting

Once a quarter the tech lead shares a short summary with the team and product:

- debt items closed, and what they changed (pipeline time, incident count, review time),
- top 5 open items and why they matter now,
- whether the 20% budget held, and if not, why.

This keeps debt a business conversation instead of an engineering complaint.

## Anti-patterns

- **The rewrite.** Almost never the answer. Strangle the old thing piece by piece behind stable
  interfaces.
- **Debt sprints.** A whole sprint of cleanup twice a year teaches the team that debt is someone
  else's job the rest of the time.
- **Unbounded refactors.** Every debt ticket has a definition of done and a size.
