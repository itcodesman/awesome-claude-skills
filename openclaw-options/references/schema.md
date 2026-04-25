# OpenClaw Scan Data Schema Reference

This document defines the structure of `LLM_YYYYMMDD_HHMM.txt` scan output files, supporting versions v2.X, v3.1.X, and v4.1.

## Metadata Section (m object)

| Field | Type | Required | Purpose | Notes |
|---|---|---|---|---|
| `m.ver` | string | Yes | Version identifier | "v4.1", "v3.1.X", "v2.X" |
| `m.ts` | ISO-8601 | Yes | Scan timestamp | UTC, e.g. "2026-04-25T14:30:00Z" |
| `m.scope` | enum | Yes | Scan coverage | "20tickers", "preferred", "full" |
| `m.market_state` | string | Yes | Current market condition | "A", "B", "C", "D" (per OpenClaw definition) |
| `m.gates_ok` | boolean | Yes | Pre-screen Gate overall pass | true/false |
| `m.gate_score` | number | 0-100 | Composite gate score | Higher = more risk |
| `m.margin` | object | No | PM account margin data | See margin object below |
| `m.ds_health` | object | No | Data source health | Only in v4.1+, see data health object |
| `m.fomc_d` / `m.boj_d` / `m.cpi_d` / `m.nfp_d` | string | Optional | Days to key macro events | Format: "2026-05-15" |

### Margin Object (m.margin)

| Field | Type | Unit | Meaning |
|---|---|---|---|
| `nlv` | number | USD | Net Liquidation Value |
| `req_maint` | number | USD | Maintenance requirement (TIMS) |
| `excess_liq` | number | USD | Excess Liquidity |
| `maint_ratio` | number | ratio | req_maint / nlv (0.0-1.0) |
| `bp_usage_pct` | number | percent | Buying power used (0-100) |

### Data Health Object (m.ds_health, v4.1+)

| Field | Meaning |
|---|---|
| `fallback_used` | true = price source downgrade (e.g., snapshot vs live), Greeks reliability decreased |
| `iv_freshness_min` | Minutes since last IV update across tickers (warn if >60) |
| `hv_lookback_complete` | true = full HV window available, false = truncated data |

---

## Signals Array (sig[])

Each element represents one potential trading opportunity.

```json
{
  "ticker": "AAPL",
  "tier": "A",
  "strike": 180.0,
  "expiry": "2026-05-16",
  "dte": 35,
  "premium": 4.50,
  "delta_sell": -0.30,
  "iv_rank": 65,
  "ivs_30": 0.28,
  "hv_20": 0.22,
  "off": true,
  "reasons": [
    "IV Rank 65 > threshold 50",
    "Delta -0.30 in recommended range [-0.35, -0.20]",
    "DTE 35 optimal opening window"
  ]
}
```

### Signal Fields Reference

| Field | Type | Meaning | Notes |
|---|---|---|---|
| `ticker` | string | Stock symbol | Must exist in routing table |
| `tier` | string | Risk tier | "A+", "A", "B", "C", "D" |
| `strike` | number | Strike price in USD | |
| `expiry` | date | Expiration YYYY-MM-DD | Usually 3rd Friday of month |
| `dte` | number | Days to expiration | Integer |
| `premium` | number | Sell Put premium per contract | Latest quote |
| `delta_sell` | number | Delta for short Put | Negative value (−0.50 to −0.15 typical) |
| `iv_rank` | number | IV Rank 0-100 | Percentile of historical IV |
| `ivs_30` | number | 30-day IV (annualized) | Decimal (0.25 = 25% IV) |
| `hv_20` | number | 20-day Historical Vol | Decimal |
| `off` | boolean | Official ticker? | true = in tickers-20pool or tickers-preferred |
| `reasons` | array | Why signal generated | List of passing conditions |

---

## Abnormal Input Handling Checklist

| Scenario | Claude Action |
|---|---|
| `m.ver` unknown | Assume v3.1.X compatibility, output notice |
| `m.scope` mismatch with sig content (e.g., scope=20tickers but NVDA present) | Warn user: ticker outside declared scope, treat as extension pool |
| `m.gates_ok=false` but gate_score ambiguous | Query which specific gate(s) failed, soft-pass if gate_score < 50 |
| `m.margin` missing entirely | Soft-pass Gate-4, warn: "Margin not reported, verify on IBKR before execution" |
| `m.ds_health.fallback_used=true` | Flag all Greeks with notice: "Data source downgraded, Delta/IV reliability reduced" |
| Empty `sig[]` array | Output: "No signals pass gate thresholds today. Market state: {state}, VIX: {vix}, IVR: {ivr}." |
| Multiple expiry dates in sig[] | OK, signal across different months allowed; note concentration risk if >80% in one month |

---

## Version Compatibility

- **v4.1**: Full schema, includes `m.ds_health`
- **v3.1.X**: No `m.ds_health`, otherwise compatible
- **v2.X**: Minimal fields, assume defaults for missing values

