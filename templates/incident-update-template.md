# Incident Update Template

Post on the cadence for the severity (SEV1 every 30 min, SEV2 every 60 min), even if nothing has
changed. Facts only, no root cause speculation. Process:
[incident response](../operations/incident-response.md).

## Internal update

```text
[SEV2] Withdrawals delayed - Update #3 - 14:45 UTC
Status:      Investigating | Identified | Mitigating | Monitoring | Resolved
Impact:      Withdrawal requests accepted but not processed since 14:02 UTC.
             ~300 requests queued. No funds lost; balances correct.
Current:     Payment provider B returning 502s. Failover to provider A in progress.
Next steps:  Complete failover (ETA 15:00), then drain queue.
Ask:         Support: use macro "withdrawal-delay" for incoming tickets.
IC:          name   Comms: name
Next update: 15:15 UTC
```

## External / status page update

```text
Title:   Delayed withdrawals
Status:  Identified

We are aware that some withdrawals are taking longer than usual to process.
Your balance is not affected, and all requests will be completed. We have
identified the cause and are working on a fix.

Next update by 15:15 UTC.
```

## Resolution update

```text
[SEV2] Withdrawals delayed - Resolved - 15:40 UTC
Impact:     Withdrawals delayed 14:02 to 15:30 UTC. ~420 requests affected, all processed.
Fix:        Failed over to provider A; provider B under investigation with vendor.
Follow-up:  Postmortem by YYYY-MM-DD, owner: name.
```

## Checklist

- [ ] Severity and update number in the title
- [ ] Impact in user terms, with numbers if known
- [ ] Whether money or data is at risk, stated explicitly
- [ ] What is being done now and what comes next
- [ ] Any ask of the reader
- [ ] Time of the next update
