# Specification: `of-sys-acct-core-banking-v1`

> Status: design draft for review. This is not an approved contract and does not describe an actual Core Banking integration. Methods, paths, fields, and example data come from the [API-Led example](../open-finance-api-led-example.md), which identifies them as fictitious.

## Purpose and context

- **Layer and domain:** Banking accounts System API (`acct`).
- **Intended consumer:** the Process API or another authorized consumer that needs account-query capabilities.
- **Purpose:** expose account reads and writes, balance queries, and auditable balance adjustments while isolating consumers from underlying banking-system details.
- **Architectural relationship:** the example mentions `of-prc-port-investment-summary-v1` as possible context. It does not assert that the Process API queries balances or define how they would be used in the summary.
- **Proposed base path:** `/open-finance/system/accounts/v1` (approval pending).
- **GitLab project:** an independent design project with this identifier; it is not created here.
- **Version:** initial contract version `1.0.0`; identifier and base path use major version `v1`. Future compatibility policy remains pending.

## Proposed operations

Read operations have no request body. Write operations use request bodies where shown and do not physically delete accounts or overwrite derived balances.

| ID | Proposed method and full path | Parameters | Proposed responses |
| --- | --- | --- | --- |
| `ACCT-OP-001` | `GET /open-finance/system/accounts/v1/customers/{customerId}/accounts` | Required path `customerId`, `string`; scope `accounts.read` | `200` collection of `AccountReference`; `400`, `401`, `403`, `404`, `429`, `500`, `503` |
| `ACCT-OP-002` | `GET /open-finance/system/accounts/v1/accounts/{accountId}` | Required path `accountId`, `string`; scope `accounts.read` | `200 AccountReference`; `401`, `403`, `404`, `429`, `500`, `503` |
| `ACCT-OP-003` | `GET /open-finance/system/accounts/v1/accounts/{accountId}/balances` | Required path `accountId`, `string`; scope `balances.read` | `200 AccountBalance`; `401`, `403`, `404`, `429`, `500`, `503` |
| `ACCT-OP-004` | `POST /open-finance/system/accounts/v1/customers/{customerId}/accounts` | Required path `customerId`; `CreateAccountRequest`; scope `accounts.write` | `201 AccountReference`; `400`, `401`, `403`, `404`, `409`, `429`, `500`, `503` |
| `ACCT-OP-005` | `POST /open-finance/system/accounts/v1/accounts/{accountId}/closure` | Required path `accountId`; scope `accounts.close` | `200 AccountReference` with `accountState: CLOSED`; `400`, `401`, `403`, `404`, `409`, `429`, `500`, `503` |
| `ACCT-OP-006` | `POST /open-finance/system/accounts/v1/accounts/{accountId}/balance-adjustments` | Required path `accountId`, `Idempotency-Key` header, and `BalanceAdjustmentRequest`; scope `balances.adjust` | `201 BalanceAdjustment`; `202` same request in progress; `400`, `401`, `403`, `404`, `409`, `429`, `500`, `503` |

Account creation accepts canonical `accountType` and `currency`; the workshop mock resolves them to a synthetic Mambu `productTypeKey` and resolves the demo `customerId` to a synthetic `accountHolderKey`. The fixture below is for mock responses only; its keys must never be sent to a real Mambu tenant. These keys are not caller inputs. An unsupported product combination returns `400`; a missing or incomplete mapping configuration returns `503`. Production integration replaces the fixtures with tenant product configuration and an authorized customer registry. Core Banking assigns `accountId` and starts in `PENDING_APPROVAL`; the caller cannot choose the initial `accountState`. Verify that the tenant product permits this state. Account deletion is represented as logical closure; no physical-delete operation is defined. Repeating closure for an already `CLOSED` account returns its current representation with `200`; an invalid transition maps to `409`. Balances remain ledger-derived: initial funding and corrections are posted as signed adjustments. Generic `PUT` replacement and `PATCH` mutation of account or balance resources are not defined because no mutable account fields or direct balance-overwrite behavior have been approved.

### Mambu API v2 source mapping

| System API operation/data | Mambu API v2 source |
| --- | --- |
| `AccountReference.accountType` | `accountType` enum, passed through using the documented Mambu values. |
| `AccountReference.accountState` | `accountState` enum, passed through using the documented Mambu values. |
| `AccountReference.accountId` | Mambu `encodedKey`; Mambu `id` is not exposed as the contract identifier. |
| `AccountReference.currency` | Mambu `currencyCode`. |
| `ACCT-OP-004` customer holder | In the workshop mock, resolve the demo `customerId` through the fixture below to a synthetic Mambu `accountHolderKey`; set `accountHolderType` to `CLIENT`. Production uses the authorized customer registry. |
| `ACCT-OP-004` product selection | In the workshop mock, resolve canonical `accountType` and `currency` through the fixture below to a synthetic `productTypeKey`; the key is not exposed to the API consumer. An unmapped combination returns `400`; missing/incomplete configuration returns `503`, with no fallback. |
| `ACCT-OP-005` logical closure | Mambu deposit account `changeState` operation with target `CLOSED`; tenant transition rules apply. |
| `ACCT-OP-006` positive amount | Mambu deposit transaction operation: `POST /deposits/{depositAccountId}/transactions/deposits`. |
| `ACCT-OP-006` negative amount | Mambu withdrawal transaction operation: `POST /deposits/{depositAccountId}/transactions/withdrawals`. |
| `ACCT-OP-006` reason/channel | Store `reason` in the System API audit record; do not send it to Mambu without a tenant OAS mapping. Resolve transaction channel from tenant configuration. |
| Correction of a previously posted transaction | Mambu `POST /deposits/transactions/{depositTransactionId}:adjust` is out of scope; correct via a new compensating `ACCT-OP-006` adjustment and preserve the original transaction. |

### Workshop mock mappings

These synthetic values let the brownfield workshop model Mambu behavior without a tenant. They are fixtures for mock responses, not valid tenant configuration, and must never be sent to a real Mambu API. The mock supports the following products in EUR:

| `accountType` | `currency` | Mock `productTypeKey` |
| --- | --- | --- |
| `CURRENT_ACCOUNT` | `EUR` | `mock-product-current-eur` |
| `REGULAR_SAVINGS` | `EUR` | `mock-product-savings-eur` |
| `FIXED_DEPOSIT` | `EUR` | `mock-product-fixed-eur` |
| `SAVINGS_PLAN` | `EUR` | `mock-product-savings-plan-eur` |
| `INVESTOR_ACCOUNT` | `EUR` | `mock-product-investor-eur` |

The demo customer mapping is `cus-100045` to `mock-client-100045`, with `accountHolderType` `CLIENT`. Other customer IDs have no mock mapping. Production replaces these fixtures with tenant product configuration and a runtime lookup in the authorized customer registry.

Mambu documents `Idempotency-Key` as a UUID, recommends version 4, and requires retries to use the exact same request. This workshop contract replays the original response for a completed identical retry, returns `409` when the same key is reused with a different payload, and returns `202` while the original request is processing. The tenant retention duration remains to be configured for production.

### Request examples

Illustrative requests omit credentials; operations require a Bearer JWT. The required scope is listed with each operation.

```http
GET /open-finance/system/accounts/v1/customers/cus-100045/accounts HTTP/1.1
Accept: application/json

GET /open-finance/system/accounts/v1/accounts/acc-50001 HTTP/1.1
Accept: application/json

GET /open-finance/system/accounts/v1/accounts/acc-50001/balances HTTP/1.1
Accept: application/json

POST /open-finance/system/accounts/v1/customers/cus-100045/accounts HTTP/1.1
Content-Type: application/json

{
  "name": "Everyday savings",
  "accountType": "REGULAR_SAVINGS",
  "currency": "EUR"
}

POST /open-finance/system/accounts/v1/accounts/acc-50001/closure HTTP/1.1
Accept: application/json

POST /open-finance/system/accounts/v1/accounts/acc-50001/balance-adjustments HTTP/1.1
Content-Type: application/json
Idempotency-Key: 8d2438e0-895b-4b24-8d72-39a52c623c18

{
  "amount": 100.00,
  "currency": "EUR",
  "reason": "Opening balance"
}
```

Example account-creation body using canonical API fields:

```json
{
  "name": "Everyday savings",
  "accountType": "REGULAR_SAVINGS",
  "currency": "EUR"
}
```

`customerId` and `accountId` are opaque strings of 1–64 ASCII letters, digits, hyphens, or underscores, matching `^[A-Za-z0-9_-]+$`. They must not embed personal data and are classified as confidential. For Mambu, `accountId` maps to the auto-generated `encodedKey`, not the customizable `id`. Raw identifiers must not be logged; mask them when logging is necessary. Each operation requires a Bearer JWT and the scope shown in the operations table, and must authorize the authenticated consumer for the customer and account. Knowing an identifier alone does not grant access. Return `404` for both nonexistent and non-visible resources to prevent resource enumeration. Consent requirements remain pending.

## Candidate canonical models

The models are candidates, not approved golden records. No corporate model revision or shared repository has been provided. The values below are documented for Mambu API v2 Deposit Accounts. Tenant product configuration and identifier mappings still require verification. See [Mambu Create deposit account](https://docs.mambu.com/api/api-v2/deposits/create/) and [Mambu Get deposit account](https://docs.mambu.com/api/api-v2/deposits/get-by-id/).

### `AccountReference`

| Field | Type/format | Meaning | Status |
| --- | --- | --- | --- |
| `accountId` | `string`, 1–64 characters, `^[A-Za-z0-9_-]+$` | Opaque Mambu `encodedKey`; no personal data is embedded. | Required; format and source mapping approved. |
| `accountType` | `string` | Mambu Deposit Account type. | Required; Mambu API v2 enum. |
| `currency` | `string`, ISO 4217 | Account currency. | Required. |
| `accountState` | `string` | Mambu Deposit Account lifecycle state. | Required; Mambu API v2 enum. |

Mambu API v2 `accountType` values:

| Value | Meaning |
| --- | --- |
| `CURRENT_ACCOUNT` | Current account. |
| `REGULAR_SAVINGS` | Regular savings account. |
| `FIXED_DEPOSIT` | Fixed-term deposit. |
| `SAVINGS_PLAN` | Savings plan. |
| `INVESTOR_ACCOUNT` | Investor account. |

Mambu API v2 `accountState` values:

| Value | Meaning |
| --- | --- |
| `PENDING_APPROVAL` | Account is awaiting approval. |
| `APPROVED` | Account has been approved. |
| `ACTIVE` | Account is active. |
| `ACTIVE_IN_ARREARS` | Account is active and in arrears. |
| `MATURED` | Account has matured. |
| `LOCKED` | Account is locked. |
| `DORMANT` | Account is dormant. |
| `CLOSED` | Account is closed. |
| `CLOSED_WRITTEN_OFF` | Account is closed and written off. |
| `WITHDRAWN` | Account was withdrawn. |
| `CLOSED_REJECTED` | Account was closed after rejection. |

These are Mambu API v2 Deposit Account values, not universal banking enums. The source field is `accountState`; `LOCKED` is a distinct source state from `DORMANT`. Tenant-specific product values must be checked against the tenant OAS. Mambu's public create-account request also defines `accountHolderType` values `CLIENT` and `GROUP`; this customer-scoped API maps its path to a `CLIENT` holder and does not currently expose group-owned accounts.

All four `AccountReference` fields are required and must not be omitted or returned as `null`. The behavior when the source system does not provide a required value remains pending approval.

### `AccountBalance`

| Field | Type/format | Meaning and unit | Status |
| --- | --- | --- | --- |
| `accountId` | `string` | Account whose balances are queried. | Suggested. |
| `asOf` | `string`, RFC 3339 `date-time` | Same UTC cut-off instant for both balances. | Approved; distinct from `valueDate`. |
| `valueDate` | `string`, `date` | Business date to which the balance applies. | Optional; include only when an approved tenant OAS/extension supplies it. |
| `currency` | `string`, ISO 4217 | Account currency used by both balances. | Approved. |
| `availableBalance` | `number` | Funds available after pending holds and restrictions, in `currency`. | Required and non-null; meaning, sign, and precision policy approved. |
| `bookBalance` | `number` | Ledger balance from posted transactions, in `currency`. | Required and non-null; meaning, sign, and precision policy approved. |

Both balance amounts are required and non-null. If either is unavailable, the `GET /balances` operation fails; it must not return `200` with a partial balance or synthesize a value. This rule also applies to closed accounts: return recorded balances when Core Banking provides them; otherwise fail under the same rule. Incomplete or invalid required data from Core Banking maps to `502 Bad Gateway`; a dependency that is unavailable maps to `503 Service Unavailable`, and a dependency timeout maps to `504 Gateway Timeout`. Both balances use the account currency and the same `asOf` instant in UTC. `valueDate` is a separate, optional business date, not interchangeable with the cut-off instant. Mambu's public balance reference does not define a generic balance `valueDate`; never infer it from `asOf` or transaction dates. Include it only when an approved tenant OAS/extension supplies it. The sign convention is from the customer's perspective: positive means funds or credit belonging to the customer; negative means debt or overdraft. Amounts have decimal semantics and explicit currency. The API preserves the decimal precision supplied by Core Banking, uses the scale applicable to the currency, and performs no additional rounding. Do not use binary floating-point for financial calculations.

### `CreateAccountRequest`

Mambu API v2 requires `name` and tenant-specific `productTypeKey` when creating a Deposit Account. This contract requires canonical `accountType` and `currency`; the System API resolves them to `productTypeKey` using tenant configuration and maps `currency` to Mambu `currencyCode`. The customer path resolves through the authorized customer registry to the required `accountHolderKey`, with `accountHolderType` set to `CLIENT`. Mambu keys are internal and are not caller inputs. Group-owned accounts are outside this customer-scoped API. `accountId` maps to Mambu `encodedKey`. The initial state is `PENDING_APPROVAL`, subject to tenant product support.

### `BalanceAdjustmentRequest` and `BalanceAdjustment`

`BalanceAdjustmentRequest` contains required `amount: number`, `currency: string` (ISO 4217), and `reason: string`. The signed amount follows the customer-perspective convention: positive credits the customer and negative debits the customer. Currency must match the account currency; cross-currency adjustments are not part of this proposal. `reason` is stored in the System API audit record and is not sent to Mambu unless the tenant OAS confirms a matching field. The transaction channel is resolved from tenant configuration, not caller input. `Idempotency-Key` is a required request header and is not part of the JSON body. Mambu recommends a UUID v4 value and requires the same key to be reused only with an identical request. A completed retry with the same key and payload replays the original response; reuse with a different payload returns `409`; a duplicate received while the original is processing returns `202`. Retention duration is tenant-configured. Correct an existing transaction by posting a new compensating adjustment; this API does not expose Mambu's transaction-adjust endpoint or rewrite the original transaction.

Approved policy: for negative adjustments, Mambu's product configuration is the authority for available funds and any permitted overdraft. The API does not bypass that limit. If a debit exceeds available funds plus an explicitly configured overdraft, reject the whole transaction; do not post a partial amount or synthesize an overdraft. Tenant overdraft limit values and the HTTP mapping for this rejection remain to be verified.

`BalanceAdjustment` is the proposed `201` response and contains `adjustmentId: string`, `accountId: string`, `amount: number`, `currency: string`, `reason: string`, and `recordedAt: string` (`date-time`). It records the adjustment; resulting balances are retrieved through `GET /accounts/{accountId}/balances`.

### Examples available in the source

`AccountReference` example for `ACCT-OP-002`:

```json
{
  "accountId": "acc-50001",
  "accountType": "CURRENT_ACCOUNT",
  "currency": "EUR",
  "accountState": "ACTIVE"
}
```

`AccountBalance` example for `ACCT-OP-003`:

```json
{
  "accountId": "acc-50001",
  "asOf": "2026-10-31T10:30:00Z",
  "currency": "EUR",
  "availableBalance": 5400.25,
  "bookBalance": 5650.25
}
```

The data is illustrative; it does not identify real accounts or define valid catalog values.

The response for `ACCT-OP-001` is a collection of `AccountReference`; the example above illustrates each item with Mambu API v2 values. The source does not provide HTTP error-response examples.

## Security, errors, and availability

Operations require a Bearer JWT and the per-operation scope in the operations table. Treat customer and account identifiers as confidential and do not log their raw values; mask them when logging is necessary. For each operation, authorize the authenticated consumer against the customer and account. Knowing an identifier alone does not grant access. Return `404` for both nonexistent and non-visible resources to prevent resource enumeration. Consent is validated by the upstream Experience/Process layer; this System API enforces token scopes and resource authorization. Account creation, closure, and balance adjustments require audit logging; adjustment requests require a reason and idempotency key. Consent evidence propagation and audit retention remain organization policies.

Errors use `application/problem+json` with the shared Problem Details schema and a `Correlation-Id` response header; the response body includes the corresponding `correlationId` extension. Proposed status meanings are `400` invalid input, `401` missing/invalid authentication, `403` missing permission, `404` nonexistent or non-visible resource, `409` state/idempotency conflict, `429` rate limit exceeded, `500` unexpected internal error, `502` invalid or incomplete upstream data, `503` unavailable dependency, and `504` dependency timeout. Rate limits, timeout durations, availability targets, and platform-specific error codes remain to be agreed.

The `GET /balances` operation fails with `502` if either required amount is absent or invalid; it never returns a partial `200` or synthesizes a balance. For closed accounts, return recorded balances only when Core Banking provides them; otherwise apply the same failure behavior. Partial availability across accounts is not applicable to this single-account operation.

## Requirements and acceptance

| Stable ID | Verifiable requirement/criterion | Operation |
| --- | --- | --- |
| `ACCT-RQ-001` | The contract provides account listing/details, balance queries, account creation/closure, and ledger balance adjustments. | `ACCT-OP-001`–`006` |
| `ACCT-RQ-002` | Each operation declares its method, path, identifiers, and proposed responses. | All |
| `ACCT-RQ-003` | `AccountBalance` distinguishes available and book balances, uses the account currency, and identifies the shared UTC cut-off. | `ACCT-OP-003` |
| `ACCT-RQ-004` | Access is authorized by customer and account; a known ID is never sufficient. | All |
| `ACCT-RQ-005` | Examples are illustrative, and account type/state values match the documented Mambu API v2 Deposit Account enums. | All |
| `ACCT-RQ-006` | `AccountReference` requires `accountId`, `accountType`, `currency`, and `accountState`. | `ACCT-OP-001`, `ACCT-OP-002` |
| `ACCT-RQ-007` | If any required `AccountReference` field is missing from source data, the affected operation fails and does not return an incomplete object or partial list. | `ACCT-OP-001`, `ACCT-OP-002` |
| `ACCT-RQ-008` | `accountState` uses the documented Mambu API v2 Deposit Account state enum, including `LOCKED` and `DORMANT`. | `ACCT-OP-001`, `ACCT-OP-002` |
| `ACCT-RQ-009` | Positive balances represent customer funds/credit; negative balances represent customer debt/overdraft. | `ACCT-OP-003` |
| `ACCT-RQ-010` | Balance precision from Core Banking is preserved without additional API rounding; optional `valueDate` is separate from `asOf` and is never inferred. | `ACCT-OP-003` |
| `ACCT-RQ-011` | `availableBalance` and `bookBalance` are required and non-null; the operation fails rather than returning a partial `200` if either is unavailable. | `ACCT-OP-003` |
| `ACCT-RQ-012` | For closed accounts, return recorded balances when available; otherwise fail without synthesizing values. | `ACCT-OP-003` |
| `ACCT-RQ-013` | Account creation requires `name`, canonical `accountType`, and `currency`; tenant Mambu keys are resolved internally and the initial state is `PENDING_APPROVAL` when supported. | `ACCT-OP-004` |
| `ACCT-RQ-014` | Account deletion is logical closure; repeat closure returns the existing `CLOSED` representation with `200`; invalid state transitions return `409`. | `ACCT-OP-005` |
| `ACCT-RQ-015` | Balance changes are appended as auditable adjustments; the contract does not directly overwrite a balance. | `ACCT-OP-006` |
| `ACCT-RQ-016` | Balance adjustments require a reason and UUID v4 `Idempotency-Key`; completed identical retries replay the original response, changed payload returns `409`, and an in-progress duplicate returns `202`. | `ACCT-OP-006` |
| `ACCT-RQ-017` | Approved: negative adjustments obey Mambu product funds/overdraft limits and fail atomically without partial posting when a limit is exceeded. | `ACCT-OP-006` |
| `ACCT-RQ-018` | The approved internal mapping resolves `customerId` to `accountHolderKey` and canonical `accountType`/`currency` to tenant `productTypeKey`; Mambu keys are not exposed to callers. | `ACCT-OP-004` |
| `ACCT-RQ-019` | `valueDate` is optional and is returned only when an approved tenant OAS/extension supplies it; it is not inferred. | `ACCT-OP-003` |
| `ACCT-RQ-020` | An unmapped accountType/currency combination returns `400`; missing or incomplete tenant mapping configuration returns `503`, with no fallback product. | `ACCT-OP-004` |
| `ACCT-RQ-021` | Adjustment `reason` is audit-only unless tenant OAS confirms a Mambu field mapping; transaction channel is resolved from tenant configuration. | `ACCT-OP-006` |
| `ACCT-RQ-022` | Existing Mambu transactions are not rewritten; corrections use a new compensating adjustment. | `ACCT-OP-006` |
| `ACCT-RQ-023` | Idempotency-key retention is configured by tenant policy. | `ACCT-OP-006` |
| `ACCT-AC-001` | The three GET operations have no request body; each path operation requires its declared identifier. | All |
| `ACCT-AC-002` | Consumers can trace requirements to an operation and candidate model by stable ID. | All |
| `ACCT-AC-003` | OpenAPI is not published until security, balance semantics, errors, and common models are approved. | All |
| `ACCT-AC-004` | The OpenAPI schema requires both balance amount fields. | `ACCT-OP-003` |
| `ACCT-AC-005` | Write operations use distinct scopes and resource-level authorization. | `ACCT-OP-004`–`006` |
| `ACCT-AC-006` | Closure preserves the account record, and balance adjustments do not overwrite ledger-derived balances. | `ACCT-OP-005`, `ACCT-OP-006` |
| `ACCT-AC-007` | `CreateAccountRequest` accepts canonical values and never requires a caller to provide tenant Mambu keys. | `ACCT-OP-004` |
| `ACCT-AC-008` | Unsupported product selection and unavailable tenant mapping configuration return their specified errors without fallback. | `ACCT-OP-004` |
| `ACCT-AC-009` | The contract does not expose Mambu's transaction-adjust endpoint or mutate the original transaction. | `ACCT-OP-006` |

## Open decisions for production approval

These decisions apply to a real tenant integration and do not block the workshop mock fixtures.

1. Approved schema and revision for `AccountReference`, `AccountBalance`, and `Money`.
2. Rate limits, availability targets, timeout durations, and platform-specific error codes.
3. Verify tenant products permit initial `accountState` `PENDING_APPROVAL` and define Mambu `changeState` transition preconditions.
4. Map adjustment `amount` and `currency` to Mambu deposit/withdrawal request fields; confirm the tenant transaction channel, overdraft limits, insufficient-funds HTTP mapping, and idempotency retention duration. `reason` remains audit-only unless tenant OAS confirms a matching Mambu field. Transaction corrections stay out of scope; any future reversal requires a separately approved operation.
5. Consent evidence propagation and audit-record retention required by organization policy.
6. Deployment host, environments, and future compatibility policy; initial SemVer is `1.0.0`.

## The 7 Cs review

| C | Application in this specification |
| --- | --- |
| Clarity | Defines the intended consumer and distinguishes account reads/writes from balance queries/adjustments. |
| Conciseness | Limits scope to six defined account and balance operations. |
| Concreteness | Specifies paths, parameters, responses, and fields with types/units. |
| Correctness | Marks banking definitions absent from the source as pending. |
| Coherence | Consistently uses `accountId`, ISO 4217, and `asOf`. |
| Completeness | Includes models, errors, security, and criteria; it is not yet publishable. |
| Courtesy | Uses neutral terms and explains access clearly. |

## Organization and traceability

This file belongs to the independent `of-sys-acct-core-banking-v1` GitLab design project. The relationship with the Process API is contextual, not an implemented call. Source: [API-Led example](../open-finance-api-led-example.md). Common and domain model revisions must be pinned before producing OpenAPI.
