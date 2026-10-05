# RFC: Title

<!--
Copy to docs/rfcs/YYYY-MM-short-title.md and open a PR. Process: process/rfc-process.md.
Aim for something readable in 15 minutes. Put detail in appendices.
-->

| | |
| --- | --- |
| **Author** | name |
| **Decider** | name |
| **Reviewers** | 2 to 4 names (owner of affected system, someone who will operate it, an outsider) |
| **Status** | Draft / In review / Accepted / Rejected / Withdrawn |
| **Review deadline** | YYYY-MM-DD |
| **Tickets** | links |

## Summary

Three to five sentences: the problem, the proposal, and the expected outcome.

## Problem

What is wrong or missing today? Who is affected, how often, and what does it cost? Include data
where you have it. Reviewers should agree on this section before judging the rest.

## Goals and non-goals

- **Goals:** what success looks like, measurable where possible.
- **Non-goals:** what this deliberately does not solve.

## Proposal

How it works. Diagrams, API shapes, data model, sequence of events. Enough detail that someone
else could start implementing.

## Alternatives considered

| Option | Pros | Cons | Why not |
| --- | --- | --- | --- |
| Do nothing | | | |
| Option B | | | |

## Risks and mitigations

| Risk | Likelihood | Impact | Mitigation | How it would be detected |
| --- | --- | --- | --- | --- |
| | | | | |

For money, auth, or personal data: threat model and abuse cases.

## Rollout and rollback

- Migration steps (expand/contract, see backend/postgres-guidelines.md)
- Feature flags and rollout stages
- How to roll back at each stage

## Operability

New metrics, alerts, dashboards, runbook entries, on-call impact.

## Open questions

- Question, and who can answer it

## Decision

Filled in by the decider: accepted or rejected, date, reasoning, and links to resulting ADRs.
