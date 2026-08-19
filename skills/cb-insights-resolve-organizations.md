---
name: cb-insights-resolve-organizations
description: >-
  Match a list of company names or domains you already hold to CB Insights orgIds without spending
  credits. This is the mandatory first step for every other CB Insights flow — no data endpoint
  except ChatCBI accepts anything but an orgId.
api: CB Insights API v2
base_url: https://api.cbinsights.com
operations:
  - POST /v2/authorize
  - POST /v2/organizations
generated: '2026-08-09'
method: generated
source: >-
  Grounded in openapi/_original/cb-insights-api-v2-openapi.json and
  https://api-docs.cbinsights.com/portal/docs/CBI-API/search-structure. The published contract
  declares no operationIds, so operations are referenced by method and path.
---

# Resolve organizations to CB Insights orgIds

## Why this skill exists

The CB Insights API is credit-metered: nearly every data call debits the account ledger. The
organization lookup operation is the documented exception — CB Insights states it "never charges
credits" and exists "to match CBI organizations with your own data prior to spending credits on
them". Resolve first; only then spend.

## Steps

1. **Get a token.** `POST /v2/authorize` with `{"clientId": ..., "clientSecret": ...}`. The response
   is `{"token": "..."}`. Cache it — it is valid 24 hours and there is no refresh token, so a fresh
   authorize on every call needlessly resends the long-lived secret.

2. **Send your identifiers.** `POST /v2/organizations` with `Authorization: Bearer <token>` and a
   body containing at least one search parameter:
   - `urls`: array of domains (e.g. `["cbinsights.com"]`) — the highest-precision key.
   - `names`: array of company names.
   - `profileUrl`: a single CB Insights profile URL. **Mutually exclusive with `names` and `urls`** —
     supplying it alongside either returns 400.
   - `limit`: 1–100, default 10.
   - `sort`: `{"field": "orgName", "direction": "asc"|"desc"}`.

3. **Read the response.** `orgs[]` carries `orgId`, `name`, `description`, `aliases` and `urls`.
   `totalHits` is the match count; `totalHitsRelation` is `eq` when exact and `gte` once the count
   passes 10,000.

4. **Page.** If `nextPageToken` is non-null, resend the same body with `nextPageToken` set. Stop when
   it is null. The token is opaque — never construct or parse one.

5. **Persist the mapping.** Store `your_identifier -> orgId` locally. Re-resolving on every run is
   free in credits but not in rate limit.

## Rules

- Batch. The lookup accepts arrays; sending one company per request burns the 100 requests/second
  budget for no reason.
- Confirm before you spend. Match on `urls` and cross-check `aliases`/`legalNames` — a name-only
  match can land on the wrong entity, and every downstream call on a wrong orgId costs credits.
- Handle `403` as "token expired": re-authorize and retry once.
- Do not retry a `424`. That is credit exhaustion, not a transient fault — the balance must be
  topped up by the account's CSM.

## References

- conventions/cb-insights-conventions.yml — pagination, filtering, metering.
- errors/cb-insights-problem-types.yml — the full status catalogue.
- data-model/cb-insights-data-model.yml — why orgId is the hub key.
