# Changelog

## Unreleased

Brings the SDK to parity with the Futures API docs v1.0.12. This is a clean
break for the changed call forms below: removed arguments raise an error
(`TypeError`/`ValueError`) and no request is sent. Only `get_order` keeps
working, with a `DeprecationWarning`.

### Changed signatures (old → new)

| Area | Old | New |
|---|---|---|
| Client currency | `TradeClient(trade_currency="INR")` raised `ValueError` (USDT only) | `trade_currency` accepts `"USDT"` or `"INR"`; other values raise `ValueError` |
| Place order | `place_order("BTCUSDT", leverage="10", quantity=..., ...)` → `POST /fapi/v1/futures/{symbol}/order` | `client.set_leverage("BTCUSDT", leverage="10")` first, then `place_order("BTCUSDT", quantity=..., ...)` → `POST /fapi/v2/futures/order`. Passing `leverage=` raises `TypeError` |
| Client order id on place | not supported | `place_order(..., client_order_id="...")` |
| Cancel | `cancel_order(order_id)` | `cancel_order(order_id)` or `cancel_order(client_order_id="...")` (exactly one) |
| Amend | `amend_order(order_id, order_price=...)` | `amend_order(order_id, order_price=..., quantity=...)`. Docs require both; the API response is surfaced unchanged |
| Spot funds | `get_wallet_funds()` | `get_wallet_funds(currency="USDT" \| "INR")`, default `"USDT"` |
| Risk orders | no client ids | `place_risk_order(..., stoploss_client_order_id=, takeprofit_client_order_id=)`; `amend_risk_order(..., stoploss_client_order_id=, takeprofit_client_order_id=, trigger_type="MARKET" \| "LIMIT")` (on edit the client ids only locate an existing leg) |
| Partial close | `close_position_partial(pid, quantity=, order_type=)`, docstring said `LONG`/`SHORT` | adds `client_order_id=`; `order_type` is `"MARKET"` or `"LIMIT"` |

### Added

- `get_order_detail(order_id=None, client_order_id=None)` — `GET /futures/orders/detail`.
- `get_execution_report(order_id)` — `GET /futures/orders/{order_id}/execution-report`.
- `get_fee(fee_id)` — `GET /futures/fee/{fee_id}`.
- `get_transactions(limit=None, trade_currency=None)` — `GET /futures/transactions`.
- `MudrexAPIError.retry_after`: seconds from the `Retry-After` header on HTTP 429 (`None` if absent). The SDK still does not retry.

### Deprecated

- `get_order(order_id)` still works but emits `DeprecationWarning`; use `get_order_detail`.

### Response shape changes

- **Square off** (`close_position`): returns `order_id` (close order UUID) and `position_id` (position UUID). Before API v1.0.12, `position_id` held the close order UUID.
- **Partial close** (`close_position_partial`): returns an object with `order_id`, `position_id`, `quantity` (was `{"result": True}`).
- `data: null` now returns `None` instead of `MudrexResponse({"result": None})`; `get_positions()` returns `[]` when there are no open positions.
- Scalar `data` is returned as-is instead of `MudrexResponse({"result": value})`, e.g. `get_liquidation_price` returns the price string. Replace `resp.result` with the value itself.

### Documentation

- README rate limits now match Authentication and Rate Limits (default 10/s, 500/min, 30000/hr; core trading adds 720000/day; wallet 2/s, 50/min, 3000/hr).
- README and docstrings describe v1.0.12 fields: `position_id` (orders, open positions, position history), `order_id` on position history rows, `previous_funding_interval` on asset endpoints.
