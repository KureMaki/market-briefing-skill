---
name: market-briefing
description: >
  Generate a structured global market daily briefing report covering US stocks, bonds,
  commodities, gold, oil, and FX. Use this skill whenever the user asks for a "market briefing",
  "market report", "daily report", "市场日报", "全球市场", "盘前简报", or anything related
  to generating a structured overview of financial market conditions. Also trigger when the
  user asks to check stock prices, market conditions, or portfolio-relevant market data
  across multiple asset classes. This skill searches financial news and produces a formatted
  4-panel report with signal detection, geopolitical context, and forward-looking analysis.
---

# Global Market Briefing Generator

You are generating a structured pre-market briefing for an investor who trades during the
JST 14:30–15:30 window. The report covers US equities, bonds, commodities, gold, oil, FX,
and geopolitical developments — focusing on what happened, why, and what it means going forward.

**Core philosophy — news-first**: Search financial news articles, extract numbers that
journalists have already verified, and structure them into an actionable report. Do NOT
independently look up ticker prices or rely on memory for market data. Every number in the
report must be traceable to a news article found in Step 1.

---

## Step 1: Search for News (4 searches, run in sequence)

All news must be from the **past 24 hours**. Reject any article older than 24 hours even if
it appears relevant — stale data is misleading.

```
Search 1: "US stock market today" OR "美股收盘" — index levels, big moves, tech stocks
Search 2: "bond market gold oil price today" — rates, gold, crude, FX moves
Search 3: "market news today Fed tariff earnings geopolitical" — catalysts and events
Search 4: "Hang Seng Nikkei A-share today USD JPY CNY" — Asia markets + FX
```

After each search, **check the publication date of every article before extracting any number**.
Only use figures from articles confirmed to be within the past 24 hours.

**24-hour rule — applied at the article level, not the search level**:
- If a search returns articles from mixed dates, only extract numbers from articles published
  within the past 24 hours. Discard numbers from older articles even if they appear in the
  same batch.
- A figure from a week-old article describing "this week's move" is stale — do not use it.
- If no 24-hour article contains a specific data point (e.g. gold price), omit that data
  point entirely. Do not substitute with numbers from older articles.
- If an entire topic only has articles from 2+ days ago, write
  "今日新闻暂缺，以下基于最近可用信息" at the top of the relevant section.

---

## Step 2: Detect Cross-Asset Signals

Based on the **directions** described in the news (not precise numbers), check for these
patterns. If a signal fires, it becomes the core of the main narrative in Panel 1.

### Cross-Asset Signals

| Signal | Pattern | Label (used in report) |
|--------|---------|------------------------|
| X01 | Bonds↑ + Gold↑ + Equities↓ | "资金流向避险资产" |
| X02 | Gold↑ + Oil↑ + Bonds↓ | "市场交易通胀预期上升" |
| X03 | Equities↑ + Bonds↓ + VIX↓ | "风险偏好回升" |
| X04 | USD↑ + Gold↓ + EM↓ | "美元走强压制非美资产" |
| X06 | Oil↑ + Equities↓ + Bonds↓ | "滞胀恐慌信号" |

### Commodity Internal Signals

| Code | Pattern | Label (used in report) |
|------|---------|------------------------|
| C01 | Gold↑ + Oil↑ | "黄金与油气同涨→通胀交易升温" |
| C02 | Gold↑ + Oil↓ | "黄金涨油气跌→避险需求驱动" |
| C03 | Gold↓ + Oil↓ | "黄金油气普跌→美元走强压制商品" |
| C04 | Gold flat/↓ + Oil↑ | "油气独立上涨→供给端扰动" |

---

## Step 3: Generate the Report

> **Language note**: Report output is in Chinese. To localize, replace the panel headers
> and template strings below with your target language equivalents.

### Report Structure (4 panels)

```
🌅 亚洲盘前Briefing | [日期] 11:00 JST
基于[上一交易日]美股收盘，数据来源：财经新闻（过去24小时）

━━ 🌐 主线叙事 ━━
━━ 📰 昨夜发生了什么 ━━   ← inline ⚠️ alerts, no separate alert panel
━━ 📊 数据速览 ━━
━━ 📋 市场总评 & 亚洲关注 ━━   ← forward-looking only, never repeats panels 1–3
```

---

### Panel 1: 主线叙事

**2–3 sentences, under 100 characters.** One cause-effect chain for the whole session.

- Find the single most important causal chain (A → B → C) with specific numbers
- If a cross-asset signal fired in Step 2, name it explicitly (e.g. "形成X01避险格局")
- No filler phrases: "市场波动加剧", "投资者情绪谨慎" → do not write these

Example:
```
━━ 🌐 主线叙事 ━━
特朗普宣布对华芯片出口新限制，科技股领跌，标普500收跌1.8%，资金同步流向国债和黄金，
形成 X01 避险格局。纳指科技权重股集体承压，半导体板块单日跌幅超3%。
```

---

### Panel 2: 昨夜发生了什么（含⚠️预警内嵌）

Five sub-modules in order. **Skip any sub-module with no relevant news — do not write a
placeholder.** Every event must include a specific actor/institution + action + number.
Do not write "markets fell on concerns" — that is paraphrasing, not reporting.

**Inline ⚠️ alert rule**: If a specific asset is mentioned in the news as having a
significant move (e.g. "NVDA fell 4.2%"), add one line immediately below the relevant
sub-module:
```
  ⚠️ TICKER: [1-sentence cause] → [1-sentence meaning for the holder]
```
No separate alert panel. No repeated explanation.

**Key price levels** (flag in the ⚠️ line if news price is within ±2% of these):
```
VOO: $545(-15%), $513(-20%) | QQQ: $541(-15%), $510(-20%), $478(-25%)
SMH: $342(-20%), $321(-25%), $300(-30%) | EWY: $116(-25%), $108(-30%)
SCHD: $29, $27 | GLDM: $85, $76, $65.84(-40%) | TLT: $85, $80, $75
```

Format:
```
━━ 📰 昨夜发生了什么 ━━

🇺🇸 美股整体（含科技股重点）
→ [index performance + tech/semis, cite numbers from news]
  ⚠️ NVDA: 半导体板块系统性承压 → 持仓者注意估值收缩风险

💰 债券 / 黄金 / 能源
→ [10Y Treasury yield, gold price, WTI/Brent — cite news numbers]
  ⚠️ 布伦特原油 +5.7% 至 $108: 输入性通胀压力上升 → 注意对Fed利率预期的影响

📌 具体事件与催化剂
1️⃣ [event: specific actor + action + number]
   → 市场反应: [link to a move in equities/gold/bonds]

2️⃣ [second event, if any]
   → 市场反应: [...]

⚔️ 地缘与战争（skip if already covered above）
→ [conflict/development, location + action, impact on commodities or safe havens]

🌏 亚洲相关（昨日表现）
→ 港股: [Hang Seng level + move]
→ 日经: [Nikkei 225 level + move]
→ A股: [Shanghai/CSI level + move]
→ 汇率: USD/JPY xxx.xx（贬值/升值x%），USD/CNY x.xxxx
```

---

### Panel 3: 数据速览

**Only show assets for which a specific number appeared in yesterday's news.** If no number
was found for an asset, omit the entire row — do not write "N/A" or "暂缺".

Asset pool (show only if data found): US equities (VOO, QQQ, NVDA, GOOGL, MSFT, SMH, EWY,
SCHD), US bonds (TLT, SHY, EDV or 10Y yield directly), commodities (WTI, gold spot, GLDM,
XLE), FX (USD/JPY, USD/CNY). VIX only when: >20, <13, or daily move >±15%.

Format (align with 2+ spaces):
```
━━ 📊 数据速览 ━━

🇺🇸 美股
QQQ(纳指100)   $xxx.xx  -x.xx%
NVDA(英伟达)   $xxx.xx  -x.xx%

📉 美债 & 利率
10年期美债收益率  x.xx%  ±x bp
TLT(长债ETF)    $xx.xx  +x.xx%

🛢️ 大宗 & 黄金
布伦特原油  $xxx.xx  +x.xx%
黄金(现货)  $x,xxx   -x.xx%
[C01–C04 signal line if detected, 1 line only]

💱 汇率
USD/JPY  xxx.xx  [贬值/升值x%]  → [one-line FX impact note]
USD/CNY  x.xxxx  [贬值/升值x%]  → [one-line FX impact note]

[VIX line — only appears when threshold triggered]
```

**FX impact reference** (use when writing the → note):
- USD/JPY rising (yen weakening): benefits Japanese exporters, dilutes real returns on JPY-denominated assets
- USD/JPY falling (yen strengthening): pressures Nikkei, often accompanies global risk-off
- USD/CNY rising (yuan weakening): foreign outflow pressure on A-shares; CNY gold price may rise more than USD gold
- USD/CNY falling (yuan strengthening): foreign inflow expectations, positive for A-shares

**VIX reference** (only when threshold triggered):

| Range | Emoji | Meaning |
|-------|-------|---------|
| < 13 | 💤 | Extremely low — possible inflection point building |
| 20–25 | 🟡 | Elevated — risk-off sentiment rising |
| 25–30 | 😱 | Fear zone |
| > 30 | 😱😱 | Extreme fear — high probability of post-spike rebound |
| ±15% daily | ⚡ | Single-day spike |

---

### Panel 4: 市场总评 & 亚洲关注

**Forward-looking analysis only. Do not repeat any numbers or events from panels 1–3.**

- **总评**: What market phase are we in? What is the main tension? What is the single most
  important variable to watch over the next 1–2 sessions?
- **亚洲关注**: Opening direction outlook for today's Asian session (do not re-cite
  yesterday's Asian numbers from Panel 2):
  - Japan: USD/JPY trend implication + Nikkei open outlook
  - HK/A-shares: USD/CNY trend implication + Hang Seng/CSI open outlook
- Strictly grounded in the news found in Step 1. Do not infer beyond what the sources support.

Format:
```
━━ 📋 市场总评 & 亚洲关注 ━━

总评: [2–3 sentences. Current phase + main tension + key variable to watch.
      No data repetition. Give a judgment, not a summary.]

亚洲关注:
① 日本: [USD/JPY trend implication + Nikkei open outlook, 1 sentence]
② 港股/A股: [USD/CNY trend implication + Hang Seng/CSI open outlook, 1 sentence]

---
⚠️ AI分析仅供参考，不构成投资建议。数据来源：过去24小时财经新闻。
```

---

## Data Integrity Rules

1. **Every number must come from a news article found in Step 1.** Do not search tickers independently. Do not estimate from memory.
2. **No data = no row.** If a number wasn't in the news, omit the asset entirely. Never write "暂缺".
3. **24-hour cutoff is strict.** Numbers from older articles are not used even if no fresher source exists.
4. **Panel 4 does not invent.** Forward-looking statements must be grounded in what the Step 1 news actually says.
5. **Events ≠ market moves.** An event entry requires a specific actor + action + number, not a restatement of price changes.

---

## Step 4: Save Report

Save the completed report as a markdown file:
- Filename: `market_briefing_[YYYY-MM-DD].md`
- Location: `market-briefing-result/` subfolder inside the current workspace
- Create the folder if it does not already exist
