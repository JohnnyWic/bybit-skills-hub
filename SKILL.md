---
name: bybit-v5
description: Bybit V5 unified API skill for spot, linear perpetuals, inverse contracts, and options trading. Supports market data, order management, position management, account operations, and more. Authentication via HMAC-SHA256 with header-based signing.
metadata:
  version: 1.0.0
  author: Bybit
license: MIT
---

# Bybit V5 Unified Skill

Unified API skill for Bybit V5 covering all product types: **spot**, **linear** (USDT/USDC perpetuals), **inverse** (inverse perpetuals/futures), and **option**. All endpoints share a common `category` parameter to specify the product type. Returns results in JSON format.

---

## Authentication

### Base URLs

| Region | Mainnet URL | Notes |
|--------|-------------|-------|
| Global (default) | `https://api.bybit.com` | Primary |
| Global (backup) | `https://api.bytick.com` | Backup |
| Netherlands | `https://api.bybit.nl` | |
| Turkey | `https://api.bybit-tr.com` | |
| Kazakhstan | `https://api.bybit.kz` | |
| Georgia | `https://api.bybitgeorgia.ge` | |
| UAE | `https://api.bybit.ae` | |
| EEA | `https://api.bybit.eu` | |
| Indonesia | `https://api.bybit.id` | |
| **Testnet** | `https://api-testnet.bybit.com` | Always this URL regardless of region |

Agent SHALL auto-select the base URL based on the account's `Region` field. If no region is set, default to `https://api.bybit.com`. Testnet accounts always use `https://api-testnet.bybit.com`.

### Required Headers

| Header | Value | Description |
|--------|-------|-------------|
| `X-BAPI-API-KEY` | Your API key | API key identifier |
| `X-BAPI-TIMESTAMP` | Unix timestamp (ms) | Current UTC time in milliseconds |
| `X-BAPI-SIGN` | HMAC-SHA256 hex string | Request signature |
| `X-BAPI-RECV-WINDOW` | `5000` (default) | Request validity window in ms |
| `Content-Type` | `application/json` | Required for POST requests |
| `User-Agent` | `bybit-v5/1.0.0 (Skill)` | Product identifier |

### Signing Process

**Step 1: Build the string to sign**

For **GET** requests:
```
{timestamp}{apiKey}{recvWindow}{queryString}
```

For **POST** requests:
```
{timestamp}{apiKey}{recvWindow}{jsonBodyString}
```

**Step 2: Generate HMAC-SHA256 signature**

```bash
echo -n "${PARAM_STR}" | openssl dgst -sha256 -hmac "${SECRET_KEY}" | cut -d' ' -f2
```

The result is a lowercase hex string.

**Step 3: Include signature in X-BAPI-SIGN header**

### Complete Examples

**GET request (authenticated):**

```bash
#!/bin/bash
API_KEY="your_api_key"
SECRET_KEY="your_secret_key"
BASE_URL="https://api.bybit.com"
RECV_WINDOW=5000
TIMESTAMP=$(date +%s000)

QUERY="category=linear&symbol=BTCUSDT"
PARAM_STR="${TIMESTAMP}${API_KEY}${RECV_WINDOW}${QUERY}"
SIGN=$(echo -n "$PARAM_STR" | openssl dgst -sha256 -hmac "$SECRET_KEY" | cut -d' ' -f2)

curl -s "${BASE_URL}/v5/position/list?${QUERY}" \
  -H "X-BAPI-API-KEY: ${API_KEY}" \
  -H "X-BAPI-TIMESTAMP: ${TIMESTAMP}" \
  -H "X-BAPI-SIGN: ${SIGN}" \
  -H "X-BAPI-RECV-WINDOW: ${RECV_WINDOW}" \
  -H "User-Agent: bybit-v5/1.0.0 (Skill)"
```

**POST request (place order):**

```bash
#!/bin/bash
API_KEY="your_api_key"
SECRET_KEY="your_secret_key"
BASE_URL="https://api.bybit.com"
RECV_WINDOW=5000
TIMESTAMP=$(date +%s000)

BODY='{"category":"spot","symbol":"BTCUSDT","side":"Buy","orderType":"Market","qty":"0.001"}'
PARAM_STR="${TIMESTAMP}${API_KEY}${RECV_WINDOW}${BODY}"
SIGN=$(echo -n "$PARAM_STR" | openssl dgst -sha256 -hmac "$SECRET_KEY" | cut -d' ' -f2)

curl -s -X POST "${BASE_URL}/v5/order/create" \
  -H "Content-Type: application/json" \
  -H "X-BAPI-API-KEY: ${API_KEY}" \
  -H "X-BAPI-TIMESTAMP: ${TIMESTAMP}" \
  -H "X-BAPI-SIGN: ${SIGN}" \
  -H "X-BAPI-RECV-WINDOW: ${RECV_WINDOW}" \
  -H "User-Agent: bybit-v5/1.0.0 (Skill)" \
  -d "${BODY}"
```

### Response Format

All Bybit V5 responses follow this structure:

```json
{
  "retCode": 0,
  "retMsg": "OK",
  "result": { },
  "retExtInfo": {},
  "time": 1672211918471
}
```

`retCode=0` means success. Non-zero indicates an error.

**Rate Limit Response Headers:**
- `X-Bapi-Limit` — Your rate limit for this endpoint
- `X-Bapi-Limit-Status` — Remaining requests in current window
- `X-Bapi-Limit-Reset-Timestamp` — When limit resets (ms)

---

## Quick Reference

### Market Data (No Authentication)

| Endpoint | Path | Method | Required | Optional | Categories |
|----------|------|--------|----------|----------|------------|
| Kline | `/v5/market/kline` | GET | symbol, interval | category, start, end, limit | spot, linear, inverse |
| Mark Price Kline | `/v5/market/mark-price-kline` | GET | category, symbol, interval | start, end, limit | linear, inverse |
| Index Price Kline | `/v5/market/index-price-kline` | GET | category, symbol, interval | start, end, limit | linear, inverse |
| Premium Index Kline | `/v5/market/premium-index-price-kline` | GET | category, symbol, interval | start, end, limit | linear |
| Instruments Info | `/v5/market/instruments-info` | GET | category | symbol, baseCoin, limit, cursor, status | spot, linear, inverse, option |
| Orderbook | `/v5/market/orderbook` | GET | category, symbol | limit | spot, linear, inverse, option |
| Tickers | `/v5/market/tickers` | GET | category | symbol, baseCoin, expDate | spot, linear, inverse, option |
| Funding Rate History | `/v5/market/funding/history` | GET | category, symbol | startTime, endTime, limit | linear, inverse |
| Recent Trades | `/v5/market/recent-trade` | GET | category, symbol | baseCoin, limit | spot, linear, inverse, option |
| Open Interest | `/v5/market/open-interest` | GET | category, symbol, intervalTime | startTime, endTime, limit, cursor | linear, inverse |
| Historical Volatility | `/v5/market/historical-volatility` | GET | category | baseCoin, period, startTime, endTime | option |
| Insurance | `/v5/market/insurance` | GET | — | coin | — |
| Risk Limit | `/v5/market/risk-limit` | GET | category | symbol | linear, inverse |
| Delivery Price | `/v5/market/delivery-price` | GET | category | symbol, baseCoin, limit, cursor | linear, inverse, option |
| Long Short Ratio | `/v5/market/account-ratio` | GET | category, symbol, period | limit | linear, inverse |
| Server Time | `/v5/market/time` | GET | — | — | — |
| Announcement | `/v5/announcements/index` | GET | — | locale, type, tag, page, limit | — |

### Trade (Authentication Required)

| Endpoint | Path | Method | Required | Optional | Rate Limit | Categories |
|----------|------|--------|----------|----------|------------|------------|
| Place Order | `/v5/order/create` | POST | category, symbol, side, orderType, qty | price, timeInForce, orderLinkId, triggerPrice, takeProfit, stopLoss, tpslMode, reduceOnly, positionIdx, ... | 10-20/s | spot, linear, inverse, option |
| Amend Order | `/v5/order/amend` | POST | category, symbol | orderId/orderLinkId, qty, price, takeProfit, stopLoss, triggerPrice, tpLimitPrice, slLimitPrice | 10/s | spot, linear, inverse, option |
| Cancel Order | `/v5/order/cancel` | POST | category, symbol | orderId/orderLinkId, orderFilter | 10-20/s | spot, linear, inverse, option |
| Get Open Orders | `/v5/order/realtime` | GET | category | symbol, baseCoin, orderId, orderLinkId, openOnly, orderFilter, limit, cursor | 50/s | spot, linear, inverse, option |
| Cancel All Orders | `/v5/order/cancel-all` | POST | category | symbol, baseCoin, settleCoin, orderFilter, stopOrderType | 10/s | spot, linear, inverse, option |
| Get Order History | `/v5/order/history` | GET | category | symbol, orderId, orderLinkId, orderFilter, orderStatus, startTime, endTime, limit, cursor | 50/s | spot, linear, inverse, option |
| Batch Place | `/v5/order/create-batch` | POST | category, request[] | — | per-order | spot, linear, inverse, option |
| Batch Amend | `/v5/order/amend-batch` | POST | category, request[] | — | per-order | spot, linear, inverse, option |
| Batch Cancel | `/v5/order/cancel-batch` | POST | category, request[] | — | per-order | spot, linear, inverse, option |
| Spot Borrow Check | `/v5/order/spot-borrow-check` | GET | category, symbol, side | — | — | spot |
| DCP | `/v5/order/disconnected-cancel-all` | POST | timeWindow | — | — | option |

### Position (Authentication Required)

| Endpoint | Path | Method | Required | Optional | Categories |
|----------|------|--------|----------|----------|------------|
| Get Position | `/v5/position/list` | GET | category | symbol, baseCoin, settleCoin, limit, cursor | linear, inverse, option |
| Set Leverage | `/v5/position/set-leverage` | POST | category, symbol, buyLeverage, sellLeverage | — | linear, inverse |
| Switch Isolated/Cross | `/v5/position/switch-isolated` | POST | category, symbol, tradeMode, buyLeverage, sellLeverage | — | linear, inverse |
| Set TP/SL Mode | `/v5/position/set-tpsl-mode` | POST | category, symbol, tpSlMode | — | linear, inverse |
| Switch Position Mode | `/v5/position/switch-mode` | POST | category, mode | coin, symbol | linear, inverse |
| Set Risk Limit | `/v5/position/set-risk-limit` | POST | category, symbol, riskId | — | linear, inverse |
| Set Trading Stop | `/v5/position/trading-stop` | POST | category, symbol, tpslMode, positionIdx | takeProfit, stopLoss, trailingStop, tpTriggerBy, slTriggerBy, activePrice, tpSize, slSize, tpLimitPrice, slLimitPrice, tpOrderType, slOrderType | linear, inverse |
| Set Auto Add Margin | `/v5/position/set-auto-add-margin` | POST | category, symbol, autoAddMargin | positionIdx | linear, inverse |
| Add/Reduce Margin | `/v5/position/add-margin` | POST | category, symbol, margin | positionIdx | linear, inverse |
| Move Position | `/v5/position/move-positions` | POST | fromUid, toUid, list[] | — | linear, inverse |
| Move Position History | `/v5/position/move-history` | GET | — | category, symbol, startTime, endTime, status, blockTradeId, limit, cursor | linear, inverse |
| Get Trade History | `/v5/execution/list` | GET | category | symbol, baseCoin, orderId, orderLinkId, startTime, endTime, execType, limit, cursor | spot, linear, inverse, option |
| Get Closed PnL | `/v5/position/closed-pnl` | GET | category, symbol | startTime, endTime, limit, cursor | linear, inverse |
| Get Closed Options | `/v5/position/get-closed-positions` | GET | category | symbol, limit, cursor | option |

### Account (Authentication Required)

| Endpoint | Path | Method | Required | Optional |
|----------|------|--------|----------|----------|
| Wallet Balance | `/v5/account/wallet-balance` | GET | accountType | coin |
| Account Info | `/v5/account/info` | GET | — | — |
| Upgrade to UTA | `/v5/account/upgrade-to-uta` | POST | — | — |
| Borrow History | `/v5/account/borrow-history` | GET | — | currency, startTime, endTime, limit, cursor |
| Set Collateral | `/v5/account/set-collateral-switch` | POST | coin, collateralSwitch | — |
| Collateral Info | `/v5/account/collateral-info` | GET | — | currency |
| Coin Greeks | `/v5/asset/coin-greeks` | GET | — | baseCoin |
| Fee Rate | `/v5/account/fee-rate` | GET | category | symbol, baseCoin |
| Transaction Log | `/v5/account/transaction-log` | GET | — | accountType, category, currency, baseCoin, type, startTime, endTime, limit, cursor |
| Set Margin Mode | `/v5/account/set-margin-mode` | POST | setMarginMode | — |
| Set MMP | `/v5/account/mmp-modify` | POST | baseCoin, window, frozenPeriod, qtyLimit, deltaLimit | — |
| Reset MMP | `/v5/account/mmp-reset` | POST | baseCoin | — |
| Get MMP State | `/v5/account/mmp-state` | GET | baseCoin | — |

### Asset (Authentication Required)

| Endpoint | Path | Method | Required | Optional |
|----------|------|--------|----------|----------|
| Coin Exchange Records | `/v5/asset/exchange/order-record` | GET | — | fromCoin, toCoin, limit, cursor |
| Delivery Record | `/v5/asset/delivery-record` | GET | category | symbol, expDate, limit, cursor |
| USDC Settlement Record | `/v5/asset/settlement-record` | GET | category | symbol, limit, cursor |
| Internal Transfer Records | `/v5/asset/transfer/query-inter-transfer-list` | GET | — | transferId, coin, status, startTime, endTime, limit, cursor |
| Asset Info (Spot) | `/v5/asset/transfer/query-asset-info` | GET | accountType | coin |
| All Coins Balance | `/v5/asset/transfer/query-account-coins-balance` | GET | accountType | memberId, coin, withBonus |
| Single Coin Balance | `/v5/asset/transfer/query-account-coin-balance` | GET | accountType, coin | memberId, toAccountType, toMemberId, withBonus, withTransferSafeAmount |
| Transferable Coins | `/v5/asset/transfer/query-transfer-coin-list` | GET | fromAccountType, toAccountType | — |
| Internal Transfer | `/v5/asset/transfer/inter-transfer` | POST | transferId, coin, amount, fromAccountType, toAccountType | — |
| Sub UID List | `/v5/asset/transfer/query-sub-member-list` | GET | — | — |
| Universal Transfer | `/v5/asset/transfer/universal-transfer` | POST | transferId, coin, amount, fromMemberId, toMemberId, fromAccountType, toAccountType | — |
| Universal Transfer Records | `/v5/asset/transfer/query-universal-transfer-list` | GET | — | transferId, coin, status, startTime, endTime, limit, cursor |
| Allowed Deposit List | `/v5/asset/deposit/query-allowed-list` | GET | — | coin, chain, cursor, limit |
| Set Deposit Account | `/v5/asset/deposit/deposit-to-account` | POST | accountType | — |
| Deposit Records | `/v5/asset/deposit/query-record` | GET | — | coin, startTime, endTime, limit, cursor |
| Sub Deposit Records | `/v5/asset/deposit/query-sub-member-record` | GET | subMemberId | coin, startTime, endTime, limit, cursor |
| Internal Deposit Records | `/v5/asset/deposit/query-internal-record` | GET | — | startTime, endTime, coin, cursor, limit |
| Master Deposit Address | `/v5/asset/deposit/query-address` | GET | coin | chainType |
| Sub Deposit Address | `/v5/asset/deposit/query-sub-member-address` | GET | coin, chainType, subMemberId | — |
| Coin Info | `/v5/asset/coin/query-info` | GET | — | coin |
| Withdrawal Records | `/v5/asset/withdraw/query-record` | GET | — | withdrawID, coin, withdrawType, startTime, endTime, limit, cursor |
| Withdrawable Amount | `/v5/asset/withdraw/withdrawable-amount` | GET | coin | — |
| Withdraw | `/v5/asset/withdraw/create` | POST | coin, chain, address, tag, amount, timestamp, forceChain, accountType | — |
| Cancel Withdrawal | `/v5/asset/withdraw/cancel` | POST | id | — |

### User (Authentication Required)

| Endpoint | Path | Method | Required | Optional |
|----------|------|--------|----------|----------|
| Create Sub Account | `/v5/user/create-sub-member` | POST | username, memberType | switch, isUta, note |
| Create Sub API Key | `/v5/user/create-sub-api` | POST | subuid, readOnly, permissions | note, ips |
| Get Sub UID List | `/v5/user/query-sub-members` | GET | — | — |
| Freeze/Unfreeze Sub | `/v5/user/frozen-sub-member` | POST | subuid, frozen | — |
| Get API Key Info | `/v5/user/query-api` | GET | — | — |
| Get Member Type | `/v5/user/get-member-type` | GET | — | — |
| Modify Master API Key | `/v5/user/update-api` | POST | — | readOnly, ips, permissions |
| Modify Sub API Key | `/v5/user/update-sub-api` | POST | apikey | readOnly, ips, permissions |
| Delete Master API Key | `/v5/user/delete-api` | POST | — | — |
| Delete Sub API Key | `/v5/user/delete-sub-api` | POST | apikey | — |
| Get Affiliate User Info | `/v5/user/aff-customer-info` | GET | uid | — |

### Spot Margin — Unified Account (Authentication Required)

| Endpoint | Path | Method | Required | Optional |
|----------|------|--------|----------|----------|
| Switch Margin Mode | `/v5/spot-margin-trade/switch-mode` | POST | spotMarginMode | — |
| Set Spot Leverage | `/v5/spot-margin-trade/set-leverage` | POST | leverage | — |
| Get VIP Margin Data | `/v5/spot-margin-trade/data` | GET | — | — |
| Interest Rate History | `/v5/spot-margin-trade/interest-rate-history` | GET | currency | startTime, endTime, vipLevel |
| Get Margin State | `/v5/spot-margin-trade/state` | GET | — | — |
| Cross Margin Loan Info | `/v5/spot-cross-margin-trade/loan-info` | GET | — | coin |
| Cross Margin Account | `/v5/spot-cross-margin-trade/account` | GET | — | — |
| Borrow | `/v5/spot-cross-margin-trade/loan` | POST | coin, qty | — |
| Repay | `/v5/spot-cross-margin-trade/repay` | POST | coin, qty | — |
| Cross Margin Switch | `/v5/spot-cross-margin-trade/switch` | POST | switch | — |

### Leverage Token (Authentication for Purchase/Redeem)

| Endpoint | Path | Method | Required | Optional | Auth |
|----------|------|--------|----------|----------|------|
| LT Info | `/v5/spot-lever-token/info` | GET | — | ltCoin | No |
| LT Market | `/v5/spot-lever-token/reference` | GET | ltCoin | — | No |
| Purchase | `/v5/spot-lever-token/purchase` | POST | ltCoin, ltAmount | serialNo | Yes |
| Redeem | `/v5/spot-lever-token/redeem` | POST | ltCoin, ltAmount | serialNo | Yes |
| Order Records | `/v5/spot-lever-token/order-record` | GET | — | ltCoin, orderId, startTime, endTime, limit, ltOrderType, serialNo | Yes |

### Broker (Authentication Required)

| Endpoint | Path | Method | Required | Optional |
|----------|------|--------|----------|----------|
| Earnings Info | `/v5/broker/earnings-info` | GET | — | bizType, startTime, endTime, limit, cursor |
| Account Info | `/v5/broker/account-info` | GET | — | — |
| Sub Deposit Records | `/v5/broker/asset/query-sub-member-deposit-record` | GET | — | subMemberId, coin, startTime, endTime, limit, cursor |
| Voucher Spec | `/v5/broker/award/info` | GET | awardId | — |
| Distribute Voucher | `/v5/broker/award/distribute-award` | POST | uid, awardId, amount, specCode | — |
| Voucher Distribution Records | `/v5/broker/award/distribution-record` | GET | — | awardId, startTime, endTime, limit, cursor |

### Earn (Authentication Required)

| Endpoint | Path | Method | Required | Optional |
|----------|------|--------|----------|----------|
| Product Info | `/v5/earn/product-info` | GET | — | category, coin |
| Stake / Redeem | `/v5/earn/create-order` | POST | productId, amount, orderType | serialNo, accountType |
| Staked Position | `/v5/earn/position` | GET | — | productId, coin, category |
| Order History | `/v5/earn/order-history` | GET | — | productId, orderId, startTime, endTime, limit, cursor |
| Yield History | `/v5/earn/yield-history` | GET | — | productId, coin, startTime, endTime, limit, cursor |

### Crypto Loan

| Endpoint | Path | Method | Required | Optional | Auth |
|----------|------|--------|----------|----------|------|
| Borrowable Coins | `/v5/crypto-loan/loan-coin` | GET | — | coin | No |
| Collateral Coins | `/v5/crypto-loan/collateral-coin` | GET | — | coin | No |
| Flexible Borrow | `/v5/crypto-loan/flexible/borrow` | POST | loanCoin | loanAmount, collateralCoin, collateralAmount | Yes |
| Flexible Repay | `/v5/crypto-loan/flexible/repay` | POST | orderId, amount | — | Yes |
| Flexible Loans | `/v5/crypto-loan/flexible/ongoing-orders` | GET | — | orderId, loanCoin, collateralCoin, limit, cursor | Yes |
| Adjust Collateral | `/v5/crypto-loan/adjust-collateral` | POST | orderId, amount, direction | — | Yes |
| Fixed Borrow | `/v5/crypto-loan/fixed/borrow` | POST | loanCoin, loanAmount, collateralCoin, termId | — | Yes |
| Fixed Repay | `/v5/crypto-loan/fixed/repay` | POST | orderId, amount | — | Yes |
| Max Loan Amount | `/v5/crypto-loan/max-loan-amount` | GET | loanCoin, collateralCoin | collateralAmount, loanAmount | Yes |
| LTV Adjust History | `/v5/crypto-loan/ltv-adjust-history` | GET | — | orderId, startTime, endTime, limit, cursor | Yes |

### Institutional Loan (Authentication Required)

| Endpoint | Path | Method | Required | Optional |
|----------|------|--------|----------|----------|
| Product Info | `/v5/ins-loan/product-infos` | GET | — | productId |
| Margin Coin Info | `/v5/ins-loan/ensure-tokens-convert` | GET | — | loanOrderId |
| Loan Orders | `/v5/ins-loan/loan-order` | GET | — | orderId, startTime, endTime, limit |
| Repay History | `/v5/ins-loan/repaid-history` | GET | — | startTime, endTime, limit |
| LTV | `/v5/ins-loan/ltv-convert` | GET | — | — |

### RFQ — Block Trading (Authentication Required, 50/s)

| Endpoint | Path | Method | Required | Optional |
|----------|------|--------|----------|----------|
| Create RFQ | `/v5/rfq/create-rfq` | POST | baseCoin, legs[] | rfqId, quoteExpiry |
| Cancel RFQ | `/v5/rfq/cancel-rfq` | POST | rfqId | — |
| Cancel All RFQs | `/v5/rfq/cancel-all-rfq` | POST | — | — |
| Create Quote | `/v5/rfq/create-quote` | POST | rfqId, legs[] | — |
| Execute Quote | `/v5/rfq/execute-quote` | POST | rfqId, quoteId | — |
| Cancel Quote | `/v5/rfq/cancel-quote` | POST | quoteId | — |
| Cancel All Quotes | `/v5/rfq/cancel-all-quotes` | POST | — | — |
| RFQ Realtime | `/v5/rfq/rfq-realtime` | GET | — | rfqId, baseCoin, side, limit |
| RFQ History | `/v5/rfq/rfq-list` | GET | — | rfqId, startTime, endTime, limit, cursor |
| Quote Realtime | `/v5/rfq/quote-realtime` | GET | — | quoteId, rfqId, baseCoin, limit |
| Quote History | `/v5/rfq/quote-list` | GET | — | quoteId, startTime, endTime, limit, cursor |
| Trade List | `/v5/rfq/trade-list` | GET | — | rfqId, startTime, endTime, limit, cursor |
| Public Trades | `/v5/rfq/public-trades` | GET | — | baseCoin, category, limit |
| Config | `/v5/rfq/config` | GET | — | — |

### Spread Trade (Authentication Required)

| Endpoint | Path | Method | Description |
|----------|------|--------|-------------|
| Order Endpoints | `/v5/spread/order/` | POST | Place, amend, cancel, query spread trade orders |
| Market Endpoints | `/v5/spread/market` | GET | Spread instrument market data |

---

## Parameters

### Common Parameters

* **category**: Product type — `spot`, `linear`, `inverse`, `option`
* **symbol**: Trading pair symbol, uppercase (e.g., `BTCUSDT`, `ETH-25DEC22-1400-C`)
* **side**: `Buy`, `Sell`
* **orderType**: `Market`, `Limit`
* **qty**: Order quantity (string)
* **price**: Order price (string, required for Limit orders)
* **timeInForce**: `GTC` (default), `IOC`, `FOK`, `PostOnly`, `RPI`
* **orderLinkId**: Custom order ID (max 36 chars, unique among open orders)
* **positionIdx**: `0` (one-way mode), `1` (hedge-mode buy side), `2` (hedge-mode sell side)
* **triggerPrice**: Conditional order trigger price (string)
* **triggerBy**: Trigger price type — `LastPrice`, `IndexPrice`, `MarkPrice`
* **reduceOnly**: `true` = reduce position only
* **closeOnTrigger**: `true` = close order behavior
* **isLeverage**: `0` (no borrow, default), `1` (margin trading, spot only)
* **orderFilter**: Spot only — `Order` (default), `tpslOrder`, `StopOrder`
* **marketUnit**: Spot market orders — `baseCoin`, `quoteCoin`. **Tip:** For spot market Buy orders, prefer `marketUnit=quoteCoin` with qty as the USDT amount (e.g., `"qty":"7000","marketUnit":"quoteCoin"`) for more reliable execution. Using `baseCoin` qty may hit "Order value exceeded lower limit" on some environments.

### TP/SL Parameters

Used on `/v5/order/create`, `/v5/order/amend`, and `/v5/position/trading-stop`:

* **takeProfit**: Take profit price (pass `"0"` to cancel)
* **stopLoss**: Stop loss price (pass `"0"` to cancel)
* **tpTriggerBy**: TP trigger price type — `LastPrice` (default), `IndexPrice`, `MarkPrice`
* **slTriggerBy**: SL trigger price type — `LastPrice` (default), `IndexPrice`, `MarkPrice`
* **tpslMode**: `Full` (entire position), `Partial` (specify size)
* **tpSize**: TP quantity for partial mode
* **slSize**: SL quantity for partial mode
* **tpLimitPrice**: Limit price when TP is triggered (partial mode, Limit order)
* **slLimitPrice**: Limit price when SL is triggered (partial mode, Limit order)
* **tpOrderType**: `Market` (default), `Limit`
* **slOrderType**: `Market` (default), `Limit`
* **trailingStop**: Trailing stop distance (pass `"0"` to cancel, position only)
* **activePrice**: Trailing stop activation price (position only)

### Enums

* **category**: `spot` | `linear` | `inverse` | `option`
* **side**: `Buy` | `Sell`
* **orderType**: `Market` | `Limit`
* **timeInForce**: `GTC` | `IOC` | `FOK` | `PostOnly` | `RPI`
* **triggerBy**: `LastPrice` | `IndexPrice` | `MarkPrice`
* **positionIdx**: `0` (one-way) | `1` (hedge-buy) | `2` (hedge-sell)
* **accountType**: `UNIFIED` | `FUND`
* **tpslMode**: `Full` | `Partial`
* **tradeMode**: `0` (cross margin) | `1` (isolated margin)
* **positionMode**: `0` (MergedSingle / one-way) | `3` (BothSide / hedge)
* **orderStatus (open)**: `New` | `PartiallyFilled` | `Untriggered`
* **orderStatus (closed)**: `Rejected` | `PartiallyFilledCanceled` | `Filled` | `Cancelled` | `Triggered` | `Deactivated`
* **execType**: `Trade` | `AdlTrade` | `Funding` | `BustTrade` | `Delivery` | `Settle` | `BlockTrade` | `MovePosition`
* **stopOrderType**: `TakeProfit` | `StopLoss` | `TrailingStop` | `Stop` | `PartialTakeProfit` | `PartialStopLoss` | `tpslOrder` | `OcoOrder`
* **cancelType**: `CancelByUser` | `CancelByReduceOnly` | `CancelByPrepareLiq` | `CancelByPrepareAdl` | `CancelByAdmin` | `CancelBySettle` | `CancelByTpSlTsClear` | `CancelBySmp` | `CancelByDCP`
* **interval** (kline): `1` | `3` | `5` | `15` | `30` | `60` | `120` | `240` | `360` | `720` | `D` | `W` | `M`
* **intervalTime** (open interest): `5min` | `15min` | `30min` | `1h` | `4h` | `1d`
* **period** (long/short ratio): `5min` | `15min` | `30min` | `1h` | `4h` | `1d`
* **setMarginMode**: `ISOLATED_MARGIN` | `REGULAR_MARGIN` | `PORTFOLIO_MARGIN`
* **collateralSwitch**: `ON` | `OFF`
* **autoAddMargin**: `0` (off) | `1` (on)
* **frozen** (sub account): `0` (unfreeze) | `1` (freeze)
* **memberType** (sub account): `1` (normal) | `6` (custodial)
* **spotMarginMode**: `0` (off) | `1` (on)
* **direction** (collateral adjust): `ADD` | `REDUCE`
* **earnOrderType**: `Stake` | `Redeem`

---

## WebSocket Reference

For agents that need real-time push notifications (e.g., always-on assistants).

### Public Streams

**URL:** `wss://stream.bybit.com/v5/public/{category}`
**Testnet:** `wss://stream-testnet.bybit.com/v5/public/{category}`

Where `{category}` = `spot`, `linear`, `inverse`, `option`

| Topic | Format | Description |
|-------|--------|-------------|
| Orderbook | `orderbook.{depth}.{symbol}` | Depth: 1, 50, 200, 500 |
| Public Trade | `publicTrade.{symbol}` | Real-time trades |
| Tickers | `tickers.{symbol}` | Ticker updates |
| Kline | `kline.{interval}.{symbol}` | Candlestick updates |
| Liquidation | `liquidation.{symbol}` | Liquidation events |
| LT Kline | `kline_lt.{interval}.{symbol}` | Leveraged token kline |
| LT Ticker | `tickers_lt.{symbol}` | Leveraged token ticker |
| LT NAV | `lt.{symbol}` | Leveraged token NAV |

### Private Streams

**URL:** `wss://stream.bybit.com/v5/private`
**Testnet:** `wss://stream-testnet.bybit.com/v5/private`

| Topic | Description |
|-------|-------------|
| `position` | Position updates |
| `execution` | Trade execution updates |
| `execution.fast` | Fast execution updates (lower latency) |
| `order` | Order status updates |
| `wallet` | Wallet balance changes |
| `greeks` | Option greeks updates |
| `dcp` | Disconnect cancel-all protection |

### Subscription Format

```json
{"op": "subscribe", "args": ["orderbook.50.BTCUSDT", "publicTrade.BTCUSDT"]}
```

**Unsubscribe:**
```json
{"op": "unsubscribe", "args": ["orderbook.50.BTCUSDT"]}
```

**Heartbeat:** Send `{"op": "ping"}` every 20 seconds.

### Private Stream Authentication

```json
{"op": "auth", "args": ["<apiKey>", "<expires>", "<signature>"]}
```

Where:
- `expires` = future timestamp in milliseconds
- `signature` = HMAC-SHA256 of `GET/realtime{expires}` using secret key

---

## Security

### Share Credentials

Users can provide Bybit API credentials by sending a file:

```
abc123...xyz
secret123...key
```

### Never Display Full Secrets

When showing credentials to users:
- **API Key:** Show first 5 + last 4 characters: `su1Qc...8akf`
- **Secret Key:** Always mask, show only last 5: `***...aws1`

Example:
```
Account: main
API Key: su1Qc...8akf
Secret: ***...aws1
Environment: Mainnet
Region: Global
```

### Listing Accounts

Show names, environment, and region only — never keys:
```
Bybit Accounts:
* main (Mainnet, Global)
* testnet-dev (Testnet)
* uae-trading (Mainnet, UAE)
```

### Transactions on Mainnet

When performing transactions on mainnet (any POST to trading/position endpoints), always confirm with the user before proceeding by asking them to write **"CONFIRM"** to proceed. Testnet does not require confirmation.

---

## Bybit Accounts

Accounts are stored in [`TOOLS.md`](./TOOLS.md), NOT in this file. See TOOLS.md for actual credentials.

### TOOLS.md Structure

```
## Bybit Accounts

### account-name
- API Key: abc123...xyz
- Secret: secret123...key
- Testnet: false
- Region: uae
- Description: UAE trading account
```

Valid Region values: `global`, `netherlands`, `turkey`, `kazakhstan`, `georgia`, `uae`, `eea`, `indonesia`

### Adding New Accounts

When user provides new credentials:
1. Ask for account name
2. Ask: Mainnet or Testnet
3. Ask: Region (or default to global)
4. Store in `TOOLS.md` with masked display confirmation

---

## Agent Behavior

1. **Category confirmation:** When a symbol like BTCUSDT can exist in multiple categories (spot, linear, inverse), the agent MUST ask the user whether they want spot (现货) or perpetual/futures (合约) before placing orders. Never assume a category.
2. **Credentials requested:** Mask secrets (API Key: first 5 + last 4, Secret: last 5 only)
3. **Listing accounts:** Show names, environment, and region — never keys
4. **Account selection:** Ask if ambiguous, default to `main`
5. **Mainnet transactions:** Require user to type "CONFIRM" before executing
6. **New credentials:** Ask for account name, environment (Mainnet/Testnet), region, then store in `TOOLS.md`
7. **Regional URL:** Auto-select based on account Region field. Testnet always uses `api-testnet.bybit.com`
8. **Response handling:** Check `retCode` — `0` means success, non-zero is an error. Show `retMsg` on errors
9. **POST requests:** Always use `Content-Type: application/json` with JSON body
10. **GET requests:** Parameters as query string
11. **User-Agent:** Include `User-Agent: bybit-v5/1.0.0 (Skill)` on all requests
12. **Spot Market Buy:** Prefer `marketUnit=quoteCoin` with USDT amount for spot market buy orders (more reliable than baseCoin qty)
13. **Hedge Mode detection:** If a linear/inverse order returns `retCode=10001` "position idx not match position mode", the account is in hedge mode. Retry with `positionIdx`: `1` for Buy/Long, `2` for Sell/Short. Agent should remember and auto-apply for subsequent orders.
