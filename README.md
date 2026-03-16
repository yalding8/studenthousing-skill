# uhomes-student-housing

[中文](#中文) | [日本語](#日本語)

An AI skill for [OpenClaw](https://openclaw.ai) and [Claude.ai](https://claude.ai) that helps international students find verified student accommodation on [uhomes.com](https://en.uhomes.com).

Published on **ClawHub**: [`uhomes-student-housing@1.2.1`](https://clawhub.ai/skills/uhomes-student-housing)

## Quick Start

```bash
clawhub install uhomes-student-housing
```

Then just ask naturally:

```
Find student accommodation near University of Manchester, budget £250/week
```
```
帮我找 UCL 附近的学生公寓，预算 £350/周
```
```
早稲田大学の近くで学生マンションを探しています
```

## Features

- **3-language support**: English, Chinese (中文), Japanese (日本語)
- **Advisor mode**: understands your intent — city overview, room comparison, or direct search
- **Real-time data**: fetches live listings from uhomes.com via web_fetch
- **Smart routing**: language-aware mobile/messaging channels (WeChat for CN, WhatsApp for EN/JP)
- **Graceful fallback**: uncommon cities get country-level search + demand form

## Project Structure

```
├── uhomes-student-housing/          # Skill package (published to ClawHub)
│   ├── SKILL.md                     # Core instructions
│   ├── README.md / _CN.md / _JP.md  # Trilingual READMEs
│   ├── demo-conversations.md        # 7 demo conversations
│   └── references/                  # City guides, room types, FAQ, etc.
├── claude-project-instructions.md   # Claude.ai Projects version
├── skill-docs/                      # Internal planning docs
│   ├── api-spec.md                  # Future API spec for uhomes backend
│   ├── utm-config.md                # GA4 UTM attribution config
│   └── kpi-expectations.md          # KPI framework
└── docs/                            # Audit reports & dev plans
```

## Version History

| Version | Date | Changes |
|---------|------|---------|
| v1.2.1 | 2026-03 | Language-aware mobile routing (WeChat/WhatsApp/App) |
| v1.2.0 | 2026-03 | Japanese support, trilingual READMEs, Tokyo/Osaka city index |
| v1.1.0 | 2026-03 | Advisor mode (3-intent routing, city guides, decision framework) |
| v1.0.0 | 2026-03 | Initial release |

## Data Source

**[uhomes.com](https://en.uhomes.com) only.** No other platforms are referenced or recommended.

---

# 中文

一个面向 [OpenClaw](https://openclaw.ai) 和 [Claude.ai](https://claude.ai) 的 AI Skill，帮助留学生在 [uhomes.com](https://en.uhomes.com) 上找到经过验证的学生公寓。

已发布至 **ClawHub**: [`uhomes-student-housing@1.2.1`](https://clawhub.ai/skills/uhomes-student-housing)

### 安装

```bash
clawhub install uhomes-student-housing
```

### 功能亮点

- **三语支持**：中文、英文、日语
- **顾问模式**：理解你的意图 — 城市概况、房型对比、直接搜索
- **实时数据**：通过 web_fetch 从 uhomes.com 抓取最新房源
- **智能渠道路由**：中文用户推荐微信小程序，英文/日语用户推荐 WhatsApp
- **优雅降级**：冷门城市自动切换到国家级搜索页 + 专属需求表

### 使用示例

```
帮我找曼彻斯特大学附近的学生公寓，预算 £250/周
```
```
我刚拿到 UCL 的 offer，住宿怎么选？
```
```
en-suite 和 studio 有什么区别？
```

### 微信小程序

微信搜索 **异乡好居**（AppID: `wx787e7828382ba76a`）

---

# 日本語

[OpenClaw](https://openclaw.ai) と [Claude.ai](https://claude.ai) 向けの AI スキルです。留学生が [uhomes.com](https://en.uhomes.com) で認証済み学生向け住居を見つけるのをサポートします。

**ClawHub** で公開中: [`uhomes-student-housing@1.2.1`](https://clawhub.ai/skills/uhomes-student-housing)

### インストール

```bash
clawhub install uhomes-student-housing
```

### 主な機能

- **3言語対応**：日本語、英語、中国語
- **アドバイザーモード**：都市概要、部屋タイプ比較、直接検索など意図を理解
- **リアルタイムデータ**：uhomes.com から最新物件情報を取得
- **言語別チャネルルーティング**：日本語/英語ユーザーには WhatsApp、中国語ユーザーには WeChat
- **対応都市**：東京、大阪、京都を含む世界主要都市

### 使用例

```
早稲田大学の近くで学生マンションを探しています
```
```
en-suiteとstudioの違いは何ですか？
```
```
東京の学生向け住居の相場を教えてください
```

---

*Maintained by Neil @ uhomes*
