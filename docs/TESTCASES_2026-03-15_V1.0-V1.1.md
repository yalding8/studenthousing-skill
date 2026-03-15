# uhomes-student-housing Skill 测试用例

**日期**：2026-03-15
**依据**：`docs/DEVPLAN_2026-03-15_V1.0-V1.1.md`
**测试方法**：对话模拟测试（在 OpenClaw / Claude.ai 中实际运行 Skill 并验证输出）

---

## 测试方法说明

### 测试类型

| 类型 | 说明 | 标记 |
|------|------|------|
| **STRUCT** | 结构验证：检查文件格式、字段、目录名等静态属性 | 可自动化 |
| **FETCH** | 数据抓取验证：web_fetch 是否能成功提取数据 | 半自动化 |
| **CONV** | 对话行为验证：模拟用户输入，检查 AI 输出是否符合 SKILL.md 指令 | 人工验证 |
| **EDGE** | 边界情况验证：异常输入、极端场景 | 人工验证 |

### 判定标准

| 结果 | 含义 |
|------|------|
| PASS | 完全符合预期 |
| PARTIAL | 核心功能正确，但细节有偏差（记录偏差内容） |
| FAIL | 不符合预期，需修复 |
| SKIP | 前置条件未满足，无法测试 |

---

## 阶段 0 测试用例

### T-001: 目录名匹配验证 [DEV-01, STRUCT]

**前置条件**：DEV-01 完成
**测试步骤**：
1. 检查项目目录结构中是否存在 `uhomes-student-housing/` 目录
2. 检查 `uhomes-student-housing/SKILL.md` 中 `name` 字段值
3. 对比目录名与 name 字段是否完全一致

**预期结果**：
- 目录名为 `uhomes-student-housing`
- `name: uhomes-student-housing`
- 两者完全匹配
- `skill/` 目录不再存在

**判定**：目录名 === name 字段 → PASS

---

### T-002: Frontmatter 完整性验证 [DEV-02, DEV-08, STRUCT]

**前置条件**：DEV-02, DEV-08 完成
**测试步骤**：
1. 解析 SKILL.md 的 YAML frontmatter
2. 逐项检查字段

**检查清单**：

| 字段 | 预期值 | 必须 |
|------|--------|------|
| `name` | `uhomes-student-housing` | Y |
| `description` | < 150 字符，包含英文和中文关键词 | Y |
| `version` | `1.0.0` | Y |
| `homepage` | `https://www.uhomes.com` | Y (DEV-08) |
| `allowed-tools` | `WebFetch WebSearch` | Y |
| `argument-hint` | `"[city or university name]"` | Y (DEV-08) |
| `metadata.openclaw.emoji` | `🏠` | Y |
| `metadata.openclaw.homepage` | `https://www.uhomes.com` | Y |
| `metadata.openclaw.requires.bins` | `[]` | Y |
| `metadata.openclaw.requires.env` | `[]` | Y |

**判定**：全部字段存在且值正确 → PASS

---

### T-003: Security Manifest 存在性验证 [DEV-03, STRUCT]

**前置条件**：DEV-03 完成
**测试步骤**：
1. 在 SKILL.md 中搜索 "Security" 或 "Privacy" 段落标题
2. 检查是否包含以下 4 个声明

**检查清单**：

| 声明项 | 关键内容 |
|--------|---------|
| 外部端点 | 提及 `en.uhomes.com` 和 `www.uhomes.com` |
| 环境变量 | 声明 "none required" |
| 文件系统 | 声明 "none" 或 "read-only" |
| 用户数据 | 声明不收集/存储/传输个人信息 |
| xcode 声明 | 声明为官方合作码，不可修改 |

**判定**：5 项全部存在 → PASS

---

### T-004: 数据处理规则存在性验证 [DEV-04, STRUCT]

**前置条件**：DEV-04 完成
**测试步骤**：
1. 在 SKILL.md Step 3 中搜索 "Data processing rules" 或等效段落
2. 检查是否包含 6 项规则

**检查清单**：

| 规则 | 关键内容 |
|------|---------|
| Filtering | 按预算和房型筛选 |
| Sorting | 距离优先，价格次之 |
| Selection | 取前 5 条 |
| Price display | 折扣价展示格式 |
| Distance display | 步行时间优先 |
| Dedup | 同品牌去重 |

**判定**：6 项规则全部存在且表述明确 → PASS

---

### T-005: Demo 数据准确性验证 [DEV-05, FETCH]

**前置条件**：DEV-05 完成
**测试步骤**：
1. 对 demo 示例 1 中出现的每个房源执行 web_fetch 验证
2. 对比 demo 中的价格与 web_fetch 返回的实时价格
3. 验证所有 URL 是否可访问

**示例 1 验证矩阵**：

| 房源名 | Demo 价格 | 实测价格 | 偏差 | URL 可访问 |
|--------|----------|---------|------|-----------|
| Dwell Manchester Student Village | £172/周 | _填入_ | _计算_ | Y/N |
| Dwell MSV South | £204/周 | _填入_ | _计算_ | Y/N |
| Canvas Manchester | £219/周 | _填入_ | _计算_ | Y/N |
| iQ Wilmslow Park | £225/周 | _填入_ | _计算_ | Y/N |

**判定**：
- 价格偏差 ≤ ±10% 且 URL 全部可访问 → PASS
- 价格偏差 ≤ ±20% 或有 1 个 URL 不可访问 → PARTIAL（记录偏差，价格波动可接受）
- 价格偏差 > ±20% 或有 2+ 个 URL 不可访问 → FAIL

---

### T-006: Demo 免责声明验证 [DEV-05, STRUCT]

**前置条件**：DEV-05 完成
**测试步骤**：
1. 检查 demo-conversations.md 文件顶部（标题之后）是否有免责声明
2. 声明中是否提及"快照"、"实时数据"、"uhomes.com"

**判定**：免责声明存在且内容完整 → PASS

---

### T-007: web_fetch 核心功能端到端测试 [FETCH]

**前置条件**：阶段 0 全部完成
**测试步骤**：
1. 使用 web_fetch 抓取 `https://en.uhomes.com/uk/manchester/university-of-manchester`
2. 按 DEV-04 的数据处理规则处理结果
3. 验证最终输出格式

**预期结果**：
- 返回 ≥ 5 条房源
- 每条包含：名称、价格、距离、特色标签、详情页 URL
- URL 格式为 `detail-apartments-{id}`
- 按距离排序

**判定**：符合全部预期 → PASS

---

### T-008: web_fetch 抓取 UCL 页面验证 [FETCH]

**测试步骤**：同 T-007，URL 改为 `https://en.uhomes.com/uk/london/university-college-london`

**额外验证**：
- 价格应为 GBP（£）
- 距离参照物应为 UCL

---

### T-009: web_fetch 抓取澳洲页面验证 [FETCH]

**测试步骤**：同 T-007，URL 改为 `https://en.uhomes.com/au/sydney/university-of-sydney`

**额外验证**：
- 价格应为 AUD（A$）
- 能提取 ≥ 3 条房源

---

## 阶段 0 对话行为测试

### T-010: 英文精准搜索对话测试 [CONV]

**用户输入**：
```
Find student accommodation near University of Manchester, budget £250/week, en-suite preferred
```

**验证点**：

| # | 检查项 | 预期 |
|---|--------|------|
| 1 | 是否展示房源列表 | 展示 3-5 条 |
| 2 | 是否按距离排序 | 最近的在前 |
| 3 | 是否过滤超预算 | 不出现 >£250/周的房源 |
| 4 | 价格格式 | "From £XXX/week"，折扣有划线原价 |
| 5 | URL 格式 | 包含 `detail-apartments-{id}` |
| 6 | URL 参数 | 包含 `xcode=000a95434637bdf71105` 和 `utm_source=openclaw` |
| 7 | 末尾"View all" | 存在且链接正确 |
| 8 | 回复语言 | 英文 |

**判定**：8 项全部通过 → PASS

---

### T-011: 中文精准搜索对话测试 [CONV]

**用户输入**：
```
帮我找UCL附近的学生公寓，预算£350/周以内
```

**验证点**：

| # | 检查项 | 预期 |
|---|--------|------|
| 1 | 回复语言 | 中文 |
| 2 | 房源名称 | 保持英文原名（如 "Scape Bloomsbury"） |
| 3 | 是否过滤超预算 | 不出现 >£350/周的房源 |
| 4 | URL 参数 | `utm_content=london` |
| 5 | 末尾"查看全部" | 存在且链接到 UCL 搜索页 |

**判定**：5 项全部通过 → PASS

---

### T-012: 冷门城市降级测试 [CONV]

**用户输入**：
```
I need housing near University of Groningen in the Netherlands
```

**验证点**：

| # | 检查项 | 预期 |
|---|--------|------|
| 1 | 不展示具体房源列表 | 正确，因为无索引数据 |
| 2 | 提供国家级搜索页链接 | `en.uhomes.com/country/netherlands` 或等效 |
| 3 | 链接包含 xcode + UTM | 是 |
| 4 | 不显示错误信息 | 语气优雅，非"加载失败" |
| 5 | 可选：提供需求表链接 | 若提及特殊需求则应包含 |

**判定**：4/5 通过 → PASS

---

### T-013: 零信息输入追问测试 [CONV]

**用户输入**：
```
帮我找学生公寓
```

**验证点**：

| # | 检查项 | 预期 |
|---|--------|------|
| 1 | 是否追问城市/大学 | 是，且仅追问一次 |
| 2 | 追问语言 | 中文（与用户一致） |
| 3 | 追问是否简洁 | 不超过 2 句话 |
| 4 | 不自行猜测城市 | 不应假设用户在某个城市 |

**判定**：4 项全部通过 → PASS

---

### T-014: 混合语言输入测试 [CONV]

**用户输入**：
```
帮我看看 Manchester 的 en-suite，预算 £200
```

**验证点**：

| # | 检查项 | 预期 |
|---|--------|------|
| 1 | 回复语言 | 中文（主要语言） |
| 2 | 专有名词 | Manchester, en-suite 保持英文 |
| 3 | 是否正确识别城市 | Manchester |
| 4 | 是否正确识别房型 | en-suite |
| 5 | 是否正确识别预算 | £200/周 |

**判定**：5 项全部通过 → PASS

---

### T-015: 需求表触发测试 [CONV]

**用户输入**：
```
我要去慕尼黑工大读研，想找允许养猫的公寓
```

**验证点**：

| # | 检查项 | 预期 |
|---|--------|------|
| 1 | 识别为冷门城市 + 特殊需求 | 是 |
| 2 | 提供需求表链接 | `https://www.uhomes.com/referral/demandForm?xcode=000a95434637bdf71105` |
| 3 | 语言 | 中文 |
| 4 | 可选：提供微信小程序信息 | 中文用户应提供 |

**判定**：3/4 通过 → PASS

---

### T-016: 微信小程序信息测试（中文用户）[CONV]

**用户输入**：
```
uhomes有微信小程序吗？
```

**验证点**：

| # | 检查项 | 预期 |
|---|--------|------|
| 1 | 提供小程序名称 | "异乡好居" |
| 2 | 提供 AppID | `wx787e7828382ba76a` |
| 3 | 提供路径 | 包含 `xcode=000a95434637bdf71105` |

**判定**：3 项全部通过 → PASS

---

### T-017: 微信小程序不应主动推送（英文用户）[CONV]

**用户输入**：
```
What options does uhomes have for University of Manchester?
```

**验证点**：

| # | 检查项 | 预期 |
|---|--------|------|
| 1 | 不主动提及微信小程序 | 正确 |
| 2 | 正常返回房源列表 | 是 |

**判定**：2 项全部通过 → PASS

---

### T-018: 超范围问题重定向测试 [CONV]

**用户输入**：
```
Can you help me apply for a UK student visa?
```

**验证点**：

| # | 检查项 | 预期 |
|---|--------|------|
| 1 | 不尝试回答签证问题 | 正确 |
| 2 | 简短承认超范围 | 是 |
| 3 | 重定向到合适资源 | 提及大学国际处或政府签证门户 |
| 4 | 引导回住宿话题 | 提供可帮助的住宿相关能力 |

**判定**：4 项全部通过 → PASS

---

### T-019: URL 参数完整性验证 [CONV]

**用户输入**：任意成功搜索（如 T-010）

**验证点**：对输出中的每一个 uhomes 链接检查

| 参数 | 预期值 |
|------|--------|
| `xcode` | `000a95434637bdf71105` |
| `utm_source` | `openclaw`（SKILL.md）或 `claude`（claude-project-instructions） |
| `utm_medium` | `ai_skill` |
| `utm_campaign` | `student-housing-skill-v1` |
| `utm_content` | 对应城市 slug（如 `manchester`、`london`） |

**判定**：全部链接的 5 个参数完整且正确 → PASS

---

## 阶段 1 测试用例

### T-020: Description 长度验证 [DEV-06, STRUCT]

**测试步骤**：计算 SKILL.md description 字段的字符数

**判定**：< 150 字符 → PASS

---

### T-021: 触发关键词覆盖验证 [DEV-06, STRUCT]

**测试步骤**：在 SKILL.md body 中搜索 "Trigger Keywords" 段落，检查以下关键词是否存在

**英文关键词**：student housing, student accommodation, student apartment, dorm, PBSA
**中文关键词**：留学公寓, 学生宿舍, 海外租房, 找房, 学生公寓

**判定**：全部关键词存在 → PASS

---

### T-022: claude-project-instructions.md 行数验证 [DEV-07, STRUCT]

**测试步骤**：`wc -l claude-project-instructions.md`

**判定**：< 160 行 → PASS

---

### T-023: claude-project-instructions.md utm_source 验证 [DEV-07, STRUCT]

**测试步骤**：
1. 搜索文件中所有 `utm_source` 出现的位置
2. 确认值为 `claude`（不是 `openclaw`）
3. 搜索文件顶部是否有 HTML 注释说明差异原因

**判定**：utm_source=claude 且有差异注释 → PASS

---

### T-024: Slash 命令直接触发测试 [DEV-08, CONV]

**用户输入**：
```
/uhomes-student-housing Manchester
```

**验证点**：
- Skill 被触发
- Manchester 被识别为搜索目标
- 返回 Manchester 房源列表

**判定**：成功触发并返回 Manchester 结果 → PASS

---

## 阶段 2 测试用例（v1.1 顾问模式）

### T-030: Intent A — 直接搜索意图识别 [DEV-10, CONV]

**用户输入**：
```
Search for en-suite rooms near University of Edinburgh, under £200/week
```

**验证点**：

| # | 检查项 | 预期 |
|---|--------|------|
| 1 | 意图判定 | Intent A（直接搜索） |
| 2 | 行为 | 直接执行 web_fetch → 展示列表 |
| 3 | 不展示城市概况 | 正确（用户已明确需求） |
| 4 | 过滤 >£200 | 是 |

**判定**：4 项全部通过 → PASS

---

### T-031: Intent B — 新生摸底意图识别 [DEV-10, DEV-11, CONV]

**用户输入**：
```
我刚拿到曼彻斯特大学的 offer，9月入学
```

**验证点**：

| # | 检查项 | 预期 |
|---|--------|------|
| 1 | 意图判定 | Intent B（Orientation） |
| 2 | 加载 city-guides.md | 是（曼彻斯特部分） |
| 3 | 提供城市住宿概况 | 提及 City Centre / Fallowfield 区域 |
| 4 | 提供价格参考区间 | 提及 £170-£340/周 en-suite 范围 |
| 5 | 给出建议 | 提及"第一年建议选 City Centre PBSA" 或等效 |
| 6 | 末尾引导 | 问用户是否要搜索 |
| 7 | 不直接甩出房源列表 | 正确 |
| 8 | 语言 | 中文 |

**判定**：7/8 通过 → PASS

---

### T-032: Intent B 后续 — 用户确认搜索 [CONV]

**前置**：T-031 完成后
**用户输入**：
```
好的，帮我搜一下 en-suite £230 以内的
```

**验证点**：

| # | 检查项 | 预期 |
|---|--------|------|
| 1 | 意图切换为 Intent A | 是 |
| 2 | 执行 web_fetch | 是 |
| 3 | 过滤 >£230 | 是 |
| 4 | 展示房源列表 | 3-5 条 |

**判定**：4 项全部通过 → PASS

---

### T-033: Intent C — 知识问答意图识别 [DEV-10, CONV]

**用户输入**：
```
What's the difference between en-suite and studio?
```

**验证点**：

| # | 检查项 | 预期 |
|---|--------|------|
| 1 | 意图判定 | Intent C（知识问答） |
| 2 | 加载 room-types.md | 是 |
| 3 | 回答准确 | en-suite（独卫+共厨）vs studio（全独立） |
| 4 | 末尾引导搜索 | "I can also help you search for..." |
| 5 | 不直接展示房源列表 | 正确 |

**判定**：5 项全部通过 → PASS

---

### T-034: Intent C 中文知识问答 [CONV]

**用户输入**：
```
PBSA 是什么意思？和普通租房有什么区别？
```

**验证点**：

| # | 检查项 | 预期 |
|---|--------|------|
| 1 | 回复语言 | 中文 |
| 2 | 解释 PBSA | Purpose-Built Student Accommodation / 专建学生公寓 |
| 3 | 对比普通租房 | 提及账单全包、安保、管理等差异 |
| 4 | 引导 | 可选，询问是否需要搜索 |

**判定**：3/4 通过 → PASS

---

### T-035: 决策框架 — 按生活方式推荐 [DEV-12, CONV]

**用户输入**：
```
我比较社恐，不太想跟人共用厨房，有什么推荐？
```

**验证点**：

| # | 检查项 | 预期 |
|---|--------|------|
| 1 | 加载 decision-guide.md | 是 |
| 2 | 推荐 Studio | 是（社恐 → 独立空间） |
| 3 | 解释推荐理由 | studio 无需共用任何空间 |
| 4 | 引导提供城市信息 | 如果用户未提供城市，追问一次 |

**判定**：4 项全部通过 → PASS

---

### T-036: 预算不匹配 — 略低 [DEV-12, CONV]

**用户输入**：
```
I want to live near UCL, budget £150/week
```

**验证点**：

| # | 检查项 | 预期 |
|---|--------|------|
| 1 | 识别预算低于伦敦最低价 | 是（伦敦 en-suite 最低 ~£200） |
| 2 | 坦诚告知预算偏紧 | 是 |
| 3 | 建议替代方案 | non-en-suite / shared / 远一点的区域 |
| 4 | 不编造低于实际的价格 | 不应出现 £150 以下的伦敦 en-suite |
| 5 | 仍提供搜索 | 以放松条件搜索 |

**判定**：5 项全部通过 → PASS

---

### T-037: 预算不匹配 — 严重不足 [CONV]

**用户输入**：
```
伦敦有没有 £80/周的学生公寓？
```

**验证点**：

| # | 检查项 | 预期 |
|---|--------|------|
| 1 | 明确告知预算远低于实际 | 是 |
| 2 | 提供伦敦实际价格范围 | 是（£200+ en-suite） |
| 3 | 不编造数据 | 不出现 £80 的伦敦房源 |
| 4 | 建议提交需求表 | 可选，让顾问帮忙找方案 |
| 5 | 语言 | 中文 |

**判定**：4/5 通过 → PASS

---

### T-038: 展示模板 — 折扣价格显示 [DEV-13, CONV]

**用户输入**：
```
Find accommodation near University of Manchester
```

**验证点**：
- 如果返回的房源有折扣（实测 Manchester 有多个折扣房源），输出中应包含划线原价格式
- 示例：`£296/week ~~£305~~` 或 `£225/week (was £230)` 或等效格式

**判定**：折扣房源有原价展示 → PASS

---

### T-039: 展示模板 — 对话引导语 [DEV-13, CONV]

**用户输入**：任意成功搜索

**验证点**：
- 房源列表之后、"View all" 链接之前，存在对话引导语
- 引导语邀请用户继续互动（如"对哪个感兴趣？"或"需要我帮你对比吗？"）

**判定**：引导语存在 → PASS

---

### T-040: 房型对比请求测试 [CONV]

**用户输入**（在 T-010 成功后追问）：
```
Compare Canvas Manchester and iQ Wilmslow Park for me
```

**验证点**：

| # | 检查项 | 预期 |
|---|--------|------|
| 1 | 给出对比分析 | 是（价格、距离、设施对比） |
| 2 | 对比格式 | 表格或并排对比 |
| 3 | 给出倾向性建议 | 可选但推荐 |
| 4 | 两个房源链接 | 均附带 xcode + UTM |

**判定**：3/4 通过 → PASS

---

### T-041: City Guide 内容准确性验证 [DEV-11, STRUCT]

**测试步骤**：对 city-guides.md 中每个城市检查

| 检查项 | 要求 |
|--------|------|
| 关键区域提及 | 至少列出 2 个学生区域 |
| 价格区间 | 与 room-types.md 不矛盾 |
| 推荐建议 | 有明确的"第一年建议" |
| 无事实错误 | 区域名称、交通信息准确 |

**覆盖城市**：London, Manchester, Edinburgh, Sydney, Melbourne, New York, Toronto, Birmingham

**判定**：8 个城市全部通过 → PASS

---

### T-042: Decision Guide 完整性验证 [DEV-12, STRUCT]

**测试步骤**：检查 decision-guide.md 是否包含

| 模块 | 要求 |
|------|------|
| 按画像推荐表 | 覆盖 ≥ 5 种用户画像 |
| 按生活方式建议 | 覆盖"社恐/爱做饭/养宠/省钱/安全" |
| 预算判断规则 | 3 级（合理/偏紧/严重不足） |
| 新生 vs 老生 | 有差异化建议 |

**判定**：4 个模块全部存在 → PASS

---

## 边界情况测试

### T-050: 不存在的大学名称 [EDGE]

**用户输入**：
```
Find housing near Hogwarts University in London
```

**验证点**：
- 不崩溃
- 可能降级到 London 城市页搜索
- 或坦诚表示找不到该大学并建议使用城市搜索

**判定**：优雅处理且不编造 → PASS

---

### T-051: 非学生住宿请求 [EDGE]

**用户输入**：
```
Find me a 3-bedroom family house in London for £2000/month
```

**验证点**：
- 识别为非典型学生住宿需求
- 可以尝试搜索但应注明 uhomes 主要面向学生
- 或引导到需求表

**判定**：不误导用户认为 uhomes 是通用租房平台 → PASS

---

### T-052: 多次追问不重复问 [EDGE]

**对话流程**：
1. 用户："找房"
2. AI 追问城市
3. 用户："嗯"（无效回复）

**验证点**：
- AI 不应再次追问城市（已追问过一次）
- 可以引导用户更具体地回答，或提供热门城市选项

**判定**：不重复追问同一问题 → PASS

---

### T-053: 用户提及竞品平台 [EDGE]

**用户输入**：
```
I saw a nice flat on Rightmove near UCL. Can you find something similar on uhomes?
```

**验证点**：
- 不推荐或引用 Rightmove
- 在 uhomes.com 上搜索 UCL 附近房源
- 可以说"我只能搜索 uhomes.com 上的房源"

**判定**：不引用竞品且成功搜索 uhomes → PASS

---

### T-054: web_fetch 失败降级测试 [EDGE]

**测试方法**：使用一个大概率无结果的 URL（如极冷门城市）
**用户输入**：
```
Find student housing in Inverness, Scotland
```

**验证点**：
- 不显示错误信息
- 提供国家级或城市级搜索页链接
- 语气优雅

**判定**：优雅降级 → PASS

---

### T-055: 连续两次不同城市搜索 [EDGE]

**对话流程**：
1. 用户："Find housing near UoM"（Manchester）
2. AI 返回 Manchester 结果
3. 用户："Now show me London options near UCL"

**验证点**：
- 第二次搜索使用 London/UCL 数据（不混入 Manchester）
- UTM utm_content 切换为 `london`
- 展示全新的房源列表

**判定**：正确切换城市且数据不混淆 → PASS

---

## 测试汇总表

| 阶段 | 用例编号 | 类型 | 对应 DEV | 说明 |
|------|---------|------|---------|------|
| 0 | T-001 | STRUCT | DEV-01 | 目录名匹配 |
| 0 | T-002 | STRUCT | DEV-02/08 | Frontmatter 完整性 |
| 0 | T-003 | STRUCT | DEV-03 | Security Manifest |
| 0 | T-004 | STRUCT | DEV-04 | 数据处理规则 |
| 0 | T-005 | FETCH | DEV-05 | Demo 数据准确性 |
| 0 | T-006 | STRUCT | DEV-05 | Demo 免责声明 |
| 0 | T-007 | FETCH | — | web_fetch Manchester |
| 0 | T-008 | FETCH | — | web_fetch UCL |
| 0 | T-009 | FETCH | — | web_fetch Sydney |
| 0 | T-010 | CONV | — | 英文精准搜索 |
| 0 | T-011 | CONV | — | 中文精准搜索 |
| 0 | T-012 | CONV | — | 冷门城市降级 |
| 0 | T-013 | CONV | — | 零信息追问 |
| 0 | T-014 | CONV | DEV-14 | 混合语言输入 |
| 0 | T-015 | CONV | — | 需求表触发 |
| 0 | T-016 | CONV | — | 微信小程序（中文） |
| 0 | T-017 | CONV | — | 微信不推送（英文） |
| 0 | T-018 | CONV | — | 超范围重定向 |
| 0 | T-019 | CONV | — | URL 参数完整性 |
| 1 | T-020 | STRUCT | DEV-06 | Description 长度 |
| 1 | T-021 | STRUCT | DEV-06 | 触发关键词覆盖 |
| 1 | T-022 | STRUCT | DEV-07 | Instructions 行数 |
| 1 | T-023 | STRUCT | DEV-07 | Instructions utm_source |
| 1 | T-024 | CONV | DEV-08 | Slash 命令触发 |
| 2 | T-030 | CONV | DEV-10 | Intent A 识别 |
| 2 | T-031 | CONV | DEV-10/11 | Intent B 识别 |
| 2 | T-032 | CONV | DEV-10 | Intent B → 搜索 |
| 2 | T-033 | CONV | DEV-10 | Intent C 识别 |
| 2 | T-034 | CONV | DEV-10 | Intent C 中文 |
| 2 | T-035 | CONV | DEV-12 | 生活方式推荐 |
| 2 | T-036 | CONV | DEV-12 | 预算略低 |
| 2 | T-037 | CONV | DEV-12 | 预算严重不足 |
| 2 | T-038 | CONV | DEV-13 | 折扣价格显示 |
| 2 | T-039 | CONV | DEV-13 | 对话引导语 |
| 2 | T-040 | CONV | — | 房型对比 |
| 2 | T-041 | STRUCT | DEV-11 | City Guide 准确性 |
| 2 | T-042 | STRUCT | DEV-12 | Decision Guide 完整性 |
| E | T-050 | EDGE | — | 虚构大学名 |
| E | T-051 | EDGE | — | 非学生住宿 |
| E | T-052 | EDGE | — | 多次追问 |
| E | T-053 | EDGE | — | 提及竞品 |
| E | T-054 | EDGE | — | web_fetch 失败 |
| E | T-055 | EDGE | — | 连续切换城市 |

**总计**：37 个测试用例（8 STRUCT + 3 FETCH + 20 CONV + 6 EDGE）

---

*文档版本：v1.0 | 2026-03-15*
*依据：DEVPLAN_2026-03-15_V1.0-V1.1.md*
