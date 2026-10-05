# NestJS Conventions

NestJS provides modules, dependency injection, and a lot of rope. These conventions keep services
consistent so anyone on the team can open any repo and find their way in minutes.

## Project layout

```text
src/
├── main.ts                      bootstrap only: app, global pipes, filters, shutdown hooks
├── app.module.ts
├── config/
│   ├── config.schema.ts         validated env schema
│   └── config.module.ts
├── common/                      cross-cutting, no business logic
│   ├── errors/                  DomainError base, problem-details filter
│   ├── idempotency/
│   ├── logging/
│   └── database/
└── modules/                     optional wrapper, see rules below
    ├── wallet/
    │   ├── wallet.module.ts
    │   ├── wallet.controller.ts
    │   ├── wallet.service.ts
    │   ├── wallet.repository.ts     optional, see rules below
    │   ├── dto/
    │   ├── domain/              pure logic, no Nest imports
    │   └── wallet.service.spec.ts
    ├── ledger/
    ├── bonus/
    └── referral/
test/
└── integration/                 Testcontainers-based tests
```

Rules:

- One folder per **business domain**, not per technical layer. No top-level `controllers/` or
  `services/` folders.
- Tests sit next to the code they test; integration tests live in `test/integration/`.
- `common/` holds plumbing only. If it knows what a bonus is, it belongs in a module.
- The `modules/` wrapper is optional. Domain folders can sit directly in `src/` (the Nest CLI
  default). Add the wrapper only when `src/` gets crowded with non-domain folders (`config/`,
  `database/`, `migrations/`, `logging/`) and the domains become hard to spot.
- A separate `<domain>.repository.ts` is optional. By default the service injects the TypeORM
  repository directly. Extract a repository class when the service gets overcrowded with query
  and persistence code, so the service keeps only the use-case logic.

## File naming

File and folder names are `snake_case`, with the Nest role as a dot suffix:

| Do | Don't |
| --- | --- |
| `withdrawal_intents/` | `withdrawal-intents/`, `withdrawalIntents/` |
| `withdrawal_intents.service.ts` | `withdrawal-intents.service.ts` |
| `create_withdrawal_intent.dto.ts` | `CreateWithdrawalIntent.dto.ts` |
| `withdrawal_intents.e2e-spec.ts` | `withdrawal-intents.e2e-spec.ts` |

Nest CLI generators emit kebab-case, so rename generated files before committing. This rule
covers files only: URL paths stay kebab-case (see [API design](api-design.md)) and DB
identifiers stay `snake_case` (see [Postgres guidelines](postgres-guidelines.md)).

## Module boundaries

| Rule | Why |
| --- | --- |
| A module exports services, never repositories or entities | Other modules depend on behavior, not tables |
| Modules talk through exported services or domain events | Keeps the dependency graph explicit |
| No circular imports; `forwardRef` needs a comment and a ticket | A cycle means a boundary is wrong |
| The ledger module is the only writer of ledger tables | One place enforces double-entry |
| Domain logic in `domain/` is plain TypeScript | Unit-testable without Nest, easy to read |

Example: the bonus module needs to credit a bonus. It calls `LedgerService.post()`, which is
exported by `LedgerModule`. It never imports `LedgerEntryRepository`.

```ts
@Module({
  imports: [LedgerModule],
  controllers: [BonusController],
  providers: [BonusService, BonusRepository],
  exports: [BonusService],
})
export class BonusModule {}
```

When a service grows past roughly 400 lines or mixes two responsibilities (settlement and
wagering, say), split it. That is a [tech debt](../process/tech-debt.md) item if it cannot be done
now.

## Layers inside a module

| Layer | Does | Does not |
| --- | --- | --- |
| Controller | HTTP mapping, DTO in/out, auth guards, calls one service method | Business rules, DB access, try/catch for flow control |
| Service | Orchestrates a use case, owns the transaction boundary | Know about HTTP (`Request`, status codes) |
| Domain | Pure rules and calculations | I/O of any kind |
| Repository (optional) | Queries and persistence | Business decisions |

## DTOs and validation

Global validation pipe in `main.ts`:

```ts
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,
    forbidNonWhitelisted: true,
    transform: true,
    transformOptions: { enableImplicitConversion: false },
  }),
);
```

- Every request body, query, and param has a DTO with `class-validator` decorators.
- Separate request and response DTOs. Never return an entity from a controller.
- Amounts are validated as integers in minor units with bounds:

```ts
export class CreateWithdrawalDto {
  @IsString() @Matches(/^wal_[0-9A-Z]{10,}$/)
  walletId!: string;

  @IsInt() @IsPositive() @Max(100_000_000)
  amountMinor!: number;

  @IsIn(['EUR', 'USD', 'PLN'])
  currency!: string;
}
```

## Configuration

- One config schema, validated at startup. The app **fails to start** on missing or invalid
  config instead of failing on the first request.
- Config is injected as a typed object, not read via `process.env` in business code.
- No defaults for secrets. Defaults for non-secret values live in the schema, visible in one place.

```ts
export const configSchema = z.object({
  NODE_ENV: z.enum(['development', 'test', 'staging', 'production']),
  PORT: z.coerce.number().int().default(3000),
  DATABASE_URL: z.string().url(),
  PAYMENT_PROVIDER_API_KEY: z.string().min(1),
  WITHDRAWAL_MANUAL_REVIEW_THRESHOLD_MINOR: z.coerce.number().int().positive(),
});
```

## Dependency injection

- Constructor injection only. No `moduleRef.get()` in business code.
- Depend on interfaces for external providers (payment, KYC, messaging) with an injection token, so
  tests and failover can swap implementations.
- Default singleton scope. Request-scoped providers need a reason; they cascade and cost
  performance.

## Error handling

- Services throw **domain errors**, not `HttpException`:

```ts
export class InsufficientFundsError extends DomainError {
  readonly code = 'INSUFFICIENT_FUNDS';
  readonly status = 422;
  constructor(readonly availableMinor: bigint, readonly requestedMinor: bigint) {
    super('Insufficient funds');
  }
}
```

- One global exception filter maps `DomainError` to RFC 9457 Problem Details, maps validation
  errors to `400` with field pointers, and maps anything unknown to a `500` with only the
  `requestId`. Format in [API design](api-design.md).
- Unknown errors are logged once, at `error`, with the stack trace, in the filter. Not logged again
  at each layer.
- Never swallow errors. A `catch` either handles the error meaningfully, adds context and rethrows,
  or does not exist.

## Async, events, and jobs

- Every promise is awaited or explicitly handled. ESLint `no-floating-promises` is on.
- Events that leave the service use the outbox table, written in the same transaction as the
  change.
- Queue consumers are idempotent: deduplicate by message ID.
- Scheduled jobs record `job_last_success_timestamp` (see
  [observability](../operations/observability.md)) and are safe to run twice.

## Lifecycle

- `app.enableShutdownHooks()`; on `SIGTERM` stop taking new requests, finish in-flight work,
  close the DB pool. Kubernetes and Docker both depend on this.
- Separate health endpoints: `/health/live` (process is up) and `/health/ready` (dependencies
  reachable). Readiness failing must not kill the pod.

## Tooling

| Tool | Setting |
| --- | --- |
| TypeScript | `strict: true`, `noUncheckedIndexedAccess: true` |
| ESLint | `@typescript-eslint` recommended + type-checked rules, `no-floating-promises`, no `any` without comment |
| Formatting | Prettier, enforced in CI |
| Package manager | One per repo, lockfile committed |
| Node | Current LTS, pinned in `.nvmrc` and the Dockerfile |
