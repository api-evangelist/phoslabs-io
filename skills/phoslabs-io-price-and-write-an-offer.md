---
generated: '2026-09-19'
method: generated
name: Price an offer and write the copy for it
description: Audit a product for free, get a behavioral pricing strategy, rewrite the offer copy, then check the result for cognitive biases — with the credit cost known up front.
api: openapi/phoslabs-io-openapi.yml
operations: [listTools, audit, pricing, copy, detectBiases]
source: >-
  Grounded in openapi/phoslabs-io-openapi.yml, captured verbatim 2026-09-19 from
  https://phoslabs.io/api/v1/openapi.json. All five operationIds verified in that spec. Prices from the live
  GET /api/v1/tools listing. Auth per authentication/phoslabs-io-authentication.yml, errors per
  errors/phoslabs-io-problem-types.yml, semantics per conventions/phoslabs-io-conventions.yml.
---

# Price an offer and write the copy for it

Total cost if every step runs: 53 credits (EUR 0.53). The first two calls are free.

## Auth
- `Authorization: Bearer <api_key>` on every POST; `listTools` is anonymous. Base URL `https://phoslabs.io/api/v1`.

## Steps

1. **Confirm prices** with `listTools` (`GET /api/v1/tools`, free). Read `credits` for `pricing`, `copy` and
   `detect-biases` from the response rather than assuming 25 / 20 / 8.
2. **Audit for free** with `audit` (`POST /api/v1/audit`, 0 credits):
   `{"product_description": "<what it is and who it is for>", "url": "<optional>", "industry": "<optional>", "goal": "<optional, e.g. increase paid conversions>"}`.
   `product_description` is required. This also proves the key works before anything is billed.
3. **Get the pricing strategy** with `pricing` (`POST /api/v1/pricing`, 25 credits):
   `{"product": "<the product>", "context": "<market, competitors, current price>", "persona": "<buyer>"}`. Only `product` is required.
4. **Rewrite the offer copy** with `copy` (`POST /api/v1/copy`, 20 credits):
   `{"draft": "<the current copy>", "goal": "<the conversion goal>"}`. Both fields are required — the call fails
   without `goal`.
5. **Check the copy for bias** with `detectBiases` (`POST /api/v1/detect-biases`, 8 credits):
   `{"text": "<the rewritten copy>"}`. Phos Labs' own terms forbid using the tools "to design deceptive
   practices, dark patterns, or manipulative interfaces"; use this step to keep the output on the right side
   of that line and show the human what it flagged.
6. **Return all four results** to the human with the `credits_remaining` from the last response.

## Rules
- Every response is `{"success", "result", "credits_remaining"}`; `result` is untyped — quote it, do not
  assume fields.
- Never retry a POST that may have succeeded: no `Idempotency-Key`, no refund path in the API.
- 401 -> stop (key missing/invalid). 402 -> stop and ask for a top-up (https://phoslabs.io/credits, EUR 5 /
  500 credits and up). No documented 429.
- The outputs are "behavioral science analysis only ... not legal, financial, medical, or other professional
  advice" (terms §6); present them as suggestions.
