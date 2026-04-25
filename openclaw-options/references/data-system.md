# OpenClaw 数据系统要点

> **用途**：Pre-screen Gate 的数据来源 + 多数据源降级策略 + IV 库冷启动 + PM 保证金数据传递
> **版本**：适配脚本 v4.1（向下兼容 v3.1.X / v2.X）
> **与 SKILL.md 的分工**：SKILL.md 给 Gate 触发规则（速查）；本文件给数据源降级细节和 PM 数据字段

---

## Pre-screen Gate 数据来源

每次扫描循环开头执行。任一失败则终止本轮全部扫描，**零期权链 API 调用**（节省 40-60% 无效请求）。

| Gate | 规则 | 数据源 | 失败动作 |
|---|---|---|---|
| 1 · 市场状态 | 非 D 级（SPX > MA50 或 MA20 向上） | yfinance `^GSPC` | 终止 |
| 2 · VIX 硬上限 | VIX ≤ 35 | CBOE 官方 JSON，fallback `^VIX` yfinance | 终止 |
| 3 · 期限结构 | VIX > VIX3M 时各标的 delta_min 收紧 +0.02（per-tier 生效；详见下方四层基准表） | CBOE（同批次） | 仅调参，不终止 |
| 4 · 保证金使用率 | **PM 账户：`margin_used < 45%`**（Reg-T：<55%） | CLI `--margin-used X` 或 `--margin-json path` | 终止；未传则软通过 |
| 5 · 三因子得分 | 得分 ≥ 50 | 本地计算 | 终止 |

**Gate-3 四层 delta_min 基准表（`--scope 20tickers` 模式）**

| 层级 | 正常期限结构 | 倒挂时（+0.02 收紧） | 备注 |
|---|---|---|---|
| A | -0.22（上限绝对值） | -0.20 | 蓝筹CSP；上移收紧安全垫 |
| B | -0.20 | -0.18 | 防御标的；与A层差距保留1格 |
| C | -0.18 | -0.16 | 高波动价差；已有安全垫要求 |
| D | -0.15 | -0.13 | 最高风险层；最保守 |

> **解读**：delta_min 为 delta 绝对值上限，开仓合约的 |Delta| 不得超过该值。倒挂时收紧意味着必须卖更深度的 OTM 合约。`preferred`/`full` 模式不区分层级，统一按 SKILL.md 的 -0.30/-0.28 基准（期限结构条件不变）。

**Gate-5 评分**：VIX（<20=35 / <28=25 / <35=15 / else=0）+ SP500（>MA200=30 / 回撤<5%=15 / else=5）+ 期限结构（NORMAL=25 / unknown=15 / INVERTED=5）。≥80 环境友好；50-79 通过但不理想。

**Gate-4 软通过提醒**：`m.margin` 缺失时，Claude 必须提醒"开仓前请在 IBKR 确认维持保证金 < NLV 的 20%（PM 账户标准）"。

---

## PM 账户保证金数据传递（v4.1+）

### 方式一：单值 CLI（最简单）

```bash
python openclaw_scan.py --margin-used 15
# 仅传入 used_pct；Claude 只能做 Gate-4 判断，无法做 PM-2 三项分析
```

### 方式二：JSON 文件（推荐，启用 PM-2 三项分析）

```bash
python openclaw_scan.py --margin-json ibkr_margin.json
```

```json
{
  "account_type": "PM",
  "nlv": 6438052,
  "maint": 942692,
  "excess": 5482033,
  "bp": 35785464,
  "bp_used": 6287811,
  "used_pct": 14.6,
  "ts": "2026-04-22 09:15:00"
}
```

**兼容性**：v3.1.X 只识别 `--margin-used`，JSON 文件不被解析（无害）。v4.1 两者共存时 JSON 优先。`m.margin.account_type` 缺失时默认按 PM 规则处理。

---

## 数据源分层（v4.1）

| 层级 | 数据源 | 额度 | 核心作用 |
|---|---|---|---|
| T0 | **Massive API** | 5 key 轮询，每 key 20s 冷却 | 完整期权链 + Greeks + K 线（主力） |
| T1 | Alpaca Paper | 200 req/min | 期权链 + 历史 |
| T2 | **Tradier** | ~200 req/min | 完整期权链 + 实时 Greeks |
| T3 | Polygon.io | 5 req/min | 期权链备份 |
| T4 | yfinance | 无官方限制 | 股票历史 + 期权链（无实时 Greeks，退化到 BS） |
| T5 | v4.1 新增层 | 依配置 | 进一步冗余路径 |

**CBOE API** 单独用于 VIX / VIX3M（Pre-screen Gate 依赖）。

**健康度机制**：成功 +5（上限 100），失败 -30（下限 0），401/403 直接 =0 永久排除。路由选 health 最高的可用源。

**Symbol 映射**：某些标的在某些源不可用（如 yfinance 无 SPX），`translate_symbol` 返回 None 时自动跳到下一源（不扣 health）；所有源都 None → status=ERROR。

**`m.ds_health`**（v4.1+ 可选）：记录本轮主力源和降级情况。缺失 = 旧版本，不作任何推断。

---

## IV 历史库冷启动

`calculate_real_ivr` 需 ≥30 条历史快照才切到真实 IVR，否则退化到 HV 代理。

**冷启动症状**：`m.hv_proxy_count` 接近总数 + 所有 signal 的 `ivs:"hv"` + `ivt:"unknown"`。
→ Claude 必须提示："本轮所有 IVR 都是近似值，决策需打折扣"

**加速**：每日至少跑一次扫描（`save_iv_snapshot` 自动写入快照），30 天后切到真实 IVR。启动时 `_iv_history_health_check()` 检查近 30 天覆盖率，<50% 红字警告。

---

## 数据库与缓存

**当前**：单表 SQLite `iv_history.db` 存 ATM IV 快照；L1 进程内存缓存（TTL 30s）避免同轮重复查询。

**核心原则**：先解决"该不该打"（Gate 前置），再解决"怎么打"（限流）。双时间戳区分 API 响应时间和数据市场时间。

**铁律**：**AI 不直接修改策略规则文件**，参数调整必须人工 `git commit`。

---

## v4.1 变更清单

| 类别 | 变更 | 影响 |
|---|---|---|
| 数据源 | 新增 T5 层、Tradier 限额提升、增强故障切换 | 可用性提升；无破坏性 schema 变更 |
| Bug 修复 | 健康度持久化、Symbol 映射边界、缓存 TTL | 稳定性提升 |
| Schema | 完全兼容 v3.1.X；新增可选字段 `m.ds_health`、`c.notional`、`sig.src` | 缺失即忽略 |
| PM 账户 | `--margin-json` 支持完整 `m.margin` 对象 | 启用 PM-2 三项指标分析 |
| 破坏性变更 | **无** | — |

升级：替换脚本文件即可；旧 `LLM_*.txt`（v3.1.X / v2.X）Claude 仍能正常解析。

---

⚠️ 本内容仅为架构参考，不构成任何投资建议。
