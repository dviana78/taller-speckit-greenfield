# Specification: `of-exp-port-investment-dashboard-v1`

> Status: design draft for review. This is not an approved contract and does not authorize implementation. Methods, paths, and models are taken from the [API-Led example](../open-finance-api-led-example.md), where they are identified as fictitious. The open decisions in this document require agreement before a publishable OpenAPI contract can be generated.

## Purpose and context

- **Layer and domain:** Experience API, portfolios and investments (`port`).
- **Consumer:** a digital channel that presents the investment dashboard to an authenticated customer.
- **Purpose:** provide the channel with a summary, a performance view, and positions for the dashboard experience.
- **Architectural relationship:** the example describes consumption of `of-prc-port-investment-summary-v1`. This is context; it does not assert that an API call has been implemented.
- **GitLab project:** an independent design project with this identifier. This file does not create the project.
- **Proposed base path:** `/open-finance/experience/v1` (not approved as a standard).
- **Version:** the identifier indicates `v1`; the SemVer version and compatibility policy must be agreed.

## Proposed operations

All operations are read-only and have no request body. `{customerId}` is required in the path and is an Open Finance customer identifier of type `string`. Its format, length, and authorization rules require domain approval.

| ID | Proposed method and full path | Query parameters | Proposed success response |
| --- | --- | --- | --- |
| `EXP-OP-001` | `GET /open-finance/experience/v1/customers/{customerId}/investment-dashboard` | `currency` opcional, código ISO 4217 | `200 InvestmentSummaryView` |
| `EXP-OP-002` | `GET /open-finance/experience/v1/customers/{customerId}/investment-dashboard/performance` | Optional `from` and `to`, `date` format (`YYYY-MM-DD`) | `200`; performance schema pending a decision |
| `EXP-OP-003` | `GET /open-finance/experience/v1/customers/{customerId}/investment-dashboard/positions` | Optional integer `page` and `pageSize` | `200`; collection and pagination metadata pending approval |

### Request examples

Illustrative requests without an authentication header because the security mechanism has not yet been approved:

```http
GET /open-finance/experience/v1/customers/cus-100045/investment-dashboard?currency=EUR HTTP/1.1
Accept: application/json

GET /open-finance/experience/v1/customers/cus-100045/investment-dashboard/performance?from=2026-01-01&to=2026-10-31 HTTP/1.1
Accept: application/json

GET /open-finance/experience/v1/customers/cus-100045/investment-dashboard/positions?page=1&pageSize=20 HTTP/1.1
Accept: application/json
```

Proposed error responses for all three operations are `400`, `401`, `403`, `404`, `429`, `500`, and `503`, using the common error component once approved. No meaning is assigned to an unagreed business error. For `from`/`to`, this specification does not assume whether both parameters must be supplied together or define behavior for an inverted range.

### Parameters

| Name | Location | Type and format | Required | Known constraints |
| --- | --- | --- | --- | --- |
| `customerId` | Path | `string` | Yes | Open Finance domain identifier; pattern and length pending. |
| `currency` | Query | `string`, ISO 4217 | No | Currency conversion, default currency, and behavior for unsupported codes pending. |
| `from` | Query | `string`, `date` | No | Inclusive/exclusive date semantics and relationship to `to` pending. |
| `to` | Query | `string`, `date` | No | Inclusive/exclusive date semantics and relationship to `from` pending. |
| `page` | Query | `integer` | No | Numbering base and minimum value pending. |
| `pageSize` | Query | `integer` | No | Default and platform maximum pending. |

## Candidate data models

The following models are candidates from the example, not approved golden records. `Money` represents a decimal amount with an explicit currency. The example uses `EUR`; `amount` is a JSON `number`, and its contractual precision and scale must be set by the data standard. Financial arithmetic must not use binary floating-point. Applicable corporate model revisions have not been provided.

### `InvestmentSummaryView`

Candidate view for `EXP-OP-001`, based on the dashboard response in the example.

| Field | Type/format | Meaning and unit | Status |
| --- | --- | --- | --- |
| `customerId` | `string` | Identifier of the queried customer. | Proposed; requiredness pending approval. |
| `asOf` | `string`, RFC 3339 `date-time` | Cut-off instant; the example uses UTC (`Z`). | Proposed; cut-off semantics pending. |
| `currency` | `string`, ISO 4217 | Currency of totals and represented values. | Proposed; does not imply conversion. |
| `totalMarketValue` | `number` | Total market value in `currency`. | Proposed; aggregation and precision pending. |
| `totalCostBasis` | `number` | Total cost basis in `currency`. | Proposed; financial definition and missing-value handling pending. |
| `unrealizedGainLoss` | `number` | Unrealized gain or loss in `currency`. | Proposed; formula and aggregation pending. |
| `unrealizedGainLossPercentage` | `number` | Gain/loss percentage; the example shows `7.21`. | Proposed; denominator, scale, and rounding pending. |
| `portfolioCount` | `integer` | Number of included portfolios. | Proposed; inclusion rule pending. |
| `portfolios` | `array<PortfolioSummaryView>` | Portfolios presented in the dashboard. | Proposed; inclusion and ordering pending. |

`PortfolioSummaryView` contains `portfolioId: string`, `name: string`, `marketValue: number`, `currency: string` (ISO 4217), `unrealizedGainLoss: number`, and `unrealizedGainLossPercentage: number`. Financial meaning, requiredness, and missing-value handling require approval. The example does not provide response models for this Experience API's performance or positions operations; none are invented here.

### Response example available in the source

Illustrative example for `EXP-OP-001`; its values are not real data or approved contractual values.

```json
{
  "customerId": "cus-100045",
  "asOf": "2026-10-31T10:30:00Z",
  "currency": "EUR",
  "totalMarketValue": 125430.75,
  "totalCostBasis": 117000.00,
  "unrealizedGainLoss": 8430.75,
  "unrealizedGainLossPercentage": 7.21,
  "portfolioCount": 2,
  "portfolios": [
    {
      "portfolioId": "prt-20001",
      "name": "Balanced portfolio",
      "marketValue": 80430.75,
      "currency": "EUR",
      "unrealizedGainLoss": 5430.75,
      "unrealizedGainLossPercentage": 7.24
    },
    {
      "portfolioId": "prt-20002",
      "name": "Conservative portfolio",
      "marketValue": 45000.00,
      "currency": "EUR",
      "unrealizedGainLoss": 3000.00,
      "unrealizedGainLossPercentage": 7.14
    }
  ]
}
```

## Security, errors, and partial data

The source does not define the authentication mechanism, per-operation permissions/scopes, consent, or customer authorization. These must be approved before finalizing the OpenAPI contract; no fictitious security scheme is declared. Knowing `customerId` alone must not grant access.

Responses to partial data, unavailable dependencies, unpriced positions, missing cost basis, or no portfolios remain undecided. This specification does not assume partial responses, degraded success, or total failure. Errors must use Problem Details and the corporate correlation header only after their components and revisions are approved.

## Requirements and acceptance

| Stable ID | Verifiable requirement/criterion | Operation |
| --- | --- | --- |
| `EXP-RQ-001` | The target consumer is the digital channel, and the API presents the investment dashboard. | All |
| `EXP-RQ-002` | The inventory contains the three proposed GET operations and paths listed in the table. | `EXP-OP-001`–`003` |
| `EXP-RQ-003` | Each parameter documents location, type, requiredness, and known constraints; undecided items are identified. | All |
| `EXP-RQ-004` | Amounts always identify their currency; no conversion or aggregation is asserted without an approved rule. | `EXP-OP-001`–`003` |
| `EXP-RQ-005` | The contract does not assign an unapproved partial-availability policy or permissions. | All |
| `EXP-AC-001` | A reviewer can trace each requirement by its ID to an associated operation. | All |
| `EXP-AC-002` | OpenAPI is not published until the blockers listed under “Open decisions” are approved. | All |
| `EXP-AC-003` | The dashboard example validates candidate names and types without treating its values as real data. | `EXP-OP-001` |

## Open decisions for approval

1. `customerId` format and limits; resource authorization and scopes.
2. Whether `currency` is supported, the default currency and conversion rules, exchange-rate source/date, and rounding.
3. Cost-basis, gain/loss, and percentage formulas; precision, scale, and units.
4. Cut-off and period dates, inclusivity, defaults, and missing-data behavior.
5. Performance and positions schemas, grouping, ordering, and pagination.
6. Partial-data and dependency errors; timeout and availability policy.
7. Security mechanism and per-operation scopes.
8. Approved revisions for `Money`, portfolio models, `Problem`, correlation, and pagination.
9. SemVer versioning, servers, page limits, and compatibility criteria.

## The 7 Cs review

| C | Application in this specification |
| --- | --- |
| Clarity | Identifies the digital channel and separates summary, performance, and positions. |
| Conciseness | Limits scope to the three Experience API reads. |
| Concreteness | Records methods, paths, parameters, types, responses, and requirements with IDs. |
| Correctness | Distinguishes example data from approved decisions. |
| Coherence | Consistently uses `customerId`, explicit currency, and source model names. |
| Completeness | Records errors, security, models, and open decisions; it is not yet publishable. |
| Courtesy | Describes responses and errors neutrally and usefully for the consumer. |

## Organization and traceability

This Markdown file is an independent functional specification for the `of-exp-port-investment-dashboard-v1` GitLab project. It does not create the project, an OpenAPI contract, or an inter-API call. The relationship with the Process API is architectural context. Source for the proposals: [API-Led example](../open-finance-api-led-example.md). Approvals and model revisions must be recorded here before generating the contract.
