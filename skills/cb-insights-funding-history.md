---
name: cb-insights-funding-history
description: >-
  Pull the full financing picture for a set of companies from CB Insights — funding rounds received,
  investments made, and portfolio exits — including investors, valuations and deal terms.
api: CB Insights API v2
base_url: https://api.cbinsights.com
operations:
  - POST /v2/authorize
  - POST /v2/organizations
  - POST /v2/financialtransactions/fundings
  - POST /v2/financialtransactions/investments
  - POST /v2/financialtransactions/portfolioexits
  - POST /v2/organizations/{orgId}/financialtransactions/fundings
  - POST /v2/organizations/{orgId}/financialtransactions/investments
  - POST /v2/organizations/{orgId}/financialtransactions/portfolioexits
generated: '2026-08-09'
method: generated
source: >-
  Grounded in openapi/cb-insights-api-v2-openapi.json (v2FinancialTransactions.* definitions) and
  https://api-docs.cbinsights.com/portal/docs/CBI-data/Company-data-and-insights/financial-transactions.
  The published contract declares no operationIds, so operations are referenced by method and path.
---

# Retrieve funding, investment and exit history

## Pick the right endpoint

CB Insights splits financial transactions by the **role the organization played in the deal**, and
gives each role both a multi-organization and a single-organization form.

| Role | Many organizations | One organization |
|---|---|---|
| Received funding | `POST /v2/financialtransactions/fundings` | `POST /v2/organizations/{orgId}/financialtransactions/fundings` |
| Made an investment | `POST /v2/financialtransactions/investments` | `POST /v2/organizations/{orgId}/financialtransactions/investments` |
| Exited a portfolio holding | `POST /v2/financialtransactions/portfolioexits` | `POST /v2/organizations/{orgId}/financialtransactions/portfolioexits` |

Use the multi-organization form when you have a cohort — one request covering N orgIds costs far
less rate-limit budget than N requests.

## Steps

1. Authenticate with `POST /v2/authorize` and cache the 24-hour token.
2. Resolve your companies to `orgId`s — see `cb-insights-resolve-organizations`. This step is free.
3. Call the appropriate operation above with `Authorization: Bearer <token>` and `orgIds` in the body.
4. Read `transactions[]`. Each transaction carries `dealId`, `date`, `amountInMillions`,
   `valuationInMillions`, `round` / `roundCategory` / `roundCategoryId` / `roundId`,
   `simplifiedRound`, `investors[]`, `recipient`, `isExit`, revenue range and multiple fields,
   `sources[]`, and AI-generated `insights`.
5. Page with `nextPageToken` until it comes back null.
6. Resolve vocabulary ids. `roundId`, `roundCategoryId` and `roundTypeId` are ids, not labels —
   map them against the published funding-types reference.

## Rules

- **Directionality matters.** A funding round appears as a *funding* on the recipient and as an
  *investment* on each investor. Do not double count when you assemble a market total.
- `valuationIsEstimate`, `valuationSourceType` and `valuationSourceUrls` exist for a reason —
  carry the provenance through into anything you publish.
- Every call here costs credits, and a retry costs them again. There is no idempotency key.
  De-duplicate before you send, and treat `424` as stop, not as retry.
- Watch `ratelimit-remaining`; the limit is 100 requests per second across the whole API.

## References

- vocabulary/cb-insights-vocabulary.yml — funding types, investor types.
- data-model/cb-insights-data-model.yml — the deal ↔ organization edges.
- rate-limits/cb-insights-rate-limits.yml — throttling and credit metering.
