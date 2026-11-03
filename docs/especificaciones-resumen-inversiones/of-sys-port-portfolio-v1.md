# Specification: `of-sys-port-portfolio-v1`

> Status: design draft for review. This is not an approved contract and does not describe the actual backend system. Methods, paths, parameters, and models are based on the [API-Led example](../open-finance-api-led-example.md), whose values are fictitious.

## Purpose and context

- **Layer and domain:** Portfolio System API (`port`).
- **Intended consumer:** a process capability that needs to query portfolios, positions, and valuations; the example mentions `of-prc-port-investment-summary-v1` as context, not as an implemented call.
- **Purpose:** expose portfolio reads while insulating consumers from backend details.
- **Proposed base path:** `/open-finance/system/portfolios/v1` (standard pending).
- **GitLab project:** an independent design project under the portfolio domain; it is not created here.
- **Version:** identifier `v1`; SemVer and compatibility are pending.

## Proposed operations

| ID | Proposed method and full path | Parameters | Proposed responses |
| --- | --- | --- | --- |
| `PORT-OP-001` | `GET /open-finance/system/portfolios/v1/customers/{customerId}/portfolios` | Required path `customerId`, `string` | `200` collection of `Portfolio`; `400`, `401`, `403`, `404`, `429`, `500`, `503` |
| `PORT-OP-002` | `GET /open-finance/system/portfolios/v1/portfolios/{portfolioId}` | Required path `portfolioId`, `string` | `200 Portfolio`; `401`, `403`, `404`, `429`, `500`, `503` |
| `PORT-OP-003` | `GET /open-finance/system/portfolios/v1/portfolios/{portfolioId}/positions` | Required path `portfolioId`; optional query `asOf` (`date`), integer `page` and `pageSize`, and string `sort` | `200` collection of `InvestmentPosition`; `400`, `401`, `403`, `404`, `429`, `500`, `503` |
| `PORT-OP-004` | `GET /open-finance/system/portfolios/v1/portfolios/{portfolioId}/valuation` | Required path `portfolioId`; optional query `asOf` (`date`) | `200 PortfolioValuation`; `400`, `401`, `403`, `404`, `429`, `500`, `503` |

### Request examples

Illustrative requests without an authentication header because the security mechanism has not yet been approved:

```http
GET /open-finance/system/portfolios/v1/customers/cus-100045/portfolios HTTP/1.1
Accept: application/json

GET /open-finance/system/portfolios/v1/portfolios/prt-20001 HTTP/1.1
Accept: application/json

GET /open-finance/system/portfolios/v1/portfolios/prt-20001/positions?asOf=2026-10-30&page=1&pageSize=20 HTTP/1.1
Accept: application/json

GET /open-finance/system/portfolios/v1/portfolios/prt-20001/valuation?asOf=2026-10-30 HTTP/1.1
Accept: application/json
```

Open constraints: identifier format and length; inclusive/exclusive semantics for `asOf`; valid `sort` values; pagination base and limits. No filters beyond those in the source are added.

## Candidate canonical models

The following schemas are candidates based on fields suggested by the example, not approved golden records. No approved version or `goldenRecords` repository has been provided. For all amounts, `currency` is an ISO 4217 code. JSON `number` fields represent decimal amounts/quantities conceptually; precision, scale, and rounding must be set by the data standard. Do not perform financial calculations using binary floating-point.

### `CustomerReference`

| Field | Type | Meaning |
| --- | --- | --- |
| `customerId` | `string` | Customer identifier in the agreed domain. |

### `Portfolio`

| Field | Type/format | Meaning | Status |
| --- | --- | --- | --- |
| `portfolioId` | `string` | Unique portfolio identifier. | Suggested field; pattern pending. |
| `customerId` | `string` | Reference to the related customer. | Suggested field; exposure and requiredness pending approval. |
| `name` | `string` | Display name, if available. | Optional according to the source. |
| `status` | `string` | Portfolio status. Example catalog values: `ACTIVE`, `INACTIVE`, `CLOSED`. | Catalog not approved. |
| `baseCurrency` | `string`, ISO 4217 | Portfolio base currency. | Suggested field. |
| `openedOn` | `string`, `date` | Opening date. | Only if provided by the source. |

### `InvestmentPosition`

| Field | Type/format | Meaning and unit | Status |
| --- | --- | --- | --- |
| `positionId` | `string` | Position identifier. | Suggested. |
| `portfolioId` | `string` | Portfolio that owns the position. | Suggested. |
| `instrumentId` | `string` | Instrument reference. | Suggested. |
| `instrumentName` | `string` | Display name. | Optional. |
| `quantity` | `number` | Quantity held; precision depends on the instrument. | Precision pending. |
| `marketPrice` | `Money` | Unit price and currency. | Price/cut-off meaning pending. |
| `marketValue` | `Money` | Market value and currency. | Valuation rule pending. |
| `valuationDate` | `string`, `date` | Valuation date. | Contractual time zone/cut-off pending. |

Candidate `Money` has `amount: number` and `currency: string` (ISO 4217). The example shows `{ "amount": 642.00, "currency": "EUR" }`.

### `PortfolioValuation`

| Field | Type/format | Meaning and unit | Status |
| --- | --- | --- | --- |
| `portfolioId` | `string` | Portfolio being valued. | Suggested. |
| `asOf` | `string`, RFC 3339 `date-time` | Calculation/cut-off instant; example uses UTC. | Semantics pending approval. |
| `currency` | `string`, ISO 4217 | Currency of the amounts. | Suggested. |
| `marketValue` | `number` | Total market value in `currency`. | Formula/precision pending. |
| `costBasis` | `number` | Cost basis in `currency`. | Fee/tax inclusion and missing-value handling pending. |
| `unrealizedGainLoss` | `number` | Unrealized difference in `currency`. | Formula pending. |
| `unrealizedGainLossPercentage` | `number` | Percentage; the example shows `7.24`. | Formula, scale, and rounding pending. |

Requiredness of candidate properties must be agreed with the system owner. Allowed statuses, quantity units, and missing-field behavior cannot be inferred from the examples.

## Examples available in the source

`Portfolio` example for `PORT-OP-002`:

```json
{
  "portfolioId": "prt-20001",
  "customerId": "cus-100045",
  "name": "Balanced portfolio",
  "status": "ACTIVE",
  "baseCurrency": "EUR",
  "openedOn": "2022-04-12"
}
```

`PortfolioValuation` example for `PORT-OP-004`:

```json
{
  "portfolioId": "prt-20001",
  "asOf": "2026-10-30T18:00:00Z",
  "currency": "EUR",
  "marketValue": 80430.75,
  "costBasis": 75000.00,
  "unrealizedGainLoss": 5430.75,
  "unrealizedGainLossPercentage": 7.24
}
```

Both examples are illustrative, not approved contractual values.

Position response example for `PORT-OP-003`:

```json
{
  "positionId": "pos-30001",
  "portfolioId": "prt-20001",
  "instrumentId": "ins-40001",
  "instrumentName": "Example investment fund",
  "quantity": 125.50,
  "marketPrice": { "amount": 642.00, "currency": "EUR" },
  "marketValue": { "amount": 80571.00, "currency": "EUR" },
  "valuationDate": "2026-10-30"
}
```

The response for `PORT-OP-001` is a collection of the `Portfolio` model; the `Portfolio` example above illustrates each item. The source does not provide HTTP error-response examples.

## Security, errors, and availability

The authentication mechanism and customer/portfolio permissions require approval. Authorization must verify access to the resource and its related customer; knowing `portfolioId` alone does not grant access. No scopes are invented.

Proposed error responses are listed in the operations table. `Problem Details`, correlation, and limits must reference approved corporate components. Behavior for an unavailable backend, invalid response, unpriced position, missing valuation, or partial data remains undecided; the example mentions `502`/`504` only as possible platform standards, not as selected responses for this API.

## Requirements and acceptance

| Stable ID | Verifiable requirement/criterion | Operation |
| --- | --- | --- |
| `PORT-RQ-001` | The API exposes the four listed portfolio reads without implying their implementation. | `PORT-OP-001`–`004` |
| `PORT-RQ-002` | Each operation declares its method, path, parameters, and proposed responses. | All |
| `PORT-RQ-003` | Models distinguish currency, amounts, dates, and quantities. | `PORT-OP-002`–`004` |
| `PORT-RQ-004` | Unconfirmed fields and catalogs remain candidates, not standards. | All |
| `PORT-RQ-005` | Authorization applies to the customer and resource; knowing an ID is insufficient. | All |
| `PORT-AC-001` | A reviewer can trace each stable requirement to an operation and model. | All |
| `PORT-AC-002` | Examples can be checked against candidate types without treating them as real data. | `PORT-OP-002`, `004` |
| `PORT-AC-003` | Publication is blocked until security, errors, constraints, and golden records are approved. | All |

## Open decisions for approval

1. Identifiers, catalogs, field requiredness, and supported filters.
2. `sort` values, `asOf` semantics, and pagination rules.
3. Conventions for amounts, quantities, prices, dates, valuation, and cost basis.
4. Missing price/valuation behavior and partial backend failures.
5. Security, scopes, and customer/portfolio authorization.
6. Common error responses and approved `Problem`/correlation/pagination revisions.
7. Approved revision for each golden record and the versioning strategy.

## The 7 Cs review

| C | Application in this specification |
| --- | --- |
| Clarity | Defines the portfolio system's read capabilities. |
| Conciseness | Limits the inventory to the example's four operations. |
| Concreteness | Specifies paths, parameters, candidate schemas, and proposed codes. |
| Correctness | Does not present fictitious catalogs or values as approved. |
| Coherence | Reuses `Money`, `Portfolio`, `InvestmentPosition`, and `PortfolioValuation` consistently. |
| Completeness | Records security, errors, examples, and blockers; it is not yet publishable. |
| Courtesy | Uses neutral descriptions aimed at technical consumers. |

## Organization and traceability

This document belongs to the independent `of-sys-port-portfolio-v1` GitLab design project. The relationship with the Process API is contextual. Source: [API-Led example](../open-finance-api-led-example.md). Model revisions must be pinned before generating an OpenAPI contract.
