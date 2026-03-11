# uhomes Skill API — 接口需求规范

> **文档性质**：面向 uhomes.com 技术团队的接口交付需求。本文档描述 `uhomes-student-housing` Skill 所需的后端接口规格，供技术评估与实现。
>
> **优先级**：此接口非发布阻断项。Skill v1.0 可使用 web_fetch 降级方案先行发布；此接口就绪后 Skill 直接升级至 v2.0，对用户无感知。

---

## 1. 接口概览

| 项目 | 内容 |
|------|------|
| 接口名称 | Property Search API for AI Skill |
| 调用方 | uhomes-student-housing Skill（运行在 OpenClaw / Claude 环境中） |
| 调用频率预估 | 初期 < 100 次/天，视安装量线性增长 |
| 认证方式 | API Key（Header 传入，Key 由 uhomes 分配给 Skill 维护方） |
| 数据源 | uhomes.com 现有房源数据库 |
| 响应格式 | JSON |
| 语言支持 | 优先英文；若后续支持中文界面，`description` 字段可扩展 |

---

## 2. 接口定义

### 2.1 搜索接口

```
GET /api/v1/skill/properties
```

#### 请求参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `city` | string | 与 `university` 二选一 | 城市英文名，如 `london`、`manchester`、`sydney` |
| `university` | string | 与 `city` 二选一 | 大学英文名或 slug，如 `university-of-manchester` |
| `room_type` | string | 否 | 枚举值，见 §3.1。可多选，逗号分隔 |
| `budget_min` | number | 否 | 最低周租金（本地货币，整数） |
| `budget_max` | number | 否 | 最高周租金（本地货币，整数） |
| `currency` | string | 否 | 货币代码，默认按城市自动推断（GBP/USD/AUD/CAD 等） |
| `move_in_month` | string | 否 | 入住月份，格式 `YYYY-MM`，如 `2025-09` |
| `limit` | integer | 否 | 返回条数，默认 `5`，最大 `10` |

#### 请求示例

```http
GET /api/v1/skill/properties?university=university-of-manchester&room_type=en-suite&budget_max=200&currency=GBP&limit=5
Authorization: Bearer sk-uhomes-skill-xxxxxx
```

---

### 2.2 响应结构

#### 成功响应（HTTP 200）

```json
{
  "status": "success",
  "query": {
    "university": "university-of-manchester",
    "city": "manchester",
    "room_type": ["en-suite"],
    "budget_max": 200,
    "currency": "GBP"
  },
  "total_found": 48,
  "returned": 5,
  "properties": [
    {
      "id": "prop_uk_mcr_001",
      "name": "iQ Manchester",
      "type": "PBSA",
      "url": "https://en.uhomes.com/uk/manchester/iq-manchester",
      "distance_to_university": {
        "value": 8,
        "unit": "minutes_walk",
        "label": "8 min walk to UoM"
      },
      "room_options": [
        {
          "room_type": "en-suite",
          "label": "En-suite Room",
          "price_per_week": 175,
          "currency": "GBP",
          "bills_included": true,
          "availability": "available",
          "contract_weeks": 51
        },
        {
          "room_type": "studio",
          "label": "Studio",
          "price_per_week": 230,
          "currency": "GBP",
          "bills_included": true,
          "availability": "limited",
          "contract_weeks": 51
        }
      ],
      "highlights": [
        "No service fee",
        "24h security",
        "Gym",
        "Near Chinese supermarket"
      ],
      "no_service_fee": true,
      "verified": true,
      "rating": 4.6,
      "review_count": 312
    }
  ],
  "search_page_url": "https://en.uhomes.com/uk/manchester/university-of-manchester",
  "meta": {
    "currency": "GBP",
    "price_unit": "per_week"
  }
}
```

#### 无结果响应（HTTP 200，results 为空）

```json
{
  "status": "success",
  "total_found": 0,
  "returned": 0,
  "properties": [],
  "search_page_url": "https://en.uhomes.com/uk/manchester",
  "fallback_message": "No properties found matching your criteria. Try adjusting your budget or room type."
}
```

#### 错误响应

| HTTP 状态码 | `error_code` | 含义 |
|-------------|--------------|------|
| 400 | `MISSING_LOCATION` | city 和 university 均未提供 |
| 400 | `INVALID_ROOM_TYPE` | room_type 枚举值不合法 |
| 401 | `UNAUTHORIZED` | API Key 无效或缺失 |
| 429 | `RATE_LIMITED` | 超出调用频率限制 |
| 500 | `SERVER_ERROR` | 服务端内部错误 |

```json
{
  "status": "error",
  "error_code": "MISSING_LOCATION",
  "message": "Either 'city' or 'university' parameter is required."
}
```

---

## 3. 枚举值定义

### 3.1 `room_type` 合法值

| 值 | 含义 |
|----|------|
| `en-suite` | 独立卫浴单间 |
| `non-en-suite` | 共用卫浴单间 |
| `studio` | 独立套间（含独立厨卫） |
| `1-bed` | 一室一厅 |
| `2-bed` | 两室一厅 |
| `shared-room` | 合租间 |

### 3.2 `availability` 合法值

| 值 | 含义 |
|----|------|
| `available` | 充足可预订 |
| `limited` | 少量剩余 |
| `waitlist` | 候补中 |
| `unavailable` | 暂不可订（可展示但不推荐） |

### 3.3 `type`（物业类型）合法值

| 值 | 含义 |
|----|------|
| `PBSA` | Purpose-Built Student Accommodation（专建学生公寓） |
| `private-rental` | 私人房东出租 |
| `managed` | 品牌管理公寓 |

---

## 4. 技术约束

### 4.1 性能要求

| 指标 | 要求 |
|------|------|
| P95 响应时间 | ≤ 1500ms |
| P99 响应时间 | ≤ 3000ms |
| 单 Key 调用限制 | 200 次/小时（超出返回 429） |

> **说明**：Skill 在对话中调用此接口，用户在等待响应。超过 3 秒会显著影响体验。

### 4.2 数据要求

- `url` 字段必须是合法的 uhomes.com 完整 URL，可直接跳转
- `price_per_week` 必须与 uhomes.com 页面展示价格一致（允许 ±5% 浮动缓冲）
- `highlights` 数组最多返回 5 条，优先返回对留学生最相关的标签（No service fee、Bills included、Near campus 优先级最高）
- 接口不返回楼层、具体房间号等过细信息（Skill 展示层不需要）

### 4.3 认证

```http
Authorization: Bearer {API_KEY}
```

API Key 由 uhomes 技术团队生成并提供给 Skill 维护方（Neil）。建议设置独立的 Skill 专用 Key，便于隔离和限流。

---

## 5. Skill 侧处理逻辑（供参考）

Skill 收到响应后的处理行为，供接口设计参考：

1. 从 `properties` 数组中取前 3-5 条展示给用户
2. 每条展示：名称、最低周租金、距学校距离、highlights 前 3 条、uhomes 直达链接（附加 UTM 参数）
3. 末尾附 `search_page_url`（同样附加 UTM 参数）作为"查看全部"入口
4. 若 `total_found = 0`，展示 `fallback_message` 并给出 `search_page_url`
5. 若接口超时或返回 5xx，降级为直接给出 `search_page_url`（Skill 内部构建）

---

## 6. 交付验收标准

- [ ] 接口在测试环境可正常调用，返回格式符合本文档定义
- [ ] 至少覆盖以下城市/大学的数据：London、Manchester、Birmingham、Edinburgh、Sydney、Melbourne、Toronto、New York（Columbia/NYU）
- [ ] `url` 字段均为有效 uhomes.com 链接，人工抽查通过率 ≥ 95%
- [ ] P95 响应时间 ≤ 1500ms（在 uhomes 服务器本地测试）
- [ ] API Key 认证机制正常工作（合法 Key 200，无效 Key 401）

---

*文档版本：v1.0 | 2026-03*
*对接联系人：Neil（uhomes 业务侧）*
