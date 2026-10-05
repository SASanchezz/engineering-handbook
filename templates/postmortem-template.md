# Postmortem: Short description of the incident

<!--
Blameless: describe what happened and why it made sense at the time. Focus on systems,
not individuals. Name roles, not people, in the timeline unless they agree otherwise.
Draft within 2 business days; review within 5. Process: operations/incident-response.md.
-->

| | |
| --- | --- |
| **Date** | YYYY-MM-DD |
| **Severity** | SEV1 / SEV2 / SEV3 |
| **Duration** | Detected HH:MM UTC, mitigated HH:MM, resolved HH:MM |
| **Incident commander** | name |
| **Author** | name |
| **Status** | Draft / Reviewed / Actions complete |

## Summary

Two to four sentences: what broke, who was affected, how long, how it was fixed.

## Impact

- Users affected (number or percentage) and what they experienced
- Money impact: transactions failed, delayed, or incorrect; amounts corrected
- SLO and error budget consumed
- Internal impact: support tickets, manual work

## Timeline (UTC)

| Time | Event |
| --- | --- |
| 14:02 | Deploy of vX.Y.Z to production |
| 14:09 | Alert fired: ... |
| 14:12 | On-call acknowledged, declared SEV2 |
| 14:20 | Mitigation: ... |
| 14:35 | Metrics back to normal |
| 14:50 | Resolved |

## Root cause and contributing factors

What in the system allowed this to happen? Ask "why" until you reach things the team can change.
Usually there is more than one factor.

1.
2.

## Detection

How was it detected? How long did it take? Would it have been noticed without a customer report?

## Response

What went well, what slowed the response down (access, missing runbook, unclear dashboards, comms).

## What went well

-

## Where luck helped

-

## Action items

| Action | Type | Owner | Due | Ticket |
| --- | --- | --- | --- | --- |
| | Prevent / Detect / Mitigate / Process | | | |

Each action has one owner and a due date. Open items are reviewed weekly until all are closed.

## Lessons

What should another team know to avoid the same problem?
