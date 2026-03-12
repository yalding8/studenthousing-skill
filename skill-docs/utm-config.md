# uhomes Skill — UTM 归因配置需求

> **文档性质**：面向 uhomes.com 数据/增长团队的 UTM 配置要求。本文档描述 `uhomes-student-housing` Skill 上线前须在 GA4（或等效 BI 系统）中完成的配置工作，以实现对 Skill 渠道流量的独立追踪与归因。
>
> **前置条件**：GA4 已部署在 uhomes.com 并正常收集事件数据。

---

## 1. 追踪参数规范

所有从 Skill 生成的 uhomes.com 链接均携带 **uhomes 合作伙伴 xcode** 和 **UTM 参数**，实现双重归因。

### 1.0 合作伙伴 xcode（uhomes 侧归因）

| 参数 | 值 | 说明 |
|------|----|------|
| `xcode` | `000a95434637bdf71105` | uhomes 合作伙伴推荐码，在 uhomes 内部系统中追踪来源 |

> **说明**：`xcode` 是 uhomes 平台原生的合作伙伴追踪参数，与 GA4 UTM 参数互补。UTM 归因于 GA4 侧，xcode 归因于 uhomes CRM/业务系统侧。两者同时使用，确保双侧均可独立统计。

### 1.1 UTM 标准参数值

| 参数 | 值 | 说明 |
|------|----|------|
| `utm_source` | `openclaw` 或 `claude` | 区分两个平台入口（见 §1.2） |
| `utm_medium` | `ai_skill` | 渠道类型：AI Skill |
| `utm_campaign` | `student-housing-skill-v1` | 活动标识，版本升级时更新 |
| `utm_content` | 城市 slug，如 `london`、`manchester`、`sydney` | 区分不同城市流量来源 |

### 1.2 `utm_source` 区分规则

| 用户入口 | `utm_source` 值 |
|----------|-----------------|
| OpenClaw Agent（本地运行） | `openclaw` |
| Claude.ai 网页/App | `claude` |

> **说明**：目前 Skill 无法在运行时动态判断入口，故 v1.0 统一使用 `openclaw`。若后续在 Claude.ai 独立配置 Skill，届时切换为 `claude`。

### 1.3 链接示例

```
# 曼彻斯特大学搜索结果页
https://en.uhomes.com/uk/manchester/university-of-manchester?xcode=000a95434637bdf71105&utm_source=openclaw&utm_medium=ai_skill&utm_campaign=student-housing-skill-v1&utm_content=manchester

# 伦敦城市搜索页
https://en.uhomes.com/uk/london/student-accommodation?xcode=000a95434637bdf71105&utm_source=openclaw&utm_medium=ai_skill&utm_campaign=student-housing-skill-v1&utm_content=london

# 具体房源页
https://en.uhomes.com/uk/manchester/iq-manchester?xcode=000a95434637bdf71105&utm_source=openclaw&utm_medium=ai_skill&utm_campaign=student-housing-skill-v1&utm_content=manchester

# 专属租房需求表（个性化需求用户）
https://www.uhomes.com/referral/demandForm?xcode=000a95434637bdf71105

# 专属网站入口
https://www.uhomes.com?xcode=000a95434637bdf71105
```

---

## 2. GA4 配置要求

### 2.1 自定义渠道分组（Channel Grouping）

在 GA4 → Admin → Data Display → **Channel Groups** 中，新增自定义规则，使 Skill 流量归入独立渠道而非被混入 Referral 或 Other。

**新增规则：**

| 规则名称 | 条件 |
|----------|------|
| AI Skill | `utm_source` 完全匹配 `openclaw` **OR** `utm_source` 完全匹配 `claude` |

> 若不配置此规则，Skill 流量将被归类为 `Unassigned` 或 `Referral`，无法独立分析。

### 2.2 关键事件（Key Events）

确认以下事件已在 GA4 中标记为 Key Event（即转化事件）。若尚未配置，需技术团队添加对应的事件追踪代码：

| 事件名 | 触发条件 | 优先级 |
|--------|----------|--------|
| `booking_initiated` | 用户点击"Book Now"或进入预订流程第一步 | 高 |
| `enquiry_submitted` | 用户提交咨询表单 | 高 |
| `property_viewed` | 用户访问具体房源详情页 | 中 |
| `search_performed` | 用户在 uhomes.com 执行搜索操作 | 中 |

> **说明**：Skill 的核心归因指标是 `booking_initiated` 和 `enquiry_submitted`。如果这两个事件未被追踪，无法评估 Skill 的实际业务贡献。

### 2.3 归因设置

| 设置项 | 建议值 | 说明 |
|--------|--------|------|
| 归因模型 | Data-Driven 或 Last Click | 留学生决策周期较长，Data-Driven 更准确，但 Last Click 更易于解释 |
| 转化回溯窗口 | 30 天 | 用户从点击 Skill 链接到完成预订可能跨越数周 |
| 参与度回溯窗口 | 7 天 | - |

---

## 3. 需配置的报告视图

在 GA4 → **Explorations** 中预建以下分析模板，供运营团队定期查看：

### 报告一：Skill 流量概览（月度）

| 维度 | 指标 |
|------|------|
| `utm_source`、`utm_campaign`、`utm_content`（城市） | 会话数、用户数、新用户占比、平均会话时长 |

**用途**：了解 Skill 带来的整体流量规模及质量趋势。

### 报告二：Skill 转化漏斗

漏斗步骤：
1. 从 `utm_source = openclaw` 进入网站
2. 触发 `property_viewed`
3. 触发 `enquiry_submitted` 或 `booking_initiated`

**用途**：追踪从 Skill 引流到实际预订意向的转化率，识别流失节点。

### 报告三：城市维度拆分

| 维度 | 指标 |
|------|------|
| `utm_content`（城市） | 会话数、`booking_initiated` 次数、转化率 |

**用途**：判断哪些城市的 Skill 推荐效果最好，指导 city-index 的优化优先级。

---

## 4. 验收标准

Skill 正式上线前，数据团队需完成以下验证：

- [ ] 使用测试链接（携带规范 UTM 参数）访问 uhomes.com，在 GA4 Real-time 报告中确认会话来源显示为 `openclaw` / `ai_skill`
- [ ] 确认自定义 Channel Group 规则已生效（测试会话归入"AI Skill"渠道而非 Unassigned）
- [ ] 确认 `booking_initiated` 和 `enquiry_submitted` 事件在测试流程中正常触发
- [ ] 确认归因回溯窗口已设置为 30 天

---

## 5. 已接入的 uhomes 合作渠道

### 5.1 专属租房需求表

| 项目 | 值 |
|------|-----|
| 链接 | `https://www.uhomes.com/referral/demandForm?xcode=000a95434637bdf71105` |
| 用途 | 用户提交个性化租房需求，uhomes 顾问一对一跟进 |
| 触发场景 | 冷门城市降级、用户有特殊需求、用户主动要求顾问协助 |

### 5.2 专属网站入口

| 项目 | 值 |
|------|-----|
| 链接 | `https://www.uhomes.com?xcode=000a95434637bdf71105` |
| 用途 | 通用入口，带合作伙伴归因 |

### 5.3 微信小程序（异乡好居）

| 项目 | 值 |
|------|-----|
| AppID | `wx787e7828382ba76a` |
| 原始 ID | `gh_ac67b37aa05a` |
| 路径 | `pages/index/index?xcode=000a95434637bdf71105` |
| 用途 | 面向中文用户的移动端入口，Skill 在用户提及微信或移动端时推荐 |

---

## 6. 后续扩展（可选，非上线阻断）

| 扩展项 | 说明 | 建议时机 |
|--------|------|----------|
| UTM 专属优惠码 | 为 Skill 渠道用户提供专属折扣码（如 `SKILL10`），实现比 UTM 更精准的转化追踪 | v1.1 稳定后 |
| 跨设备追踪 | 用户在 OpenClaw 上看到推荐，在手机上完成预订——当前 UTM 无法追踪跨设备行为 | 需评估实现成本 |
| Webhook 回调 | 预订完成后通过 Webhook 通知 Skill 维护方，实现预订级别的精准归因 | API 接口就绪后 |
| 小程序数据打通 | 将微信小程序内的 xcode 转化数据与 GA4 UTM 数据关联 | 小程序流量规模起来后 |

---

*文档版本：v1.0 | 2026-03*
*对接联系人：Neil（uhomes 业务侧）*
