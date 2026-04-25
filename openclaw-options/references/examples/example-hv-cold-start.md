# Example · IV 冷启动日（HV 代理，全量 IVR 打折）

> **目的**：演示 IV 历史库冷启动期（`hv_proxy_count` 接近总数）时 Claude 的保守输出。
> **场景**：IV 库刚启用 12 天 · 所有 `ivs="hv"` · `ivt="?"` · Gate 通过 · PM 账户健康。
> **关键约束**：即便 `opp.t=true`，**禁止**放宽 IVR 5pp（违反 `schema.md §opp 决策权重 边界条件 2`）。

---

## 输入样例（精简版 `LLM_20260422_0930.txt`）

```json
{
  "m": {
    "ts": "2026-04-22 09:30:00",
    "ver": "v4.1",
    "vix": 19.8,
    "vix3m": 20.5,
    "term": "NORMAL",
    "sp_ma200": true,
    "sp_dd20": 1.2,
    "gates_ok": true,
    "gate_score": 65,
    "fomc_d": 14,
    "scope": "20tickers",
    "off_cnt": 20,
    "sig_cnt": "2/20",
    "hv_proxy_count": 20,
    "hv_proxy_sig_count": 2,
    "margin": {
      "account_type": "PM",
      "nlv": 6438052,
      "maint": 970000,
      "excess": 5400000,
      "used_pct": 15.1
    },
    "ds_health": { "primary": "massive", "fallback_used": false }
  },
  "sig": [
    {
      "tkr": "AAPL", "g": "A", "off": true, "sec": "Tech",
      "px": 245.10, "dte": 35, "exp": "2026-05-27",
      "ivr": 33, "ivs": "hv", "ivt": "?",
      "earn": 18,
      "th": { "ivr_min": 30, "ann_min": 10, "otm_buf": 10 },
      "c": [ { "k": 230, "del": -0.18, "yld": 10.4, "prem": 2.25, "oi": 8200, "gs": "r", "str": "csp", "notional": 23000 } ],
      "warn": ["hv_proxy", "ivt_unknown"],
      "opp": { "t": true, "cd": 3, "p5d": -5.2, "c1_5d": true, "d5": 12 }
    },
    {
      "tkr": "HON", "g": "A", "off": true, "sec": "Industrial",
      "px": 218.50, "dte": 35,
      "ivr": 32, "ivs": "hv", "ivt": "?",
      "earn": 28,
      "th": { "ivr_min": 30, "ann_min": 9, "otm_buf": 10 },
      "c": [ { "k": 205, "del": -0.17, "yld": 9.6, "prem": 1.90, "gs": "r", "str": "csp", "notional": 20500 } ],
      "warn": ["hv_proxy", "ivt_unknown"]
    }
  ],
  "skip": {
    "count": 18,
    "ivrl": ["MSFT(26)", "GOOG(24)", "MA(25)", "V(22)"],
    "ivu": ["LLY", "UNH", "HCA", "KLAC", "GE"],
    "nt": ["COST", "AMD", "PANW", "CRWD", "SNOW", "TSLA", "COIN", "SMCI", "HOOD"]
  }
}
```

---

## Claude 期望输出

### 本轮异常 / 降级提示

⚠️ **IV 历史库冷启动**：`hv_proxy_count=20/20`，所有标的 IVR 均为 **HV 代理近似值**，非真实 IVR。本轮所有信号决策需**打折看待**，推送数量与仓位规模**主动降一档**。

⚠️ **IV 趋势未知**：全部标的 `ivt="?"`，无法判断 IV 是上升还是下降窗口。

⚠️ **Opportunity Alert 5pp 放宽 · 本轮禁用**：AAPL 虽 `opp.t=true`，但 `ivs="hv"` + `ivt_unknown` + `warn` 含 `hv_proxy` 已违反边界条件 2/4（见 `schema.md §opp 决策权重`），5pp 放宽**不启用**；`opp.t` 仅作展示标签。

### 本轮扫描概览

- 扫描时间：2026-04-22 09:30 SGT · 脚本 v4.1 · 范围 `20tickers`
- Gate：通过（score=65，及格但不理想）
- PM 账户：维持保证金 15.1% · 剩余流动性 83.9%（健康）
- 宏观：距 FOMC 14 天（无宏观阻断）

### 推荐信号（2 个 · 保守建议）

**建议 1 · AAPL**（官方标的 · A 层）

- 合约：$230 Put · DTE 35 · Delta -0.18
- 权利金 $2.25 · 年化 10.4%（勉强过门槛 10%）
- 推荐原因（结构化）：
  - IV 状态：IVR=33%（门槛 30%） · **ivs=hv**（近似值） · ivt=?
  - 趋势状态：VIX=19.8 · 期限正常 · SPX > 200MA
  - 宏观验证：距 FOMC 14 天 · 距财报 18 天
  - 标的级验证：接货市值 $23K（< $60K 遗产税线）
  - PM 验证：开仓后维持保证金 ≈ 15.3%，单标 0.36%
  - 年化：10.4%
  - Opportunity：5 项全中（`cd=3 / p5d=-5.2% / d5=+12pp / c1_5d`）但因 HV 代理禁用 5pp 放宽
- **冷启动降级建议**：仓位规模按常规 × 0.7；IVR=33 与门槛 30 差距过小，建议再等待 1–2 天观察真实 IVR 到位后再开仓。

**建议 2 · HON**（官方标的 · A 层）

- 合约：$205 Put · DTE 35 · Delta -0.17 · 权利金 $1.90 · 年化 9.6%
- **年化 9.6% < HON 门槛 9% 勉强通过**；结合 HV 代理不确定性，实际偏差可能使其低于门槛，**本轮列为"观察"**不推建仓。

### 跳过原因汇总

| 类别 | 标的 | 说明 |
|---|---|---|
| IVR 低于门槛 | MSFT(26) / GOOG(24) / MA(25) / V(22) | HV 代理下绝对值无参考意义，真实 IVR 可能更高或更低 |
| IVR 缺失 | LLY / UNH / HCA / KLAC / GE | 数据不足，本轮跳过 |
| 无合格合约 | COST / AMD / PANW / CRWD / SNOW / TSLA / COIN / SMCI / HOOD | 权利金或 Delta 未达筛选门槛 |

### IBKR 验证清单

□ TWS 查询 AAPL / HON 真实 IV Rank（**重点**：本轮脚本值为 HV 代理）
□ 若 IBKR 真实 IVR 比脚本值低 >5pp → 放弃本笔开仓
□ 所有 Greeks 偏差核查 < 10%
□ 财报窗口 ≥ 10 天（AAPL=18 / HON=28）
□ PM 维持保证金 15.1%（安全）
□ 本轮信号仓位规模 ×0.7（冷启动保守）
□ 建议同时运行 `save_iv_snapshot` 以加速 IV 库到 30 天阈值

### 关键提示

- IV 库预计再跑 **18 天**（30 - 12）后切到真实 IVR；在此之前建议每日都跑扫描。
- 当前所有 `opp.t=true` 的机会信号都被 HV 代理边界条件拦截，**这是设计意图**，不是误拦截。
- 若用户坚持在冷启动期开仓，应书面确认"已知 IVR 为近似值，决策风险自担"。

**本内容由RHCLOUD生成，不构成任何投资建议**
