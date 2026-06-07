# 🍔 基于Gemma 4的营养成分以及开销伴随智能体

> **赛道**: A (AI Agent) | **队伍**: EatOrNot | **模型**: Gemma 4 31B Dense (`gemma-4-31b-it`)

## 项目简介

**面向程序员的终端饮食智能体** — 解决高强度工作下饮食不规律的问题。

程序员生活节奏快、压力大，经常忘记点饭、凭情绪暴饮暴食、不知不觉超预算。本智能体通过 **8 个专业 Agent 的圆桌辩论**，自主完成营养分析、预算控制、习惯学习、下单推荐全流程，成为用户的"饮食营养伴随"。

### 与 Gemma 4 的深度结合

| Gemma 4 能力 | 本项目实现 |
|-------------|-----------|
| **原生函数调用 (Native Function Calling)** | Agent 通过 Function Calling 自动选择工具：查询菜单、获取营养、计算价格、创建订单 |
| **多步规划 (Multi-step Planning)** | Orchestrator 根据用户场景动态调度 3-5 个 Agent，并行分析后 4 阶段辩论 |
| **Agent Memory** | SQLite 持久化饮食偏好/习惯模式/预算行为，持续学习用户画像 |
| **Tool Calling** | 麦当劳 MCP 接入真实菜单/价格/下单 API，实现自主下单闭环 |

---

## 核心架构

### 决策流程

```
用户输入 "想吃麦当劳但我在减肥"
    │
    ▼
MealDecisionFlow (场景检测 + 记忆召回)
    │
    ▼
SupervisorAgent
    ├── OrchestratorAgent (Gemma 4 智能调度 → 选 3-5 个相关 Agent)
    │       │
    │       ▼ asyncio.gather() 并行分析
    │   [档案] [减脂] [营养] [预算] [食欲] [时间] [安全] [未来模拟]
    │       │
    │       ▼
    │   DebateEngine (4 阶段圆桌辩论)
    │       ├── 第一轮：各 Agent 发表初始意见
    │       ├── 第二轮：发现冲突（减脂 vs 食欲、预算 vs 营养）
    │       ├── 第三轮：形成妥协方案
    │       └── 第四轮：最终投票
    │       │
    │       ▼
    │   3 个方案: 💪自律 / 💰省钱 / 🍔犒劳
    │
    ▼
AutoDraftService → 自动生成订单草稿
```

### Agent Memory 实现

```python
# memory_service.py — 饮食习惯长期记忆
class MemoryService:
    """基于 SQLite 的饮食习惯学习引擎"""

    async def record_preference(self, user_id, food, rating):
        """记录每次用餐反馈，构建偏好画像"""

    async def get_habits(self, user_id):
        """召回用户习惯：偏好口味、回避食材、预算模式、用餐时段"""

    async def detect_pattern(self, user_id):
        """检测行为模式：如 '工作日倾向重口味'、'周末放松控制'"""
```

记忆在每次决策时被召回，影响 Orchestrator 的 Agent 选择权重和各 Agent 的分析基线。

### Tool Calling 实现

```python
# mcdonalds_mcp_provider.py — 麦当劳 MCP 接入
class McDonaldsMCPProvider(FoodProvider):
    """通过 Model Context Protocol 接入真实麦当劳 API"""

    async def search_menu(self, keyword: str) -> list[Meal]:
        """搜索菜单 → Gemma 4 Function Calling 触发"""

    async def get_nutrition(self, product_code: str) -> Nutrition:
        """获取营养成分 → 营养 Agent 分析用"""

    async def calculate_price(self, items, store_code) -> PriceResult:
        """计算价格（含优惠） → 预算 Agent 用"""

    async def create_order(self, items, store_code, ...) -> OrderResult:
        """创建订单 → 用户确认后自主下单"""
```

Agent 通过 Gemma 4 的 Function Calling 自动选择合适的工具：
- `search_menu` → 营养 Agent 需要餐品数据
- `get_nutrition` → 减脂 Agent 计算热量
- `calculate_price` → 预算 Agent 检查是否超支
- `create_order` → 用户确认后自主下单

---

## 技术栈

| 层 | 技术 | 说明 |
|----|------|------|
| **LLM** | Google Gemini API → Gemma 4 31B Dense | 原生 Function Calling |
| **后端** | Python 3.12 + FastAPI | 异步 Agent 编排 |
| **前端** | Vue 3 + Vite 6 + TypeScript | 响应式 UI |
| **数据库** | SQLAlchemy + aiosqlite | 用户档案 + 饮食记忆 |
| **Agent 框架** | Google ADK 2.1 | Agent Development Kit |
| **外部工具** | 麦当劳 MCP | Model Context Protocol |
| **UI** | Tailwind CSS + shadcn-vue | 组件库 |
| **部署** | Docker Compose | 一键启动 |

---

## 快速启动

### 方式 A：Docker 一键启动（推荐）

```bash
# 1. 配置环境变量
cp .env.example .env
# 编辑 .env，填入 GEMINI_API_KEY（必填）

# 2. 一键启动
docker compose up -d --build

# 3. 访问
# 前端: http://localhost:5173
# 后端: http://localhost:8000/health
```

### 方式 B：本地开发

```bash
# 后端
cd apps/api
pip install -r requirements.txt
uvicorn main:app --host 127.0.0.1 --port 8000

# 前端 (另一个终端)
cd apps/web
pnpm install
pnpm dev
```

访问 http://localhost:5173

### 终端点餐（Claude Code Skill）

```powershell
# 安装 skill + 权限
.\scripts\claude-setup\install.ps1

# 使用
帮我点午餐     # AI 推荐 + 辩论 + 下单
领券           # 自动领取麦当劳优惠券
```

### 环境变量

| 变量 | 必填 | 说明 |
|------|------|------|
| `GEMINI_API_KEY` | ✅ | [Google AI Studio](https://aistudio.google.com/apikey) 获取 |
| `GEMINI_MODEL` | ❌ | 默认 `gemma-4-31b-it` |
| `MCD_MCP_TOKEN` | ❌ | 麦当劳 MCP Token（无则用 Mock 数据） |
| `DATABASE_URL` | ❌ | 默认 `sqlite+aiosqlite:///./data/eatornot.db` |

---

## Agent 列表

| Agent | 职责 | Gemma 4 Function Calling |
|-------|------|--------------------------|
| 🧑 档案Agent | 分析用户画像 | 调用 `get_user_profile` |
| 🔥 减脂Agent | 热量目标 & 减脂策略 | 调用 `calculate_tdee` |
| 🥗 营养Agent | 营养素评估 | 调用 `get_nutrition` |
| 💰 预算Agent | 预算控制 | 调用 `calculate_price` |
| 🍫 食欲Agent | 情绪性进食分析 | 记忆检索 |
| ⏰ 时间Agent | 时间压力评估 | 调用 `get_meals` |
| ⚠️ 安全Agent | 过敏 & 饮食安全 | 安全规则匹配 |
| 🔮 未来模拟Agent | 用餐后影响预测 | 热量-预算平衡模型 |

---

## 项目结构

```
eatornot/
├── apps/api/                          # FastAPI 后端
│   ├── agents/                        # Agent 层 (核心)
│   │   ├── orchestrator_agent.py      # LLM 智能调度器
│   │   ├── supervisor_agent.py        # 方案构建器
│   │   ├── debate_engine.py           # 4 阶段圆桌辩论
│   │   └── 8 个专业 Agent
│   ├── services/                      # 业务服务
│   │   ├── memory_service.py          # 饮食习惯长期记忆 ★
│   │   ├── meal_decision_flow.py      # Dual-Trigger 决策流
│   │   └── auto_draft_service.py      # 自动订单草稿
│   ├── providers/                     # 食物数据源
│   │   ├── mcdonalds_mcp_provider.py  # 麦当劳 MCP ★
│   │   └── mock_mcdonalds_provider.py # Mock 降级
│   └── core/
│       └── llm_client.py              # Gemma 4 调用封装
├── apps/web/                          # Vue 3 前端
├── knowledge/                         # 营养学知识库 (115 条)
├── skills/                            # ADK Skills (5 个)
└── scripts/claude-setup/             # 终端点餐 Skill
```

★ = 赛道 A 重点考察文件

---

## 运行日志示例

### Agent Memory 记忆学习

```
[Memory] 记录用餐反馈: user=dev_coder, food=巨无霸, rating=4
[Memory] 检测到模式: 用户偏好高蛋白餐品 (置信度 0.82)
[Memory] 召回习惯: 偏好=[辣味, 高蛋白], 回避=[生菜], 预算模式=[工作日节省]
[Orchestrator] 基于记忆增加减脂Agent和预算Agent权重
```

### Tool Calling 工具调用

```
[NutritionAgent] 调用 get_nutrition(product_code="big_mac")
  → 热量: 563kcal, 蛋白质: 33g, 脂肪: 33g, 碳水: 39g

[BudgetAgent] 调用 calculate_price(items=[...], store_code="12345")
  → 总价: ¥42.00, 可用优惠券: 满减券-¥5

[OrderDraft] 调用 create_order(items=[...], store_code="12345")
  → 订单创建成功, order_id=ORD20260607001
```

---

## 文档

- [技术报告](TECHNICAL_REPORT.md) — 模型选型与架构设计
- [架构文档](docs/architecture.md) — 详细系统架构
- [演示视频脚本](docs/demo_video_script.md) — 5 分钟 demo 流程
- [开发指南](CLAUDE.md) — 项目结构与协作规范

## License

MIT
