---
name: options-playbooks
description: >
  期权交易策略操作手册合集——包含11份机构级客户交付文档，覆盖Wheel卖Put、
  Covered Call、波动率套利、事件驱动等多策略体系。按标的分为：综合手册（wheel/open）、
  指数策略（SPX/QQQ）、科技蓝筹（AAPL/NVDA/TSLA）、加密相关（COIN/CRCL/IBIT）、
  及客户定制方案（KAISHANG/MR GENG）。
  当用户提到以下任何内容时触发此技能：期权操作手册、策略手册、客户交付文档、
  波动率策略操作、NRA期权合规、AAPL/NVDA/TSLA/COIN/CRCL/IBIT的具体操作规程、
  Wheel执行手册、备兑开仓手册、三级熔断机制、月度操作SOP、绩效追踪台账、
  期权策略架构（三层/五层收益结构）、EV门槛、Delta矩阵操作细节、
  市场状态引擎操作、财报季操作规程、心理纪律守则、资金分配铁律、
  VIX仓位矩阵操作、KAISHANG方案、MR GENG方案、期权套利自动化、
  或需要参考已有策略手册来生成新的客户方案/操作手册时。
  也适用于用户要求创建新标的手册、修改已有手册、对比不同标的策略差异、
  提取手册中的具体参数或规则等场景。
  注意：本技能侧重「操作执行层」的完整手册，与openclaw-options技能（侧重系统参数与信号引擎）互补。
  如果用户问的是开仓参数配置、信号推送格式、Pre-screen Gate等系统层逻辑，优先使用openclaw-options；
  如果问的是某标的的完整操作规程、客户手册内容、或需要生成新手册，使用本技能。
---

# 期权交易策略操作手册

## JSON 扫描结果分析工作流

**所需文件：SKILL.md + `references/04-tickers.md`。税务身份默认NRA，无需读取其他reference。**

### Step 1：Pre-screen Gates

读取 `data.pre_screen_gates`，全部 `passed=true` 才继续。任一失败 → 输出"扫描终止"+原因。

### Step 2：记录市场环境

```
VIX = data.market.vix
VIX3M = data.market.vix3m（如有）
三因子得分 = Gate-5 中的得分
```

### Step 3：逐标的淘汰筛选

对 `data.tickers` 中每个标的，依次检查（任一不通过即淘汰）：

| 序号 | 检查项 | JSON字段 | 淘汰条件 |
|------|--------|---------|---------|
| 1 | 状态 | `status` | ≠ "OK" |
| 2 | IVR达标 | `ivr_meets_threshold` | = false |
| 3 | 财报黑名单 | `in_earnings_blackout` | = true |
| 4 | 标的特殊前置条件 | 需查 04-tickers.md | 未满足（如GLD需VIX>22等） |
| 5 | 有合规合约 | `best_contracts` | 为空 |

**Step 4 需搜索确认**（不在JSON中）：
- FOMC日期：SPY/QQQ前3天、XLU前5天停
- 各标的财报日期：按黑名单天数检查
- 地缘/监管事件

### Step 4：合约筛选

对存活标的的 `best_contracts`，逐条验证：

| 检查项 | JSON字段 | 要求 |
|--------|---------|------|
| OTM缓冲 | `otm_pct` | ≥ 该标的OTM门槛（查04-tickers.md） |
| Delta | `delta` | 在允许区间内 |
| 流动性 | `spread_pct` | < 15% |
| 年化收益 | `annualized_yield` | ≥ 该标的年化门槛 |

不达标 → 降档尝试下一个合约。全部不达标 → 该标的本轮无信号。

### Step 5：输出

对通过的合约，用可视化信号卡片展示：
```
【Wheel卖Put信号】
标的代码：{TICKER}（{等级}级）
行权价：${STRIKE}（OTM {otm_pct}%）
行权日期：{EXPIRY}（DTE {X}天）
年化收益：{annualized_yield}%（权利金 ${mid}/张）
Delta：{delta}
结构：{CSP / Bull Put Spread}
⚠️ 需IBKR验证：实时Delta、精确IV、OI数量
```

无任何标的通过 → 输出"本轮无合格信号"+ 各标的淘汰原因汇总。

---

## Reference 文件导航

| 文件 | 何时读取 | 内容概要 |
|------|---------|---------|
| `04-tickers.md` | **分析JSON时必读** | 20个标的的特点与交易参数 |
| `01-tax-framework.md` | 问税务/合规/遗产税时 | NRA税务框架、W-8BEN、遗产税红线 |
| `02-revenue-risk.md` | 问收益结构/EV/Delta/风控时 | 五层收益、EV公式、Delta矩阵、持仓管理 |
| `03-operations-manual.md` | 问熔断/SOP/台账/纪律时 | 市场引擎、三因子、熔断、月度SOP、台账、纪律 |

**与 openclaw-options 的分工**：
- 分析JSON → options-playbooks（本技能的SKILL.md + 04-tickers.md）
- 查系统参数/信号格式/Pre-screen → openclaw-options
- 生成新手册 → 本技能全部references + openclaw-options最新参数

⚠️ 本内容仅为数据参考，不构成任何投资建议。期权交易具有高风险，可能导致全部本金损失。
