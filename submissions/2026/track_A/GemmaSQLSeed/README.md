# GemmaSQLSeed

基于 Gemma 大模型的智能 SQLite 测试数据生成工具。

## 项目简介

GemmaSQLSeed 是一款基于 Gemma 大模型的智能 SQLite 测试数据生成工具。通过 MCP（Model Context Protocol）协议，AI Agent 可以直接与数据库交互，实现 schema 自动分析、数据配置智能生成和批量数据填充。

## 核心创新点

1. **Gemma 驱动的 SchemaAnalyzer**：自动识别表结构并推荐数据生成策略
2. **自纠正流程（AiConfigRefiner）**：Gemma 生成配置后自动验证并修正，最多3轮自纠正
3. **MCP Server 标准化工具接口**：支持任何兼容 MCP 的 AI Agent 零代码接入
4. **9级列映射策略链 + 31种数据生成器 + DAG依赖排序**：确保跨表外键完整性

## 技术架构

- **核心引擎**：Python 3.10+，声明式数据生成框架
- **AI 集成**：Gemma 4 模型驱动 Schema 分析与配置生成
- **MCP 协议**：FastMCP 实现标准化工具接口
- **数据生成**：9级列映射策略链，31种生成器，DAG 依赖排序
- **数据库适配**：SQLite（支持 sqlite-utils 和原生适配器）

## 环境安装

```bash
# 克隆主仓库
git clone https://github.com/sunbos/sqlseed.git
cd sqlseed

# 安装（含所有可选依赖）
pip install -e ".[dev,all]"

# 安装 AI 插件（需要 Gemma API key）
pip install -e "./plugins/sqlseed-ai"

# 安装 MCP Server
pip install -e "./plugins/mcp-server-sqlseed"
```

## 快速开始

### 命令行使用

```bash
# 零配置填充 10000 行数据
sqlseed fill app.db -t users -n 10000

# 预览生成数据
sqlseed preview app.db -t users -n 5

# 查看列映射策略
sqlseed inspect app.db --table users --show-mapping

# AI 驱动的智能配置生成
sqlseed ai-suggest app.db --table users
```

### Python API

```python
import sqlseed

# 一行代码生成数据
result = sqlseed.fill("test.db", table="users", count=100_000)
print(result)
# → GenerationResult(table=users, count=100000, elapsed=2.34s, speed=42735 rows/s)

# 使用配置文件
sqlseed.fill_from_config("config.yaml")

# 预览模式
preview = sqlseed.preview("test.db", table="users", count=5)
```

### MCP Server

```bash
# 启动 MCP Server
mcp-server-sqlseed
```

AI Agent 可通过 MCP 协议调用以下工具：
- `inspect_schema`：分析数据库 schema
- `generate_yaml_config`：AI 生成数据配置
- `fill_table`：批量填充数据

## 项目结构

```
sqlseed/
├── src/sqlseed/          # 主包
│   ├── core/             # 编排器、映射器、Schema推断、约束求解、DAG
│   ├── generators/       # 数据提供者：base, faker, mimesis
│   ├── database/         # SQLite 适配器
│   ├── plugins/          # 插件系统
│   ├── config/           # Pydantic 配置模型
│   ├── cli/              # Click 命令行
│   └── _utils/           # 内部工具
├── plugins/
│   ├── sqlseed-ai/       # Gemma 驱动的 AI 插件
│   └── mcp-server-sqlseed/  # MCP Server
├── tests/                # 测试套件
└── docs/                 # 文档
```

## Gemma 模型使用说明

本项目深度利用 Gemma 4 的**原生函数调用（Native Function Calling）**能力：

1. **SchemaAnalyzer**：Gemma 分析数据库 schema，识别列语义，推荐数据生成策略
2. **AiConfigRefiner**：自纠正流程，Gemma 生成配置后自动验证并修正
3. **MCP Tool Calling**：通过 MCP 协议暴露标准化工具接口，支持 Agent 多步规划

## 依赖

- Python 3.10+
- pydantic, pluggy, structlog, pyyaml, click, rich, simpleeval, rstr
- 可选：faker, mimesis, sqlite-utils
- AI 插件：openai（兼容 Gemma API）

## 许可证

AGPL-3.0-or-later
