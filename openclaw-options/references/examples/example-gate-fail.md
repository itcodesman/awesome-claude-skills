# Example · Gate 失败日（VIX 闸门触发，零信号）

> **目的**：演示 Pre-screen Gate 硬阻断时 Claude 的输出规范——**不推任何信号、不做妥协**。
> **场景**：市场 C 级 · VIX=37.4（> 35 硬红线）· 期限倒挂 · PM 账户尚未触及警戒。

---

## 输入样例（精简版 `LLM_20260422_1530.txt`）

```json
{
  "m": {
    "ts": "2026-04-22 15:30:00",
    "ver": "v4.1",
    "vix": 37.4,
    "vix3m": 29.8,
    "term": "INVERTED",
    "sp_ma200": false,
    "sp_dd20": 8.5,
    "gates_ok": false,
    "gate_score": 15,
    "fomc_d": 4,
    "cpi_d": 2,
    "scope": "20tickers",
    "off_cnt": 20,
    "sig_cnt": "0/20",
    "hv_proxy_count": 0,
    "margin": {
      "account_type": "PM",
      "nlv": 6438052,
      "maint": 1287610,
      "excess": 4150441,
      "used_pct": 20.0
    }
  },
  "sig": [],
  "skip": { "count": 20, "nt": "全部标的因 Gate 失败跳过" }
}
```

---

## Claude 期望输出

### 本轮异常

⚠️ **Pre-screen Gate 失败**：`gates_ok=false`，`gate_score=15`（<50 硬红线）。VIX=37.4 > 35（Gate-2 终止）+ VIX > VIX3M（期限倒挂）+ SPX < 200MA + CPI 距今 2 天。

### 本轮扫描概览

- 扫描时间：2026-04-22 15:30 SGT · 脚本 v4.1 · 范围 `20tickers`
- **Gate 状态**：失败，本轮不推送任何新开仓信号
- PM 账户：维持保证金 20.0%（刚触警戒边界）· 剩余流动性 64.5%（仍绿）
- 宏观：距 FOMC 4 天 / CPI **2 天**（极近）

### 决策

❌ **本轮零开仓建议**

**失败原因（按严重度排序）**：

1. `VIX=37.4 > 35` — PM-3 规则进入 Gate-2 终止区间，且 VIX > 40 时还需主动平仓至敞口 ≤ NLV × 0.5（本轮尚未到此）。
2. `term=INVERTED` — 期限倒挂，即便通过 Gate 也需 per-tier delta 收紧 +0.02。
3. `sp_ma200=false` + `sp_dd20=8.5%` — 市场 C/D 特征明显。
4. `cpi_d=2` — 距 CPI 仅 2 天，按宏观窗口已足以阻断 SPY/QQQ 等。

### 主动提示（不构成交易建议）

- 当前 PM 维持保证金 20.0% 恰触警戒，若 VIX 继续跳升 > 40 → 按 PM-3 主动平仓至敞口 ≤ NLV × 0.5；请在 IBKR 提前准备减仓清单。
- 现有持仓请按 **DTE + 权利金回撤 50%** 规则评估是否提前平仓。
- 建议等待 VIX 回落 < 30、`gates_ok=true` 且 `gate_score ≥ 50` 后再扫描。

### IBKR 验证清单（减仓/守仓专用）

□ 导出当前持仓列表（按 Delta 绝对值降序）
□ 标记 |Delta| > 0.25 的头寸为"优先评估"
□ 检查 Exposure Fee 预估（PM 集中度风险在 VIX>35 时惩罚最重）
□ 准备若 VIX > 40 时平仓至敞口 ≤ NLV × 0.5 的候选清单

**本内容由RHCLOUD生成，不构成任何投资建议**
