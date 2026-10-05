# Security Baseline

The minimum every service meets before it goes to production. It is not a full security program;
it is the floor. For systems that hold money, parts of this floor are higher than usual, and those
parts are called out.

## Secrets

| Rule | Detail |
| --- | --- |
| No secrets in git | Not in code, config, tests, Dockerfiles, or CI logs. Secret scanning runs pre-commit and in CI |
| Secrets come from a manager | Injected at runtime from the secret store; never baked into images |
| Scoped per service and environment | The staging wallet service cannot read production secrets |
| Rotatable | Every secret can be rotated without a code change. Rotate on any suspected leak and when people leave |
| No shared human accounts | Personal accounts with SSO and MFA; break-glass accounts are audited |

If a secret lands in git, treat it as leaked: rotate first, then clean history. Removing the
commit alone does not help.

## Dependencies and supply chain

- Lockfiles committed; CI installs with `--frozen-lockfile`.
- Automated dependency update PRs weekly, grouped by ecosystem.
- `npm audit` or equivalent in CI. High or critical vulnerabilities in runtime dependencies block
  merge unless there is a documented exception with an expiry date.
- Base images pinned by digest and rebuilt at least monthly.
- Images scanned before push to GHCR. Only images built by CI are deployable.
- New dependencies on a critical path get a quick check: maintenance activity, download count,
  license, transitive size.
- Package names suggested by an AI assistant are checked before install. Models invent plausible
  package names, and attackers register them with malicious code. A package that is days old,
  has almost no downloads, or does not match its claimed repository does not get installed.
- MCP servers, assistant plugins, and shared skills from outside the company are dependencies:
  pinned versions, a named owner, and the same review before they are added (see
  [AI agent guardrails](../process/ai-agent-guardrails.md#mcp-servers)).

## Least privilege

- **Services:** each has its own database role with only the grants it needs. Application roles
  cannot run DDL; migrations run as a separate role.
- **People:** production access is read-only by default and time-boxed for anything more, with a
  ticket reference. Direct writes to production databases need two people.
- **CI:** pipeline credentials are scoped to the registry and environments the pipeline deploys
  to. Production promotion requires a human approval.
- **AI agents:** no production credentials in their environment. Committed permission rules deny
  secret files, database clients, and pushes. Unattended agents use a bot identity, so their
  actions are attributable in the audit trail (see
  [AI agent guardrails](../process/ai-agent-guardrails.md)).

Access is reviewed quarterly and removed the same day someone leaves.

## Prompt injection

Any agent or LLM feature that reads text from outside the team can be steered by that text:
tickets, PR comments, support messages, log lines, documents, web pages. Treat it like injection
into SQL: the input is data, never instructions.

- No single agent session has untrusted input, access to private data, and a way to send data
  out at the same time. Remove at least one.
- Agents processing external content run without credentials beyond read access and without
  open outbound network.
- Instructions found inside processed content are reported to a human, not followed.

Details and examples: [AI agent guardrails](../process/ai-agent-guardrails.md#prompt-injection).

## API security basics

Mapped to the most common API risks:

| Risk | Rule |
| --- | --- |
| Broken object-level authorization | Every query is scoped to the caller: `WHERE wallet.user_id = :callerId`. Never trust an ID from the path alone |
| Broken authentication | Short-lived access tokens, rotating refresh tokens, rate-limited login, MFA for back office |
| Broken function-level authorization | Admin endpoints on a separate route prefix and guard; deny by default |
| Mass assignment | DTOs with explicit allowed fields; `whitelist: true, forbidNonWhitelisted: true` in validation |
| Excessive data exposure | Response DTOs, never raw entities |
| Lack of rate limiting | Rate limits per user and per IP on auth, money, and expensive endpoints |
| Injection | Parameterized queries only. No string-built SQL, including in "internal" tools |
| Security misconfiguration | Strict CORS allowlist, security headers, no stack traces in error responses |
| SSRF | Outbound calls only to allowlisted hosts; no user-supplied URLs fetched server-side |
| Logging sensitive data | See [observability](observability.md): no secrets, masked PII |

## Money-handling specifics

These apply to any code that moves balances: deposits, withdrawals, bets, bonuses, wagering,
referral payouts.

### Double-entry ledger

- Every money movement is a transaction made of ledger entries that **sum to zero**.
- Balances are derived from entries (or a cached projection updated in the same transaction), never
  set directly.
- Entries are append-only. Corrections are new, compensating entries that reference the original.

```text
Bet placed, stake 20.00 EUR (amounts in minor units)
  tx_9001  wallet:user_123:cash     -2000
  tx_9001  house:bets_in_play       +2000
                                     ----
                                        0

Bet won, payout 45.00 EUR
  tx_9002  house:bets_in_play       -2000
  tx_9002  house:revenue            -2500
  tx_9002  wallet:user_123:cash     +4500
                                     ----
                                        0
```

Bonus funds live in their own accounts (`wallet:user_123:bonus`) so wagering rules can tell
cash from bonus money, and conversion from bonus to cash is itself a ledger transaction.

### Idempotency

- Every money-moving request and message carries an idempotency key.
- The key, a hash of the request, and the result are stored in the same database transaction as
  the ledger entries. A retry returns the stored result; it never posts twice.
- Same key with a different payload is rejected with `422` (see
  [API design](../backend/api-design.md)).
- Consumers of provider webhooks deduplicate by the provider's event ID.

### Concurrency

- Balance-changing operations lock the wallet row (`SELECT ... FOR UPDATE`) or use a serializable
  transaction with retry. Pick one per service and document it.
- Database constraints back up code: `CHECK (balance_minor >= 0)` where overdraft is not allowed.

### Amounts

- Integer minor units (`bigint`) with a currency code, or `numeric` with fixed scale. Never
  `float`, never JavaScript `number` arithmetic on amounts beyond safe integer range. See
  [Postgres guidelines](../backend/postgres-guidelines.md).
- Rounding rules are explicit and tested, and happen in one place.

### Audit log

- Every change to money, limits, bonuses, KYC status, and permissions writes an audit record:
  who (user, service, or admin), what, before/after, when, why (request ID, ticket, or reason).
- Audit records are append-only; the application role cannot update or delete them.
- Back-office actions require a reason field. "Manual adjustment" with no reason is rejected.

### Abuse

Bonus and referral systems attract abuse. Designs include from day one: limits per user, device,
and payment method; velocity checks; and a manual review queue for payouts above a threshold.

## Before going to production

- [ ] No secrets in the repo; secret scanning enabled
- [ ] Dependencies scanned; no unaddressed critical issues
- [ ] Service has its own least-privilege database role
- [ ] AuthN and AuthZ on every endpoint; object-level checks tested
- [ ] Input validation with whitelisting on every DTO
- [ ] Rate limits on auth and money endpoints
- [ ] Money paths: ledger, idempotency, locking, and audit log in place and tested
- [ ] Logs reviewed for PII and secrets
- [ ] Threat model discussed in the RFC for anything touching money or auth
- [ ] Repo passes the [agent checklist](../process/ai-agent-guardrails.md#before-enabling-agents-in-a-repo)
      if AI agents work in it
