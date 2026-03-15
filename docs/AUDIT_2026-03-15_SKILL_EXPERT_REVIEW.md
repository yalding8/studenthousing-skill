# uhomes-student-housing Skill 专家评审报告

**日期**：2026-03-15
**审核范围**：全部 Skill 文件（SKILL.md、references/*、demo-conversations.md、README.md、claude-project-instructions.md）及配套文档（skill-docs/*）
**评审基准**：Agent Skills 开放标准规范（agentskills.io）、ClawHub 发布要求、OpenClaw 13 点 Skill 质量清单

---

## 专家面板

| # | 专家角色 | 专业领域 |
|---|---------|---------|
| 1 | OpenClaw Skill 架构专家 | Agent Skills 标准规范、SKILL.md 格式合规性、ClawHub 发布流程 |
| 2 | Claude Code 集成专家 | Claude.ai Project Instructions、web_fetch 行为、Skill 触发机制 |
| 3 | AI 对话设计专家 | Slot filling 工作流、降级策略、多语言对话质量 |
| 4 | 增长/归因专家 | UTM 归因、xcode 追踪、KPI 设计、GA4 配置 |
| 5 | 安全与合规专家 | API Key 管理、用户数据、链接安全、Skill 安全审查 |

---

## 评审总评

| 维度 | 分数 (1-10) | 说明 |
|------|------------|------|
| 结构合规性 | 7.5 | 基本符合标准，有几处需修正 |
| 内容质量 | 8.5 | 工作流清晰、降级策略完善 |
| 技术可行性 | 6.5 | web_fetch 方案有实质性风险 |
| 商业完整度 | 9.0 | UTM/KPI/API 文档非常专业 |
| 安全性 | 7.0 | 有几处需关注的安全问题 |
| **综合均分** | **7.7** | |

---

## 发现清单

### 严重度定义

- **P0 (阻断)**: 发布前必须修复，否则 Skill 无法正常工作或无法通过 ClawHub 审核
- **P1 (重要)**: 强烈建议修复，影响用户体验或追踪准确性
- **P2 (改进)**: 建议修复，提升质量但不阻断发布
- **P3 (建议)**: 锦上添花，可后续迭代

---

### P0 — 阻断级问题

#### F-01: `metadata.openclaw` 应改为 `metadata.clawdbot`
- **发现者**: 专家 1 (OpenClaw 架构)
- **文件**: `skill/SKILL.md` L6-11
- **问题**: ClawHub 发布系统实际解析的 metadata 字段名是 `clawdbot`，而非 `openclaw`。使用 `openclaw` 可能导致运行时需求声明（bins/env）不被解析，在某些环境下 Skill 被错误地标记为不可用。
- **修复建议**: 将 `metadata.openclaw` 改为 `metadata.clawdbot`，或同时声明两者以兼容。发布前用 `clawhub validate` 验证。
- **状态**: ❌ 未修

#### F-02: Skill 目录名与 `name` 字段不匹配
- **发现者**: 专家 1 (OpenClaw 架构)
- **文件**: 目录结构 + `skill/SKILL.md` L2
- **问题**: Agent Skills 标准要求 `name` 字段必须与父目录名一致。当前目录名为 `skill/`，但 `name` 为 `uhomes-student-housing`。如果以 `skill/` 目录直接发布，将不通过 ClawHub 验证。
- **修复建议**: 将 `skill/` 目录重命名为 `uhomes-student-housing/`，或在发布时确保包含正确的目录名映射。
- **状态**: ❌ 未修

#### F-03: web_fetch 对 uhomes.com SPA/SSR 页面的可靠性未验证
- **发现者**: 专家 2 (Claude Code 集成)
- **文件**: `skill/SKILL.md` L57-71
- **问题**: Skill 核心依赖 `web_fetch` 从 uhomes.com 页面抓取房源数据。但 uhomes.com 大概率使用 CSR (Client-Side Rendering) 或重度 JS 渲染，`web_fetch` 只能获取初始 HTML，无法执行 JavaScript。这意味着：
  - 房源列表可能完全不在初始 HTML 中
  - 价格、距离等关键数据由 API 异步加载
  - **Skill 的"正常路径"很可能永远走不通，用户始终看到 fallback**
- **修复建议**:
  1. **立即验证**: 用 `web_fetch` 实际抓取几个 uhomes 页面，确认是否能拿到房源数据
  2. 如果不行，v1.0 应明确定位为 **"导航型 Skill"**（直接给链接），而非"数据型 Skill"（展示房源列表）
  3. 在 SKILL.md 中将 fallback 路径提升为主路径，避免用户看到"加载失败"的体验
- **状态**: ❌ 未修

---

### P1 — 重要问题

#### F-04: `description` 字段过长（超出最佳实践）
- **发现者**: 专家 1 (OpenClaw 架构)
- **文件**: `skill/SKILL.md` L3
- **问题**: 当前 `description` 约 600+ 字符，虽未超过 1024 字符硬限制，但远超推荐的 20-40 词。过长的 description 会：
  - 增加每次对话的 token 消耗（description 被注入 system prompt）
  - 降低触发精度（关键词被稀释）
- **修复建议**: 将 description 精简到约 150 字符的核心描述，将触发关键词列表移到 SKILL.md body 中的"触发规则"段落。
- **状态**: ❌ 未修

#### F-05: `claude-project-instructions.md` 与 `SKILL.md` 内容大量重复
- **发现者**: 专家 2 (Claude Code 集成)
- **文件**: `claude-project-instructions.md` vs `skill/SKILL.md` + `skill/references/*`
- **问题**: `claude-project-instructions.md`（293 行）几乎是 SKILL.md + 所有 references 文件的合并副本。这意味着：
  - 维护负担翻倍：每次更新 Skill 都需同步两份
  - 容易出现不一致（已有不一致，见 F-06）
  - Claude.ai 用户和 OpenClaw 用户可能看到不同版本的指令
- **修复建议**: `claude-project-instructions.md` 应精简为仅包含 Claude.ai 特有的差异化配置（如 `utm_source=claude`），其余内容引用 SKILL.md 为 single source of truth。
- **状态**: ❌ 未修

#### F-06: UTM `utm_source` 在两个文件中不一致
- **发现者**: 专家 4 (增长/归因)
- **文件**: `skill/SKILL.md` L51 vs `claude-project-instructions.md` L35
- **问题**:
  - SKILL.md 使用 `utm_source=openclaw`
  - claude-project-instructions.md 使用 `utm_source=claude`
  - 这是设计意图（区分平台入口），但文档中没有明确说明这一差异的原因
  - utm-config.md §1.2 有解释，但 Skill 文件本身没有注释说明
- **修复建议**: 在两个文件中各添加一行注释，说明 utm_source 值的差异是刻意的平台区分。
- **状态**: ❌ 未修

#### F-07: 缺少 `allowed-tools` 声明
- **发现者**: 专家 1 (OpenClaw 架构)
- **文件**: `skill/SKILL.md` frontmatter
- **问题**: Skill 依赖 `web_fetch`（或 `WebFetch`）工具抓取页面，但 frontmatter 中未声明 `allowed-tools`。在严格权限模式下，用户每次使用 Skill 都需手动批准工具调用，破坏自动化体验。
- **修复建议**: 添加 `allowed-tools: WebFetch WebSearch` 到 frontmatter。
- **状态**: ❌ 未修

#### F-08: Demo 对话中的 URL 是硬编码的虚构数据
- **发现者**: 专家 3 (对话设计)
- **文件**: `skill/demo-conversations.md`
- **问题**: 示例对话中的房源名称、价格、URL 都是手动编写的，部分 URL 可能并不存在于 uhomes.com（如 `iq-manchester`、`chapter-spitalfields` 等 slug 未在 city-index 中验证）。如果用户基于 demo 尝试访问这些链接，可能返回 404。
- **修复建议**:
  1. 验证所有 demo 中的 URL 确实可访问
  2. 或在 demo 文件顶部添加声明："示例中的房源数据为展示目的，实际结果以 uhomes.com 实时数据为准"
- **状态**: ❌ 未修

---

### P2 — 改进级问题

#### F-09: 缺少 `homepage` 字段
- **发现者**: 专家 5 (安全与合规)
- **文件**: `skill/SKILL.md` frontmatter
- **问题**: `homepage` 字段虽然在 `metadata.openclaw` 中存在，但 Agent Skills 标准建议将 `homepage` 作为顶级 frontmatter 字段。ClawHub 安全审查会检查 homepage 并降低"可疑分数"。缺少顶级 homepage 会增加审查阻力。
- **修复建议**: 在 frontmatter 顶级添加 `homepage: https://www.uhomes.com`
- **状态**: ❌ 未修

#### F-10: 未声明外部端点（安全清单缺失）
- **发现者**: 专家 5 (安全与合规)
- **文件**: `skill/SKILL.md`
- **问题**: ClawHub 13 点清单建议 Skill 包含安全清单（Security Manifest），声明：
  - 访问的外部端点（`en.uhomes.com`、`www.uhomes.com`）
  - 读写的文件（无）
  - 使用的环境变量（无）
  未声明可能导致安全审查时被标记为"行为未声明"。
- **修复建议**: 在 SKILL.md 末尾或单独文件中添加安全清单段落。
- **状态**: ❌ 未修

#### F-11: 价格数据硬编码，存在过期风险
- **发现者**: 专家 3 (对话设计)
- **文件**: `skill/references/room-types.md` L56-68, `claude-project-instructions.md` L239-248
- **问题**: 价格参考范围（如 "London En-suite £180-£280"）是硬编码的静态数据。如果 uhomes.com 价格变动（通常每年涨价），Skill 给出的参考价可能误导用户。
- **修复建议**:
  1. 在价格表前添加 "Last updated: 2026-03" 标注
  2. 在 SKILL.md 的工作流中强调"价格仅供参考，以 uhomes.com 实时数据为准"
  3. 设置半年一次的价格更新提醒
- **状态**: ❌ 未修

#### F-12: SKILL.md 总行数偏多（159 行）
- **发现者**: 专家 1 (OpenClaw 架构)
- **文件**: `skill/SKILL.md`
- **问题**: 虽然未超过 500 行上限，但 SKILL.md 包含了较多细节。每次 Skill 被激活时，整个 SKILL.md 都会被注入 context，消耗 token。参考文件的加载指令（L149-159）设计良好，但工作流部分可以更精简。
- **修复建议**: 考虑将 WeChat Mini Program 段落移到 `references/wechat.md`，仅在 SKILL.md 中保留触发条件。
- **状态**: ❌ 未修

#### F-13: 缺少版本变更日志
- **发现者**: 专家 1 (OpenClaw 架构)
- **文件**: 项目根目录
- **问题**: 项目声明为 v1.0.0 但没有 CHANGELOG.md。当 v1.1 或 v2.0 发布时，用户无法了解变更内容。
- **修复建议**: 创建 CHANGELOG.md，至少记录 v1.0.0 的初始功能集。
- **状态**: ❌ 未修

---

### P3 — 建议级问题

#### F-14: 缺少 `license` 字段
- **发现者**: 专家 1 (OpenClaw 架构)
- **文件**: `skill/SKILL.md` frontmatter
- **问题**: 未声明许可证。ClawHub 不强制要求，但声明许可证有助于企业用户评估是否可以使用此 Skill。
- **修复建议**: 添加 `license: MIT` 或适合的许可证。
- **状态**: ❌ 未修

#### F-15: KPI 文档中 OpenClaw 生态数据可能过时
- **发现者**: 专家 4 (增长/归因)
- **文件**: `skill-docs/kpi-expectations.md` L13-17
- **问题**: 文档中引用的 "GitHub Stars ~25 万" 和 "ClawHub 约 3,200+ Skills" 是撰写时的数据快照。OpenClaw 增长很快，这些数据可能在发布时已过时。
- **修复建议**: 标注数据来源日期，或使用"截至 2026 年 3 月"的限定语（已部分做到）。
- **状态**: ⏭️ 已有（部分标注）

#### F-16: API 接口文档中的错误码设计可更细化
- **发现者**: 专家 2 (Claude Code 集成)
- **文件**: `skill-docs/api-spec.md`
- **问题**: API 规范是面向未来 v2.0 的，设计质量很高，但缺少分页参数（offset/cursor）。当某城市有数百个房源时，只靠 limit 可能不够。
- **修复建议**: v2.0 开发时考虑添加分页。不影响当前发布。
- **状态**: ⏭️ 无需改（v2.0 范畴）

#### F-17: 示例对话缺少 web_fetch 失败的完整交互示例
- **发现者**: 专家 3 (对话设计)
- **文件**: `skill/demo-conversations.md`
- **问题**: 示例 5 展示了冷门城市降级，但没有展示"热门城市 + web_fetch 技术失败"的场景。考虑到 F-03 的分析（web_fetch 大概率失败），这个场景反而是最常见的。
- **修复建议**: 添加一个示例，展示热门城市查询时 web_fetch 失败后的优雅降级。
- **状态**: ❌ 未修

---

## 修复优先级排序

| 优先级 | 编号 | 修复内容 | 预计工作量 |
|--------|------|---------|-----------|
| 1 | F-03 | 验证 web_fetch 可行性，调整主路径 | 2h |
| 2 | F-02 | 修正目录名匹配 | 5min |
| 3 | F-01 | metadata 字段名修正 | 5min |
| 4 | F-07 | 添加 allowed-tools | 5min |
| 5 | F-04 | 精简 description | 30min |
| 6 | F-05 | 消除重复内容 | 1h |
| 7 | F-06 | UTM source 添加注释 | 10min |
| 8 | F-08 | 验证 demo URL | 30min |
| 9 | F-09 | 添加顶级 homepage | 5min |
| 10 | F-10 | 添加安全清单 | 20min |

---

## 各专家综合评语

### 专家 1 — OpenClaw Skill 架构专家 (评分: 7.5/10)

> 这是一个结构良好的 Skill，工作流设计清晰，参考文件拆分合理。主要问题集中在 frontmatter 合规性上：`metadata` 字段名、目录名匹配、`allowed-tools` 缺失。这些都是快速修复项。description 过长会影响触发效率，建议精简。整体是 ClawHub 上垂直类 Skill 的优秀范例。

### 专家 2 — Claude Code 集成专家 (评分: 6.5/10)

> **最大风险是 web_fetch 的可行性问题。** 如果 uhomes.com 使用前端渲染（SPA），Skill 的核心功能形同虚设。建议在发布前做一次实际验证。claude-project-instructions.md 的维护成本问题也需要解决——两份几乎相同的文件是技术债务的典型来源。Claude.ai 的 Project Instructions 功能限制了文件大小，当前 293 行的文件已经接近上限。

### 专家 3 — AI 对话设计专家 (评分: 8.5/10)

> 对话设计是本项目的亮点。Slot filling 策略简洁有效（只要求一个必填字段），降级策略层次分明（大学→城市→国家→需求表），中英双语处理规则清晰。Demo 对话覆盖了核心场景。主要建议是增加 web_fetch 失败场景的示例，以及验证 demo 中的 URL 有效性。

### 专家 4 — 增长/归因专家 (评分: 9.0/10)

> UTM + xcode 双重归因设计非常专业，GA4 配置文档可以直接交付给数据团队执行。KPI 框架的分层设计（可见性→参与→转化）和"不应使用的对比基准"段落展现了成熟的增长思维。utm_source 区分 openclaw/claude 的设计是正确的，但需要在 Skill 文件中添加注释避免维护时被误认为 bug。建议 v1.1 增加 utm_term 参数追踪具体搜索关键词。

### 专家 5 — 安全与合规专家 (评分: 7.0/10)

> 没有重大安全问题。Skill 不要求任何 env var 或 API Key（v1.0），不写入文件，不执行脚本。xcode 是公开的合作伙伴码，不构成泄露风险。主要关注点：
> 1. 缺少安全清单（Security Manifest）可能增加 ClawHub 审查阻力
> 2. 应在 SKILL.md 中明确声明"此 Skill 仅读取公开网页数据，不收集或存储用户个人信息"
> 3. demo 中的链接如果指向不存在的页面，可能被安全扫描误判为钓鱼

---

## 下一步行动项

1. **立即**: 用 web_fetch 测试 `https://en.uhomes.com/uk/manchester/university-of-manchester`，确认是否能提取房源数据 (F-03)
2. **发布前**: 修复所有 P0 问题 (F-01, F-02, F-03)
3. **发布前建议**: 修复 P1 问题 (F-04 ~ F-08)
4. **发布后迭代**: 处理 P2/P3 问题

---

*报告生成方式：5 位领域专家模拟评审*
*报告版本：v1.0 | 2026-03-15*
