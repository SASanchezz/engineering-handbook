# Observability

Anyone should be able to answer "is it working, and if not, why?" from dashboards and logs, without
deploying new code. Observability is part of the feature, not an afterthought (see
[definition of done](../process/definition-of-done.md)).

## The three signals

| Signal | Answers | Default |
| --- | --- | --- |
| Logs | What happened in this request? | Structured JSON to stdout, collected centrally |
| Metrics | How is the system behaving overall? | Prometheus-style counters, histograms, gauges |
| Traces | Where did the time go across services? | OpenTelemetry, propagated via W3C `traceparent` |

All three share a correlation ID so anyone can jump from a metric spike to traces to the logs of one
request.

## Logs

### Format

One JSON object per line. Example from a NestJS service:

```json
{
  "level": "info",
  "time": "2026-03-04T10:15:22.481Z",
  "service": "wallet-api",
  "version": "v1.14.0",
  "env": "production",
  "traceId": "4bf92f3577b34da6a3ce929d0e0e4736",
  "requestId": "req_01HQ8Z6Y3K",
  "msg": "bet settled",
  "betId": "bet_8812",
  "walletId": "wal_2231",
  "outcome": "won",
  "payoutMinor": 4500,
  "currency": "EUR",
  "durationMs": 38
}
```

### Rules

- **Structured fields, not string interpolation.** `msg` is a constant phrase; data goes in
  fields. That makes logs searchable and aggregatable.
- **Levels mean something:**

| Level | Use for | Example |
| --- | --- | --- |
| `error` | Something failed and needs attention | Payment provider returned 500 after retries |
| `warn` | Unexpected but handled | Retried a serialization failure |
| `info` | Business events worth keeping | Deposit completed, bonus granted |
| `debug` | Developer detail, off in production by default | Query parameters, cache hits |

- **Never log:** secrets, tokens, passwords, full card data, documents, full request bodies on
  money or auth endpoints. Email and phone are masked. See
  [security baseline](security-baseline.md).
- Log once per failure, at the boundary that handles it. Not at every layer it passes through.

## Metrics

### What every service exposes

Use RED for request-driven services and USE for resources.

| Metric | Type | Labels |
| --- | --- | --- |
| `http_requests_total` | counter | `route`, `method`, `status_class` |
| `http_request_duration_seconds` | histogram | `route`, `method` |
| `db_query_duration_seconds` | histogram | `operation` |
| `db_pool_in_use` / `db_pool_waiting` | gauge | |
| `queue_messages_processed_total` | counter | `queue`, `result` |
| `queue_lag_seconds` | gauge | `queue` |
| `job_last_success_timestamp` | gauge | `job` |

### Business metrics

Technical metrics tell you the service is up. Business metrics tell you it is working.

- deposits, withdrawals, and bets per minute, by result,
- bonuses granted, wagering completed, bonuses expired,
- referral sign-ups and payouts,
- ledger integrity: unbalanced transactions (should be zero, always).

A drop in deposits per minute with a green HTTP dashboard is still an incident.

### Label discipline

Never use unbounded values (user IDs, wallet IDs, raw URLs) as labels. They explode cardinality and
cost. Use them in logs and traces instead.

## Traces

- Auto-instrument HTTP, Postgres, Redis, and queue clients with OpenTelemetry.
- Add manual spans around meaningful domain steps: `ledger.post`, `bonus.evaluate`,
  `provider.payment.authorize`.
- Propagate context through queues by putting `traceparent` in message headers.
- Sample: 100% of errors, a percentage of successes tuned to cost.

## SLOs

Set SLOs for user-facing journeys, not for individual services.

| Journey | SLI | SLO (28-day window) |
| --- | --- | --- |
| Place a bet | Requests answered successfully in < 500 ms | 99.5% |
| Deposit | Deposits completed without platform error | 99.9% |
| Withdrawal request | Accepted without platform error | 99.9% |
| Balance read | Successful in < 200 ms | 99.9% |
| Bonus wagering progress | Updated within 60 s of bet settlement | 99% |

These are starting points; tune them to what users and the business actually need. A 100% SLO
is a refusal to ship.

### Error budgets

- Budget = 1 - SLO. For 99.9% over 28 days that is about 40 minutes of failure.
- Budget healthy: ship freely.
- Budget below 25% remaining: risky changes need a second approval; reliability work moves up.
- Budget exhausted: feature releases to that journey pause until it is back in budget, except
  fixes.

## Alerting

Page on SLO burn rate, not on raw thresholds. Multi-window burn rate alerts catch both fast
outages and slow leaks while staying quiet on blips.

| Alert | Condition | Action |
| --- | --- | --- |
| Fast burn | 2% of monthly budget spent in 1 hour (and confirmed over 5 min) | Page |
| Slow burn | 10% of monthly budget spent in 3 days | Ticket |
| Ledger imbalance | Any unbalanced transaction | Page, SEV1 |
| Job missed | `job_last_success_timestamp` older than 2x its interval | Ticket, page if money job |
| Queue lag | Lag above the journey's SLO target for 10 min | Page |

Alert rules follow the hygiene rules in [on-call](on-call.md): every page links to a runbook and
has an owner.

## Dashboards

Each service has one overview dashboard with the same layout, top to bottom:

1. SLO status and remaining error budget
2. Traffic, errors, latency (RED)
3. Business metrics for that service
4. Dependencies: database, cache, queues, external providers
5. Saturation: CPU, memory, pool usage
6. Deploy markers with version tags

Same layout everywhere means on-call can read any service's dashboard without learning it first.
