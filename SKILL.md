---
name: tossinvest-openapi-integration
description: Use when integrating Toss Securities Open API for market data, holdings, order workflows, and approval-gated execution.
version: 0.1.0
author: kuil09
license: MIT
metadata:
  hermes:
    tags: [tossinvest, openapi, trading, investing, brokerage]
    related_skills: [investment-monitoring-and-trade-briefs]
---

# TossInvest OpenAPI Integration

## Overview

This skill packages agent-agnostic Toss Securities Open API guidance for Hermes. The same operational rules also work for other AI agents and automation systems. It covers authentication, account header requirements, read-only data access, approval-gated order flows, rate limits, and operational pitfalls such as IP allowlisting.

## When to use

Use this skill when working on any Toss Securities Open API task, including:

- Reading or summarizing Toss Open API docs
- Building auth/token helpers
- Fetching market data, stock info, exchange rates, or market calendars
- Querying accounts, holdings, commissions, buying power, or sellable quantity
- Creating, modifying, canceling, or reviewing orders
- Designing an investment agent that uses Toss account data or execution

## Core rules

1. Treat the canonical OpenAPI spec as the source of truth:
   - `https://openapi.tossinvest.com/openapi-docs/latest/openapi.json`
2. Human-readable overview:
   - `https://openapi.tossinvest.com/openapi-docs/overview.md`
3. Browser docs:
   - `https://developers.tossinvest.com/docs`
4. Base API server:
   - `https://openapi.tossinvest.com`
5. Use OAuth 2.0 Client Credentials Grant for every API flow.
6. For account, asset, and order APIs, send both:
   - `Authorization: Bearer {access_token}`
   - `X-Tossinvest-Account: {accountSeq}`
7. For any account-specific trade execution workflow, preserve a human approval gate before live order placement unless the user explicitly changes that policy.

## Workflow

## Preferred local interface: `tossinvest` CLI

Recommended companion CLI commands:

- `tossinvest` — raw/programmatic API wrapper
- `tossinvest-snapshot` — holdings + buying power + FX snapshot helper
- `tossinvest-order-approval` — Korean approval-packet generator for proposed live orders

Use it as the default programmable interface before hand-writing `curl` unless a task specifically requires raw HTTP.

Safe starter commands:

```bash
tossinvest env-check
tossinvest doctor
tossinvest token
tossinvest accounts
tossinvest holdings
tossinvest market prices --symbols 005930,AAPL
tossinvest market stocks --symbols 005930,AAPL
tossinvest market exchange-rate --base-currency KRW --quote-currency USD
tossinvest order-info buying-power --currency KRW
tossinvest-snapshot
tossinvest-order-approval --symbol 005930 --side BUY --order-type LIMIT --quantity 1 --price 350000
```

Order mutations are preview-only by default. The CLI only sends live create/modify/cancel requests when both flags are present:

```bash
tossinvest orders create ... --execute --confirm-live
tossinvest orders modify ... --execute --confirm-live
tossinvest orders cancel ... --execute --confirm-live
```

It supports:

- token caching under `~/.cache/tossinvest-openapi/token.json`
- account auto-resolution when only one brokerage account exists
- reading credentials from environment variables or `~/.zshrc`
- raw endpoint access via `tossinvest raw ...`
- diagnostic checks via `tossinvest doctor`
- public-IP visibility so allowlist failures can be diagnosed quickly

## 1) Establish credentials and storage

Preferred env var names if you are wiring local scripts:

- `TOSSINVEST_API_KEY`
- `TOSSINVEST_SECRET_KEY`

Map them to the API's required token request fields:

- `client_id = TOSSINVEST_API_KEY`
- `client_secret = TOSSINVEST_SECRET_KEY`

If the user asks to store credentials in shell config, remember that protected credential files may reject direct file-edit tools. If so, use a safe file-writing path that still verifies the final file content.

If you build local wrappers around this API, prefer environment-variable injection over hardcoded secrets. Shell-config fallbacks are a convenience for local development, not a requirement for portable automation.

## 2) Get an access token

Request:

```bash
curl -s -X POST 'https://openapi.tossinvest.com/oauth2/token' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=client_credentials' \
  -d "client_id=$TOSSINVEST_API_KEY" \
  -d "client_secret=$TOSSINVEST_SECRET_KEY"
```

Response fields to expect:

- `access_token`
- `token_type` = `Bearer`
- `expires_in`

## 3) Resolve accountSeq before account-scoped calls

Call:

```bash
curl -s 'https://openapi.tossinvest.com/api/v1/accounts' \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

Use `accountSeq` from the response as the value for `X-Tossinvest-Account`.

## 4) Separate public-data vs account-scoped APIs

### Token only

- Market Data
- Stock Info
- Market Info

### Token + `X-Tossinvest-Account`

- Account
- Asset
- Order
- Order History
- Order Info

## 5) Be precise about order models

### Quantity-based orders

Supports KR and US flows.

Fields commonly used:

- `clientOrderId` (idempotency key)
- `symbol`
- `side` = `BUY` or `SELL`
- `orderType` = `LIMIT` or `MARKET`
- `timeInForce` = commonly `DAY` or `CLS`
- `quantity`
- `price` for limit orders
- `confirmHighValueOrder` when required

### Amount-based orders

Only for US market buy flows and `MARKET` orders.

Common fields:

- `clientOrderId`
- `symbol`
- `side`
- `orderType` = `MARKET`
- `orderAmount`
- `confirmHighValueOrder`

## 6) Respect rate limits

Use the response headers to throttle dynamically:

- `X-RateLimit-Limit`
- `X-RateLimit-Remaining`
- `X-RateLimit-Reset`
- `Retry-After` on 429

Important documented limits include:

- `AUTH`: 5 TPS
- `ACCOUNT`: 1 TPS
- `ASSET`: 5 TPS
- `STOCK`: 5 TPS
- `MARKET_INFO`: 3 TPS
- `MARKET_DATA`: 10 TPS
- `MARKET_DATA_CHART`: 5 TPS
- `ORDER`: 6 TPS, but 09:00~09:10 KST is 3 TPS
- `ORDER_HISTORY`: 5 TPS
- `ORDER_INFO`: 6 TPS, but 09:00~09:10 KST is 3 TPS

Apply backoff and jitter on 429.

## 6.5) Diagnose allowlist and network-access issues early

When auth fails, do not stop at "403". Inspect the error body and surface the operational next step.

Important case:

- `403 access_denied` with `error_description = IP address not allowed`

In that case:

1. Fetch and display the current public egress IP.
2. Tell the user to whitelist that IP in Toss Securities WTS > Open API settings.
3. Re-run a small diagnostic (`tossinvest doctor` or equivalent token probe) after the allowlist change.
4. Warn that home-network public IPs are often dynamic, so allowlist breakage can recur later even after a successful setup.

For recurring automated investment workflows, prefer a stable egress IP (fixed-IP line, VPS, or other fixed outbound address) over a consumer dynamic-IP path.

## 7) Handle errors by code, not only HTTP status

Common important codes:

- `invalid-token`
- `expired-token`
- `account-header-required`
- `account-not-found`
- `insufficient-buying-power`
- `order-hours-closed`
- `request-in-progress`
- `already-filled`
- `already-canceled`
- `already-processing`
- `rate-limit-exceeded`

Preserve `requestId` for support/debugging.

## Common Pitfalls

- Toss may reject valid credentials with `403 access_denied` and `IP address not allowed`. In that case, whitelist the current public IP in Toss Securities WTS > Open API settings before retrying.
- Do not assume token-only access works for holdings or orders; account header is required.
- Do not assume order modification/cancel returns the original `orderId`; the API can return a newly issued order identifier.
- Do not assume amount-based orders work for KR stocks; the documented amount-based model is US MARKET only.
- Do not treat `status=OPEN|CLOSED` query values as the same enum as `orders[].status`; the list filter is a grouped lifecycle label.
- Do not hardcode rate limits without checking headers; the docs explicitly allow operational adjustments.
- If the browser docs say they are JS-rendered, switch to `llms.txt`, overview markdown, or the canonical OpenAPI JSON instead of scraping rendered HTML.

## Deliverables to prefer

For implementation tasks, prefer producing:

- a tiny token helper
- a verified account discovery call
- a holdings fetcher
- a market-data fetcher
- a dry-run or approval-gated order path

## Verification Checklist

- [ ] Canonical OpenAPI spec consulted for any implementation change
- [ ] Credentials injected via environment variables, not committed files
- [ ] `X-Tossinvest-Account` present for account-scoped endpoints
- [ ] Live order flows remain approval-gated unless explicitly changed
- [ ] Rate-limit and allowlist failure handling documented

## References

- `references/tossinvest-openapi-summary.md` — concise endpoint/auth/rate-limit notes captured from the official docs.
- `references/local-cli-cookbook.md` — local CLI usage examples, including snapshot and approval helper commands.
