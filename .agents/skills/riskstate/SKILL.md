---
name: riskstate
description: Deterministic risk governance API for crypto trading agents. Returns position size limits, allowed/blocked actions, and policy constraints computed from 30+ real-time market signals. Use when an agent needs to check how much risk is allowed before opening, sizing, or managing crypto positions (BTC, ETH).
license: Proprietary
compatibility: Requires network access to riskstate.netlify.app. Works with any agent that can make HTTP POST requests.
metadata:
  author: RiskState
  version: "1.1.1"
  homepage: "https://riskstate.ai"
---

# RiskState — Risk Governor for Crypto Trading Agents

## What it does

Returns **operational risk permissions** for crypto trading. A deterministic policy engine computes how much exposure is allowed based on 30+ real-time signals across macro, on-chain, derivatives, and DeFi health.

The response tells you:
- **max_size_fraction**: Maximum position size as fraction of portfolio (0.0-1.0)
- **allowed_actions / blocked_actions**: What the agent MAY and MUST NOT do
- **risk_flags**: Structural blockers (hard stop) vs contextual risks (reduce conviction)
- **binding_constraint**: Which cap is limiting and why

## What it does NOT do

- No trade signals, no entry/exit prices, no predictions
- No order execution or routing

This is a **risk governor**, not a trading oracle.

## When to call

- **Before opening or sizing positions** — check permissions first
- **Periodically during holds** — every 5 min for active trading, every 4h for holding
- **After significant market moves** — cache invalidates after 60s

## Authentication

Request a free API key at https://riskstate.ai (waitlist, email only).

```
Authorization: Bearer <your_api_key>
```

## Quick start

### Minimal request (BTC)

```bash
curl -X POST https://riskstate.netlify.app/v1/risk-state \
  -H "Authorization: Bearer $RISKSTATE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"asset": "BTC"}'
```

### With scoring breakdown

```bash
curl -X POST https://riskstate.netlify.app/v1/risk-state \
  -H "Authorization: Bearer $RISKSTATE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"asset": "ETH", "include_details": true}'
```

## Binding precedence

Agents MUST evaluate response fields in this order:

1. `risk_flags.structural_blockers` — if non-empty, ABORT new entries
2. `exposure_policy.blocked_actions` — actions the agent MUST NOT take
3. `exposure_policy.reduce_recommended` — reduce exposure if true
4. `exposure_policy.max_size_fraction` — maximum position size
5. `exposure_policy.max_leverage` — maximum leverage allowed

## Policy levels

| Level | Label | Key constraints |
|-------|-------|-----------------|
| 1 | BLOCK Survival | No new positions, max leverage 0x |
| 2 | BLOCK Defensive | Wait or hedge only, max leverage 1x |
| 3 | CAUTIOUS | DCA with R:R >2:1, max leverage 1x |
| 4 | GREEN Selective | Trade with confirmation, max leverage 1.5x |
| 5 | GREEN Expansion | Full operations, max leverage 2x |

## Failure modes

| Condition | Agent behavior |
|-----------|----------------|
| `stale_fields` contains core indicators | Downgrade conviction |
| `data_quality_score` < 50 | Do not open new positions |
| HTTP 500 or timeout | Assume BLOCK. Retry after 60s |

## Example response

```json
{
  "exposure_policy": {
    "max_size_fraction": 0.42,
    "leverage_allowed": false,
    "max_leverage": "1x",
    "direction_bias": "LONG_PREFERRED",
    "reduce_recommended": false,
    "allowed_actions": ["DCA", "WAIT", "LIGHT_ACCUMULATION", "RR_GT_2"],
    "blocked_actions": ["LEVERAGE", "AGGRESSIVE_LONG", "ALL_IN"]
  },
  "policy_level": 3,
  "confidence_score": 0.72,
  "data_quality_score": 85,
  "risk_flags": {
    "structural_blockers": [],
    "context_risks": ["HIGH_FUNDING"]
  },
  "asset": "BTC",
  "ttl_seconds": 60,
  "timestamp": "2026-03-13T14:30:00.000Z"
}
```

See [references/api-v1.md](references/api-v1.md) for full field documentation.
