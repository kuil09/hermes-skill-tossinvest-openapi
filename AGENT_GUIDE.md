# TossInvest OpenAPI Agent Guide

This document is framework-neutral guidance for any AI agent or automation system that needs to work with the Toss Securities Open API.

## Goal

Operate Toss Open API integrations safely and programmatically for:

- market data retrieval
- account and holdings inspection
- buying power / commissions / sellable quantity checks
- order review workflows
- approval-gated live execution

## Non-negotiable rules

1. Treat the canonical OpenAPI spec as the source of truth:
   - `https://openapi.tossinvest.com/openapi-docs/latest/openapi.json`
2. Use OAuth 2.0 Client Credentials Grant.
3. Never embed secrets in code, prompts, or committed files.
4. Inject credentials via environment variables:
   - `TOSSINVEST_API_KEY`
   - `TOSSINVEST_SECRET_KEY`
   - optional `TOSSINVEST_ACCOUNT_SEQ`
5. For account, holdings, and order endpoints, send both:
   - `Authorization: Bearer {access_token}`
   - `X-Tossinvest-Account: {accountSeq}`
6. Keep live order execution behind an explicit human approval gate unless the operator intentionally changes that policy.

## Recommended operating sequence

1. Verify credentials are present.
2. Issue or refresh an access token.
3. Resolve `accountSeq` with `GET /api/v1/accounts` before account-scoped calls.
4. Separate public-data endpoints from account-scoped endpoints.
5. Fetch holdings, buying power, commissions, sellable quantity, market data, and exchange rates as needed.
6. Produce a human-readable review packet before any live order mutation.
7. Only after explicit approval, send live create / modify / cancel requests.

## Preferred companion CLI

If available, prefer the companion CLI instead of hand-writing HTTP calls:

- `tossinvest`
- `tossinvest-snapshot`
- `tossinvest-order-approval`

Useful starter commands:

```bash
tossinvest env-check
tossinvest doctor
tossinvest token
tossinvest accounts
tossinvest holdings
tossinvest market prices --symbols 005930,AAPL
tossinvest order-info buying-power --currency KRW
tossinvest-snapshot
tossinvest-order-approval --symbol 005930 --side BUY --order-type LIMIT --quantity 1 --price 350000
```

## Approval-gated execution policy

Preview live order mutations by default.
Only execute create / modify / cancel when the operator clearly authorizes execution.

Example execution command:

```bash
tossinvest orders create ... --execute --confirm-live
```

## Error handling priorities

Do not stop at HTTP status alone. Inspect structured error codes and bodies.
High-signal cases include:

- `invalid-token`
- `expired-token`
- `account-header-required`
- `account-not-found`
- `insufficient-buying-power`
- `order-hours-closed`
- `rate-limit-exceeded`
- `access_denied` with `IP address not allowed`

## Allowlist diagnosis

If token issuance fails with `403 access_denied` and `error_description = IP address not allowed`:

1. Determine the current public egress IP.
2. Ask the operator to whitelist that IP in Toss Securities WTS > Open API settings.
3. Re-run a token or doctor check.
4. Warn that consumer-network IPs may change later.

## Rate limits

Throttle dynamically using response headers:

- `X-RateLimit-Limit`
- `X-RateLimit-Remaining`
- `X-RateLimit-Reset`
- `Retry-After`

Apply backoff and jitter on `429`.

## Deliverables to prefer

For implementation tasks, prefer producing:

- a token helper
- an account discovery call
- a holdings fetcher
- a market-data fetcher
- a dry-run or approval-packet workflow
- an approval-gated live-order path

## References

- `references/tossinvest-openapi-summary.md`
- `references/local-cli-cookbook.md`
- companion CLI: `https://github.com/kuil09/tossinvest-openapi-cli`
