# Example · 正常日（scope=20tickers, 5 通过 / 3 跳过）

> **目的**：演示 Claude 收到一份典型 `LLM_*.txt` 后的完整输出格式。
> **场景**：市场 A 级 · VIX=17 · 期限正常 · PM 账户健康 · 无数据降级。

---

## 输入样例（精简版 `LLM_20260422_0930.txt`）

```json
{
  "m": {
    "ts": "2026-04-22 09:30:00",
    "ver": "v4.1",
    "vix": 17.2,
    "vix3m": 18.1,
    "term": "NORMAL",
    "sp_ma200": true,
    "sp_dd20": 0.8,
    "gates_ok": true,
    "gate_score": 78,
    "fomc_d": 12,
    "cpi_d": 8,
    "nfp_d": 20,
    "scope": "20tickers",
    "off_cnt": 20,
    "sug_cnt": 0,
    "sig_cnt": "5/20",
    "hv_proxy_count": 0,
    "hv_proxy_sig_count": 0,
    "margin": {
      "account_type": "PM",
      "nlv": 6438052,
      "maint": 942692,
      "excess": 5482033,
      "bp": 35785464,
      "bp_used": 6287811,
      "used_pct": 14.6
    }
  },
  "sig": [
    {
      "tkr": "AAPL", "g": "A", "off": true, "sec": "Tech",
      "px": 245.10, "dte": 35, "exp": "2026-05-27",
      "ivr": 45, "ivs": "real", "ivt": "fl",
      "earn": 18,
      "th": { "ivr_min": 30, "ann_min": 10, "otm_buf": 10 },
      "c": [ { "k": 230, "del": -0.18, "yld": 12.4, "otm": 6.2, "prem": 2.70, "spd": 1.8, "oi": 8200, "vol": 1340, "gs": "r", "str": "csp", "notional": 23000 } ]
    },
    {
      "tkr": "MSFT", "g": "A", "off": true, "sec": "Tech",
      "px": 432.50, "dte": 35, "exp": "2026-05-27",
      "ivr": 38, "ivs": "real", "ivt": "fl",
      "earn": 24,
      "th": { "ivr_min": 30, "ann_min": 10, "otm_buf": 10 },
      "c": [ { "k": 410, "del": -0.19, "yld": 11.2, "otm": 5.2, "prem": 4.30, "spd": 1.5, "oi": 5600, "vol": 900, "gs": "r", "str": "csp", "notional": 41000 } ]
    },
    {
      "tkr": "UNH", "g": "B", "off": true, "sec": "Healthcare",
      "px": 548.00, "dte": 38, "exp": "2026-05-30",
      "ivr": 42, "ivs": "real", "ivt": "fl",
      "earn": 22,
      "th": { "ivr_min": 35, "ann_min": 8, "otm_buf": 12 },
      "c": [ { "k": 510, "del": -0.20, "yld": 9.8, "otm": 6.9, "prem": 5.20, "spd": 2.1, "oi": 2800, "vol": 410, "gs": "r", "str": "csp", "notional": 51000 } ]
    },
    {
      "tkr": "AMD", "g": "C", "off": true, "sec": "Semiconductors",
      "px": 148.20, "dte": 40, "exp": "2026-06-01",
      "ivr": 58, "ivs": "real", "ivt": "up",
      "earn": 30,
      "th": { "ivr_min": 50, "ann_min": 14, "otm_buf": 17 },
      "c": [ { "k": 122, "del": -0.22, "yld": 16.8, "otm": 17.7, "prem": 2.20, "spd": 2.6, "oi": 14200, "vol": 2100, "gs": "r", "str": "bps", "notional": 500 } ],
      "warn": ["iv_rising"]
    },
    {
      "tkr": "TSLA", "g": "D", "off": true, "sec": "Auto",
      "px": 232.40, "dte": 38, "exp": "2026-05-30",
      "ivr": 72, "ivs": "real", "ivt": "fl",
      "earn": 26,
      "th": { "ivr_min": 65, "ann_min": 15, "otm_buf": 22 },
      "c": [ { "k": 180, "del": -0.14, "yld": 17.5, "otm": 22.5, "prem": 2.80, "spd": 3.1, "oi": 28000, "vol": 4500, "gs": "r", "str": "fbps", "notional": 1500 } ]
    }
  ],
  "skip": {
    "count": 3,
    "ivrl": ["GOOG(28)", "COST(30)"],
    "ebl": ["LLY(6)"]
  }
}
```

---

## Claude 期望输出

### 本轮扫描概览

- 扫描时间：2026-04-22 09:30 SGT · 脚本 v4.1
- 范围：`20tickers` 精选池（20 标的）
- Gate：全部通过（score=78，环境良好）
- PM 账户：维持保证金 14.6% · 剩余流动性 85% · 购买力使用 17.6%（**全绿**）
- 宏观：距 FOMC 12 天 / CPI 8 天 / NFP 20 天（无阻断）

### 推荐信号（5 个）

```
【Wheel卖Put信号】

标的代码：AAPL（官方标的）
行权价：$230
行权日期：2026-05-27（DTE 35 天）
年化收益：12.4%（权利金 $2.70/张）
Delta：-0.18
名义敞口：$23,000（占 NLV 的 0.36%）
推荐原因:
  IV 状态: "IVR=45%（门槛 30%），ivs=real，ivt=fl"
  趋势状态: "VIX=17 / 期限正常 / SPX>200MA"
  宏观验证: "距 FOMC 12 天，距 CPI 8 天，距财报 18 天（安全）"
  标的级验证: "接货后正股市值预估 $23K（<$60K 遗产税线）"
  PM 验证: "开仓后维持保证金 14.6%→约 14.9%，单标 0.36%，科技行业合计约 1.0%"
  年化: "12.4%（权利金 $2.70/张，DTE 35 天）"
  Opportunity: "—"
⚠️ 结构：csp
⚠️ PM 提醒：无触发
```

```
【Wheel卖Put信号】MSFT · $410 Put · DTE 35 · 年化 11.2% · Delta -0.19 · 名义 $41K（0.64%）
【Wheel卖Put信号】UNH  · $510 Put · DTE 38 · 年化 9.8%  · Delta -0.20 · 名义 $51K（0.79%）
【Wheel卖Put信号】AMD  · $122/$112 Bull Put Spread · DTE 40 · 年化 16.8% · 名义 $500（<0.01%）
【Wheel卖Put信号】TSLA · $180/$165 Bull Put Spread · DTE 38 · 年化 17.5% · 名义 $1.5K · ⚠️ 张数≤2 张（见 §PM-1 硬编码张数上限表）
```

（简写处仅示范；生产环境每条都按 AAPL 范例完整展开。）

### 跳过原因（3 个）

| 标的 | 原因 | 数值 |
|---|---|---|
| GOOG | IVR 低于门槛 | 当前 IVR=28 / 门槛 35 |
| COST | IVR 低于门槛 | 当前 IVR=30 / 门槛 35 |
| LLY | 财报阻断 | 距财报 6 天（禁区 10 天） |

### IBKR 验证清单

□ TWS 期权链确认行权价/到期日/权利金
□ Greeks 偏差 < 10%（AAPL -0.18 / MSFT -0.19 / UNH -0.20 / AMD -0.22 / TSLA -0.14）
□ IVR 复核（45 / 38 / 42 / 58 / 72）
□ 财报窗口复核（AAPL ≥10 / MSFT ≥10 / UNH ≥10 / AMD ≥14 / TSLA ≥14）
□ 当前维持保证金 / NLV < 20% ✅
□ 当前剩余流动性 / NLV > 50% ✅
□ 拟开仓后保证金占用增量 TWS "Check Margin"
□ 加密组合合计 ≤ NLV × 4%（本轮无加密标的，N/A）
□ Exposure Fee 预估 = $0
□ 【D 层专项】TSLA 张数 ≤ 2 张
□ 【接货警告】若 MSFT/UNH 接货后美国正股市值超 $60K，推遗产税警告

**本内容由RHCLOUD生成，不构成任何投资建议**
