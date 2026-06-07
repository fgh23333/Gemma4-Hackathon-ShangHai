# TODO - 下一步工作

## 当前状态

### ✅ Phase 1-4: 已全部完成
- ✅ FastAPI 后端 + Vue 3 前端 + SQLite
- ✅ 8 Agent 圆桌辩论 + Orchestrator 智能调度
- ✅ 双模式入口 + 多轮精炼 + 长期记忆
- ✅ 真实麦当劳 MCP + Gemini 官方 API
- ✅ 饭点提醒 + 今日仪表盘 + 订单草稿
- ✅ 终端 Skill + Cloudflare 部署

### ✅ 已清理的技术债务
- ✅ ~~Orchestrator 关键词映射 → LLM 驱动 + 关键词降级~~
- ✅ ~~前端 React → Vue 3~~
- ✅ ~~MCP Mock only → 真实麦当劳 MCP~~
- ✅ ~~CF AIG → Gemini 官方 API~~
- ✅ ~~ConversationService 内存 → SQLite 持久化~~
- ✅ ~~ActivePlanService 内存 → SQLite 持久化~~
- ✅ ~~demo-user 硬编码 → 动态 getUserId()~~
- ✅ ~~CORS 硬编码 → 环境变量~~
- ✅ ~~BalanceMode mood → 已接入情绪输入~~

---

## 待完成

### 🔥 功能增强

- [ ] **拍照分析入口**
  - 图片上传组件
  - Gemini Vision API 识别食物
  - 营养范围估算（非精确值）
  - 用户确认流程

### 📸 体验优化

- [ ] **推荐结果快捷回复** — "换一个" / "少油" / "更便宜" 按钮
- [ ] **移动端响应式适配**
- [ ] **用户认证（轻量级）** — 简单 session/token，Hackathon 够用

### 🔧 后续扩展（Hackathon 后）

- [ ] **更多餐饮平台** — FoodProvider 接口已就绪，接入美团/饿了么
- [ ] **RAG 向量检索** — 当前知识库用关键词匹配，升级到 ChromaDB
- [ ] **测试覆盖** — pytest 单元测试 + API 集成测试 + E2E 测试

---

## 如何贡献

1. 克隆项目
2. 创建分支: `git checkout -b feat/xxx`
3. 提交代码: `git commit -m "feat: xxx"`
4. 创建 PR → 合并到 main

## 联系方式

- 项目地址: https://github.com/fgh23333/eatornot
- 问题反馈: https://github.com/fgh23333/eatornot/issues
