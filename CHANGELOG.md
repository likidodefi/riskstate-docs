# Changelog

All notable changes to the RiskState API will be documented in this file.

## [1.2.0] - 2026-03-19

### Added
- **Usage tracking** — Monthly and total API call counts per key, visible in admin panel
- **Bybit V5 + OKX V5 fallback chain** — Funding rate and OI now have 4-level fallback: Binance → OKX → Bybit → CoinGlass → default. 98%+ uptime for positioning data
- **DXY in API** — Frankfurter EUR/USD proxy added to API (was missing, affected regime classification)
- **Aave V3 support** — DeFi position monitoring now supports both Spark Protocol and Aave V3 with per-collateral liquidation thresholds

### Fixed
- **BTC cycle phase alignment** — API now replicates exact dashboard 9-branch priority order (was simplified 6-branch, causing POST-PEAK vs CORRECTION divergence)
- **Regime classification fix** — Missing DXY caused false BEAR regime in API (DXY defaulted to 100 → extra bearSignal)
- **ETH structural score** — Now fetches real data (was hardcoded null → default 50). Lido staking, burn rate, DEX volume, fees, stablecoin TVL all live
- **Macro regime alignment** — API now calls macro.js as single source of truth (was divergent inline computation)

## [1.1.1] - 2026-03-16

### Added
- **English standardization** — All API responses in English (was mixed Spanish/English)
- **ETH Structural Score v2.2** — Widened issuance bands, lending heat directional scoring, hard/soft downgrade reclassification
- **Weighted warnings in rules cap** — Structural warnings (slow-moving) penalize 0.5x vs tactical 1.0x

### Fixed
- **False BLOCK prevention** — Mild ETH inflation (+0.3%/yr) no longer triggers defensive policy
- **CoinGlass fallback fixes** — OI, L/S ratio, MVRV fallback chains corrected (wrong field names, unused data)

## [1.1.0] - 2026-03-13

### Added
- **Deterministic API contract** — `allowed_actions` and `blocked_actions` use uppercase enum tokens (DCA, WAIT, LEVERAGE_GT_2X) instead of free-text
- **`reason_codes`** — Machine-parseable tokens in `binding_constraint` (e.g., MACRO_RISK_OFF, COUPLING_NORMAL)
- **`ttl_seconds`** — Cache TTL in response (60s) for agent scheduling
- **`binding_constraint.source` uppercased** — RULES, DEFI, MACRO, CYCLE (consistent enum style)

### Changed
- **SKILL.md rewritten** — Binding precedence section, decision rules table, failure modes table, updated example response

## [1.0.0] - 2026-03-12

### Added
- **Initial release** — `POST /v1/risk-state` endpoint
- **5-level policy engine** — BLOCK (1-2) → CAUTIOUS (3) → GREEN (4-5)
- **Multi-asset support** — BTC and ETH with asset-specific scoring
- **4-cap system** — Rules, DeFi, Macro, Cycle caps × quality × volatility adjustment
- **30+ real-time signals** — Macro, on-chain, derivatives, DeFi health, sentiment
- **DeFi-aware** — Optional wallet parameter for Spark/Aave V3 health factor integration
- **SHA-256 policy hash** — Deterministic audit trail for non-repudiation
- **Hierarchical risk flags** — `structural_blockers` (hard stop) vs `context_risks` (reduce conviction)
- **60s Blob cache** — Skip cache when wallet parameter provided
- **Bearer auth** — Fail-closed authentication
- **SKILL.md** — Agent discovery file for skills.sh and agentskills.io ecosystems
- **Full API documentation** — docs/api-v1.md with field types, error codes, interpretation guide
