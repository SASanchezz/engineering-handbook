# ADR 0001: Record architecture decisions

- **Status:** Accepted
- **Date:** 2026-01-12
- **Deciders:** Engineering lead, tech leads
- **Related:** [RFC process](../../process/rfc-process.md),
  [ADR template](../../templates/adr-template.md)

## Context

Decisions about the company's services were made in calls, chat threads, and PR comments. Six months later
nobody could say why the wallet service locks rows instead of using serializable transactions, or
why bonus balances are separate ledger accounts. New engineers either repeated old debates or
changed things without knowing the original constraints.

The team is remote and spread across time zones. Context that lives in someone's memory does not reach the
people who need it, and it leaves when they do.

The team needs a lightweight way to record significant decisions next to the code they affect, without
turning every choice into a formal document.

## Decision

Record architecturally significant decisions as Architecture Decision Records in
`docs/adr/` in the repository they affect, using the
[ADR template](../../templates/adr-template.md).

- ADRs are numbered sequentially (`0001`, `0002`, ...) and never renumbered.
- An ADR is short: one page covering context, decision, options, and consequences.
- Once accepted, an ADR is immutable. To change a decision, write a new ADR that supersedes it and
  update the old one's status line only.
- ADRs are reviewed through a normal PR. The decider named in the ADR merges it.
- Larger proposals go through an [RFC](../../process/rfc-process.md) first; the RFC produces one or
  more ADRs.

A decision is significant enough for an ADR when it is hard to reverse, affects more than one
module or team, or would make a new engineer ask "why is it like this?"

## Options considered

| Option | Pros | Cons |
| --- | --- | --- |
| ADRs in the repo (chosen) | Versioned with code; reviewed like code; found with `grep` | Discipline needed to write them |
| Wiki pages | Easy to edit, rich formatting | Drift from code; hard to find; no review |
| Decisions in Jira tickets | Already part of the workflow | Buried in comments; tickets get closed and forgotten |
| Nothing formal | No overhead | The problem as it stands today |

## Consequences

- **Positive:** decisions and their reasoning survive people leaving; onboarding includes reading
  recent ADRs; disagreements resurface with new information instead of from scratch.
- **Negative:** some overhead per decision; a risk of writing ADRs for trivial choices. The team accepts
  the first and push back on the second in review.
- **Follow-ups:** each service repo gets a `docs/adr/` folder with this ADR copied as `0001`;
  [onboarding](../../people/onboarding.md) includes reading the last few ADRs.
