# 🌏 Global Market Pre-Market Briefing — Claude Cowork Skill

*[中文版 →](README.md)*

Automatically searches financial news every weekday morning and generates a structured briefing covering US equities, bonds, commodities, gold, oil, and FX — to support trading decisions in Asian markets.

**Report language: Chinese | Schedule: Tue–Sat 09:40 JST | Platform: Claude Cowork**

→ [View sample report](example/market_briefing_sample.md)

---

## Design: News-First

This skill does not call market data APIs. Instead, it extracts numbers from financial news articles — figures that journalists have already verified — and structures them into a report.

Benefit: no API keys, no rate limits, every data point traceable to a source.  
Trade-off: if an asset wasn't mentioned in yesterday's news, it won't appear in the report. No placeholders, no hallucinated numbers.

---

## How to Use

### Option 1: Claude Cowork (recommended — supports scheduled auto-run)

1. Copy `SKILL.md` into your Cowork skills directory (typically `.claude/skills/market-briefing/SKILL.md`)
2. Restart Cowork or reload skills
3. Say to Claude: **"Generate market briefing"** or **"市场日报"**

Reports are saved to `market-briefing-result/` in your workspace.

To run on a schedule: create a Scheduled Task in Cowork with cron `40 9 * * 2-6` (Asia/Tokyo timezone) and the prompt `Run the market-briefing skill and save the report`.

### Option 2: Any Claude interface (manual)

Copy the full contents of `prompt.md` and paste it into any Claude conversation — claude.ai web, desktop, or mobile. No installation needed.

---

## Report Structure (4 panels)

| Panel | Content |
|-------|---------|
| 🌐 主线叙事 | The single most important cause-effect chain from the previous session (2–3 sentences). Names the cross-asset signal if one fired. |
| 📰 昨夜发生了什么 | 5 sub-modules: US equities · Bonds/Gold/Energy · Specific events · Geopolitics · Asia markets. Significant movers get an inline ⚠️ alert. |
| 📊 数据速览 | Data snapshot — only assets with numbers confirmed in yesterday's news. |
| 📋 市场总评 & 亚洲关注 | Forward-looking only. Market phase + key variable to watch + Nikkei/Hang Seng/A-share open outlook. Never repeats panels 1–3. |

### Cross-Asset Signal Detection

| Code | Pattern | Report label |
|------|---------|-------------|
| X01 | Bonds↑ + Gold↑ + Equities↓ | Risk-off flow |
| X02 | Gold↑ + Oil↑ + Bonds↓ | Inflation trade |
| X03 | Equities↑ + Bonds↓ + VIX↓ | Risk-on |
| X04 | USD↑ + Gold↓ + EM↓ | Dollar squeeze |
| X06 | Oil↑ + Equities↓ + Bonds↓ | Stagflation signal |

---

## Customization

**Key price levels** (used for ⚠️ proximity alerts — update in SKILL.md Panel 2 to match your holdings):

```
VOO: $545(-15%), $513(-20%) | QQQ: $541(-15%), $510(-20%), $478(-25%)
SMH: $342(-20%), $321(-25%), $300(-30%) | TLT: $85, $80, $75
```

**Asset coverage**: US equities (VOO/QQQ/NVDA/GOOGL/MSFT/SMH/EWY/SCHD), US bonds (TLT/SHY/EDV), commodities (gold/oil), FX (USD/JPY, USD/CNY). Edit the asset pool in SKILL.md Panel 3.

**Language**: Report output is in Chinese. To localize, replace the panel headers and template strings in SKILL.md Step 3.

---

## Requirements

- [Claude Cowork](https://claude.ai) (desktop app, Cowork mode) — for Option 1
- Web Search must be enabled
- No additional API keys required

---

## Background

This skill started from a simple goal: get a structured daily market briefing without signing up for data APIs, managing keys, or writing any code.

The news-first approach makes that possible — just Claude and search access, zero configuration. The trade-off in data completeness is acceptable for the use case of quickly understanding what happened in the previous session.

A production version is planned: direct market data feeds (YFinance / AkShare), scheduled via GitHub Actions, with push delivery to messaging apps.
