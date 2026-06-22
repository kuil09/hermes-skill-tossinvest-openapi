# Toss Securities Open API summary

## Canonical documentation

- Browser docs: `https://developers.tossinvest.com/docs`
- Agent hint: `https://developers.tossinvest.com/llms.txt`
- Overview markdown: `https://openapi.tossinvest.com/openapi-docs/overview.md`
- Canonical OpenAPI JSON: `https://openapi.tossinvest.com/openapi-docs/latest/openapi.json`
- Base API server: `https://openapi.tossinvest.com`
- Observed OpenAPI version during review: `1.1.1`

## Authentication

- Token endpoint: `POST /oauth2/token`
- Grant type: `client_credentials`
- Required form fields:
  - `grant_type=client_credentials`
  - `client_id`
  - `client_secret`
- Token response includes:
  - `access_token`
  - `token_type=Bearer`
  - `expires_in`

## Account header rule

The following categories require both bearer token and:

- `X-Tossinvest-Account: {accountSeq}`

Get `accountSeq` from:

- `GET /api/v1/accounts`

## Endpoint groups

### Market data / reference data

- `GET /api/v1/orderbook`
- `GET /api/v1/prices`
- `GET /api/v1/trades`
- `GET /api/v1/price-limits`
- `GET /api/v1/candles`
- `GET /api/v1/stocks`
- `GET /api/v1/stocks/{symbol}/warnings`
- `GET /api/v1/exchange-rate`
- `GET /api/v1/market-calendar/KR`
- `GET /api/v1/market-calendar/US`

### Account / holdings

- `GET /api/v1/accounts`
- `GET /api/v1/holdings`

### Order / order history / order info

- `POST /api/v1/orders`
- `POST /api/v1/orders/{orderId}/modify`
- `POST /api/v1/orders/{orderId}/cancel`
- `GET /api/v1/orders`
- `GET /api/v1/orders/{orderId}`
- `GET /api/v1/buying-power`
- `GET /api/v1/sellable-quantity`
- `GET /api/v1/commissions`

## Important request-model notes

### Create order

Two documented request shapes:

1. Quantity-based:
   - `symbol`
   - `side` = `BUY|SELL`
   - `orderType` = `LIMIT|MARKET`
   - `timeInForce` often `DAY|CLS`
   - `quantity`
   - optional `price`
   - optional `clientOrderId`
   - optional `confirmHighValueOrder`

2. Amount-based:
   - US only
   - `orderType=MARKET`
   - `orderAmount`
   - optional `clientOrderId`
   - optional `confirmHighValueOrder`

### Modify order

- Path param: `orderId`
- Header: `X-Tossinvest-Account`
- Body fields:
  - `orderType`
  - optional `quantity`
  - optional `price`
  - optional `confirmHighValueOrder`

### Cancel order

- Path param: `orderId`
- Header: `X-Tossinvest-Account`
- JSON body exists in the API shape even though it is logically a cancel call

## Important query parameters

### Holdings

- optional `symbol`

### Orders list

- required `status = OPEN | CLOSED`
- optional `symbol`
- optional `from`
- optional `to`
- optional `cursor`
- optional `limit`

Note: `OPEN|CLOSED` is a grouping filter, not the same enum system as per-order detailed status.

### Candles

- required `interval = 1m | 1d`
- optional `count`
- optional `before`
- optional `adjusted`
- documented max return: 200 candles

## Rate limits captured from overview

- `AUTH`: 5 TPS
- `ACCOUNT`: 1 TPS
- `ASSET`: 5 TPS
- `STOCK`: 5 TPS
- `MARKET_INFO`: 3 TPS
- `MARKET_DATA`: 10 TPS
- `MARKET_DATA_CHART`: 5 TPS
- `ORDER`: 6 TPS, reduced to 3 TPS during 09:00~09:10 KST
- `ORDER_HISTORY`: 5 TPS
- `ORDER_INFO`: 6 TPS, reduced to 3 TPS during 09:00~09:10 KST

Response headers to use:

- `X-RateLimit-Limit`
- `X-RateLimit-Remaining`
- `X-RateLimit-Reset`
- `Retry-After` on 429

## Error envelope

```json
{
  "error": {
    "requestId": "...",
    "code": "invalid-request",
    "message": "...",
    "data": {}
  }
}
```

## High-signal documented error codes

- `invalid-request`
- `confirm-high-value-required`
- `account-header-required`
- `invalid-token`
- `expired-token`
- `login-user-not-found`
- `forbidden`
- `stock-not-found`
- `exchange-rate-not-found`
- `account-not-found`
- `order-not-found`
- `request-in-progress`
- `already-filled`
- `already-canceled`
- `already-modified`
- `already-rejected`
- `already-processing`
- `insufficient-buying-power`
- `order-hours-closed`
- `stock-restricted`
- `price-out-of-range`
- `opposite-pending-order-exists`
- `order-type-not-allowed`
- `prerequisite-required`
- `market-not-supported-for-stock`
- `investor-exchange-not-integrated`
- `amount-order-outside-regular-hours`
- `modify-restricted`
- `cancel-restricted`
- `rate-limit-exceeded`
- `internal-error`
- `maintenance`

## Practical integration stance

For investment-agent work, a good minimum sequence is:

1. issue token
2. fetch accounts
3. resolve `accountSeq`
4. fetch holdings and account-scoped info
5. fetch market data and exchange rate
6. compute recommendation
7. require explicit human approval before live order submission
