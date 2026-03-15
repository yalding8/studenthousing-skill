# uhomes-student-housing Skill 规划文档

`uhomes-student-housing` 是一个面向 OpenClaw / Claude 生态的 AI Skill，帮助留学生通过 AI 对话找到 [uhomes.com](https://www.uhomes.com) 上的海外住宿，并引导完成预订。

## 文档目录

| 文件 | 内容 | 面向对象 |
|------|------|----------|
| [api-spec.md](./api-spec.md) | 后端搜索接口需求规范（JSON 格式、字段定义、错误码） | uhomes 技术团队 |
| [utm-config.md](./utm-config.md) | GA4 UTM 归因配置要求（参数规范、渠道分组、报告视图） | uhomes 数据/增长团队 |
| [kpi-expectations.md](./kpi-expectations.md) | KPI 预期框架（与 OpenClaw 生态阶段匹配的分层指标） | 业务决策层 |

## 项目背景

- **Skill 名称**：`uhomes-student-housing`
- **目标平台**：OpenClaw (ClawHub) + Claude.ai
- **数据来源**：唯一来源为 [uhomes.com](https://en.uhomes.com)
- **商业目标**：通过 AI 对话为 uhomes.com 带来有意向的住宿搜索流量

## 当前状态

- [x] 整体规划方案完成
- [x] API 接口需求文档
- [x] UTM 归因配置文档（含 xcode 双重归因）
- [x] KPI 预期框架
- [x] SKILL.md v1.0 完成（含核心工作流、降级策略、语言处理）
- [x] 参考文件完成（url-patterns / city-index / room-types / faq / city-guides / decision-guide）
- [x] 示例对话 6 组（英文/中文/对比/顾问摸底/降级/需求表+小程序）— 基于实测数据重写
- [x] uhomes 合作渠道整合（xcode 追踪 / 专属需求表 / 微信小程序）
- [x] Skill 目录名修正（`uhomes-student-housing/`）
- [x] 专家评审通过（技术 8.2 / 战略 6.8→顾问模式升级中）
- [ ] uhomes 侧 GA4 配置验收（utm-config.md §4 验收清单）
- [ ] uhomes 侧确认 xcode 归因正常工作
- [ ] uhomes 技术接口对接（非发布阻断，v2.0 升级项）
- [ ] ClawHub 发布

---

*维护者：Neil @ uhomes*
