# OpenClaw 扫描器输出 Schema

> **版本**：v4.1（兼容 v3.1.X / v3.0 / v2.1 / v2.0）·对应脚本 `openclaw_scan.py`
> **触发**：文件名形如 `LLM_YYYYMMDD_HHMM.txt`，内容以 `{"m":{...` 开头
> **v4.1 变更摘要**：新增/扩展数据源路由、修复若干 bug；**无破坏性 schema 变更**。字段语义与 v3.1.X 完全一致；若遇 v3.1.X/v2.X 输出，按旧版规则解析。

---

## 文件整体结构

```
{
  "m":      { ...meta... },      // 市场环境 + Gate 状态 + 扫描范围
  "sig":    [ ...signals... ],   // 通过筛选的标的（可能为空）
  "skip":   { ...skip_info... }, // 被跳过标的汇总（v3.x/v4.x 压缩分类；v2.1 文字）
  "legend": { ... }              // 可选，仅 --with-legend 时出现
}
```

**紧凑化规则**：值为 `null` 的字段被省略。字段不存在 ≠ 数据异常。

---

## 版本兼容矩阵

| 版本 | 识别 | 关键差异 | 解析策略 |
|---|---|---|---|
| **v4.1**（当前） | `m.ver == "v4.1"` | 数据源增加 + bug 修复；schema 与 v3.1.X 同构 | 全字段支持 |
| v3.1 / v3.1.X | `m.ver == "v3.1"` 或以 `"v3.1."` 开头 | 完整 v3 字段集 | 全字段支持 |
| v3.0 | `m.ver == "v3.0"` | 缺少极少数 v3.1 新增字段 | 按字段存在性判断 |
| v2.1 | `m.ver == "v2.1"` | `skip` 为字符串；部分枚举用全称 | 降级解析（见下） |
| v2.0 或更早 | `m.ver` 缺失或以 `v2.0` 开头 | 无 `scope` / 无宏观日历字段 | 视 scope=preferred；宏观事件字段视为空 |

**降级处理原则**：字段缺失不报错、视为 `null`；枚举值遇到未知短码回退到默认分支。Claude 在报告开头提示："兼容模式解析 (vX.X)，新字段缺失属正常"。

---

## `m` (meta) 字段

| 字段 | 类型 | 含义 | 自版本 |
|---|---|---|---|
| `ts` | string | 扫描时间戳 `YYYY-MM-DD HH:MM:SS` | v2.0+ |
| `ver` | string | 脚本版本号（v4.1/v3.1.X/v3.0/v2.1/...） | v2.0+ |
| `vix` | number | VIX 现价（CBOE 官方优先） | v2.0+ |
| `vix3m` | number | VIX3M 现价（用于期限结构） | v2.0+ |
| `term` | string | 期限结构 `NORMAL` / `INVERTED` | v2.0+ |
| `sp_ma200` | bool | SP500 是否在 200MA 上方 | v2.0+ |
| `sp_dd20` | number | SP500 相对 20 日峰值回撤%（正=回撤；0=高点；负=创新高） | v2.0+ |
| `gates_ok` | bool | 5 项 Pre-screen Gate 是否全通过（false 时 sig 为空） | v2.0+ |
| `gate_score` | int | 三因子得分（0-100；≥50 通过；≥80 友好） | v2.0+ |
| `fomc_d` | int | 距下次 FOMC 天数 | v2.0+ |
| `cpi_d` | int | 距下次 CPI 天数 | v3.0+ |
| `nfp_d` | int | 距下次非农天数 | v3.0+ |
| `boj_d` | int | 距下次 BOJ 天数 | v3.0+ |
| `scope` | string | 扫描范围 `20tickers` / `preferred` / `full`（v2.1 缺失视为 preferred；v3.0 仅有 preferred/full；`20tickers` 自 v3.1 起支持） | v3.0+ |
| `off_cnt` | int | 官方标的数量 | v3.0+ |
| `sug_cnt` | int | 扩展标的数量 | v3.0+ |
| `sig_cnt` | string | `"{信号数}/{总扫描数}"` | v2.0+ |
| `hv_proxy_count` | int | 走 HV 代理的标的数 | v2.0+ |
| `hv_proxy_sig_count` | int | 信号列表中走 HV 代理的数量 | v2.0+ |
| `margin` | object | 保证金状态（见下表）；**整对象缺失 = Gate-4 软通过** | v2.0+ |
| `d_adj` | number | DELTA_MIN 收紧量（仅期限结构倒挂时出现） | v2.0+ |
| `ds_health` | object | 数据源健康度快照（见下表） | v4.1+（可选） |

### `m.margin` 子字段（PM 账户关键）

> **所有子字段可选**。用户通过 `--margin-used X` 或 `--margin-json path` 提供则填入；否则整对象缺失。

| 子字段 | 类型 | 含义 | 备注 |
|---|---|---|---|
| `used_pct` | number | 已使用保证金 / 可用保证金 × 100 | Gate-4 依据；PM 账户门槛 45% |
| `nlv` | number | Net Liquidation Value（净清算价值） | PM-6 $100K 红线依据 |
| `maint` | number | Maintenance Margin（维持保证金美元） | PM-2 指标 1 分子 |
| `excess` | number | Excess Liquidity（剩余流动性美元） | PM-2 指标 2 分子 |
| `bp` | number | Buying Power（购买力美元） | PM-2 指标 3 分母 |
| `bp_used` | number | 已占用购买力美元 | PM-2 指标 3 分子 |
| `account_type` | string | `"PM"` / `"RegT"` | 标识账户类型 |
| `ts` | string | 保证金快照时间 | 用户手工更新易过期，提醒复核 |

**Claude 读 `m.margin` 的分析优先级**：
1. 先查 `account_type`。是 `"PM"` 则应用 SKILL.md 的 PM 风控铁律；是 `"RegT"` 则用 Reg-T 规则。
2. 计算 PM-2 三项指标：
   - 维持保证金比 = `maint / nlv`
   - 剩余流动性比 = `excess / nlv`
   - 购买力使用率 = `bp_used / bp`
3. 报告所处区域（安全/警戒/危险）并决定是否推信号。

### `m.ds_health` 子字段（v4.1+ 可选）

| 子字段 | 类型 | 含义 |
|---|---|---|
| `primary` | string | 本轮扫描主力数据源名称 |
| `fallback_used` | bool | 是否触发了降级到备用源 |
| `sources` | object | 各源 health 分数快照 `{"massive": 100, "tradier": 85, ...}` |

**缺失兼容**：v3.1.X 及以下版本无此对象，Claude 不作任何断言，视为"数据源状态未报告"。

---

## `sig` 数组字段

### 顶层字段

| 字段 | 含义 | 自版本 |
|---|---|---|
| `tkr` | 股票代码 | v2.0+ |
| `g` | 风险评级 `A+`/`A`/`B`/`C` | v2.0+ |
| `off` | 是否官方标的（**键不存在=扩展池**；v2.1 缺失视为 true） | v3.0+ |
| `sec` | 行业分类 | v3.0+ |
| `cc` | 合规/上下文规则标签 | v2.0+ |
| `px` | 标的现价 | v2.0+ |
| `dte` | 到期天数 | v2.0+ |
| `exp` | 到期日 `YYYY-MM-DD` | v2.0+ |
| `ivr` | IVR（0-100） | v2.0+ |
| `ivs` | IVR 来源 `real` / `hv` | v2.0+ |
| `ivt` | IV 趋势（v3.x/v4.x 短码：`?`/`up`/`dn`/`fl`；v2.1 全称：`unknown`/`rising`/`falling`/`flat`） | v2.0+ |
| `earn` | 距财报天数（正=前；负=后；null=未知） | v2.0+ |
| `earn_note` | 财报上下文（仅 earn<0 时出现） | v2.0+ |
| `opp` | Opportunity Alert 摘要（见下） | v3.0+ |
| `warn` | 警告代码列表 | v2.0+ |
| `th` | 筛选阈值 `{ivr_min, ann_min, otm_buf}`（分析时对比实际值） | v2.0+ |
| `c` | 候选合约列表 | v2.0+ |
| `src` | 该标的数据来源（v4.1+ 可选） | v4.1+ |

### `opp` 子字段

| 字段 | 含义 |
|---|---|
| `t` | 全部 5 项条件同时触发（强机会信号） |
| `p` | 仅 C1 触发（弱机会信号，参考用） |
| `cd` | 连续收阴天数 |
| `p5d` / `p3d` | 5日/3日价格涨跌幅%（负=下跌） |
| `c1_5d` / `c1_3d` | 是否满足 5日/3日跌幅门槛 |
| `d5` | IVR 5日变化（百分点） |
| `fail` | `p=true` 时列出未满足条件 key |

**Opportunity Alert 5 项条件定义（`opp.t=true` 须全部满足）**

| 条件 | 字段 | 触发标准 | 含义 |
|---|---|---|---|
| C1 | `c1_5d` 或 `c1_3d` | 5日跌幅 ≤ -5% 或 3日跌幅 ≤ -3% | 近期价格超跌，溢价提升 |
| C2 | `d5` | IVR 5日内上升 ≥ 10 百分点 | IV 快速抬升，卖方窗口开启 |
| C3 | `cd` | 连续收阴 ≥ 3 天 | 持续下行压力，恐慌情绪积累 |
| C4 | `ivr` vs `th.ivr_min` | 当前 IVR ≥ 标的门槛 | IV 绝对水平已达开仓标准 |
| C5 | `earn` | 距财报天数 ≥ 标的禁区天数 | 无财报黑天鹅风险 |

**Claude 决策权重（含边界条件）**：

- `opp.t=true`：视为加分项；**允许**在其他条件边界时放宽 IVR 容忍度 **5pp**，但必须**同时满足**以下全部边界条件：
  1. 标的层级为 **A 层或 B 层**（C/D 层禁止放宽；高波动层放宽反而放大尾部风险）
  2. `ivs == "real"`（HV 代理下禁止放宽，即 `ivs=="hv"` 时忽略此加分）
  3. `m.ds_health.fallback_used != true`（数据源降级下禁止放宽）
  4. `warn` 列表中不含 `iv_rising` / `iv_rising_blocked` / `hv_proxy` / `ds_degrade`
  5. **永远不得**绕过 PM-1~PM-6 铁律（保证金、名义敞口、加密总控等硬约束）
  6. 任一条件不满足 → `opp.t` 仅作为展示标签，不触发 5pp 放宽
- `opp.p=true`（仅 C1 触发）：仅作参考，不调整任何阈值
- `opp` 字段缺失：视为无机会信号，正常按标准门槛评估
- `drop` 标记（见 `skip.drop`）：有机会但被其他规则阻断，**必须**在输出中单独列出供人工关注

---

## `c` (contract) 数组字段

| 字段 | 含义 | 自版本 |
|---|---|---|
| `k` | 行权价 | v2.0+ |
| `del` | Delta（卖 PUT 为负） | v2.0+ |
| `yld` | 年化收益%（CSP：`prem/k×365/dte×100`；价差：`净权利金/宽度×365/dte×100`） | v2.0+ |
| `otm` | OTM 深度%：`(px-k)/px×100` | v2.0+ |
| `prem` | 中间价权利金 | v2.0+ |
| `spd` | 买卖价差% | v2.0+ |
| `oi` | 持仓量 | v2.0+ |
| `vol` | 当日成交量 | v2.0+ |
| `gs` | Greeks 来源：`r`=真实；`bs`=Black-Scholes | v2.0+ |
| `str` | 策略结构短码 | v2.0+ |
| `notional` | 名义敞口美元（v4.1+ 可选：`k*100`） | v4.1+ |

**`str` 策略短码**：
- `csp` = 现金担保卖 PUT
- `csp_spread` = CSP 但建议改价差
- `bps` = Bull Put Spread
- `fbps` = 强制 Bull Put Spread
- `fspread` = 强制价差（通用）

**`notional` 缺失兼容**：v3.1.X 及以下版本无此字段，Claude 自行用 `k * 100` 计算（CSP）或 `(短腿k - 长腿k) * 100`（价差）。

---

## `skip` 字段

### v3.x / v4.x 压缩分类

| 字段 | 含义 |
|---|---|
| `count` | 被跳过总数 |
| `nc` | 无合格合约（ticker 列表） |
| `ebl` | 财报阻断：`SYM(days)` |
| `eunk` | 财报日期未知，保守阻断 |
| `pebl` | `--block-post-earnings` 阻断 |
| `ivrl` | IVR 低于门槛：`SYM(current_ivr)` |
| `ivu` | IVR 缺失/无法计算 |
| `gld` | GLD VIX 闸门阻断：`GLD(vix)` |
| `fomc` / `cpi` / `nfp` / `boj` | 宏观阻断窗口：`SYM(days)` |
| `nt` | 无信号触发（兜底） |
| `err` | 数据异常：`SYM:message` |
| `drop` | ⚡ 标的触发了 Opportunity Alert **但因其他原因（财报/宏观/IVR不足）被跳过**：`SYM:type(pct%)`。注意：`drop` 不表示因"机会"被过滤，而是"有机会信号却无法开仓"，应重点标记供人工关注。 |

### v2.1 文字格式

逗号分隔字符串，例：`"EWJ(无合格合约), GLD(gld_vix(18.1)), TSLA(earn_bl(6d))"`。按 `SYM(原因)` 逐条拆分，⚡ 后缀=Opportunity Alert。

---

## Warn Codes

| 代码 | 含义 |
|---|---|
| `earn_bl` | 在财报阻断窗口 |
| `near_blackout` | 接近财报阻断（缓冲期内） |
| `earn_unk` | 财报日获取失败，保守处理 |
| `post_earnings_vol` | 财报后 IV crush 窗口 |
| `post_earn_bl` | 被 `--block-post-earnings` 阻断 |
| `fomc_bl` / `cpi_bl` / `nfp_bl` / `boj_bl` | 宏观阻断窗口 |
| `gld_vix_blocked` | GLD 被 VIX<22 机制阻断 |
| `hv_proxy` | IVR 用 HV 代理（非真实 IV） |
| `iv_rising` | IV 趋势上升（卖方不利） |
| `iv_rising_blocked` | IV 上升 + strict flag 共同阻断 |
| `ivt_unknown` | IV 趋势数据不足 |
| `dte:N` | DTE 落在 fallback 区间 |
| `cc:价差` | 合规规则要求价差结构 |
| `cc:暂停` | 合规规则要求暂停 |
| `ds_degrade` | 数据源降级（v4.1+ 可选） |

---

## 关键解析注意事项

1. **版本识别优先**：先读 `m.ver` 确认版本；v4.1 和 v3.1.X 的 schema 同构，解析逻辑完全一致；v2.x 走降级分支。
2. **`off` 判断**：v3.x/v4.x 键存在=官方；键不存在=扩展池，参数推断必须注明。v2.1 全部视为官方。
3. **`scope` + `sug_cnt`**：preferred + sug_cnt=0=纯核心池；full + sug_cnt>0=含扩展，官方优先。
4. **`sp_dd20`**：正=回撤 | 0=高点 | 负=创新高。
5. **HV 代理全覆盖**：`hv_proxy_count` 接近总数时，IVR 仅供粗略参考，明确提示"本轮所有 IVR 都是近似值，决策打折"。
6. **`margin` 缺失**：每次必提醒"Gate-4 未核验，开仓前请在 IBKR 确认维持保证金 < NLV 的 20%（PM 账户标准）"。
7. **`margin` 存在但 `account_type` 缺失**：默认按 PM 账户规则应用（符合当前用户账户情况）。
8. **v2.x 降级**：检测 `m.ver == "v2.1"` 或更早版本时提示"兼容模式解析，新字段缺失属正常"，其余逻辑不变。
9. **`ds_health` 缺失**：v3.1.X 及以下无此字段，不作任何数据源状态推断。

---

## 异常输入处理清单

下列情况按表处理；每条异常在输出开头**必须**用 ⚠️ 前缀单独列出，不可静默。

| # | 异常现象 | 判定条件 | 处理动作 |
|---|---|---|---|
| E1 | 多版本并存 | 用户一次性上传多份 `LLM_*.txt`，`m.ver` 不同 | 以**最新**扫描时间 `m.ts` 为准；旧文件仅作对比参考；输出提示"本轮以 {最新 ts} 为主" |
| E2 | 同版本双份文件（扩展 + 精选） | `scope == "20tickers"` 与 `scope == "preferred"/"full"` 各一份 | 两份独立解析、独立输出；参数**不跨池合并**；最终合并 IBKR 验证清单 |
| E3 | scope 与 sig 内 ticker 不一致 | `scope == "20tickers"` 但 `sig[i].tkr` 不在 20 池内 | 对该 ticker 降级为"扩展池·参数推断"，按 `tickers-preferred.md` 参数评估；输出提示"scope 与 tkr 不匹配，已降级处理" |
| E4 | `m.ver` 缺失或未知 | `m.ver` 为空 / 非 `v2.x/v3.x/v4.x` 前缀 | 视为 v2.0 兼容模式，`scope` 默认 `preferred`；输出提示"版本未知，已按兼容模式解析" |
| E5 | `m.scope` 缺失 | 无 `scope` 字段 | 默认 `preferred`；输出提示"scope 缺失，已按 preferred 处理" |
| E6 | `gates_ok=false` 但 `sig` 非空 | 逻辑矛盾（gate 失败不该有信号） | 一律忽略 `sig`；按 gate 失败处理；输出提示"文件状态不一致，已按 Gate 失败处理" |
| E7 | `c` 数组为空但该 `sig` 未被跳过 | 合约列表空 | 视为该 ticker 无合格合约；并入 `skip.nc` 处理 |
| E8 | `m.margin.account_type` 与当前账户不符 | 值为 `"RegT"` 而用户账户为 PM（或相反） | 以**用户账户类型**为准（SKILL.md 用户配置 = PM）；输出提示"扫描文件账户类型与用户配置不一致，已按 PM 规则处理" |
| E9 | 时间戳过期 | `m.ts` 距当前 SGT > 6 小时 | 输出红字警告"扫描数据可能过期，建议重新扫描后决策" |
| E10 | 文件非 JSON 或解析失败 | 首行非 `{"m":{` 或 JSON 解析异常 | 拒绝分析；要求用户重新上传 `LLM_YYYYMMDD_HHMM.txt` |

**处理原则**：异常不终止分析（除 E10）；但所有异常必须**显式**出现在输出开头的"本轮异常"区，不允许合并到普通文本中。

---

⚠️ 本文档仅描述扫描器输出格式，不构成任何投资建议。
