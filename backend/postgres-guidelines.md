# PostgreSQL Guidelines

PostgreSQL is the default datastore. These rules keep schema changes safe on live tables, keep
money correct, and keep queries fast as data grows.

## Schema basics

| Rule | Why |
| --- | --- |
| `snake_case` tables (plural) and columns | Postgres folds unquoted identifiers to lowercase |
| Every table has a primary key | Replication, tooling, and sanity |
| `created_at timestamptz not null default now()` and `updated_at` | Debugging and audits |
| `timestamptz`, never `timestamp` | Time zones bite eventually |
| `text` with a `CHECK` instead of `varchar(n)` | Same performance, explicit constraint |
| Foreign keys on by default, indexed | Integrity; unindexed FKs make deletes and joins slow |
| `NOT NULL` unless null has a meaning | Nulls spread bugs |
| Constraints in the database, not only in code | Code has bugs; constraints hold |

## Money

- Store amounts as `bigint` in **minor units** (`amount_minor bigint not null`) plus a
  `currency char(3) not null`. This is the default.
- Use `numeric(20, 8)` or similar only when sub-minor precision is needed (FX rates, crypto
  amounts). Never `real`, `double precision`, or `money`.
- In TypeScript, read `bigint` columns as `bigint` or strings, not `number`. The driver must be
  configured for this explicitly.
- Back invariants with constraints:

```sql
CREATE TABLE ledger_entries (
  id             bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  transaction_id text   NOT NULL,
  account_id     text   NOT NULL,
  amount_minor   bigint NOT NULL CHECK (amount_minor <> 0),
  currency       char(3) NOT NULL,
  created_at     timestamptz NOT NULL DEFAULT now()
);

CREATE INDEX ledger_entries_account_created_idx
  ON ledger_entries (account_id, created_at DESC);
```

Ledger design is described in the [security baseline](../operations/security-baseline.md).

## Migrations: expand and contract

Every migration must be **backward compatible with the release currently in production**. That is
what makes deploys and rollbacks safe (see
[branching and releases](../process/branching-and-releases.md)).

Never rename or drop a column in one step. Example: rename `users.name` to `users.full_name`.

| Step | Release | Change |
| --- | --- | --- |
| 1. Expand | `v1.20.0` | `ALTER TABLE users ADD COLUMN full_name text;` (nullable, no default rewrite) |
| 2. Dual write | `v1.20.0` | Code writes both `name` and `full_name`, reads `name` |
| 3. Backfill | Job after deploy | Copy in batches: `UPDATE ... WHERE id BETWEEN ...` with pauses |
| 4. Switch reads | `v1.21.0` | Code reads `full_name`, still writes both |
| 5. Constrain | `v1.21.0` | Add `NOT NULL` via a validated `CHECK` constraint (see below) |
| 6. Stop old writes | `v1.22.0` | Code no longer touches `name` |
| 7. Contract | `v1.23.0` | `ALTER TABLE users DROP COLUMN name;` |

Each step is safe to roll back to the previous release. Steps 6 and 7 are separate releases so a
rollback from step 7 never meets code that needs `name`.

## Locks: what blocks production

Most outages from migrations are lock queues, not slow DDL. An `ALTER TABLE` waiting for an
`ACCESS EXCLUSIVE` lock blocks every query behind it.

Every migration session sets:

```sql
SET lock_timeout = '3s';
SET statement_timeout = '15min';
```

If it cannot get the lock in 3 seconds, it fails and is retried, instead of freezing the table.

### Safe and unsafe operations

| Operation | Safe? | Do this instead |
| --- | --- | --- |
| `ADD COLUMN` nullable, no default | Safe | |
| `ADD COLUMN ... DEFAULT <constant>` | Safe on modern Postgres | Volatile defaults (`now()`, random) still rewrite; avoid |
| `CREATE INDEX` | **Blocks writes** | `CREATE INDEX CONCURRENTLY` (outside a transaction) |
| `DROP INDEX` | Blocks | `DROP INDEX CONCURRENTLY` |
| `ADD FOREIGN KEY` | Blocks both tables while validating | Add `NOT VALID`, then `VALIDATE CONSTRAINT` separately |
| `SET NOT NULL` | Full scan under exclusive lock | Add `CHECK (col IS NOT NULL) NOT VALID`, validate, then `SET NOT NULL` |
| `ALTER COLUMN TYPE` | Usually rewrites the table | New column + expand/contract |
| `RENAME COLUMN` / `DROP COLUMN` | Breaks running code | Expand/contract |
| Large `UPDATE` / `DELETE` | Long locks, bloat, replication lag | Batches of 1k to 10k rows with pauses |

Example: adding an index and a foreign key without blocking writes:

```sql
-- migration 1 (no transaction wrapper)
CREATE INDEX CONCURRENTLY IF NOT EXISTS bonuses_wallet_id_idx ON bonuses (wallet_id);

-- migration 2
ALTER TABLE bonuses
  ADD CONSTRAINT bonuses_wallet_id_fk FOREIGN KEY (wallet_id) REFERENCES wallets (id) NOT VALID;

-- migration 3
ALTER TABLE bonuses VALIDATE CONSTRAINT bonuses_wallet_id_fk;
```

If a `CONCURRENTLY` build fails, it leaves an `INVALID` index. Drop it and retry.

## Transactions

- **Short.** No network calls (payment providers, other services, queues) inside a database
  transaction. Use the transactional outbox pattern to publish events.
- **Explicit isolation.** Default `READ COMMITTED`. For balance changes, either lock rows with
  `SELECT ... FOR UPDATE` or use `SERIALIZABLE` with a retry on `40001`. Document which one the
  module uses.
- **Consistent lock order.** Lock wallets in ascending ID order when touching more than one, to
  avoid deadlocks.
- **Idempotency in the same transaction.** The idempotency record and the ledger entries commit
  together or not at all.
- Set `idle_in_transaction_session_timeout` so a forgotten transaction cannot hold locks forever.

## Queries and indexes

- Every new query on a large table gets an `EXPLAIN (ANALYZE, BUFFERS)` against realistic data in
  the PR when it is on a hot path.
- Index for the `WHERE` and `ORDER BY` you actually run. Composite index column order matters.
- Partial indexes for hot subsets: `WHERE status = 'pending'`.
- No `SELECT *` in application code.
- Watch for N+1 from the ORM in list endpoints; fetch in one query or batch.
- `pg_stat_statements` is enabled everywhere; review the top 10 by total time monthly.

## Operations

- Connection pooling (PgBouncer or the driver pool) sized deliberately; total connections across
  all pods stay well below `max_connections`.
- Point-in-time recovery enabled; **restores are tested** quarterly, not assumed.
- Separate roles: migration role (DDL) and application role (DML only).
- Long-running analytics go to a replica, never the primary.

## Migration PR checklist

- [ ] Backward compatible with the release in production
- [ ] `lock_timeout` set; no unsafe operation from the table above
- [ ] Indexes created `CONCURRENTLY`
- [ ] Backfills batched and timed against production-sized data
- [ ] Destructive step scheduled as a separate, later release with its own ticket
- [ ] Reviewed by a second engineer (migrations on hot tables need 2 approvals, see
      [code review](../process/code-review.md))
