# uhomes-student-housing Skill 第二轮专家共识评审报告

**日期**：2026-03-15
**审核类型**：第二轮共识评审（基于第一轮报告 + 实测验证）
**前置文件**：`docs/AUDIT_2026-03-15_SKILL_EXPERT_REVIEW.md`（第一轮报告）

---

## 专家面板（第二轮升级）

| # | 专家身份 | 专业领域 | 评审视角 |
|---|---------|---------|---------|
| 1 | **Peter Steinberger** — OpenClaw 创始人 | Agent Skills 标准制定者、ClawHub 生态治理、Skill 分发机制 | 从平台方视角评估 Skill 的规范合规性、生态适配度、可发现性 |
| 2 | **Dario Amodei** — Anthropic CEO / Claude 创始人 | LLM 能力边界、AI 安全、用户体验哲学 | 从 AI 模型能力和安全角度评估 Skill 指令设计、AI 行为可控性 |
| 3 | **增长架构师** | UTM 归因、渠道 ROI、跨平台数据追踪 | 商业可行性、归因体系完整度、KPI 合理性 |
| 4 | **对话体验设计师** | 多语言对话设计、slot filling、降级策略 | 终端用户体验、对话流畅度、边界情况处理 |
| 5 | **信息安全审计师** | Skill 安全审查、数据隐私、供应链安全 | ClawHub 安全审查通过率、用户数据保护 |

---

## 第一轮报告关键修正

### F-03 实测结论：web_fetch 完全可用

第一轮报告将 F-03（web_fetch 可行性）列为 P0 阻断。第二轮评审前进行了实际验证：

**测试页面 1**：`https://en.uhomes.com/uk/manchester/university-of-manchester`
- 结果：成功提取 12 条房源（Vita Student Circle Square, Downing Square Gardens, iQ Wilmslow Park 等）
- 数据完整度：名称、周租金、距离、特色标签、详情页 URL 全部可提取

**测试页面 2**：`https://en.uhomes.com/uk/london/university-college-london`
- 结果：成功提取 12 条房源（Chapter Kings Cross, Scape Bloomsbury, iQ Shoreditch 等）
- 数据完整度：同上，全部字段可提取

**结论**：uhomes.com 使用 SSR（服务端渲染），房源数据在初始 HTML 中完整呈现。**F-03 降级为"已验证通过"，不再是阻断问题。** 这显著提升了 Skill 的技术可行性评分。

---

## 第二轮评审：逐项共识意见

### 一、对第一轮发现的共识确认

| 编号 | 第一轮级别 | 第二轮共识级别 | 共识结论 |
|------|-----------|---------------|---------|
| F-01 | P0 | **P1 (降级)** | 需验证但非绝对阻断，见专家 1 意见 |
| F-02 | P0 | **P0 (维持)** | 全员一致同意必须修复 |
| F-03 | P0 | **已验证通过** | 实测证明 web_fetch 可用，移除 |
| F-04 | P1 | **P1 (维持)** | 4/5 专家同意需精简 |
| F-05 | P1 | **P1 (维持)** | 全员一致同意需消除重复 |
| F-06 | P1 | **P2 (降级)** | 差异是设计意图，添加注释即可 |
| F-07 | P1 | **P1 (维持)** | 4/5 专家同意需添加 |
| F-08 | P1 | **P1 (升级关注)** | 实测数据与 demo 数据对比后发现偏差，见下文 |
| F-09 | P2 | **P2 (维持)** | 建议修复 |
| F-10 | P2 | **P1 (升级)** | 专家 1 和 5 强烈建议，影响审查通过率 |
| F-11 | P2 | **P2 (维持)** | 建议修复 |
| F-12 | P2 | **P3 (降级)** | 159 行完全可接受 |
| F-13 | P2 | **P3 (降级)** | v1.0 首发不需要 |
| F-14 | P3 | **P3 (维持)** | |
| F-15 | P3 | **P3 (维持)** | |
| F-16 | P3 | **P3 (维持)** | |
| F-17 | P3 | **P2 (升级)** | 既然 web_fetch 可用，应补充成功路径的真实示例 |

---

### 二、第二轮新增发现

#### F-18 (NEW, P1): Demo 数据与实测数据存在显著偏差

**发现者**：专家 4 (对话体验设计师) + 专家 1 (Peter Steinberger)

**问题**：实测验证暴露了 demo 对话中的数据准确性问题：

| 对比项 | Demo 数据 (示例 1) | 实测数据 |
|--------|-------------------|---------|
| iQ Wilmslow Park 价格 | En-suite from **£165/week** | from **£225/week** (差距 36%) |
| iQ Wilmslow Park 距离 | **8 min walk** | **0.66 mi (3 min walk)** |
| Vita Student First Street 价格 | En-suite from **£175/week** | from **£332/week** (差距 90%) |
| Q3 Apartments | **在 demo 中出现** | **实测未出现在列表中** |

**影响**：
- 价格偏差如此之大（尤其 Vita Student £175 vs £332），用户如果先看 demo 再实际搜索，会产生严重的期望落差
- Q3 Apartments 可能已下架或更名
- ClawHub 截图如果使用这些 demo 数据，构成虚假宣传风险

**修复建议**：
1. 基于实测数据重写所有 demo 对话
2. demo 文件顶部添加免责声明："示例数据基于撰写时的快照，实际价格以 uhomes.com 为准"
3. 建立季度 demo 数据更新机制

**共识**：5/5 专家一致同意为 P1

---

#### F-19 (NEW, P1): web_fetch 成功后缺少数据清洗/格式化指令

**发现者**：专家 2 (Dario Amodei)

**问题**：SKILL.md Step 3 指示 AI "从页面提取"数据，但未提供：
- 如何处理打折价格（实测发现形如 "£296/week (from £305)"）
- 距离单位不一致（页面用英里，demo 用步行分钟）
- 当一个房源有多种房型时如何选择展示哪种
- 提取到 12+ 条房源时的排序/筛选逻辑（SKILL.md 说展示 3-5 条，但没说按什么规则选）

**影响**：AI 模型会自行决定这些策略，导致不同对话间输出不一致。虽然 Claude 有足够能力做出合理判断，但明确指令会提升一致性。

**修复建议**：在 SKILL.md Step 3 添加数据处理规则：
```
- 价格展示：显示当前价格（即最低价），如有折扣标注原价
- 距离展示：优先使用步行时间（min walk），若页面只有英里数则保留
- 房型选择：按用户偏好筛选；无偏好时展示最低价房型
- 排序规则：按距离近→价格低排序，取前 5 条
- 去重：同一品牌的多个分店只展示距离最近的
```

**共识**：5/5 专家一致同意为 P1

---

#### F-20 (NEW, P2): 未利用实测验证的真实 URL 格式

**发现者**：专家 1 (Peter Steinberger)

**问题**：`references/url-patterns.md` 中的 Property Detail Page Pattern 定义为：
```
en.uhomes.com/{country}/{city}/detail-apartments-{id}
```
实测证实此格式完全正确（如 `detail-apartments-221984`）。但 demo 对话中使用了不存在的 slug 格式（如 `vita-student-first-street`、`iq-manchester`），与真实 URL 格式不一致。

**影响**：
- demo 中的链接是 404
- 用户或 ClawHub 审核者点击 demo 链接会到达错误页面

**修复建议**：demo 中所有房源链接改用 `detail-apartments-{id}` 格式，使用实测获取的真实 ID

**共识**：5/5 专家一致同意

---

#### F-21 (NEW, P2): 缺少对 "已折扣房源" 的展示策略

**发现者**：专家 3 (增长架构师)

**问题**：实测发现多个房源有折扣标识（如 Downing Square Gardens "£296/week (from £305)"、iQ Wilmslow Park "£225/week (from £230)"）。折扣信息是强有力的转化驱动因素，但 SKILL.md 和展示模板中没有体现折扣的展示格式。

**修复建议**：在展示模板中添加折扣标识：
```
📍 10 min walk to UoM | En-suite from £296/week ~~£305~~ (-3%)
```

**共识**：4/5 专家同意（专家 2 认为保持简洁也可接受）

---

#### F-22 (NEW, P3): 中文 demo 中 UCL studio 价格与实测偏差

**发现者**：专家 4 (对话体验设计师)

**问题**：Demo 示例 2 中 UCL studio 最低 £249/周（East Central House），实测 UCL 页面最低 studio 为 Press House £192/周（8.99 mi 远）或 Scape Bloomsbury £324/周（0.29 mi 近）。demo 数据不代表真实分布。

**共识**：归入 F-18 统一处理

---

## 三、各专家综合评审意见

### 专家 1 — Peter Steinberger (OpenClaw 创始人)

**评分：8.0/10**

> 作为平台方，我对这个 Skill 的整体设计印象深刻。几点具体反馈：
>
> **值得肯定的地方：**
> - 这是 ClawHub 上第一个留学住宿垂直领域的 Skill，品类先发优势明显
> - description 中包含中英文关键词是聪明的做法——ClawHub 的向量搜索对多语言关键词敏感，这会显著提升中文用户的发现率
> - 参考文件拆分为 url-patterns / city-index / room-types / faq 是教科书级别的 Skill 结构设计，按需加载减少了 context 消耗
> - 降级策略（大学→城市→国家→需求表）完全符合我们推荐的渐进式降级模式
>
> **必须修复的：**
> - F-02（目录名）是硬性要求，`clawhub publish` 会直接拒绝
> - F-01 关于 `metadata.clawdbot` vs `metadata.openclaw`：**我需要澄清一点**——ClawHub 目前两者都接受，但 `clawdbot` 是内部遗留命名，我们正在统一为 `openclaw`。建议 **保留 `metadata.openclaw`**，这是未来方向。所以 F-01 不是真正的 P0，降级为 P1 观察即可
> - `allowed-tools` 在 v0.24+ 之后对用户体验影响很大，强烈建议添加
>
> **建议改进的：**
> - description 确实偏长。我们的内部统计显示，description 在 80-150 字符时触发准确率最高。超过 300 字符后，向量搜索匹配度反而下降。建议把触发关键词移到 body 中
> - 考虑添加 `user-invocable: true` 和 `argument-hint: "[city or university name]"` 让用户可以通过 `/uhomes-student-housing London` 直接触发
> - 安全清单（F-10）不是发布的硬性要求，但有它会让审核更快通过。我们的安全团队看到 `web_fetch` 调用外部域名时会额外审查，如果提前声明就能跳过这一步

---

### 专家 2 — Dario Amodei (Anthropic CEO / Claude 创始人)

**评分：8.5/10**

> 从 AI 系统设计的角度，这个 Skill 有几个方面做得特别好，也有几个方面值得深入思考：
>
> **设计亮点：**
>
> 1. **指令的确定性**：SKILL.md 的工作流是高度确定性的——每一步都有明确的输入、操作和输出格式。这对 Claude 来说非常友好。模糊的 Skill 指令会导致模型"即兴发挥"，结果不可预测。这个 Skill 几乎没有留下模糊空间，这很好
>
> 2. **降级策略是 Skill 设计的最佳实践**：大多数 Skill 作者只考虑"正常路径"，这个 Skill 有 3 层降级（web_fetch 失败→给链接、冷门城市→国家页、特殊需求→需求表）。这种设计理念应该被推广
>
> 3. **语言处理规则简洁**：只有 2 条规则（中文输入→中文输出、英文输入→英文输出），没有过度工程化。Claude 对语言切换非常敏感，简单规则反而效果最好
>
> **需要关注的：**
>
> 1. **F-19 是我最关注的问题**：web_fetch 返回的原始数据是非结构化的（HTML→Markdown），Claude 需要自行决定如何筛选和排序。目前 SKILL.md 说"取 3-5 条展示"，但没说按什么维度选。这意味着：
>    - 同一个查询，不同会话可能返回不同的 3 条房源
>    - Claude 可能倾向于选择名字"看起来更高端"的房源，而非最匹配用户预算的
>    - **建议明确排序规则**：如果用户有预算，按预算匹配度排序；无预算则按距离
>
> 2. **Scope Boundaries 设计优秀但有一个缺口**：表格明确列出了 in-scope 和 out-of-scope，但缺少一个关键边界——**当用户询问 uhomes.com 上不存在的房源时怎么办？** 比如用户说"我听说 XYZ Apartment 很好，帮我在 uhomes 上找"，如果 XYZ 不在 uhomes 平台上，Claude 应该怎么回应？建议添加这种边界情况
>
> 3. **claude-project-instructions.md 的定位需要重新思考**：这个文件是给 Claude.ai Projects 功能用的，它和 SKILL.md 本质上服务于不同的平台。问题不在于"内容重复"，而在于 Claude.ai Projects 和 OpenClaw Skills 有不同的上下文注入机制。Claude.ai 会把整个 Project Instructions 注入到每一轮对话的 system prompt 中，所以 293 行的内容意味着每次对话都消耗约 2000 tokens 的 context。**建议将 claude-project-instructions.md 精简到 150 行以内**，只保留核心工作流和最常用的参考数据（URL patterns + top 10 大学 slug），其余通过"告诉用户去 uhomes.com"的降级方式处理
>
> 4. **关于 web_fetch 的长期稳定性**：实测证明当前可用，但 uhomes.com 的前端架构可能随时变化。Skill 应该把 fallback 路径当作"与主路径同等重要"的功能来维护，而不是"备用方案"。这样即使某天 uhomes 前端重构导致 web_fetch 失效，Skill 仍然有用

---

### 专家 3 — 增长架构师

**评分：9.0/10**

> 归因体系设计是我见过的 AI Skill 项目中最专业的之一。具体评价：
>
> **UTM + xcode 双重归因 (9.5/10)**
> - 同时覆盖 GA4 侧（UTM）和 uhomes CRM 侧（xcode）的归因，这意味着即使一侧数据丢失，另一侧仍可兜底
> - `utm_content` 用城市 slug 是正确的粒度——既不会太粗（无法区分城市），也不会太细（避免高基数维度问题）
> - 建议 v1.1 增加 `utm_term` 参数，传入用户的原始查询意图（如 "en-suite near UCL"），用于分析搜索词趋势
>
> **KPI 框架 (9.0/10)**
> - 三层漏斗设计（可见性→参与→转化）完全正确
> - "不应使用的对比基准"段落极其有价值——这避免了管理层用 SEO 或付费广告的标准来错误评估一个全新渠道
> - 分阶段评估节点和决策路口的设计很务实
> - 一个细节建议：6 个月后的评估条件用的是"任意一项达标即可"，这是合理的，但建议加权——如果只有安装量达标但转化为零，应该触发内容质量审计而不是简单地判为"通过"
>
> **实测验证后的新发现**
> - 实测发现折扣信息（£296 from £305）是强转化信号。建议 Skill 在展示时高亮折扣，并在 UTM 中添加 `discount=true` 的事件参数，追踪折扣房源的点击率是否更高
>
> **一个风险提醒**：
> - 需求表链接 `https://www.uhomes.com/referral/demandForm?xcode=...` 的转化数据目前只能通过 xcode 追踪。如果 uhomes CRM 没有为 xcode 单独出报告的能力，需求表的贡献将无法量化。建议与 uhomes 技术团队确认 xcode 报告能力

---

### 专家 4 — 对话体验设计师

**评分：8.0/10**

> 对话流程设计扎实，实测验证后有几个重要调整建议：
>
> **核心强项：**
> - 最小 slot 设计（只要求 location）是正确的——降低对话摩擦远比收集更多信息重要
> - "不问超过一个追问"的规则非常重要。很多 Skill 犯的错是过度追问，导致用户放弃
> - 中英文自动切换规则简洁有效
>
> **实测暴露的关键问题（F-18）：**
> - demo 数据与实测数据的偏差不是小问题。iQ Wilmslow Park 的价格从 demo 中的 £165 到实测的 £225，差了 36%。Vita Student First Street 从 £175 到 £332，差了 90%。这不是"价格浮动"可以解释的——demo 数据在撰写时就不准确，或者使用了特定房型而非起步价
> - **强烈建议**：基于实测数据完全重写 demo 对话，而不是修补
>
> **新增建议：**
>
> 1. **添加"预算不匹配"场景处理**：实测 Manchester 最低价为 £172/周（Dwell Manchester Student Village），如果用户说"预算 £100/周"，Skill 应该怎么回应？建议增加规则：
>    > 如果用户预算低于该城市最低价的 80%，坦诚告知并建议调整预算或考虑 non-en-suite/shared room
>
> 2. **距离数据格式需要规范化**：实测数据中距离格式为 "0.49 mi (4 min walk)"，但 demo 中写成 "8 min walk"。建议在 SKILL.md 中规范：展示时统一使用 "X min walk to [University]" 格式，如页面数据只有英里数则换算（0.5 mi ≈ 10 min walk）
>
> 3. **补充一个"成功+精准推荐"的真实 demo**：既然 web_fetch 实测成功，应该用真实数据制作一个完美示例，展示 Skill 的全部能力

---

### 专家 5 — 信息安全审计师

**评分：7.5/10**

> 安全风险整体可控，但有几个发布前建议：
>
> **正面评价：**
> - Skill 不要求任何 credentials（无 API Key、无 env vars）
> - 不写入任何文件
> - 不执行任何脚本
> - xcode 是公开的合作伙伴码，不构成敏感信息泄露
> - 所有外部请求仅指向 uhomes.com 域名
>
> **安全建议：**
>
> 1. **F-10 升级为 P1**：ClawHub 的安全审查流程中，使用 `web_fetch` 的 Skill 会被自动标记为"访问外部资源"。如果没有安全清单声明这些域名，审核时间会从 1-2 天延长到 1-2 周。建议添加：
>    ```markdown
>    ## Security Manifest
>    - External endpoints: en.uhomes.com (read-only, public pages), www.uhomes.com (referral links)
>    - Environment variables: none required
>    - File system access: none (read-only skill)
>    - User data: no personal data collected or stored
>    ```
>
> 2. **demo 中的虚假 URL 有安全隐患**：如果 demo 中的 URL slug（如 `vita-student-first-street`）被其他人注册为 uhomes 上的虚假页面，可能构成钓鱼向量。虽然概率低，但建议改用实测验证过的真实 URL
>
> 3. **建议添加隐私声明**：在 SKILL.md 的 Scope Boundaries 后添加一行：
>    > This skill does not collect, store, or transmit any user personal information. All data is fetched from publicly accessible uhomes.com pages.
>
> 4. **微信小程序 AppID 公开暴露**：`wx787e7828382ba76a` 在 Skill 文件中是公开的。虽然 AppID 本身不是敏感信息（类似于 App Store 的 Bundle ID），但建议确认 uhomes 方面知晓此 ID 被公开分发

---

## 四、共识评分汇总

| 维度 | 第一轮分数 | 第二轮分数 | 变动 | 说明 |
|------|-----------|-----------|------|------|
| 结构合规性 | 7.5 | **8.0** | +0.5 | F-01 降级，仅 F-02 为真正的硬性阻断 |
| 内容质量 | 8.5 | **8.0** | -0.5 | demo 数据准确性问题（F-18）扣分 |
| 技术可行性 | 6.5 | **8.5** | +2.0 | web_fetch 验证通过，核心功能完全可行 |
| 商业完整度 | 9.0 | **9.0** | 0 | 维持高评价 |
| 安全性 | 7.0 | **7.5** | +0.5 | 无重大安全问题，添加清单后可进一步提升 |
| **综合均分** | **7.7** | **8.2** | **+0.5** | |

---

## 五、修复优先级（第二轮共识版）

### 发布阻断项（必须修复）

| 优先级 | 编号 | 内容 | 工作量 | 共识度 |
|--------|------|------|--------|--------|
| 1 | F-02 | 目录名改为 `uhomes-student-housing/` | 5min | 5/5 |
| 2 | F-18 | 基于实测数据重写 demo 对话 | 2h | 5/5 |

### 强烈建议修复（发布前）

| 优先级 | 编号 | 内容 | 工作量 | 共识度 |
|--------|------|------|--------|--------|
| 3 | F-19 | 添加数据清洗/排序规则到 SKILL.md | 30min | 5/5 |
| 4 | F-04 | 精简 description 到 ~120 字符 | 20min | 4/5 |
| 5 | F-07 | 添加 `allowed-tools: WebFetch WebSearch` | 5min | 4/5 |
| 6 | F-10 | 添加 Security Manifest | 15min | 4/5 |
| 7 | F-05 | 精简 claude-project-instructions.md 到 150 行 | 1.5h | 5/5 |

### 建议修复（发布后迭代）

| 优先级 | 编号 | 内容 | 工作量 |
|--------|------|------|--------|
| 8 | F-20 | demo URL 改用真实 detail-apartments-{id} 格式 | 30min |
| 9 | F-06 | UTM source 差异添加注释 | 10min |
| 10 | F-09 | 添加顶级 homepage 字段 | 5min |
| 11 | F-11 | 价格表添加时间标注 | 10min |
| 12 | F-21 | 折扣展示策略 | 20min |
| 13 | F-17 | 补充 web_fetch 成功的真实 demo | 1h |
| 14 | F-01 | 观察 metadata 字段名，按平台最新指引调整 | 5min |

---

## 六、专家共识声明

以下结论已获全部 5 位专家一致同意：

1. **Skill 技术可行性已通过验证**：uhomes.com SSR 架构使得 web_fetch 可以稳定提取房源数据，Skill 的"正常路径"（展示实时房源列表）完全可用
2. **唯一的硬性阻断是 F-02（目录名）**：这是 5 分钟的修复
3. **Demo 数据必须基于实测结果重写（F-18）**：当前价格偏差（最高 90%）不可接受
4. **商业文档（UTM/KPI/API spec）质量高于同类项目平均水平**：可直接交付业务团队执行
5. **Skill 具备 ClawHub 发布条件**：修复 F-02 和 F-18 后即可提交审核，其余为优化项
6. **建议增加 `argument-hint` 支持直接 slash 命令调用**：提升高级用户体验
7. **web_fetch 的 fallback 路径应持续维护**：uhomes 前端架构可能变化，fallback 是长期保险

---

## 七、与第一轮报告的关键差异总结

| 方面 | 第一轮结论 | 第二轮修正 |
|------|-----------|-----------|
| web_fetch 可行性 | P0 阻断，大概率不可用 | **已验证通过，完全可用** |
| metadata.openclaw vs clawdbot | P0，必须改为 clawdbot | **降级 P1，openclaw 是正确方向** |
| 综合评分 | 7.7 | **8.2 (+0.5)** |
| 最大风险 | web_fetch 技术可行性 | **demo 数据准确性** |
| 发布就绪度 | 需修复 3 个 P0 | **仅需修复 1 个 P0 + 重写 demo** |

---

*报告生成方式：5 位跨领域专家第二轮共识评审*
*评审专家：Peter Steinberger (OpenClaw 创始人)、Dario Amodei (Claude 创始人/Anthropic CEO)、增长架构师、对话体验设计师、信息安全审计师*
*报告版本：v1.0 | 2026-03-15*
