# 学生住宿推荐算法 — 专家组讨论纪要

**日期**：2026-03-16
**议题**：uhomes-student-housing Skill 的推荐排序算法应该如何设计？
**背景**：当前使用固定权重线性加分模型（distance 40 + budget 25 + must_stay 20 + quality 15 = 100），需要评估其合理性并探讨优化方向。

---

## 专家面板

| # | 专家 | 身份与专长 | 讨论视角 |
|---|------|----------|---------|
| 1 | **张一鸣** | 字节跳动创始人，构建了全球最大的推荐系统（今日头条/抖音/TikTok） | 推荐系统工程化、冷启动、用户画像、兴趣探索与利用平衡 |
| 2 | **Elon Musk** | X/Twitter 推荐算法开源者，Tesla FSD 多目标决策系统设计者 | 推荐透明性、开源算法哲学、多目标优化、简单性原则 |
| 3 | **Jure Leskovec** | Stanford 教授，Pinterest 首席科学家，图神经网络推荐（PinSage）发明者 | 图结构推荐、嵌入学习、学术理论与工业实践结合 |
| 4 | **Ed Chi** | Google DeepMind 推荐研究负责人，Wide & Deep / DCN 模型共同发明者 | 深度学习推荐、特征交叉、大规模系统设计 |
| 5 | **项亮** | 《推荐系统实践》作者，前微软/Hulu 推荐算法负责人，中文推荐系统领域权威 | 推荐系统落地实践、冷启动策略、中国市场推荐系统设计 |

---

## 第一环节：现有算法诊断

### 当前算法

```
score = distance_score(0-40) + budget_score(0-25) + must_stay_bonus(0-20) + quality_score(0-15)
```

### 各专家诊断意见

#### 张一鸣

> 先说结论：**这个算法在当前阶段是合理的，但它的根本问题不在权重，而在于没有反馈闭环。**
>
> 在字节，我们做推荐的第一性原理是：**推荐系统的本质不是排序算法，而是一个持续学习的系统。** 没有用户行为反馈的推荐，本质上是编辑推荐——你在替用户做判断。
>
> 你目前的场景有一个根本约束：**没有用户行为数据**。Skill 在对话中运行，没有点击率、停留时长、预订转化率这些信号。这意味着协同过滤、CTR 预估、强化学习这些主流推荐方法全部不可用。
>
> 在这个约束下，固定权重的线性模型是**正确的选择**。但我有三个具体问题：
>
> **问题 1：权重是怎么来的？**
>
> distance_score 占 40%，budget_score 占 25%——这个比例的依据是什么？是直觉还是数据？如果是直觉，那就需要验证。我在字节最早做头条推荐的时候，初始权重也是拍脑袋定的，但上线第一周就根据 A/B 测试调了 3 次。
>
> 你可以做一个简单验证：拿 5 个真实用户场景，手动跑一遍算法，看排序结果是否符合一个"有经验的留学顾问"的判断。如果不符合，调权重；如果符合，你的直觉就对了。
>
> **问题 2：距离分的阶梯太粗。**
>
> 5 分钟步行得 40 分，10 分钟得 30 分——差了 10 分。但 5 分钟和 10 分钟的体验差距，远不如 10 分钟和 30 分钟的差距大。后者只差 20 分。建议用**非线性衰减**：
>
> ```
> distance_score = 40 × e^(-0.05 × walk_minutes)
> ```
>
> 这让 5 分钟 ≈ 31 分，10 分钟 ≈ 24 分，15 分钟 ≈ 19 分，30 分钟 ≈ 9 分——更符合真实体验曲线。
>
> **问题 3：缺少"多样性"机制。**
>
> 当前算法会让排名前 5 全部是高分但同质化的房源（比如全是距离近的 PBSA）。但一个好的推荐应该包含一些差异化选项——比如一个稍远但便宜很多的、一个评分特别高的、一个有独特卖点的。抖音的推荐之所以让人刷不停，不是因为每条都精准，而是因为有**多样性**。
>
> 建议在取 top 5 时加入 MMR（Maximal Marginal Relevance）：
> ```
> 选第 1 个：纯按 score 最高
> 选第 2-5 个：score × λ + diversity × (1-λ)
> ```
> 其中 diversity 衡量候选项与已选项的差异度（价格区间不同、房型不同、距离区间不同）。λ = 0.7 是一个合理的起点。

---

#### Elon Musk

> I'll cut straight to the point.
>
> **The algorithm is overthinking it.** When we open-sourced the Twitter recommendation algorithm, the biggest lesson was that **simple, transparent algorithms outperform complex ones when you don't have enough data to train the complex ones.** You have zero training data. So keep it simple.
>
> Three specific comments:
>
> **1. The Must-Stay bonus is the most important signal — and it's underweighted.**
>
> The Must-Stay list is uhomes' editorial curation based on actual booking data, user reviews, and partner quality. It represents **institutional knowledge** that your algorithm doesn't have. You're giving it 20 out of 100 — only 20%. I'd argue it should be **the primary signal, not a bonus.**
>
> Think of it like this: if a Michelin guide says a restaurant is 3-star, you don't then re-rank it based on your own distance calculation. The Michelin rating IS the most valuable signal.
>
> I'd restructure:
> ```
> If Must-Stay data available:
>   primary_sort = must_stay_rank (1-10)
>   secondary_sort = distance_to_university
>   Show top 3 Must-Stay + top 2 non-Must-Stay (for diversity)
>
> If Must-Stay data NOT available:
>   fall back to current distance + budget + quality scoring
> ```
>
> **2. Make the algorithm transparent to the user.**
>
> When we open-sourced Twitter's algorithm, engagement went UP, not down. People trust what they can see. Your Skill should tell users **why** each property is recommended:
>
> - "🏆 #3 on uhomes Must-Stay list — rated 4.7 by 210 students"
> - "📍 Closest to campus — 3 min walk"
> - "💰 Best value in your budget — £172/week"
>
> This isn't just UX — it makes the algorithm self-correcting. When users see the reasoning, they'll tell you when it's wrong.
>
> **3. Don't build for scale you don't have.**
>
> You have ~12 properties per university page. You're not ranking 10 million items. With 12 candidates and no behavioral data, a simple rule-based system will outperform any ML model. Save the ML for when you have 10,000+ interactions.

---

#### Jure Leskovec (Stanford / Pinterest)

> From an academic perspective, your problem is a **cold-start item ranking problem with side information**. Let me frame it precisely.
>
> **Problem formulation:**
> - Items: ~12 properties per query
> - User features: location (university), budget, room type preference, language
> - Item features: price, distance, rating, review count, Must-Stay rank, highlights
> - Objective: rank items to maximize booking probability
> - Constraint: no historical interaction data
>
> **Why your current approach is the right family of solutions:**
>
> In recommendation systems, we broadly have:
> 1. Collaborative filtering (needs user-item interactions → you don't have)
> 2. Content-based filtering (uses item features → you have this)
> 3. Knowledge-based (uses explicit rules → you have this)
> 4. Hybrid approaches
>
> Your linear scoring model is a content-based/knowledge-based hybrid. This is correct for cold-start.
>
> **Where I see room for improvement:**
>
> **A. Feature interactions matter.**
>
> Your current model is purely additive: `score = f(distance) + f(budget) + f(must_stay) + f(quality)`. This misses **interactions** between features. For example:
>
> - A student with a tight budget (£150/week) cares more about price than distance
> - A student with a generous budget (£400/week) cares more about quality and distance
> - A first-year student values Must-Stay more (trust signal) than a returning student
>
> You can model this without ML by making weights **context-dependent**:
>
> ```
> If budget is tight (< 80% of city median):
>   budget_weight = 35, distance_weight = 30, must_stay = 20, quality = 15
>
> If budget is generous (> 120% of city median):
>   budget_weight = 15, distance_weight = 35, must_stay = 20, quality = 30
>
> Default:
>   budget_weight = 25, distance_weight = 40, must_stay = 20, quality = 15
> ```
>
> **B. The Must-Stay rank should be normalized.**
>
> Your current model gives rank 1-3 = 20 points, 4-7 = 15, 8-10 = 10. But the difference between rank 1 and rank 3 is likely larger than between rank 8 and rank 10. Use:
>
> ```
> must_stay_score = 20 × (1 - (rank - 1) / 10)
> ```
>
> This gives: rank 1 = 20, rank 2 = 18, rank 5 = 12, rank 10 = 2 — a smooth decay.
>
> **C. When you get data, here's the upgrade path.**
>
> Once uhomes can share click-through or booking data:
> 1. Phase 1: Use the data to **calibrate weights** (simple logistic regression)
> 2. Phase 2: Move to a **learning-to-rank** model (LambdaMART)
> 3. Phase 3: If property catalog grows, use **embedding-based retrieval** (two-tower model)
>
> But phase 1-3 all require data you don't have yet. Your current approach is the correct pre-data solution.

---

#### Ed Chi (Google DeepMind)

> At Google, we've learned that recommendation systems go through predictable maturity stages. Your system is at Stage 0 — and that's fine. Let me explain the stages and where your decisions matter.
>
> **Stage 0: Rule-based (you are here)**
> - No behavioral data
> - Hand-tuned weights
> - Works when candidate set is small (< 100 items)
>
> **Stage 1: Feature-weighted (your next step)**
> - Use A/B testing or offline evaluation to tune weights
> - Still linear model but data-informed
>
> **Stage 2: ML ranking (requires data)**
> - CTR prediction or booking probability estimation
> - Wide & Deep or DCN for feature crossing
>
> **Stage 3: Multi-objective optimization (mature)**
> - Balance relevance, diversity, freshness, monetization
> - Reinforcement learning for long-term engagement
>
> **My specific recommendations for Stage 0:**
>
> **1. Separate recall from ranking.**
>
> Right now you do everything in one pass: fetch 12 properties → score → take top 5. This is fine for 12 items but won't scale. Better structure:
>
> ```
> Recall stage: Fetch A (university page) + Fetch B (Must-Stay)
>   → Merge into candidate pool (remove duplicates)
>   → Pool size: ~15-20 items
>
> Ranking stage: Score each candidate
>   → Apply scoring algorithm
>
> Re-ranking stage: Apply diversity & business rules
>   → Ensure mix of price ranges
>   → Ensure Must-Stay properties are represented
>   → Take top 5
> ```
>
> **2. Your quality_score needs a Bayesian correction.**
>
> A property with 5.0 rating and 2 reviews is NOT better than 4.5 rating with 200 reviews. Use a Bayesian average:
>
> ```
> adjusted_rating = (C × M + R × N) / (C + N)
>
> Where:
>   R = property's average rating
>   N = number of reviews
>   C = confidence threshold (e.g., 20)
>   M = global average rating across all properties (e.g., 4.2)
> ```
>
> With C=20, M=4.2:
> - Property A: 5.0 stars, 2 reviews → adjusted = (20×4.2 + 5.0×2) / 22 = 4.27
> - Property B: 4.5 stars, 200 reviews → adjusted = (20×4.2 + 4.5×200) / 220 = 4.47
>
> Property B correctly ranks higher. This is the same method IMDB uses for its Top 250.
>
> **3. Add a "freshness" signal.**
>
> Properties that were recently updated or have "new" or "2025" tags are likely actively managed. This correlates with better service quality. Give a small bonus (5 points) for properties with freshness indicators.

---

#### 项亮（《推荐系统实践》作者）

> 我从中国留学生使用场景出发来分析这个问题。
>
> **核心观点：这不是一个经典推荐问题，而是一个"辅助决策"问题。**
>
> 经典推荐（抖音、淘宝、Netflix）的目标是**最大化点击/观看/购买概率**。但学生找房不是"刷"出来的——这是一个**高参与度、低频次、高风险**的决策场景。一个学生一年只找一次房，错误的推荐可能导致一整年住得不开心。
>
> 这改变了算法设计的根本目标：
>
> | | 信息流推荐（抖音） | 住宿推荐（uhomes Skill） |
> |--|--|--|
> | 目标 | 最大化点击率/停留时长 | 最大化用户满意度/决策质量 |
> | 频次 | 每天数百次 | 一年一次 |
> | 容错性 | 推错一条无所谓 | 推错影响很大 |
> | 用户心态 | 被动消费 | 主动决策 |
> | 反馈速度 | 秒级（看不看就知道了） | 月级（住进去才知道好不好） |
>
> **基于此，我的建议是：**
>
> **1. 不要追求"精确排序"，而要追求"候选集质量"。**
>
> 你推 5 个房源给用户，用户不会严格按照你的排序来选。他们会自己比较。所以重要的不是第 1 名和第 2 名的顺序，而是这 5 个里面有没有一个真正适合他的。
>
> 这意味着 **多样性比精度更重要**。我建议的 top 5 构成：
>
> ```
> 位置 1: 综合最优（score 最高）
> 位置 2: Must-Stay 最高排名（如果不在位置 1）
> 位置 3: 距离最近（如果不在位置 1-2）
> 位置 4: 价格最低（给用户一个"底线选项"）
> 位置 5: 评分最高（给用户一个"品质选项"）
> ```
>
> 每个位置用不同的排序维度选出，确保 5 个房源覆盖了用户可能的不同偏好。这比纯 score 排序产生的同质化结果好得多。
>
> **2. 利用"解释"来提升信任。**
>
> 中国留学生在做租房决策时，一个非常典型的行为是**去小红书/知乎搜"XXX 公寓怎么样"**。他们不信算法，但信"有人推荐过"。
>
> Must-Stay 榜单就是"有人推荐过"的数字化形式。所以 Must-Stay 的价值不在于给 20 分加成，而在于**提供一个可解释的推荐理由**。你应该把它当作解释信号而非排序信号：
>
> - "🏆 入选 uhomes 曼彻斯特必住榜第 3 名（基于 236 条学生评价）"
>
> 这一句话比 20 分的隐式加权有效 10 倍。
>
> **3. 考虑"负面信号"筛除。**
>
> 当前算法只有正面加分，没有负面筛除。但对留学生来说，某些负面信号是一票否决的：
>
> - 评分 < 3.5（通常意味着严重管理问题）
> - 距离 > 60 分钟（不切实际）
> - 价格是预算的 2 倍以上（浪费推荐位）
>
> 这些应该在 scoring 之前就被硬性排除，而不是通过低分自然沉底。

---

## 第二环节：专家共识讨论

### 议题 1：权重分配是否合理？

| 专家 | 观点 |
|------|------|
| 张一鸣 | 距离权重合理但应改为非线性衰减；需要验证 |
| Musk | Must-Stay 严重低估，应为主信号而非 bonus |
| Leskovec | 权重应随用户 context 动态调整（预算紧 vs 宽裕） |
| Ed Chi | 结构合理但 quality_score 需要贝叶斯修正 |
| 项亮 | 权重问题不大，核心是多样性不足 |

**共识**：固定权重作为 Stage 0 可接受，但需要 3 个改进：
1. Must-Stay 作为推荐理由（解释性）的价值大于作为排序权重的价值
2. Quality score 需要贝叶斯修正防止少量高分误导
3. 距离分应改为非线性衰减

### 议题 2：应该追求精确排序还是候选集多样性？

| 专家 | 观点 |
|------|------|
| 张一鸣 | 需要 MMR 多样性机制 |
| Musk | 简单规则：3 个 Must-Stay + 2 个非 Must-Stay |
| Leskovec | 学术上多样性对低频决策更重要 |
| Ed Chi | 召回-排序-重排三阶段，重排阶段加多样性 |
| 项亮 | **5 个位置用 5 个不同维度选** — 多样性 > 精度 |

**共识 (5/5)**：**多样性比精确排序更重要**。5 个推荐位应覆盖不同维度，而非全部按同一个 score 排序。项亮的"多维度选位"方案获得全票支持。

### 议题 3：Must-Stay 应该是排序信号还是解释信号？

| 专家 | 观点 |
|------|------|
| 张一鸣 | 两者都是，但解释性更关键 |
| Musk | 排序信号为主，直接按 Must-Stay rank 排 |
| Leskovec | 作为 feature 参与排序，同时作为解释展示 |
| Ed Chi | 排序信号，但权重应该更高 |
| 项亮 | **解释信号为主** — 中国留学生信"有人推荐过"而非隐式权重 |

**共识 (4/5)**：Must-Stay 应**同时作为排序信号和解释信号**，但展示给用户的解释（"必住榜第 3 名"）比隐式的 20 分加权更能影响用户决策。Musk 认为应该更激进地把 Must-Stay 作为主排序维度，其余 4 位认为作为加权因子 + 强解释是更好的平衡。

### 议题 4：无反馈数据时如何验证算法质量？

| 专家 | 方案 |
|------|------|
| 张一鸣 | 5 个真实场景人工走查 |
| Musk | 把算法开源，让用户告诉你哪里不对 |
| Leskovec | 离线评估：让 3 个"目标用户"（留学生）手动排序，对比算法排序的 NDCG |
| Ed Chi | 用 uhomes 的 Must-Stay 作为 ground truth，评估算法是否把 Must-Stay 房源排在前面 |
| 项亮 | **A/B 测试替代方案**：同一个查询跑两种算法，让用户选"这组推荐哪个更好" |

**共识**：在无行为数据阶段，结合 3 种验证方式：
1. 人工走查（5 个典型场景）
2. Must-Stay 作为弱 ground truth（必住榜房源是否出现在推荐中）
3. 用户偏好测试（如果有渠道触达目标用户）

---

## 第三环节：推荐算法重新设计

### 最终方案：多维度选位 + 必住榜解释 + 贝叶斯品质分

综合 5 位专家的建议，重新设计推荐算法如下：

#### 阶段一：筛选（Hard Filter）

在 scoring 之前硬性排除：
- 评分 < 3.5 的房源（严重品质问题）
- 距离 > 60 分钟步行的房源
- 价格 > 用户预算 × 150% 的房源（允许 50% 弹性，因为用户预算常常偏低）
- 用户指定房型但房源不匹配的

#### 阶段二：评分（Scoring）

对每个通过筛选的房源计算综合分：

```
score = distance_score + budget_score + must_stay_score + quality_score

distance_score (0-40):
  40 × e^(-0.05 × walk_minutes)
  // 非线性衰减：5min≈31, 10min≈24, 15min≈19, 30min≈9

budget_score (0-25):
  If no budget specified → 15 (neutral)
  If within budget → 25
  If over by ≤ 20% → 15
  If over by ≤ 50% → 5

  // Context-dependent weight adjustment:
  If user budget < 80% of city median → budget_weight ×1.4, distance_weight ×0.75
  If user budget > 120% of city median → budget_weight ×0.6, distance_weight ×1.0, quality_weight ×2.0

must_stay_score (0-20):
  20 × (1 - (rank - 1) / 10)
  // Smooth decay: rank 1=20, rank 2=18, rank 5=12, rank 10=2
  // Not on list → 0
  // List unavailable → 0 (no penalty)

quality_score (0-15):
  Use Bayesian average rating:
    adjusted_rating = (20 × 4.2 + rating × review_count) / (20 + review_count)
    // C=20 (confidence threshold), M=4.2 (global mean estimate)

  adjusted_rating ≥ 4.5  → 15
  adjusted_rating ≥ 4.2  → 10
  adjusted_rating ≥ 3.8  → 5
  Below 3.8 or no data   → 2
```

#### 阶段三：多维度选位（Slot-based Selection）

不按 score 直接取 top 5。而是用**5 个插槽**，每个插槽用不同策略选出最佳候选：

```
Slot 1 — 综合最优: score 最高的房源
Slot 2 — 必住推荐: Must-Stay rank 最高的（如果 ≠ Slot 1）；如果无 Must-Stay 数据，选 quality_score 最高的
Slot 3 — 距离优先: 距离最近的（如果 ≠ Slot 1-2）
Slot 4 — 价格优先: 价格最低的（如果 ≠ Slot 1-3）
Slot 5 — 品质优先: adjusted_rating 最高的（如果 ≠ Slot 1-4）

如果某 Slot 的最佳候选已被占用：选该维度的次优候选
如果筛选后候选 < 5 个：有多少展示多少，不凑数
```

#### 阶段四：解释生成（Explanation）

每个推荐房源附带**推荐理由标签**，根据它被选入的 Slot 决定：

| Slot | 标签 | 示例 |
|------|------|------|
| 1 | 综合推荐 | "⭐ Top pick — best overall match for your criteria" |
| 2 | 必住推荐 | "🏆 #3 on uhomes Must-Stay list (236 student reviews)" |
| 3 | 距离最近 | "📍 Closest to campus — 3 min walk" |
| 4 | 价格最优 | "💰 Best value — £172/week" |
| 5 | 口碑最佳 | "❤️ Highest rated — 4.8★ from 312 reviews" |

---

## 第四环节：与当前算法的对比

| 维度 | 当前算法 (v1.4) | 新算法 (提案) |
|------|----------------|--------------|
| 排序方式 | 单一 score 排序取 top 5 | 多维度选位，5 个 Slot 各用不同策略 |
| 距离分 | 线性阶梯（40/30/20/10） | 指数衰减 `40×e^(-0.05×min)` |
| 品质分 | 原始评分直接映射 | 贝叶斯修正（防少量高分误导） |
| Must-Stay | 20 分隐式加权 | 评分加权 + **显式推荐理由** |
| 预算权重 | 固定 25% | 随用户预算松紧动态调整 |
| 多样性 | 无 | Slot 机制天然保证多样性 |
| 解释性 | 无推荐理由 | 每个房源有标签说明为什么推荐 |
| 负面筛除 | 仅预算超标排除 | 评分<3.5、距离>60min、价格>150%预算均排除 |

---

## 第五环节：实施建议

### 共识实施优先级

| 优先级 | 改进项 | 复杂度 | 影响 |
|--------|--------|--------|------|
| **1** | 多维度选位（5 Slot） | 低 — 纯规则重写 | 高 — 直接解决同质化问题 |
| **2** | 推荐理由标签 | 低 — 展示层变更 | 高 — 显著提升用户信任 |
| **3** | 贝叶斯品质分 | 中 — 公式替换 | 中 — 防止少量高分误导 |
| **4** | 距离非线性衰减 | 低 — 公式替换 | 中 — 更真实的距离体验映射 |
| **5** | 预算动态权重 | 中 — 需要城市中位数数据 | 中 — 更好地适配不同预算用户 |
| **6** | 硬性负面筛除 | 低 — 增加 filter 规则 | 低 — 边界场景改善 |

---

## 各专家最终评语

### 张一鸣
> 多维度选位方案很漂亮。它用最简单的方式解决了多样性问题，而不需要引入 MMR 这样的复杂机制。在没有反馈数据之前，这是我见过的最务实的推荐策略。

### Elon Musk
> Much better. The slot-based approach with explicit labels is exactly what I meant by transparency. Users can see WHY each property is there. One concern: don't over-engineer the Bayesian correction for 12 items. A simpler "require at least 10 reviews for high confidence" rule would work too.

### Jure Leskovec
> The staged architecture (filter → score → slot-select → explain) is clean and follows established information retrieval patterns. The context-dependent weight adjustment is a nice touch — it's the simplest form of personalization that doesn't require user history.

### Ed Chi
> The Bayesian rating correction is correctly specified. I'd just note that M=4.2 (the global mean) should be recalibrated periodically based on actual uhomes data if available. The overall design positions you well for a Stage 0→1 transition when data becomes available.

### 项亮
> 推荐理由标签是这次讨论最大的收获。从中国留学生的使用习惯来看，"必住榜第 3 名"这个标签的转化推动力，远大于算法多加 20 分。期待看到上线后的数据表现。

---

*讨论纪要版本：v1.0 | 2026-03-16*
*参与专家：张一鸣、Elon Musk、Jure Leskovec (Stanford/Pinterest)、Ed Chi (Google DeepMind)、项亮（推荐系统实践）*
