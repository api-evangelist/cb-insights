---
name: cb-insights-company-outlook
description: >-
  Read CB Insights' proprietary predictive scores for a company — Mosaic Score, Commercial Maturity
  and Exit Probability, current and historical — and generate an AI scouting report or strategy map
  on top of them.
api: CB Insights API v2
base_url: https://api.cbinsights.com
operations:
  - POST /v2/authorize
  - POST /v2/organizations
  - POST /v2/outlook
  - POST /v2/organizations/{orgId}/outlook
  - POST /v2/organizations/{orgId}/mosaichistory
  - POST /v2/organizations/{orgId}/commercialmaturityhistory
  - POST /v2/organizations/{orgId}/exitprobabilityhistory
  - POST /v2/outlook/fundingwindow
  - POST /v2/organizations/{orgId}/fundingwindow
  - POST /v2/organizations/{orgId}/scoutingreport
  - POST /v2/organizations/{orgId}/scoutingreportstream
  - POST /v2/organizations/{orgId}/strategymap
generated: '2026-08-09'
method: generated
source: >-
  Grounded in openapi/cb-insights-api-v2-openapi.json (v2Outlook.*, v2ScoutingReports.*,
  v2StrategyMap.* definitions) and
  https://api-docs.cbinsights.com/portal/docs/CBI-data/Company-data-and-insights/outlook.
  The published contract declares no operationIds, so operations are referenced by method and path.
---

# Score a company and generate an outlook

## Steps

1. Authenticate — `POST /v2/authorize`, cache the 24-hour token.
2. Resolve to `orgId`s — see `cb-insights-resolve-organizations` (free).
3. **Current scores.** `POST /v2/outlook` with `orgIds` for a cohort, or
   `POST /v2/organizations/{orgId}/outlook` for one company. The response carries `mosaicScore`,
   `commercialMaturity` and `exitProbability`, each with its own signals/insights breakdown.
4. **Trend.** For direction of travel rather than a point reading, call the history operations —
   `/mosaichistory`, `/commercialmaturityhistory`, `/exitprobabilityhistory` — with `startDate` and
   `endDate`.
5. **Funding window.** `POST /v2/outlook/fundingwindow` (cohort) or
   `POST /v2/organizations/{orgId}/fundingwindow` (single) for the predicted next-raise window.
6. **Narrative.** `POST /v2/organizations/{orgId}/scoutingreport` returns an AI-generated report as
   both `reportJson` and `reportMarkdown`. Use `/scoutingreportstream` when you want incremental
   output for a UI. `POST /v2/organizations/{orgId}/strategymap` returns the categorised strategy
   map — categories, companies, connections, financial events and business relationships.

## Rules

- **Do not send `mosaicScoreVersion`.** It is marked deprecated in the contract on every request
  body that accepts it: "Deprecated. Only Mosaic v2.1 is served."
- Scores are model output, not fact. Carry the score version and the retrieval date into anything
  you store or publish, and re-pull rather than caching indefinitely — the history endpoints exist
  precisely because these values move.
- Scouting reports are **generated on demand**, which makes them the most expensive calls in the
  API in credit terms. Generate once, store the `reportMarkdown`, and do not regenerate to re-read.
- No idempotency key exists. A retried scouting-report request generates and bills a second report.
- `424` means the credit balance is exhausted — stop, do not retry.
- `403` after a period of success means the 24-hour token expired; re-authorize and retry once.

## References

- vocabulary/cb-insights-vocabulary.yml — Mosaic model versions and scoring references.
- conventions/cb-insights-conventions.yml — streaming endpoints, metering, pagination.
- lifecycle/cb-insights-lifecycle.yml — the deprecated fields in this flow.
