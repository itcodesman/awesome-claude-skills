# 标的参数入口（路由索引）

> **作用**：本文件仅作**路由**。Claude 收到扫描文件后，按 `m.scope` 选择**唯一一份**参数文件加载，不做跨文件合并。
> **历史兼容**：SKILL.md 与老版 prompt 仍指向 `tickers.md`，本文件保持路径不变；详细参数已拆分到下方三个子文件。

---

## 一、参数优先级路由（硬规则）

| `m.scope` 取值 | 加载的**唯一**参数源 | 禁止加载 |
|---|---|---|
| `"20tickers"` | `references/tickers-20pool.md` | preferred.md |
| `"preferred"` | `references/tickers-preferred.md` | 20pool.md |
| `"full"` | `references/tickers-preferred.md`（含扩展池） | 20pool.md |
| 缺失或未知 | 默认按 `preferred` 处理 + 输出兼容模式提示 | — |

> **同标的两表参数冲突时**：只认 `m.scope` 指定的那张表，不做取更严、不做均值、不做跨表参考。例：`scope=20tickers` 时 `TSLA` 用 IVR≥65% / 年化≥15%；`scope=preferred`/`full` 时 `TSLA` 用 IVR≥80% / 年化≥20%。

> **非 scope 标的混入**：`scope=20tickers` 时若 `sig` 中出现 20 池外的 ticker（如 NVDA/BABA），按 `references/schema.md §异常输入处理清单` 处理。

---

## 二、子文件导航

| 文件 | 内容 | 加载时机 |
|---|---|---|
| `references/tickers-20pool.md` | 20 标的参数总表 + 20tickers 追加约束 | `m.scope == "20tickers"` |
| `references/tickers-preferred.md` | 全量池特殊规则 + 全量速查总表（A+/A/B/C） | `m.scope ∈ {"preferred", "full"}` |
| `references/tickers-playbook.md` | 结构化推荐原因模板 + 财报禁区最终值速查 | 每次输出信号时 |

**通用规则**（跨池复用，不放在任一子文件内）：

- 加密组合总控 ≤ NLV × 4% → `SKILL.md §PM-1 · 加密子条款`（唯一权威定义）
- 硬编码合约张数上限 → `SKILL.md §PM-1 · 硬编码张数上限表`（唯一权威定义）
- Opportunity Alert 决策边界 → `references/schema.md §opp 决策权重`

---

## 三、Worked Examples

不确定某种输入下该输出什么时，参考：

- `references/examples/example-normal-day.md` · 正常日：信号 + 跳过原因 + IBKR 清单
- `references/examples/example-gate-fail.md` · Gate 失败日：零信号 + 解释
- `references/examples/example-hv-cold-start.md` · IV 冷启动：所有 IVR 打折

---

⚠️ 本内容仅为数据参考，不构成任何投资建议。
