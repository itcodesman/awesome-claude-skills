# Claude Skill Suite — Private Wealth & Structured Finance

> **Institutional-grade AI skill packages for Claude, purpose-built for high-net-worth wealth management, cross-border tax planning, options strategy execution, and trust architecture.**

---

## Overview

This repository hosts a curated collection of **Claude Skill packages** designed for professionals operating at the intersection of private wealth, structured finance, and cross-border compliance. Each skill encodes deep domain expertise — transforming Claude into a specialized advisor capable of reasoning through complex, multi-jurisdictional scenarios with the rigor of an institutional practitioner.

These skills are not general-purpose. They are built for:

- **Family offices and private wealth managers** structuring cross-border portfolios
- **Options traders and volatility strategists** running systematic income strategies
- **Tax counsel and compliance teams** navigating US, Singapore, Hong Kong, and China regulatory environments
- **Trust & estate professionals** designing asset protection structures for ultra-high-net-worth clients

---

## Skill Packages

### 🏦 `openclaw-options` — Systematic Options Income Engine
*v10.0 · Last updated April 2026*

A production-ready skill for executing the **OpenClaw Wheel strategy** — a disciplined, rules-based framework for selling cash-secured Puts on a curated 20-ticker universe. Encodes pre-screen logic, signal classification, and position sizing directly into Claude's reasoning layer.

| Component | Description |
|-----------|-------------|
| `schema.md` | Scan data format spec (v2.x / v3.x / v4.1 compatible) |
| `tickers-20pool.md` | Full parameter matrix for the core 20-ticker universe |
| `tickers-playbook.md` | Per-ticker strategy cards with volatility regime context |
| `data-system.md` | Pre-screen Gate logic, data source hierarchy, and degradation rules |

**Key capabilities:** Signal interpretation from scan output · Short Put target selection · Greeks-aware position sizing · PM-level risk gate decisions

---

### 📋 `options-playbooks` — Institutional Strategy Playbook Library
*v2.5 · 11 playbooks · Institutional standard*

A structured library of **11 institutional-grade options strategy playbooks**, spanning core income strategies to volatility arbitrage. Each playbook follows a consistent operational format — entry conditions, management rules, exit triggers, and client documentation templates.

**Strategies covered:** Wheel · Covered Call · Volatility Arbitrage · Ticker-specific execution guides (AAPL, NVDA, TSLA, COIN, and more)

**Key capabilities:** Trade execution guidance · Risk scenario walkthroughs · Client-ready deliverable generation

---

### 🌏 `global-tax-expert` — Multi-Jurisdiction Tax Intelligence
*v4.0 · US · Singapore · Hong Kong · China · CRS deep module included*

A comprehensive tax planning knowledge base built for **cross-border wealth structures**. Covers the full compliance surface across four major jurisdictions — from NRA withholding mechanics to CRS automatic exchange logic and FATCA reporting obligations. The v4.0 CRS module includes look-through analysis for layered offshore structures.

| Reference | Coverage |
|-----------|----------|
| `us-federal-tax.md` | NRA tax planning, withholding, treaty positions |
| `singapore-tax.md` | SG sourcing rules, MAS frameworks, territorial principles |
| `hong-kong-tax.md` | HK profits tax, offshore claims, MPF considerations |
| `crs-deep-dive.md` | CRS mechanics, automatic exchange, beneficial ownership look-through |
| `fatca-guide.md` | FATCA classification, W-8/W-9 logic, FFI obligations |
| `estate-tax-planning.md` | US estate exposure for non-residents, treaty elections |

**Key capabilities:** NRA tax optimization · CRS exposure mapping · Cross-border succession structuring · Offshore entity design and compliance review

---

### 🏛 `us-trust-advisor` — Trust Architecture & Asset Protection
*v1.0 · Initial release*

A specialized skill for designing and evaluating **US-based and offshore trust structures** in the context of cross-border wealth planning. Covers the full spectrum from domestic revocable trusts to Dynasty and offshore asset protection structures, with integrated CRS look-through and KYC compliance guidance.

| Reference | Coverage |
|-----------|----------|
| `trust-types.md` | Revocable · Irrevocable · ILIT · Dynasty · SLAT · DAPT |
| `trust-architecture.md` | Formation, governance, trustee selection, protector roles |
| `jurisdictions.md` | South Dakota · Nevada · Cook Islands · Cayman · BVI — comparative analysis |
| `tax-implications.md` | Grantor trust rules, DNI, CRS look-through on trust structures |
| `compliance-kyc.md` | Trust KYC/AML due diligence, UBO disclosure, FATF standards |

**Key capabilities:** Bespoke trust structure design · Jurisdiction selection analysis · CRS risk mitigation · HNW succession and asset protection planning

---

## Getting Started

### Installation

```bash
# Clone the repository
git clone https://github.com/your-org/claude-skill-suite.git

# Extract the skill package(s) you need
tar -xzf openclaw-options.tar.gz
tar -xzf global-tax-expert.tar.gz
```

### Integration

**Claude API (Recommended)**
Upload the extracted skill folder to your Claude project's file system. The `SKILL.md` file serves as the skill manifest — Claude will automatically reference the `references/` directory for domain knowledge.

```
your-project/
├── skills/
│   ├── openclaw-options/
│   │   ├── SKILL.md
│   │   └── references/
│   └── global-tax-expert/
│       ├── SKILL.md
│       └── references/
```

**Claude.ai**
Follow the Anthropic Custom Skills import workflow. Upload the extracted folder via the Skills interface.

**Local Deployment**
Copy the `references/` subdirectory to your application's configured skill path.

---

## Technical Specifications

| Attribute | Detail |
|-----------|--------|
| File format | Plain text Markdown (`.md`) |
| Encoding | UTF-8, no binary dependencies |
| Compatibility | Claude API · Claude.ai · Local Claude deployments |
| Schema versioning | Backward compatible across v2.x → v4.1 |

---

## Packages at a Glance

| Package | Version | Domain | Size |
|---------|---------|--------|------|
| `openclaw-options` | v10.0 | Systematic options income | 30K |
| `options-playbooks` | v2.5 | Multi-strategy playbooks | 13K |
| `global-tax-expert` | v4.0 | Cross-border tax planning | 33K |
| `us-trust-advisor` | v1.0 | Trust & asset protection | 14K |
| · |  |  |  |

---

## Disclaimer

These skill packages encode domain knowledge for **informational and decision-support purposes only**. They do not constitute legal, tax, or financial advice. Outputs should be reviewed by qualified professionals before being acted upon. Users are responsible for ensuring compliance with applicable laws and regulations in their jurisdiction.

---

*Built for practitioners who demand institutional-grade precision from AI.*
