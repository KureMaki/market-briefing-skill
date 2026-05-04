# 使用说明

将以下内容完整复制，粘贴到任意 Claude 对话窗口（claude.ai 网页版、桌面端、手机 App 均可），发送后 Claude 会自动搜索新闻并生成当日市场简报。

**需要开启 Web Search。不需要任何 API key 或额外配置。**

---

# Global Market Briefing Generator

Generate a structured pre-market briefing for an investor who trades during the JST 14:30–15:30
window. The report covers US equities, bonds, commodities, gold, oil, FX, and geopolitical
developments — focusing on what happened, why, and what it means going forward.

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
- If no 24-hour article contains a specific data point, omit it entirely. Do not substitute
  with numbers from older articles.
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

> Report output is in Chinese. To localize, replace the panel headers and template strings
> below with your target language equivalents.

### Report Structure (4 panels)

```
🌅 亚洲盘前Briefing | [日期] 11:00 JST
基于[上一交易日]美股收盘，数据来源：财经新闻（过去24小时）

━━ 🌐 主线叙事 ━━
━━ 📰 昨夜发生了什么 ━━
━━ 📊 数据速览 ━━
━━ 📋 市场总评 & 亚洲关注 ━━
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

Five sub-modules in order. **Skip any sub-module with no relevant news.** Every event must
include a specific actor/institution + action + number. Do not write "markets fell on concerns."

**Inline ⚠️ alert rule**: If a specific asset is mentioned as having a significant move,
add one line immediately below:
```
  ⚠️ TICKER: [1-sentence cause] → [1-sentence meaning for the holder]
```

**Key price levels** (flag in ⚠️ line if news price is within ±2% of these):
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

📌 具体事件与催化剂
1️⃣ [event: specific actor + action + number]
   → 市场反应: [link to a move in equities/gold/bonds]

⚔️ 地缘与战争（skip if already covered above）
→ [conflict/development, impact on commodities or safe havens]

🌏 亚洲相关（昨日表现）
→ 港股: [Hang Seng level + move]
→ 日经: [Nikkei 225 level + move]
→ A股: [Shanghai/CSI level + move]
→ 汇率: USD/JPY xxx.xx（贬值/升值x%），USD/CNY x.xxxx
```

---

### Panel 3: 数据速览

**Only show assets for which a specific number appeared in yesterday's news.** Omit the
entire row if no number was found — do not write "N/A" or "暂缺".

Format:
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
[C01–C04 signal line if detected]

💱 汇率
USD/JPY  xxx.xx  [贬值/升值x%]  → [one-line FX impact note]
USD/CNY  x.xxxx  [贬值/升值x%]  → [one-line FX impact note]

[VIX line — only when: >20, <13, or daily move >±15%]
```

**FX impact reference**:
- USD/JPY rising (yen weakening): benefits Japanese exporters; dilutes real returns on JPY assets
- USD/JPY falling (yen strengthening): pressures Nikkei; often accompanies global risk-off
- USD/CNY rising (yuan weakening): foreign outflow pressure on A-shares
- USD/CNY falling (yuan strengthening): foreign inflow expectations, positive for A-shares

**VIX reference** (only when threshold triggered):

| Range | Emoji | Meaning |
|-------|-------|---------|
| < 13 | 💤 | Extremely low — possible inflection point building |
| 20–25 | 🟡 | Elevated — risk-off sentiment rising |
| 25–30 | 😱 | Fear zone |
| > 30 | 😱😱 | Extreme fear — high rebound probability |
| ±15% daily | ⚡ | Single-day spike |

---

### Panel 4: 市场总评 & 亚洲关注

**Forward-looking only. Do not repeat any numbers or events from panels 1–3.**

- **总评**: Current market phase + main tension + single most important variable to watch
  over the next 1–2 sessions
- **亚洲关注**: Today's Asian session opening outlook (do not re-cite yesterday's Asian numbers):
  - Japan: USD/JPY trend implication + Nikkei open outlook
  - HK/A-shares: USD/CNY trend + Hang Seng/CSI open outlook
- Strictly grounded in Step 1 news. Do not infer beyond what sources support.

Format:
```
━━ 📋 市场总评 & 亚洲关注 ━━

总评: [2–3 sentences. Phase + tension + key variable. Give a judgment, not a summary.]

亚洲关注:
① 日本: [USD/JPY trend + Nikkei open outlook, 1 sentence]
② 港股/A股: [USD/CNY trend + Hang Seng/CSI open outlook, 1 sentence]

---
⚠️ AI分析仅供参考，不构成投资建议。数据来源：过去24小时财经新闻。
```

---

## Data Integrity Rules

1. Every number must come from a news article found in Step 1. Do not search tickers independently.
2. No data = no row. Never write "暂缺".
3. 24-hour cutoff is strict. Numbers from older articles are not used.
4. Panel 4 does not invent. Forward-looking statements must be grounded in Step 1 news.
5. Events ≠ market moves. An event entry requires actor + action + number.
