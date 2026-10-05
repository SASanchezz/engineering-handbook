# Testing Strategy

Tests exist to make changing code safe. A test suite that is slow, flaky, or coupled to
implementation details reduces confidence and gets ignored. Optimize for fast, deterministic
tests that fail only when behavior breaks.

## The target shape

For NestJS services backed by PostgreSQL, the classic pyramid is slightly bent: integration tests
against a real database give more confidence per line than mocked unit tests for most
data-heavy code.

| Level | Share of tests | Runs | Speed target | Tools |
| --- | --- | --- | --- | --- |
| Unit | ~60% | Every commit, locally and in CI | Full suite < 30 s | Jest or Vitest |
| Integration | ~30% | Every PR in CI | Full suite < 5 min | Jest + Testcontainers (Postgres, Redis) |
| Contract | Per API boundary | Every PR on either side | < 2 min | Pact or OpenAPI schema checks |
| End-to-end | A handful of critical journeys | Post-deploy to staging | < 10 min | Supertest or Playwright against staging |

## Unit tests

**Test:** pure domain logic. Wagering progress, bonus eligibility, fee calculation, state
machines, mappers, validators.

**Style:**

- Arrange / act / assert, one behavior per test.
- Name tests as sentences: `it('does not count voided bets towards wagering progress')`.
- No database, no network, no clock. Inject a clock; never call `Date.now()` in domain code.
- Mock only what you own and only at module boundaries. Mocking an ORM call chain is a sign the
  logic belongs in a pure function.

```ts
describe('WageringCalculator', () => {
  it('counts only settled bets at odds >= the bonus minimum', () => {
    const bonus = aBonus({ wageringMultiplier: 10, minOdds: 1.5, amountMinor: 1_000 });
    const bets = [
      aSettledBet({ stakeMinor: 2_000, odds: 2.0 }),
      aSettledBet({ stakeMinor: 5_000, odds: 1.2 }), // below min odds
      aVoidedBet({ stakeMinor: 3_000, odds: 3.0 }),  // voided
    ];

    expect(calculateProgress(bonus, bets)).toEqual({ wageredMinor: 2_000, requiredMinor: 10_000 });
  });
});
```

## Integration tests

**Test:** anything that talks to Postgres or another real dependency. Repositories, transactions,
locking, migrations, idempotency, queue consumers, the HTTP layer through Nest's testing module.

**Rules:**

- **Real Postgres via Testcontainers**, same major version as production. Never SQLite, never
  `pg-mem` for anything with transactions or locks.
- One container per test run, migrations applied once. Each test runs in a transaction that is
  rolled back, or truncates its tables. Pick one approach per repo.
- Use builders/factories for data, not shared fixtures that every test mutates.
- Test the failure paths: unique violation, serialization failure, lock timeout, duplicate
  message.

Things that must have integration tests on money paths:

- [ ] The same idempotency key twice returns the same result and writes once.
- [ ] Two concurrent requests on the same wallet do not overdraw it.
- [ ] A failure mid-transaction leaves no partial ledger entries.
- [ ] Ledger entries for each operation sum to zero.

## Contract tests

When service A calls service B, a change to B can break A without any test in B failing.

- Consumer-driven contracts (Pact) for internal service-to-service HTTP calls.
- For public APIs, the OpenAPI spec is the contract. CI fails if a PR introduces a breaking change
  to the spec without a new version (see [API design](../backend/api-design.md)).
- Events on a queue have a versioned schema. Consumers test against recorded examples of each
  version they support.

## End-to-end tests

Few, slow, and valuable only for the journeys that would be a disaster if broken:

- sign up, deposit, place bet, settle, withdraw,
- claim a bonus and complete wagering,
- referral sign-up and payout.

They run after every deploy to staging. A failing E2E blocks promotion to production.

## What not to test

| Not worth testing | Why |
| --- | --- |
| Framework behavior (Nest DI wiring, class-validator itself) | Tested upstream; test the service's DTO rules instead |
| Private methods directly | Test through the public behavior |
| Trivial getters, DTO shape-only classes | No logic, no risk |
| Exact log messages | Brittle; assert on structured fields only if they matter |
| Snapshot tests of large JSON | They get updated blindly; assert on the fields that matter |
| Third-party APIs live in CI | Use recorded fixtures or a sandbox in a separate, non-blocking job |

## Coverage

Track coverage; do not target it. Line coverage below 70% on a service is worth a look;
above that, the question is whether the risky paths are covered, not the percentage. Money paths
are expected to be close to fully covered by integration tests.

## Flaky tests

A flaky test is a bug. Policy:

1. First flake: open a ticket, label `flaky`.
2. Second flake in a week: quarantine it (skip with a ticket link) and fix within the sprint.
3. Never "retry until green" in CI without a ticket.

Most flakes in this stack come from shared database state, real clocks, and unawaited promises.
Check those first.

## Testing AI-written code

Code generated with an assistant gets the same bar. The risk is different, though: an agent that
writes both the code and the tests can make them agree with each other instead of with the
requirement. The rules below keep tests anchored to what the business actually needs.

### Tests are the spec

- A person writes or approves the test cases before the agent implements. In the
  [Jira-to-code workflow](ai-assisted-development.md#the-jira-to-code-workflow) they are part of
  the plan: names of the test cases, as sentences, before any code.
- On money paths (ledger, wagering, bonus, payouts), the integration test cases from
  [the list above](#integration-tests) are written or reviewed line by line by a human.
- The agent may add tests. It may not delete tests, remove assertions, loosen expected values, or
  add `.skip` to get to green. A hook blocks this on money paths (see
  [AI agent guardrails](ai-agent-guardrails.md#hooks)); review catches it elsewhere.
- Snapshots are never regenerated by an agent. A changed snapshot is reviewed field by field.

### Property-based tests for money logic

Example-based tests check the cases someone thought of. Property-based tests check rules that
must hold for any input, which is where generated code tends to be subtly wrong. Use
[fast-check](https://fast-check.dev/) for invariants such as:

- every ledger transaction sums to zero,
- wagering progress never decreases and never exceeds the requirement,
- applying the same idempotency key twice changes nothing the second time.

```ts
it('posts balanced ledger entries for any bet settlement', () => {
  fc.assert(
    fc.property(aStakeMinor(), anOdds(), aSettlementOutcome(), (stakeMinor, odds, outcome) => {
      const entries = settleBet({ stakeMinor, odds, outcome });

      expect(sumMinor(entries)).toBe(0n);
    }),
  );
});
```

### Mutation testing on critical modules

Coverage says a line ran; it does not say a test would notice if the line were wrong. Run
mutation testing ([Stryker](https://stryker-mutator.io/)) on money-path modules nightly, not on
every PR. A surviving mutant in the ledger or wagering code is a missing test and gets a ticket.
This is the most reliable way to catch tests that assert the implementation instead of the
requirement.
