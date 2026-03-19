<p align="center">
  <img src="https://riskstate.ai/logo-r-grey.svg" width="48" alt="RiskState" />
</p>

<h1 align="center">RiskState</h1>

<p align="center">
  <strong>Risk governance API for autonomous crypto trading agents</strong>
</p>

<p align="center">
  <a href="https://riskstate.ai/v1/risk-state"><img src="https://img.shields.io/badge/API-v1.2.0-blue?style=flat-square" alt="API Version" /></a>
  <a href="https://riskstate.ai"><img src="https://img.shields.io/badge/status-beta-green?style=flat-square" alt="Status" /></a>
  <a href="#supported-assets"><img src="https://img.shields.io/badge/assets-BTC%20%7C%20ETH-orange?style=flat-square" alt="Assets" /></a>
  <a href="#pricing"><img src="https://img.shields.io/badge/pricing-free%20beta-brightgreen?style=flat-square" alt="Pricing" /></a>
</p>

<p align="center">
  <a href="https://riskstate.ai">Website</a> · <a href="docs/api-v1.md">API Reference</a> · <a href="SKILL.md">SKILL.md</a> · <a href="https://x.com/riskstate_ai">X/Twitter</a>
</p>

---

## What is RiskState?

A deterministic policy engine that tells AI trading agents **how much risk is allowed** — not what to trade.

One API call returns position limits, allowed actions, and policy constraints computed from **30+ real-time signals** across macro, on-chain, derivatives, and DeFi health.

```bash
curl -X POST https://riskstate.ai/v1/risk-state \
  -H "Authorization: Bearer $RISKSTATE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"asset": "BTC"}'
```

```json
{
  "exposure_policy": {
    "max_size_fraction": 0.42,
    "leverage_allowed": true,
    "allowed_actions": ["DCA", "LONG_SHORT_CONFIRMED"],
    "blocked_actions": ["ALL_IN", "LEVERAGE_GT_2X"]
  },
  "policy_level": 4,
  "risk_flags": {
    "structural_blockers": [],
    "context_risks": ["HIGH_COUPLING"]
  },
  "binding_constraint": {
    "source": "MACRO",
    "reason_codes": ["MACRO_NEUTRAL", "COUPLING_NORMAL"]
  }
}
```

Your agent reads `max_size_fraction`, checks `structural_blockers`, and acts. No parsing. No interpretation.

## Why?

Autonomous trading agents can execute trades. None of them know when to **stop**.

| Without governance | With RiskState |
|---|---|
| Agent sizes position based on signal confidence | Agent caps position at `max_size_fraction` |
| No awareness of macro regime | `RISK-OFF` → `blocked_actions: ["AGGRESSIVE_LONG"]` |
| DeFi health factor ignored | Wallet health feeds directly into position limit |
| Leverage unbounded | `max_leverage: "1x"` enforced per policy level |
| No circuit breaker | `structural_blockers` non-empty → halt |

## Quick Start

### 1. Get an API key

Sign up at [riskstate.ai](https://riskstate.ai) — email only, free during beta.

### 2. Query the API

```bash
curl -X POST https://riskstate.ai/v1/risk-state \
  -H "Authorization: Bearer $RISKSTATE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"asset": "BTC"}'
```

### 3. Enforce in your agent

```python
import requests

policy = requests.post(
    "https://riskstate.ai/v1/risk-state",
    headers={"Authorization": f"Bearer {API_KEY}"},
    json={"asset": "BTC"}
).json()

# Hard stop: structural blockers
if policy["risk_flags"]["structural_blockers"]:
    return  # Do not trade

# Size cap
max_size = policy["exposure_policy"]["max_size_fraction"]
position_size = min(desired_size, portfolio_value * max_size)

# Action filter
if "LEVERAGE" in policy["exposure_policy"]["blocked_actions"]:
    leverage = 1.0
```

## Policy Levels

| Level | Label | Max Size | What your agent can do |
|-------|-------|----------|----------------------|
| 1 | BLOCK Survival | <15% | Reduce exposure, hedge only |
| 2 | BLOCK Defensive | <35% | Wait, hedge, small scalps |
| 3 | CAUTIOUS | <60% | DCA, R:R >2:1 only |
| 4 | GREEN Selective | <80% | Trade with confirmation |
| 5 | GREEN Expansion | ≥80% | Full operations, leverage up to 2x |

## Supported Assets

- **BTC** — Full signal coverage (30+ indicators)
- **ETH** — Full coverage including ETH structural score, ETH/BTC ratio analysis, staking dynamics

## Integration Paths

### REST API
Direct HTTP calls. Any language, any framework.

### SKILL.md
Drop [`SKILL.md`](SKILL.md) into your agent's repo. Compatible with Claude Code, Copilot, Cursor, and Gemini via [skills.sh](https://skills.sh).

### MCP Server
Coming soon.

## Binding Precedence

When consuming the response, agents **must** evaluate fields in this order:

1. `risk_flags.structural_blockers` — if non-empty, **abort** new entries
2. `exposure_policy.blocked_actions` — actions the agent must not take
3. `exposure_policy.reduce_recommended` — reduce exposure if `true`
4. `exposure_policy.max_size_fraction` — maximum position size (0.0–1.0)
5. `exposure_policy.max_leverage` — maximum leverage allowed

## Data Sources

RiskState ingests 30+ real-time signals from:

- **Price & Derivatives** — Binance, OKX, Bybit (funding, OI, basis, L/S ratio)
- **On-chain** — MVRV, NUPL, exchange netflow, supply metrics (CoinGlass)
- **Macro** — DXY, US yields, S&P 500, Gold, Fed balance sheet (FRED, Yahoo Finance)
- **DeFi** — Spark Protocol, Aave V3 health factor and liquidation thresholds
- **Sentiment** — Fear & Greed, ETF flows, institutional treasuries
- **ETH-specific** — Staking ratio, burn rate, DEX volume, fees, stablecoin TVL

## Pricing

**Free during beta.** Rate limit: 60 requests/minute.

| Tier | Calls/month | Price |
|------|------------|-------|
| Free | 100 | $0 |
| Builder | 5,000 | $49/mo |
| Growth | 25,000 | $149/mo |
| Scale | 100,000 | $399/mo |

Paid tiers coming after beta. [Sign up now](https://riskstate.ai) to lock in early access.

## Documentation

- [**API Reference**](docs/api-v1.md) — Full endpoint documentation, field types, error codes
- [**SKILL.md**](SKILL.md) — Agent discovery file with decision rules and failure modes
- [**Website**](https://riskstate.ai) — Landing page with interactive examples

## Links

- Website: [riskstate.ai](https://riskstate.ai)
- X/Twitter: [@riskstate_ai](https://x.com/riskstate_ai)
- API: `POST https://riskstate.ai/v1/risk-state`

---

<p align="center">
  <sub>Built by <a href="https://github.com/likidodefi">likidodefi</a> · © 2026 Digital Venture Asset LLC</sub>
</p>
