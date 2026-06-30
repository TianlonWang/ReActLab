# ReActLab — AI Agent 实验平台

> 基于 LangGraph 的可复用 Agent 开发与评测平台 | MCP 工具集成 · RAG 知识检索 · Benchmark 评测

[![Python](https://img.shields.io/badge/Python-3.10+-blue.svg)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.109+-green.svg)](https://fastapi.tiangolo.com/)
[![LangChain](https://img.shields.io/badge/LangChain-latest-orange.svg)](https://www.langchain.com/)
[![LangGraph](https://img.shields.io/badge/LangGraph-latest-purple.svg)](https://langchain-ai.github.io/langgraph/)
[![MCP](https://img.shields.io/badge/MCP-1.0+-red.svg)](https://modelcontextprotocol.io/)
[![Milvus](https://img.shields.io/badge/Milvus-2.4+-brightgreen.svg)](https://milvus.io/)

---

## 核心能力

| 能力 | 说明 |
|------|------|
| **Agent Runtime** | Plan -> Execute -> Replan -> Report 多阶段工作流，支持动态重规划与循环控制 |
| **MCP 工具集成** | 基于 MCP 协议开发的 Tool 服务，Agent 可自主发现并调用外部工具 |
| **RAG 知识检索** | Milvus 向量库 + Embedding 语义检索，上下文增强生成 |
| **Benchmark 评测** | 100 条 RAG Query + 10 个端到端 Agent Task，量化评估系统能力 |
| **流式交互** | FastAPI + SSE 实现 Agent 执行过程实时可视化 |
| **工程化部署** | Docker Compose 一键启动，指数退避重试 + 超时熔断 |

---

## 技术栈

| 层级 | 技术 |
|------|------|
| Agent 框架 | LangChain + LangGraph |
| API 框架 | FastAPI |
| LLM | 阿里云 DashScope（通义千问 qwen-max） |
| 工具协议 | MCP（Model Context Protocol） |
| 向量数据库 | Milvus 2.4+ |
| Embedding | text-embedding-v4 |
| 部署 | Docker Compose + uv |

---

## 架构概览

```text
                         +------------------+
                         |   LangGraph      |
 User Request ---------> |   Agent Runtime  |
                         |                  |
                         | Planner          |
                         |   |              |
                         | Executor --------+----> MCP Tool Servers
                         |   |              |      (日志/监控/时间)
                         | Replanner        |
                         |   |              |
                         | Reporter <-------+----> RAG Pipeline
                         |                  |      (Milvus + Embedding)
                         +------------------+
                                  |
                                  v
                         +------------------+
                         |   LLM            |
                         | (DashScope)      |
                         +------------------+
```

---

## 快速开始

### 环境要求

- Python 3.10+
- Docker Desktop
- [阿里云 DashScope API Key](https://dashscope.aliyun.com/)

### Windows 一键启动

```powershell
git clone https://github.com/wfang0314/ReActLab.git
cd ReActLab

# 编辑 .env，填入你的 API Key
notepad .env

# 一键启动所有服务
.\start-windows.bat
```

启动后访问：

| 服务 | 地址 |
|------|------|
| Web 界面 | http://localhost:9900 |
| API 文档 (Swagger) | http://localhost:9900/docs |

### 手动启动

```bash
pip install uv
uv venv && uv sync

docker compose -f vector-database.yml up -d

python mcp_servers/cls_server.py &
python mcp_servers/monitor_server.py &

python -m uvicorn app.main:app --host 0.0.0.0 --port 9900
```

---

## API 接口

| 功能 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 快速对话 | POST | /api/chat | RAG + Agent 一次性返回 |
| 流式对话 | POST | /api/chat_stream | SSE 流式输出 |
| Agent 诊断 | POST | /api/aiops | Plan-Execute-Replan 流式诊断 |
| 文档上传 | POST | /api/upload | 上传文档建立向量索引 |
| 健康检查 | GET | /api/health | 服务状态检查 |

```bash
# 流式对话
curl -X POST http://localhost:9900/api/chat_stream   -H "Content-Type: application/json"   -d '{"Id":"s1","Question":"你好"}' --no-buffer

# Agent 诊断
curl -X POST http://localhost:9900/api/aiops   -H "Content-Type: application/json"   -d '{"session_id":"s1"}' --no-buffer
```

---

## 项目结构

```text
ReActLab/
|
+-- app/                            # 应用核心
|   +-- main.py                     # FastAPI 入口
|   +-- config.py                   # 配置加载
|   |
|   +-- agent/                      # Agent Runtime 层
|   |   +-- aiops/
|   |       +-- planner.py          # 任务规划节点
|   |       +-- executor.py         # 工具调度执行节点
|   |       +-- replanner.py        # 评估 + 动态重规划节点
|   |       +-- state.py            # Agent 状态定义 (TypedDict)
|   |       +-- utils.py            # 诊断报告生成
|   |
|   +-- api/                        # API 路由层
|   |   +-- chat.py                 # RAG + ReAct Agent 对话
|   |   +-- aiops.py                # Agent 诊断 (SSE 流式)
|   |   +-- file.py                 # 文档上传与索引
|   |   +-- health.py               # 健康检查
|   |
|   +-- services/                   # 业务服务层
|   |   +-- aiops_service.py        # Agent 工作流编排 (LangGraph)
|   |   +-- rag_agent_service.py    # RAG + ReAct Agent
|   |   +-- vector_index_service.py # 向量索引管理
|   |   +-- vector_search_service.py# 向量语义检索
|   |   +-- document_splitter_service.py  # 文档切片
|   |   +-- vector_embedding_service.py   # Embedding 向量化
|   |   +-- vector_store_manager.py # 向量库生命周期管理
|   |   +-- session_store.py        # 会话状态管理
|   |
|   +-- core/                       # 基础设施
|   |   +-- llm_factory.py          # LLM 工厂
|   |   +-- milvus_client.py        # Milvus 连接管理
|   |
|   +-- tools/                      # Agent 可调用工具
|   |   +-- knowledge_tool.py       # RAG 知识检索
|   |   +-- time_tool.py            # 时间戳工具
|   |
|   +-- models/                     # Pydantic 数据模型
|   |   +-- aiops.py / request.py / response.py / document.py
|   |
|   +-- utils/                      # 工具函数
|       +-- logger.py               # Loguru 日志
|
+-- mcp_servers/                    # MCP 工具服务
|   +-- cls_server.py               # 日志查询 Server
|   +-- monitor_server.py           # 监控数据 Server
|
+-- static/                         # Web 前端
+-- aiops-docs/                     # RAG 知识库文档
+-- portfolio/                      # 作品集 PDF
|
+-- eval_benchmark.py               # Benchmark 评测脚本
+-- benchmark_results.json          # 评测结果
+-- gen_portfolio.py                # 作品集生成
|
+-- vector-database.yml             # Milvus Docker Compose
+-- start-windows.bat               # 一键启动
+-- stop-windows.bat                # 一键停止
+-- pyproject.toml                  # 项目配置
+-- uv.lock                         # 依赖锁
```

---

## Demo 场景：智能运维诊断

以运维场景演示 Agent Runtime 的完整能力，**Runtime 本身与场景无关，可复用于其他领域**。

### 工作流

```text
用户输入: 服务器 CPU 告警
        |
        v
  Planner     解析意图 -> 生成分步诊断计划
        |
        v
  Executor    调用 MCP 工具链
        |     + 查询 CPU 监控指标
        |     + 检索历史故障案例 (RAG)
        |     + 搜索相关错误日志
        v
  Replanner   评估结果 -> 继续/调整/生成报告
        |
        v
  Reporter    输出结构化报告: 根因分析 + 处理建议 + 风险评估
```

### 工具生态

- **日志查询** — MCP CLS Server，按时间范围、关键词搜索
- **监控数据** — MCP Monitor Server，CPU/内存/磁盘指标
- **知识检索** — Milvus RAG，检索历史案例和文档
- **时间工具** — 时间戳转换，辅助时间范围计算

---

## Benchmark 评测

| 指标 | 数值 | 说明 |
|------|------|------|
| Top-1 Accuracy | **86%** | RAG 检索首位命中率 |
| Recall@3 | **94%** | RAG 检索前 3 召回率 |
| MRR | **0.887** | 平均倒数排名 |
| Agent Success Rate | **100%** | 10 个端到端任务全部完成 |
| Retry Recovery | **25% -> 75%** | 指数退避重试后工具调用恢复率 |

```bash
python eval_benchmark.py
```

---

## 扩展开发

Agent Runtime 设计为**场景无关的可复用架构**：

- **接入新工具** — 按 MCP 协议开发 Tool Server，Agent 自动发现和调度
- **替换 LLM** — 修改 llm_factory.py，支持任意 OpenAI 兼容接口
- **自定义工作流** — 基于 LangGraph StateGraph 扩展 Planner/Executor
- **新增 Demo 场景** — 复用 Runtime，只需更换 Prompt 和工具集

---

## 许可证

MIT License
