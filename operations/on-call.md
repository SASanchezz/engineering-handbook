# On-Call

On-call exists so that production problems get a fast, competent response, and so that the people
who build a system feel its pain directly. It should be a sustainable part of the job, not a tax
on the few people who know the system.

## Rotation

| Setting | Default |
| --- | --- |
| Shift length | 1 week, Monday to Monday at a time both sides are awake |
| Roles | Primary and secondary |
| Team size needed | At least 5 engineers in the rotation; below that, share with another team |
| Frequency | No one is primary more than 1 week in 5 |
| Follow the sun | When the team spans EU and US, split the day so nobody is paged overnight as routine |
| Acknowledge time | 5 minutes for pages (SEV1/SEV2), next business morning for tickets |
| Who rotates | Everyone who ships to the service, including seniors and the tech lead |

New engineers join the rotation after onboarding: first as secondary shadowing a primary, then
primary with an experienced secondary.

## Responsibilities

**Primary**

- Acknowledge pages within 5 minutes; act as IC by default (see
  [incident response](incident-response.md)).
- Triage incoming alerts and bugs tagged for on-call.
- Keep a short log of what fired and what you did.
- Improve one thing during the week: a noisy alert, a missing runbook step, a dashboard.

**Secondary**

- Back up the primary if they do not acknowledge within 10 minutes or need help.
- Take over if the primary is in an incident and a second one starts.

**Not on-call's job**

- Feature work deadlines. The primary's planned work is reduced; sprints are planned with that in
  mind.
- Fixing every bug found. Triage, mitigate, ticket. The owning team prioritizes the fix.

## Handoff

At the end of each shift, outgoing primary posts a handoff note in the on-call channel and walks
the incoming primary through it for 15 minutes.

```text
On-call handoff: 2026-03-02 -> 2026-03-09
Pages: 3 (2 actionable, 1 noise)
Incidents: 1 SEV3 - referral payout job timed out, rerun manually, PAY-622 opened
Ongoing: payment provider B intermittent 502s, watching, vendor ticket open
Noisy alerts: "wallet-api p99 > 500ms" fires on nightly batch, PAY-623 to fix threshold
Changes this week: v1.14.0 released Tue, flag wallet.wagering-progress-v2 at 50%
Watch out for: month-end payout run on Saturday
```

## Load and fairness

These lines hold because burned-out on-call engineers make poor decisions at 3 a.m.

- **Compensation.** On-call is paid or given time off in line with local rules and company
  policy. It is never "part of the salary" by assumption.
- **Night pages earn rest.** Paged after midnight? Start late the next day, no questions.
- **Out-of-hours paging target:** fewer than 2 out-of-hours pages per week per rotation. If a rotation goes
  over for two weeks running, reducing pages becomes the team's top priority.
- **Swaps are easy.** Arrange a swap with a colleague and update the schedule; no approval needed.
- **Life comes first.** Travel, illness, and family events take priority; the lead finds cover.

## Alert hygiene

Every page should be **urgent, actionable, and real.** If it is not all three, it is not a page.

| Rule | Why |
| --- | --- |
| Page on symptoms users feel (error rate, latency, failed payments), not causes (CPU high) | Causes do not always hurt users; symptoms always do |
| Every alert links to a runbook entry | On-call should not have to guess at 3 a.m. |
| Every alert has an owner team | Orphaned alerts rot |
| Non-urgent alerts create tickets, not pages | Pages are for now; tickets are for this week |
| An alert that fired and needed no action is fixed or deleted within the week | Noise trains people to ignore pages |
| Thresholds are reviewed after every incident they missed or falsely triggered | Alerts drift as traffic changes |

See [observability](observability.md) for how alerts are defined from SLOs.

## Runbooks

Runbooks live in the service repo under `docs/runbooks/` and are linked from each alert. Each entry
has:

```text
Alert:     WalletLedgerImbalance
Meaning:   Sum of ledger entries for a transaction is not zero
Severity:  SEV1 - money may be wrong
First 5 minutes:
  1. Pause withdrawals: set flag wallet.withdrawals-enabled = false
  2. Find affected transactions: dashboard "Ledger integrity", panel "Unbalanced tx"
  3. Declare incident, page secondary and wallet owner
Do not:    edit balances or ledger rows by hand
Escalate:  wallet tech lead, then engineering lead
```

## Monthly review

Once a month the team spends 30 minutes on on-call health:

- pages per week, out-of-hours pages, time to acknowledge,
- top 3 noisiest alerts and what is being done about them,
- runbook gaps found during the month,
- whether the load is fair across the rotation.
