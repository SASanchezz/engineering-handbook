# Incident Response

An incident is any unplanned event that degrades service for users or puts money or data at risk.
When in doubt, declare. Declaring an incident that turns out minor costs ten minutes. Not
declaring one that turns out major costs trust.

## Severity levels

| Severity | Definition | Examples | Response | Updates |
| --- | --- | --- | --- | --- |
| **SEV1** | Critical user impact or money/data at risk | Platform down; deposits or withdrawals failing broadly; wrong balances; data leak | Page on-call immediately, IC assigned, all hands as needed, 24/7 | Every 30 min |
| **SEV2** | Major feature degraded for many users, no money at risk | Bet placement latency 10x; bonus claims failing; one payment provider down with fallback | Page on-call, IC assigned, business hours plus on-call after hours | Every 60 min |
| **SEV3** | Minor degradation or a small group of users affected | Referral dashboard stale; one region slow; non-critical job failing | Handled by on-call in working hours | Start and end |
| **SEV4** | No user impact yet, but a risk | Disk at 80%; certificate expiring in 7 days; error rate creeping up | Ticket, fix within the sprint | None |

Rules:

- **Anything touching balances, ledgers, or payouts is at least SEV2**, and SEV1 if money may
  have moved incorrectly.
- Severity can go up or down during the incident. The IC decides.
- Security incidents follow this process, with a smaller channel and the security owner looped in
  from the start.

## Roles

For SEV1 and SEV2, assign roles explicitly. One person can hold more than one role in a small
team, but the IC is never also the person typing the fix on a SEV1.

| Role | Responsibilities | Does not |
| --- | --- | --- |
| **Incident Commander (IC)** | Owns the incident. Sets severity, assigns roles, decides on actions, calls for help, declares resolved | Debug hands-on during SEV1 |
| **Operations lead** | Investigates and applies fixes. May pull in subject experts | Communicate externally |
| **Communications lead** | Posts updates to stakeholders and status page on cadence, shields responders from questions | Speculate on root cause |
| **Scribe** | Keeps a timestamped timeline in the incident doc: actions, findings, decisions | Filter; they write everything down |

The on-call engineer is IC by default until they hand it off. Handing off is explicit:
"Handing IC to Maria." "Taking IC."

## Timeline of a typical incident

```text
T+0      Alert fires or someone reports a problem
T+5m     On-call acknowledges, assesses, declares severity
         Creates channel #inc-YYYYMMDD-short-name and incident doc
T+10m    IC assigns roles (SEV1/SEV2); first update posted
T+...    Mitigate first: rollback, flag off, failover, scale. Root cause later.
         Updates on cadence, even if "no change"
Resolve  IC confirms metrics back to normal for 15+ minutes; declares resolved
         Final update posted
+2 days  Postmortem draft (SEV1/SEV2)
+5 days  Postmortem review meeting; action items in Jira with owners
```

## Mitigate first

The first goal is to stop the bleeding, not to understand it. In order of preference:

1. **Turn off the feature flag** or kill switch. Target: under 2 minutes.
2. **Roll back** to the previous release tag. Target: under 10 minutes (see
   [branching and releases](../process/branching-and-releases.md)).
3. **Fail over or disable** the broken dependency: switch payment provider, pause a consumer,
   put a non-critical feature in maintenance mode.
4. **Scale** if it is load-related.
5. **Fix forward** only when the fix is small, understood, and faster than the above.

### Money-specific steps

If balances or ledgers may be wrong:

- Pause the affected money flows (withdrawals first) via kill switch before investigating.
- Do not "correct" balances manually in the database. Corrections are compensating ledger
  entries, reviewed by two engineers, with a reference to the incident.
- Snapshot the relevant tables and audit log rows before any correction.
- Involve finance or operations early; they own customer-facing decisions on corrections.

## Communication

### Where

| Audience | Channel |
| --- | --- |
| Responders | `#inc-...` channel and a call for SEV1 |
| Engineering and product | `#incidents` summary updates |
| Customer support and business | Their agreed channel, from the comms lead |
| Customers | Status page, wording approved by the IC |

### Cadence

- SEV1: every 30 minutes. SEV2: every 60 minutes. Even when nothing changed. Silence reads as
  chaos.
- Use the [incident update template](../templates/incident-update-template.md). Facts, impact,
  what is being done, next update time.
- Do not post root cause guesses externally.

## After the incident

- **Postmortem required** for every SEV1 and SEV2, and for any SEV3 that touched money or repeated
  within 30 days. Use the [postmortem template](../templates/postmortem-template.md).
- The IC owns the postmortem being written; the scribe usually drafts it.
- Postmortems are **blameless**: describe what people did and why it made sense at the time,
  and focus on what in the system allowed the failure.
- Action items go into Jira with one owner and a due date. Open items are reviewed weekly until
  closed.
- Share postmortems with the whole engineering org. They are the cheapest training available.

## Practice

- New on-call engineers shadow one incident or run one game day before taking the primary pager
  (see [on-call](on-call.md)).
- Once a quarter, run a game day: inject a failure in staging (kill the database primary, break
  a payment provider, fill a disk) and run the full process including comms.

## Checklist for the IC

- [ ] Severity declared and posted
- [ ] Channel and incident doc created
- [ ] Roles assigned (SEV1/SEV2)
- [ ] First update sent within 15 minutes of declaration
- [ ] Mitigation options listed and one chosen
- [ ] Money flows paused if balances may be affected
- [ ] Updates on cadence
- [ ] Resolution confirmed with metrics, final update sent
- [ ] Postmortem owner and date set
