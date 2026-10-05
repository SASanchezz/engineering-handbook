# API Design

Public and internal HTTP APIs follow the same conventions so that clients, including other
teams and future maintainers, can guess how an endpoint behaves before reading its docs.

## Resources and URLs

| Rule | Good | Bad |
| --- | --- | --- |
| Plural nouns for collections | `/v1/wallets/{walletId}/transactions` | `/v1/getWalletTransactions` |
| kebab-case paths | `/v1/bonus-campaigns` | `/v1/bonusCampaigns` |
| camelCase JSON fields | `"createdAt"` | `"created_at"` |
| Nest at most one level | `/v1/wallets/{id}/transactions` | `/v1/users/{id}/wallets/{id}/transactions/{id}/entries` |
| Actions that are not CRUD as sub-resources | `POST /v1/withdrawals/{id}/cancellation` | `POST /v1/cancelWithdrawal` |
| Prefixed, opaque IDs | `"wal_01HQ8Z6Y3K"` | `"12345"` |

Prefixed IDs make logs readable and stop a bet ID from being passed where a wallet ID is expected.

## Methods and status codes

| Method | Use | Success |
| --- | --- | --- |
| `GET` | Read; safe and cacheable | `200` |
| `POST` | Create or trigger an action | `201` with `Location`, or `202` for async |
| `PATCH` | Partial update | `200` |
| `PUT` | Full replace (rare) | `200` |
| `DELETE` | Remove | `204` |

Common error codes:

| Code | When |
| --- | --- |
| `400` | Malformed request (bad JSON, wrong types) |
| `401` | Not authenticated |
| `403` | Authenticated but not allowed |
| `404` | Not found, or exists but caller may not see it |
| `409` | Conflicts with current state (e.g., withdrawal already processed) |
| `422` | Valid shape, but fails business rules (insufficient funds, bonus not eligible) |
| `429` | Rate limited; include `Retry-After` |
| `500` | Server bug |
| `503` | Temporarily unavailable; include `Retry-After` if known |

## Errors: RFC 9457 Problem Details

Every error response uses `application/problem+json`
([RFC 9457](https://www.rfc-editor.org/rfc/rfc9457)). Clients get one shape for every failure.

```http
HTTP/1.1 422 Unprocessable Content
Content-Type: application/problem+json
```

```json
{
  "type": "https://api.example.com/problems/insufficient-funds",
  "title": "Insufficient funds",
  "status": 422,
  "detail": "Available balance is 12.50 EUR; withdrawal requested 20.00 EUR.",
  "instance": "/v1/withdrawals",
  "code": "INSUFFICIENT_FUNDS",
  "requestId": "req_01HQ8Z6Y3K",
  "availableMinor": 1250,
  "requestedMinor": 2000,
  "currency": "EUR"
}
```

Validation errors list every field problem at once:

```json
{
  "type": "https://api.example.com/problems/validation-error",
  "title": "Request validation failed",
  "status": 400,
  "code": "VALIDATION_ERROR",
  "requestId": "req_01HQ8Z7A1B",
  "errors": [
    { "pointer": "/amountMinor", "detail": "must be a positive integer" },
    { "pointer": "/currency", "detail": "must be one of EUR, USD, PLN" }
  ]
}
```

Rules:

- `code` is a stable, machine-readable string clients can switch on. `title` and `detail` are for
  humans and may change.
- Never leak stack traces, SQL, or internal hostnames. `500` responses say "Internal error" plus
  the `requestId`.
- One global exception filter produces this shape (see
  [NestJS conventions](nestjs-conventions.md)).

## Pagination

Cursor-based by default. Offset pagination breaks on large or fast-changing tables, which is every
transaction table.

```http
GET /v1/wallets/wal_2231/transactions?limit=50&cursor=eyJpZCI6InR4XzkwMDEifQ
```

```json
{
  "data": [ { "id": "tx_9001", "type": "bet_stake", "amountMinor": -2000, "currency": "EUR" } ],
  "page": {
    "nextCursor": "eyJpZCI6InR4Xzg5NTAifQ",
    "hasMore": true
  }
}
```

- `limit` defaults to 20, max 100.
- Cursors are opaque to clients (base64 of the sort key) and backed by an index on the sort key.
- Every list endpoint is paginated from day one, even if the list is small today.
- Filters are query parameters: `?type=bet_stake&createdAfter=2026-03-01T00:00:00Z`.

## Idempotency keys

Any `POST` that creates something or moves money accepts an `Idempotency-Key` header. For money
endpoints it is **required**.

```http
POST /v1/withdrawals
Idempotency-Key: 6f1c2a9e-7d4b-4c55-9a1e-2f4b8d3c0e71
Content-Type: application/json

{ "walletId": "wal_2231", "amountMinor": 2000, "currency": "EUR" }
```

Server behavior:

| Situation | Response |
| --- | --- |
| First request with this key | Process, store key + request hash + response, return result |
| Retry with same key and same body, completed | Return the stored response, same status code |
| Retry with same key while first is in progress | `409` with `code: REQUEST_IN_PROGRESS` |
| Same key, different body | `422` with `code: IDEMPOTENCY_KEY_REUSED` |

Keys are scoped per client, stored at least 24 hours, and written in the same database transaction
as the effect. Details in the [security baseline](../operations/security-baseline.md).

## Money in APIs

- Amounts are integers in minor units with an explicit currency: `"amountMinor": 2000,
  "currency": "EUR"`. Never decimals as JSON numbers.
- If a client needs a display value, send a separate formatted string; do not make clients parse
  it back.

## Dates and time

- ISO 8601 in UTC with `Z`: `"2026-03-04T10:15:22.481Z"`.
- Field names end in `At` for instants (`settledAt`) and `On` for dates (`expiresOn`).

## Versioning

- Major version in the path: `/v1/...`. Bump it only for breaking changes.
- **Non-breaking (no new version):** adding endpoints, optional request fields, response fields,
  new enum values only if clients were told to tolerate unknowns.
- **Breaking (new version):** removing or renaming fields, changing types or meaning, making an
  optional field required, changing error codes.
- Old versions get a `Deprecation` and `Sunset` header and at least 3 months notice for external
  clients.
- CI diffs the OpenAPI spec on every PR and fails on breaking changes to an existing version (see
  [testing strategy](../process/testing-strategy.md)).

## Documentation

- OpenAPI spec generated from code (Nest Swagger decorators) and committed, so diffs are reviewable.
- Every endpoint documents its error `code`s, not only success.
- Example requests and responses for every money endpoint.

## Checklist for a new endpoint

- [ ] Resource-oriented path, correct method and status codes
- [ ] Request and response DTOs, validated and documented
- [ ] Authorization checks scoped to the caller
- [ ] Errors as Problem Details with stable `code`s
- [ ] Pagination if it returns a list
- [ ] `Idempotency-Key` if it creates or moves money
- [ ] Rate limit if it is expensive or sensitive
- [ ] No breaking change to an existing version
