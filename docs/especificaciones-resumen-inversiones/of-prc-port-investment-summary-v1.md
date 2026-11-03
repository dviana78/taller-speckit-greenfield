# Specification: `of-prc-port-investment-summary-v1`

> Status: design draft for review. This is not an approved contract and does not authorize implementation. Paths and models are proposals from the [API-Led example](../open-finance-api-led-example.md), which identifies them as fictitious. Open business decisions are not inferred.

## Purpose and context

- **Layer and domain:** Process API, portfolios and investments (`port`).
- **Consumer:** the `of-exp-port-investment-dashboard-v1` Experience API, which consumes an investment-summary capability.
- **Purpose:** define a read interface for the summary, performance over a period, and positions.
- **Architectural relationship:** the example identifies `of-sys-port-portfolio-v1` and `of-sys-acct-core-banking-v1` as related capabilities. This is context, not a call specification or an assertion that the Process API must query accounts for every operation.
- **GitLab project:** an independent design project with this identifier; it is not created here.
- **Proposed base path:** `/open-finance/process/v1` (not approved).
- **Version:** the identifier indicates `v1`; SemVer and the compatibility policy are pending.

## Proposed operations

Read-only operations with no request body. `customerId` is required in the path, has type `string`, and is an Open Finance domain identifier. Its pattern, length, and authorization require agreement.

| ID | Proposed method and full path | Query parameters | Proposed success response |
| --- | --- | --- | --- |
| `PRC-OP-001` | `GET /open-finance/process/v1/customers/{customerId}/investment-summary` | Optional `currency`, ISO 4217 | `200 InvestmentSummary` |
| `PRC-OP-002` | `GET /open-finance/process/v1/customers/{customerId}/investment-summary/performance` | Optional `from` and `to`, `date` (`YYYY-MM-DD`) | `200`; performance model pending |
| `PRC-OP-003` | `GET /open-finance/process/v1/customers/{customerId}/investment-summary/positions` | Optional integer `page` and `pageSize` | `200`; collection/pagination schema pending |

### Request examples

Illustrative requests without an authentication header because the security mechanism has not yet been approved:

```http
GET /open-finance/process/v1/customers/cus-100045/investment-summary?currency=EUR HTTP/1.1
Accept: application/json

GET /open-finance/process/v1/customers/cus-100045/investment-summary/performance?from=2026-01-01&to=2026-10-31 HTTP/1.1
Accept: application/json

GET /open-finance/process/v1/customers/cus-100045/investment-summary/positions?page=1&pageSize=20 HTTP/1.1
Accept: application/json
```

Proposed error responses for each operation are `400`, `401`, `403`, `404`, `429`, `500`, and `503`, as in the example. Partial-failure semantics are not defined, and `502`/`504` are not added without a platform decision.

### Parameters

| Name | Location | Type/format | Required | Known constraints |
| --- | --- | --- | --- | --- |
| `customerId` | Path | `string` | Yes | Open Finance identifier; pattern and length pending. |
| `currency` | Query | `string`, ISO 4217 | No | Conversion, default, and supported currencies pending. |
| `from` | Query | `string`, `date` | No | Inclusivity and relationship to `to` pending. |
| `to` | Query | `string`, `date` | No | Inclusivity and relationship to `from` pending. |
| `page` | Query | `integer` | No | Numbering base, minimum, and default pending. |
| `pageSize` | Query | `integer` | No | Maximum and default pending. |

## Candidate canonical model

The fields below derive from the example's `InvestmentSummary`. They are a proposed contract model, not an approved golden-record revision. No shared repositories or revisions to pin have been provided.

### `InvestmentSummary`

| Field | Type/format | Meaning and unit | Status |
| --- | --- | --- | --- |
| `customerId` | `string` | Customer to which the summary applies. | Proposed; requiredness/pattern pending. |
| `asOf` | `string`, RFC 3339 `date-time` | Reference instant; example uses UTC (`Z`). | Proposed; valuation/cut-off date pending. |
| `currency` | `string`, ISO 4217 | Currency of the totals. | Proposed; does not define conversion. |
| `totalMarketValue` | `number` | Total market value in `currency`. | Proposed; precision and aggregation pending. |
| `totalCostBasis` | `number` | Total cost basis in `currency`. | Proposed; definition and missing-value behavior pending. |
| `unrealizedGainLoss` | `number` | Unrealized gain or loss in `currency`. | Proposed; formula and aggregation pending. |
| `unrealizedGainLossPercentage` | `number` | Gain/loss percentage; the example shows `7.21`. | Proposed; formula, scale, and rounding pending. |
| `portfolios` | `array<PortfolioSummary>` | Portfolios included in the summary. | Proposed; filter, cardinality, and ordering pending. |

`PortfolioSummary` contains `portfolioId: string`, `marketValue: number`, `currency: string` (ISO 4217), and, optionally according to the example, a name and gain/loss metrics. The source does not define the requiredness of each property; this must be approved.

`Money` could represent amounts as `{ "amount": 125430.75, "currency": "EUR" }`, but the example's `InvestmentSummary` uses flat amounts with a shared currency. The canonical representation must resolve this difference before OpenAPI is generated. Decimal semantics and explicit currency are proposed for amounts; precision, scale, and rounding depend on the approved standard. Do not use binary floating-point for financial calculations.

No model is specified for performance over a period, and no rule is defined for aggregating positions. Neither is created by inference.

### Summary model example

Illustrative source example for `PRC-OP-001`; the values are not real data or approved rules.

```json
{
  "customerId": "cus-100045",
  "asOf": "2026-10-31T10:30:00Z",
  "currency": "EUR",
  "totalMarketValue": 125430.75,
  "totalCostBasis": 117000.00,
  "unrealizedGainLoss": 8430.75,
  "unrealizedGainLossPercentage": 7.21,
  "portfolios": [
    { "portfolioId": "prt-20001", "marketValue": 80430.75, "currency": "EUR" },
    { "portfolioId": "prt-20002", "marketValue": 45000.00, "currency": "EUR" }
  ]
}
```

## Rules, errors, and security

Being a Process API does not authorize inventing aggregation policies. In particular, the contract does not decide whether accounts are part of the summary or how to handle unvalued portfolios, different currencies, unpriced positions, stale data, or missing cost basis. It also does not define whether a partial dependency failure produces a degraded response or a total error.

Security: authentication, per-operation scopes/permissions, consent, and customer/portfolio/account-level authorization require approval. The example recommends resource-level authorization; knowing an identifier must not be sufficient. No fictitious OAuth/OIDC scheme is assigned.

Proposed errors: `400` invalid input; `401` invalid authentication; `403` access denied; `404` resource does not exist or is not visible under the approved policy; `429` limit exceeded; `500` unclassified error; `503` unavailable. The Problem Details schema, correlation, timeouts, retries, and dependency-failure codes must be agreed with the platform.

## Requirements and acceptance

| Stable ID | Verifiable requirement/criterion | Operation |
| --- | --- | --- |
| `PRC-RQ-001` | The capability exposes the summary to the identified Experience API. | All |
| `PRC-RQ-002` | The inventory contains exactly the three proposed GET operations listed in the table. | `PRC-OP-001`–`003` |
| `PRC-RQ-003` | Parameters state location, type, format, requiredness, and known limits. | All |
| `PRC-RQ-004` | Amounts include currency context; different currencies are not summed without an approved rule. | All |
| `PRC-RQ-005` | Composition does not imply a call to or mandatory dependency on accounts. | `PRC-OP-001` |
| `PRC-RQ-006` | Partial-error behavior and aggregation policy require explicit approval. | All |
| `PRC-AC-001` | Each requirement can be traced by ID to an operation and a contract check. | All |
| `PRC-AC-002` | Examples match the candidate model and are marked as illustrative. | `PRC-OP-001` |
| `PRC-AC-003` | OpenAPI is not considered publishable while blocking decisions remain open. | All |

## Open decisions for approval

1. Which banking data, if any, belongs in the summary and when it is required.
2. Definitions of valuation, cost basis, performance, and percentages.
3. Aggregation, currency, conversion, exchange-rate, cut-off, and rounding policies.
4. Missing-data behavior and partial-failure handling for each dependency.
5. Performance and positions schemas, ordering, and pagination.
6. Parameter and identifier constraints.
7. Security mechanism, scopes, consent, and resource authorization.
8. Standard errors, Problem Details, correlation, timeouts, and limits.
9. Approved golden-record and commons revisions; SemVer, servers, and compatibility.

## The 7 Cs review

| C | Application in this specification |
| --- | --- |
| Clarity | Defines the Process API consumer, purpose, and boundaries. |
| Conciseness | Keeps scope to three summary operations. |
| Concreteness | Includes methods, paths, parameters, candidate model, and stable IDs. |
| Correctness | Explicitly separates fictitious examples from approved rules. |
| Coherence | Preserves source resource and field names. |
| Completeness | Records data, errors, security, and open decisions; it is not yet publishable. |
| Courtesy | Documents the contract and errors in neutral, useful language. |

## Organization and traceability

This file belongs to the independent `of-prc-port-investment-summary-v1` GitLab design project. The System APIs and Experience API are contextual relationships, not implemented calls. Source for proposed methods and models: [API-Led example](../open-finance-api-led-example.md). This deliverable does not generate executable endpoints or OpenAPI files.
