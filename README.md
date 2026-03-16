# uhomes-student-housing

[中文版](./README_CN.md) | [日本語版](./README_JP.md)

An AI skill for [OpenClaw](https://openclaw.ai) and [Claude.ai](https://claude.ai) that helps international students find verified student accommodation on [uhomes.com](https://en.uhomes.com).

Published on **ClawHub**: [`uhomes-student-housing@1.2.1`](https://clawhub.ai/skills/uhomes-student-housing)

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

## What It Does

1. **Understands your intent** — whether you need a city overview, a specific property search, or a room type comparison
2. **Fetches live listings** from uhomes.com in real time
3. **Presents 3–5 verified properties** with price, distance, highlights, and a direct booking link
4. **Offers personalised advice** — budget guidance, area recommendations, room type comparisons
5. **Links to the full search page** on uhomes.com for all available options

## Supported Languages

This skill responds in English, Chinese, and Japanese. It detects the language you use and replies in the same language.

## Mobile Access

- **WhatsApp**: [+44 207 631 5139](https://wa.me/442076315139)
- **uhomes App**: [Download](https://static.uhzcdn.com/static/webapp/09/html/openinstall/uhomes.html?channelCode=webapp_blog_download_cn)
- **Live chat**: available 24/7 at [en.uhomes.com](https://en.uhomes.com)

## Coverage

Currently indexed: UK (London, Manchester, Birmingham, Edinburgh + 10 more), Australia (Sydney, Melbourne + 4 more), US (New York, Boston + 4 more), Canada (Toronto, Vancouver, Montreal), Japan (Tokyo, Osaka, Kyoto), Ireland, Singapore, Hong Kong.

For unlisted cities, the skill falls back to the country-level search page on uhomes.com.

## Project Structure

```
├── uhomes-student-housing/          # Skill package (published to ClawHub)
│   ├── SKILL.md                     # Core skill instructions
│   ├── demo-conversations.md        # 7 demo conversations
│   └── references/                  # City guides, room types, FAQ, etc.
├── claude-project-instructions.md   # Claude.ai Projects version
├── skill-docs/                      # Internal planning docs
└── docs/                            # Audit reports & dev plans
```

## Version History

| Version | Date | Changes |
|---------|------|---------|
| v1.2.1 | 2026-03 | Language-aware mobile routing (WeChat / WhatsApp / App) |
| v1.2.0 | 2026-03 | Japanese language support, trilingual READMEs |
| v1.1.0 | 2026-03 | Advisor mode with intent routing, city guides, decision framework |
| v1.0.0 | 2026-03 | Initial release |

## Data Source

**[uhomes.com](https://en.uhomes.com) only.** No other platforms are referenced or recommended.

---

*Maintained by Neil @ uhomes*
