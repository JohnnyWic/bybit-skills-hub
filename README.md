# Bybit AI Skills

Trade Bybit with natural language — covering **spot**, **linear perpetuals**, **inverse contracts**, and **options** through ~145 API endpoints.

## What is a Skill?

A Skill is a structured markdown file that teaches AI agents (Claude, OpenClaw, etc.) how to interact with external APIs. The agent reads the skill definition and uses it to make authenticated API calls on your behalf via `curl`.

## Repository Structure

```
bybit-skills-hub/
├── README.md
├── skills/
│   └── bybit/
│       └── v5/
│           └── SKILL.md    ← Skill definition (~145 endpoints)
└── TOOLS.md                ← Your API keys (local only, gitignored)
```

## Quick Start

### 1. Install the Skill

**Option A — One-command install:**

```bash
npx skills add https://github.com/JohnnyWic/bybit-skills-hub --skill bybit-v5
```

This copies `SKILL.md` into your project's skill directory automatically.

**Option B — Manual (Claude Desktop, Cursor, any agent):**

1. Download [`SKILL.md`](skills/bybit/v5/SKILL.md) from this repo
2. Place it where your AI agent reads skill files:

| Agent | Location |
|-------|----------|
| Claude Code | `<project>/.claude/skills/bybit-v5/SKILL.md` |
| Claude Desktop | Add the file path to your Project Knowledge |
| Cursor | Add to `.cursor/rules/` or reference in system prompt |
| Other agents | Any path the agent can read — then tell it to read the file |

### 2. Configure Credentials

Create a `TOOLS.md` file **in the same directory as SKILL.md** with your Bybit API keys:

```markdown
## Bybit Accounts

### main
- API Key: your_api_key_here
- Secret: your_api_secret_here
- Testnet: false
- Region: global
- Description: Primary trading account
```

> **Tip:** You can also just tell the agent your credentials — it will create `TOOLS.md` for you automatically.

### 3. Start Trading

Ask the agent:

```
> Check BTCUSDT price
> Place a market buy 0.1 ETH on linear
> Show my wallet balance
> Set 10x leverage on BTCUSDT
```

## Supported Endpoints (~145)

| Module | Endpoints | Auth | Description |
|--------|-----------|------|-------------|
| Market | 17 | No | Klines, orderbook, tickers, funding rates, instruments info |
| Trade | 11 | Yes | Place/amend/cancel orders, batch operations, TP/SL on orders |
| Position | 14 | Yes | Positions, leverage, margin mode, trading stop (TP/SL), PnL |
| Account | 13 | Yes | Wallet balance, fees, collateral, margin mode, MMP |
| Asset | 24 | Yes | Transfers, deposits, withdrawals, balances |
| User | 11 | Yes | Sub accounts, API key management |
| Spot Margin | 10 | Yes | Margin trading, borrow/repay |
| Leverage Token | 5 | Mixed | LT info, purchase, redeem |
| Broker | 6 | Yes | Earnings, vouchers |
| Earn | 5 | Yes | Staking, yield |
| Crypto Loan | 10 | Mixed | Flexible/fixed loans, collateral |
| RFQ | 14 | Yes | Block trading |
| Spread Trade | 2 | Yes | Spread instruments |
| Institutional Loan | 5 | Yes | Institutional lending |

## Key Features

### Unified Category System

All endpoints use a `category` parameter: `spot`, `linear`, `inverse`, `option`.

```bash
# Same endpoint, different products
GET /v5/market/tickers?category=spot&symbol=BTCUSDT
GET /v5/market/tickers?category=linear&symbol=BTCUSDT
```

### Native TP/SL Support

Set take-profit and stop-loss directly on orders or existing positions:

```json
{
  "category": "linear",
  "symbol": "BTCUSDT",
  "side": "Buy",
  "orderType": "Limit",
  "qty": "0.1",
  "price": "50000",
  "takeProfit": "55000",
  "stopLoss": "48000",
  "tpslMode": "Partial",
  "tpSize": "0.05"
}
```

### Batch Operations

Place, amend, or cancel up to 20 orders in a single request:

```
POST /v5/order/create-batch
POST /v5/order/amend-batch
POST /v5/order/cancel-batch
```

### Regional URL Auto-Selection

Configure your account region and the agent auto-selects the optimal endpoint:

| Region | URL |
|--------|-----|
| Global | `api.bybit.com` |
| Netherlands | `api.bybit.nl` |
| Turkey | `api.bybit-tr.com` |
| Kazakhstan | `api.bybit.kz` |
| Georgia | `api.bybitgeorgia.ge` |
| UAE | `api.bybit.ae` |
| EEA | `api.bybit.eu` |
| Indonesia | `api.bybit.id` |
| Testnet | `api-testnet.bybit.com` |

### WebSocket Reference

The skill includes WebSocket stream documentation for agents that need real-time push notifications:

- **Public:** orderbook, trades, tickers, klines, liquidations
- **Private:** positions, executions, orders, wallet, greeks

## Authentication

Header-based HMAC-SHA256 signing. The agent handles this automatically:

```
X-BAPI-API-KEY      → Your API key
X-BAPI-TIMESTAMP    → UTC timestamp (ms)
X-BAPI-SIGN         → HMAC-SHA256 signature
X-BAPI-RECV-WINDOW  → Request validity window
```

Signing formula:
- **GET:** `HMAC(timestamp + apiKey + recvWindow + queryString, secret)`
- **POST:** `HMAC(timestamp + apiKey + recvWindow + jsonBody, secret)`

## Security

- API keys are stored in `TOOLS.md` (separate from skill definition)
- Keys are never displayed in full — masked as `su1Qc...8akf` / `***...aws1`
- Mainnet transactions require explicit "CONFIRM" from user
- Testnet transactions proceed without confirmation
- `TOOLS.md` should be gitignored in production

## Agent Behavior Highlights

- Asks **spot or futures** when a symbol exists in multiple categories
- Auto-detects **hedge mode** and retries with correct `positionIdx`
- Prefers `marketUnit=quoteCoin` for spot market buy orders
- Masks credentials in all displays
- Confirms before mainnet transactions

## License

MIT
