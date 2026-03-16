# uhomes-student-housing

[中文版](./README_CN.md) | [日本語版](./README_JP.md)

An AI skill for [OpenClaw](https://openclaw.ai) and [Claude.ai](https://claude.ai) that helps international students find verified student accommodation on [uhomes.com](https://en.uhomes.com).

Published on **ClawHub**: [`uhomes-student-housing@1.5.1`](https://clawhub.ai/skills/uhomes-student-housing)

## Install

```bash
clawhub install uhomes-student-housing
```

Then ask naturally:

```
Find student accommodation near University of Manchester, budget £250/week
```

```
I'm going to UCL next year — what's the housing situation like?
```

```
Compare en-suite vs studio near King's College London
```

## How It Works

### 1. Understands your intent

The skill detects whether you need a city overview, a direct property search, or a room type explanation — and responds accordingly.

### 2. Fetches live data from two sources

- **University listing page** on uhomes.com — real-time property data
- **uhomes Must-Stay list** (必住榜) — editorially curated top-rated properties per city/university

### 3. Scores and ranks with a 4-stage algorithm

```
Stage 1  Hard Filter     → Remove low-rated, too far, over-budget properties
Stage 2  Scoring         → Distance (exponential decay) + Budget match
                           + Must-Stay rank bonus + Bayesian quality rating
Stage 3  Slot Selection  → Pick 5 properties across different dimensions
Stage 4  Explanation     → Label each with why it was recommended
```

### 4. Presents 5 diverse recommendations

Each recommendation fills a different slot — so you always get variety, not 5 similar options:

| Slot | What it optimizes | Example label |
|------|------------------|---------------|
| ⭐ Top Pick | Highest overall score | "Best overall match" |
| 🏆 Must-Stay | Highest rank on uhomes curated list | "#1 on Must-Stay list" |
| 📍 Closest | Shortest walk to campus | "3 min walk to UoM" |
| 💰 Best Value | Lowest price | "£172/week" |
| ❤️ Top Rated | Highest Bayesian-adjusted rating | "4.6★ from 349 reviews" |

### 5. Links you to uhomes.com

Every property includes a direct booking link. The full Must-Stay list and search page are linked at the end.

## Supported Languages

English, Chinese, and Japanese. The skill detects your language and replies in the same language. Property names and room type terms (en-suite, studio, PBSA) stay in English.

## Mobile Access

- **WhatsApp**: [+44 207 631 5139](https://wa.me/442076315139)
- **uhomes App**: [Download](https://static.uhzcdn.com/static/webapp/09/html/openinstall/uhomes.html?channelCode=webapp_blog_download_cn)
- **Live chat**: available 24/7 at [en.uhomes.com](https://en.uhomes.com)

## Must-Stay Coverage

The recommendation algorithm integrates uhomes Must-Stay data for **22 universities** and **17 UK cities + Boston**. Properties on the Must-Stay list receive a ranking bonus and a 🏆 badge.

## City Coverage

UK (London, Manchester, Birmingham, Edinburgh, Glasgow, Oxford, Cambridge, Bristol, Leeds, Southampton, Sheffield, Liverpool, Durham, Nottingham, Coventry + more), Australia (Sydney, Melbourne + 4 more), US (New York, Boston + 4 more), Canada (Toronto, Vancouver, Montreal), Japan (Tokyo, Osaka, Kyoto), Ireland, Singapore, Hong Kong.

For unlisted cities, the skill falls back to the country-level search page.

## Project Structure

```
├── uhomes-student-housing/          # Skill package (published to ClawHub)
│   ├── SKILL.md                     # Core instructions + scoring algorithm
│   ├── demo-conversations.md        # 7 demo conversations (EN/CN/JP)
│   └── references/
│       ├── must-stay-config.md      # Must-Stay city/university ID mapping
│       ├── city-guides.md           # City housing overviews (10 cities)
│       ├── decision-guide.md        # Room type decision framework
│       ├── city-index.md            # Verified city/university URL slugs
│       ├── room-types.md            # Room type definitions (EN/CN/JP)
│       ├── url-patterns.md          # uhomes URL construction rules
│       └── faq.md                   # Contracts, payments, bills
├── claude-project-instructions.md   # Claude.ai Projects version
├── skill-docs/                      # Internal: API spec, UTM config, KPI
└── docs/                            # Audit reports, dev plans, algorithm discussion
```

## Version History

| Version | Date | Changes |
|---------|------|---------|
| v1.5.1 | 2026-03 | Fix slot dedup, Must-Stay badge stacking, city_id guard |
| v1.5.0 | 2026-03 | 4-stage recommendation algorithm (filter→score→slot→explain) |
| v1.4.0 | 2026-03 | Complete Must-Stay mapping — 22 universities + 17 cities |
| v1.3.0 | 2026-03 | Must-Stay list integration with weighted scoring |
| v1.2.1 | 2026-03 | Language-aware mobile routing (WeChat / WhatsApp / App) |
| v1.2.0 | 2026-03 | Japanese language support, trilingual READMEs |
| v1.1.0 | 2026-03 | Advisor mode with 3-intent routing, city guides, decision framework |
| v1.0.0 | 2026-03 | Initial release |

## Data Source

**[uhomes.com](https://en.uhomes.com) only.** No other platforms are referenced or recommended.

---

*Maintained by Neil @ uhomes*
