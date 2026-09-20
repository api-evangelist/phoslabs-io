---
generated: '2026-09-19'
method: generated
name: Diagnose a funnel drop-off and redesign the checkout
description: Use Phos Labs' contracted REST operations to find where a funnel loses people and get a behaviorally-grounded redesign, spending credits deliberately.
api: openapi/phoslabs-io-openapi.yml
operations: [listTools, diagnose, fixCheckout]
source: >-
  Grounded in openapi/phoslabs-io-openapi.yml, captured verbatim 2026-09-19 from
  https://phoslabs.io/api/v1/openapi.json. All three operationIds verified in that spec. Prices from the live
  GET /api/v1/tools listing (plans/phoslabs-io-api-v1-tools.json). Auth per
  authentication/phoslabs-io-authentication.yml, errors per errors/phoslabs-io-problem-types.yml, semantics per
  conventions/phoslabs-io-conventions.yml.
---

# Diagnose a funnel drop-off and redesign the checkout

Phos Labs' REST API is prepaid and metered per call. This skill spends 40 credits (EUR 0.40) end to end,
so confirm the balance before you start and never retry a POST that returned 200.

## Auth
- `Authorization: Bearer <api_key>` on every POST. Keys come from `POST https://phoslabs.io/api/trial`
  (`{"email": "...", "name": "..."}`) or from buying credits at https://phoslabs.io/credits. Ask the human for
  the key; do not provision one on their behalf.
- Base URL: `https://phoslabs.io/api/v1`. `GET /api/v1/tools` (`listTools`) needs no key.

## Steps

1. **Read the price list first** with `listTools` (`GET /api/v1/tools`). It returns every tool with `credits`,
   `price` and `currency` (EUR) and the input field names. Prices are the provider's live numbers and have
   changed since the README was written; trust this call, not memory.
2. **Diagnose** with `diagnose` (`POST /api/v1/diagnose`, 10 credits):
   `{"funnel_description": "<the funnel, step by step, with the observed conversion at each step>", "data": {<optional analytics>}, "industry": "<optional>"}`.
   `funnel_description` is required. The response is `{"success": true, "result": {...}, "credits_remaining": n}`;
   `result` is tool-specific and untyped in the contract — surface it to the human rather than parsing fields
   you cannot rely on.
3. **Check the balance.** If `credits_remaining` is below 30, stop and tell the human: the next call will fail
   with HTTP 402 `Insufficient credits`, and there is no free retry.
4. **Redesign** with `fixCheckout` (`POST /api/v1/fix-checkout`, 30 credits):
   `{"checkout_flow": "<the current flow as text, informed by the diagnosis>"}`. `checkout_flow` is required.
5. **Hand over, do not act.** Both results are analyses. Nothing in this API changes the human's product;
   the recommended flow is theirs to implement.

## Rules
- **No idempotency.** There is no `Idempotency-Key`; a retried POST is a second billable call. On a network
  timeout, check `credits_remaining` with your next successful call before repeating anything.
- **No reversal.** Spent credits are not refunded through the API; terms §3 say to email
  support@phoslabs.io and "a person reads it and replies", with no stated window. Treat every paid call as
  final.
- **Errors** are `{"error": "<message>", "credits_remaining": n|null}`: 401 means missing or invalid key
  (stop, do not loop), 402 means insufficient credits (stop, ask the human to top up). No 429 is documented;
  the terms say rate limits exist but publish none.
- **Free rehearsal:** `audit` (`POST /api/v1/audit`) costs 0 credits and can be used to sanity-check the
  key and the product description before spending.
