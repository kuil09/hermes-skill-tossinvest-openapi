# Local TossInvest CLI cookbook

CLI command:

- `tossinvest`

## Environment

Credentials expected in environment:

- `TOSSINVEST_API_KEY`
- `TOSSINVEST_SECRET_KEY`
- optional: `TOSSINVEST_ACCOUNT_SEQ`

Inject credentials via environment variables in shells, CI, or agent runtimes.

## First diagnostics

```bash
tossinvest env-check
tossinvest doctor
```

`doctor` also reports the current public IP and will surface the Toss error hint when the IP is not whitelisted.

## Snapshot and approval helpers

```bash
tossinvest-snapshot
tossinvest-snapshot --format json

tossinvest-order-approval \
  --symbol 005930 \
  --side BUY \
  --order-type LIMIT \
  --quantity 1 \
  --price 350000
```

`tossinvest-order-approval` produces a Korean approval request with:

- background context
- purpose of the request
- expected result
- live command to run only after approval

## Read-only flows

```bash
tossinvest token
tossinvest accounts
tossinvest holdings
tossinvest holdings --symbol AAPL

tossinvest market stocks --symbols 005930,AAPL
tossinvest market prices --symbols 005930,AAPL
tossinvest market orderbook 005930
tossinvest market trades AAPL --count 20
tossinvest market price-limits 005930
tossinvest market candles AAPL --interval 1d --count 30
tossinvest market exchange-rate --base-currency KRW --quote-currency USD
tossinvest market market-calendar KR --date 2026-06-22

tossinvest order-info buying-power --currency KRW
tossinvest order-info sellable-quantity --symbol AAPL
tossinvest order-info commissions

tossinvest orders list --status OPEN
tossinvest orders list --status CLOSED --limit 50
tossinvest orders get <order_id>
```

## Safe mutation previews

Preview only, no live request sent:

```bash
tossinvest orders create \
  --symbol 005930 \
  --side BUY \
  --order-type LIMIT \
  --quantity 1 \
  --price 70000

tossinvest orders modify \
  <order_id> \
  --order-type LIMIT \
  --price 70500

tossinvest orders cancel <order_id>
```

## Live mutation gate

Only send live order requests when the user explicitly wants execution:

```bash
tossinvest orders create ... --execute --confirm-live
tossinvest orders modify ... --execute --confirm-live
tossinvest orders cancel ... --execute --confirm-live
```

## Raw endpoint access

```bash
tossinvest raw GET /api/v1/stocks --query symbols=005930,AAPL
tossinvest raw GET /api/v1/holdings
tossinvest raw POST /api/v1/orders --body @order.json
```
