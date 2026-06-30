# ReActLab — AI Agent 实验平台

> 基于 LangGraph 的 Agent 开发与评测平台 | 可复用 Agent Runtime · MCP 工具集成 · RAG 知识检索 · Benchmark 评测

[![Python](https://img.shields.io/badge/Python-3.10+-blue.svg)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.109+-green.svg)](https://fastapi.tiangolo.com/)
[![LangChain](https://img.shields.io/badge/LangChain-latest-orange.svg)](https://www.langchain.com/)
[![LangGraph](https://img.shields.io/badge/LangGraph-latest-purple.svg)](https://langchain-ai.github.io/langgraph/)
[![MCP](https://img.shields.io/badge/MCP-1.0+-red.svg)](https://modelcontextprotocol.io/)
[![Milvus](https://img.shields.io/badge/Milvus-2.4+-brightgreen.svg)](https://milvus.io/)

## ✨ 核心能力

| 能力 | 说明 |
|------|------|
| 🤖 **Agent Runtime** | Plan → Execute → Replan → Report 多阶段 Agent 工作流，支持动态重规划与循环控制 |
| 🔌 **MCP 工具集成** | 基于 MCP 协议开发的 Tool 服务，支持 Agent Tool Calling 自主调度 |
| 📚 **RAG 知识检索** | Milvus 向量库 + Embedding 语义检索，上下文增强生成 |
| 📊 **Benchmark 评测** | 100 条 RAG Query + 10 个端到端 Agent Task，量化评估系统能力 |
| 🌐 **流式交互** | FastAPI + SSE 实现 Agent 执行过程实时可视化 |
| 🐳 **工程化部署** | Docker Compose 一键启动，失败重试 + 超时熔断 |

## 🛠️ 技术栈

- **框架**: FastAPI + LangChain + LangGraph
- **LLM**: 阿里云 DashScope (通义千问 qwen-max)
- **向量库**: Milvus 2.4+
- **工具协议**: MCP (Model Context Protocol)
- **Embedding**: text-embedding-v4
- **部署**: Docker Compose + uv

## 🏗️ 架构概览

`
User → FastAPI Gateway → Agent Runtime (Plan/Execute/Replan/Report)
                              ├── MCP Tools (日志查询 · 监控数据 · 时间工具)
                              ├── RAG Pipeline (文档切片 → Embedding → Milvus 检索)
                              └── LLM (DashScope qwen-max)
`

## 🚀 快速开始

### 环境要求
- Python 3.10+
- Docker Desktop
- 阿里云 DashScope API Key（[获取地址](https://dashscope.aliyun.com/)）

### 一键启动（Windows）

`powershell
# 1. 克隆项目
git clone https://github.com/TianlonWang/ReActLab.git
cd ReActLab

# 2. 编辑配置文件，填入你的 API Key
notepad .env

# 3. 一键启动所有服务
.\start-windows.bat
`

启动后访问:
- **Web 界面**: http://localhost:9900
- **API 文档**: http://localhost:9900/docs

### 手动启动

`ash
# 安装依赖
pip install uv
uv venv && uv sync

# 启动 Docker 服务 (Milvus)
docker compose -f vector-database.yml up -d

# 启动 MCP 工具服务
python mcp_servers/cls_server.py &
python mcp_servers/monitor_server.py &

# 启动主服务
python -m uvicorn app.main:app --host 0.0.0.0 --port 9900
`

## 📡 API 接口

| 功能 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 快速对话 | POST | /api/chat | RAG + Agent 一次性返回 |
| 流式对话 | POST | /api/chat_stream | SSE 流式输出 |
| Agent 诊断 | POST | /api/aiops | Plan-Execute-Replan 流式诊断 |
| 文档上传 | POST | /api/upload | 上传文档建立向量索引 |
| 健康检查 | GET | /api/health | 服务状态检查 |

`ash
# 流式对话
curl -X POST "http://localhost:9900/api/chat_stream" \
  -H "Content-Type: application/json" \
  -d '{\"Id\":\"s1\", \"Question\":\"你好\"}' --no-buffer

# Agent 诊断
curl -X POST "http://localhost:9900/api/aiops" \
  -H "Content-Type: application/json" \
  -d '{\"session_id\":\"s1\"}' --no-buffer
`

## 📁 项目结构

`
ReActLab/
├── app/                          # 应用核心
│   ├── agent/                    # Agent Runtime
│   │   └── aiops/                # Planner / Executor / Replanner
│   │       ├── planner.py        # 任务规划节点
│   │       ├── executor.py       # 工具调度执行节点
│   │       ├── replanner.py      # 评估 + 动态重规划节点
│   │       ├── state.py          # Agent 状态定义
│   │       └── utils.py          # 报告生成工具
│   ├── api/                      # API 路由层
│   │   ├── chat.py               # 对话接口
│   │   ├── aiops.py              # Agent 诊断接口
│   │   └── file.py               # 文件上传接口
│   ├── services/                 # 业务服务层
│   │   ├── aiops_service.py      # Agent 工作流编排
│   │   ├── rag_agent_service.py  # RAG + ReAct Agent
│   │   ├── vector_index_service.py   # 向量索引管理
│   │   └── vector_search_service.py  # 向量检索
│   ├── core/                     # 核心组件
│   │   ├── llm_factory.py        # LLM 工厂（DashScope）
│   │   └── milvus_client.py      # Milvus 客户端
│   ├── tools/                    # Agent 工具
│   │   ├── knowledge_tool.py     # 知识库检索工具
│   │   └── time_tool.py          # 时间工具
│   └── models/                   # Pydantic 数据模型
├── mcp_servers/                  # MCP 工具服务
│   ├── cls_server.py             # 日志查询 MCP Server
│   └── monitor_server.py         # 监控数据 MCP Server
├── static/                       # Web 前端
├── eval_benchmark.py             # Benchmark 评测脚本
├── benchmark_results.json        # 评测结果
├── vector-database.yml           # Milvus Docker Compose
├── start-windows.bat             # Windows 一键启动
├── stop-windows.bat              # Windows 一键停止
└── pyproject.toml                # 项目配置
`

## 🎯 Demo 场景：智能运维诊断

以运维场景演示 Agent Runtime 的完整能力：

### 工作流
`
1. Planner  → 解析用户意图，生成分步诊断计划
2. Executor → 按计划调用 MCP 工具（日志查询、监控数据、知识检索）
3. Replanner → 评估执行结果，决定继续 / 调整计划 / 生成报告
4. Reporter → 输出结构化诊断报告（根因分析 + 处理建议）
`

### 工具生态
- **日志查询** — MCP CLS Server，支持按时间范围、关键词搜索
- **监控数据** — MCP Monitor Server，CPU / 内存 / 磁盘指标查询
- **知识检索** — Milvus RAG，检索历史故障案例和运维文档
- **时间工具** — 获取当前时间戳，辅助时间范围计算

## 📊 Benchmark 评测

| 指标 | 数值 | 说明 |
|------|------|------|
| Top-1 Accuracy | **86%** | RAG 检索首位命中率 |
| Recall@3 | **94%** | RAG 检索前 3 召回率 |
| MRR | **0.887** | 平均倒数排名 |
| Agent Success Rate | **100%** | 10 个端到端任务全部完成 |
| Retry Recovery | **25%→75%** | 指数退避重试后工具调用恢复率 |

`ash
# 运行评测
python eval_benchmark.py
`

## ⚙️ 配置说明

通过 .env 文件配置（复制 .env.example 并修改）：

`ash
# LLM 配置（必填）
DASHSCOPE_API_KEY=your-api-key
DASHSCOPE_MODEL=qwen-max
DASHSCOPE_EMBEDDING_MODEL=text-embedding-v4

# 向量库配置
MILVUS_HOST=localhost
MILVUS_PORT=19530

# RAG 配置
RAG_TOP_K=3
CHUNK_MAX_SIZE=800
CHUNK_OVERLAP=100
`

## 🧪 扩展开发

Agent Runtime 设计为**场景无关的可复用架构**，你可以：

- **接入新工具**: 按 MCP 协议开发 Tool Server，Agent 自动发现和调度
- **替换 LLM**: 修改 llm_factory.py，支持任意 OpenAI 兼容接口
- **自定义工作流**: 基于 LangGraph StateGraph 扩展 Planner/Executor 逻辑
- **新增 Demo 场景**: 复用 Runtime，只需更换 Prompt 和工具集

## 📄 许可证

MIT License
