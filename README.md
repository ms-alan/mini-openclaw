# Mini-OpenClaw 中文翻译

# Mini-OpenClaw

> 一个实操性的人工智能智能体教学与研究项目——通过研究 OpenClaw 架构，从零构建一个高性能的智能体。
>
> 

## 项目介绍

Mini-OpenClaw 是一个面向**学习和实验**构建的全栈人工智能智能体系统。它实现了3个逐步复杂的智能体引擎、一个多层级记忆系统以及一个现代化的聊天界面——所有代码均具备高可读性和教学价值。

本项目通过研究[OpenClaw](https://github.com/openclaw/openclaw) 和 [nanobot](https://github.com/HKUDS/nanobot) 设计而成，提取其核心概念并从零重新实现。

## 架构设计

```plain text
前端 (Next.js)  ←→  后端 (FastAPI)
                          ├── 3 个智能体引擎 (LangGraph / create_react_agent / 原生循环)
                          ├── 5节点状态图 (检索 → 推理 → 行动 → 反思 → 刷新)
                          ├── 工具系统 (Python 交互环境、终端、网络请求、文件读写)
                          ├── 多层级记忆 (每日日志 → MEMORY.md → 检索增强生成)
                          └── 多提供商大语言模型 (智谱, DeepSeek, OpenRouter, OpenAI, Ollama, SiliconFlow)
```

### 三个智能体引擎

| 引擎                     | 文件路径                                     | 用途                                                |
| ------------------------ | -------------------------------------------- | --------------------------------------------------- |
| **LangGraph**            | backend/graph/engines/langgraph_engine.py    | 教学核心——包含检索/推理/行动/反思/刷新的5节点状态图 |
| **create_react_agent**   | backend/graph/engines/create_agent_engine.py | 生产模式——LangGraph 预构建的 ReAct 智能体           |
| **原生循环（Raw Loop）** | backend/graph/engines/raw_loop_engine.py     | 极简约设计（约100行循环代码），无 LangChain 依赖    |

三个引擎均支持通过服务器向客户端推送（SSE）实现**真实的token级流式输出**。

## 快速开始

### 前置条件

- Python 3.12+

- Node.js 18+

- 一个大语言模型 API 密钥（智谱 GLM 免费额度可直接使用）

### 1. 后端部署

```bash
cd backend
pip install -r requirements.txt

# 配置 API 密钥
cp .env.example .env
# 编辑 .env 文件——至少需设置 ZHIPUAI_API_KEY（智谱API密钥）

# 启动服务器
python -m uvicorn app:app --host 0.0.0.0 --port 8002
```

### 2. 前端部署

```bash
cd frontend
npm install
npm run dev
# 打开 http://localhost:3000 即可访问前端界面
```

### 3. 测试验证

```bash
cd backend
pytest tests/ -v                              # 运行所有测试用例（共65个）
pytest tests/test_api_chat.py -v              # 仅运行聊天API相关测试
pytest tests/test_config.py -k test_config_defaults  # 运行单个指定测试用例
```

## 项目结构

```plain text
mini-openclaw/
├── backend/                  # FastAPI 后端
│   ├── app.py               # 入口文件、生命周期管理
│   ├── config.py             # 基于Pydantic的配置（支持JSON持久化）
│   ├── api/                  # REST API + SSE 接口
│   │   ├── chat.py           # POST /api/chat（流式输出接口）
│   │   ├── sessions.py       # 会话增删改查 + 自动标题生成
│   │   └── config_api.py     # 引擎/记忆/检索增强生成模式切换
│   ├── graph/                # 智能体核心模块
│   │   ├── agent.py          # AgentManager — 统一入口
│   │   ├── engines/          # 3个可互换的智能体引擎
│   │   ├── nodes/            # 推理、行动、检索、反思、记忆刷新节点
│   │   ├── session_manager.py
│   │   └── prompt_builder.py
│   ├── providers/            # 大语言模型提供商注册中心
│   │   ├── registry.py       # get_llm(), get_embeddings() 方法
│   │   └── base.py           # ProviderSpec 数据类
│   ├── tools/                # 智能体工具集
│   │   ├── python_repl_tool.py  # 沙箱化的Python交互环境（eval/exec）
│   │   ├── terminal_tool.py     # 基于白名单的终端工具
│   │   ├── fetch_url_tool.py    # 异步HTTP请求工具
│   │   └── file_tools.py        # 文件读写/列表工具
│   ├── memory/               # 多层级记忆系统
│   │   └── native/           # 每日日志 → MEMORY.md → 检索增强生成
│   ├── rag/                  # 混合BM25 + 向量检索（检索增强生成）
│   └── tests/                # 65个测试用例
├── frontend/                 # Next.js 14 + Tailwind 前端
│   └── src/
│       ├── app/              # App 路由
│       ├── components/       # 聊天界面、侧边栏、消息气泡等组件
│       └── lib/              # api.ts, store.tsx（状态管理）
├── docs/                     # 参考文档
│   ├── plans/                # 设计与实现方案
│   └── *.pdf / *.png         # 产品需求文档、架构图
├── nanobot/                  # 参考资料：nanobot 完整源码
└── docker-compose.yml        # Docker 编排配置
```

## 支持的大语言模型提供商

| 提供商                  | 默认模型        | 是否需要API密钥                  |
| ----------------------- | --------------- | -------------------------------- |
| 智谱 (Zhipu)            | glm-4.7-flash   | 是（需配置 ZHIPUAI_API_KEY）     |
| DeepSeek                | deepseek-chat   | 是（需配置 DEEPSEEK_API_KEY）    |
| OpenRouter              | claude-sonnet-4 | 是（需配置 OPENROUTER_API_KEY）  |
| OpenAI                  | gpt-4o          | 是（需配置 OPENAI_API_KEY）      |
| SiliconFlow（硅基流动） | Qwen2.5-7B      | 是（需配置 SILICONFLOW_API_KEY） |
| Ollama                  | qwen2.5:7b      | 否（本地部署）                   |

在 .env 文件中配置相关密钥——详见 .env.example 文件中的所有可配置项。

## 核心功能

- **3个智能体引擎**——通过侧边栏下拉菜单可在运行时切换

- **真实流式输出**——从大语言模型到浏览器的token级服务器向客户端推送（SSE）

- **工具调用**——沙箱化Python交互环境、白名单终端、网络请求、文件读写

- **多层级记忆**——每日日志通过大语言模型整理后自动写入 MEMORY.md

- **检索增强生成（RAG）模式**——混合BM25 + 向量检索（支持Milvus/FAISS/pgvector）

- **会话管理**——支持会话创建、重命名、删除及自动标题生成

- **模型厂商热切换**——无需重启即可切换大语言模型服务

## 参考资料

docs/ 与 nanobot/ 目录包含项目开发前期的参考资料：

- docs/Mini-OpenClaw 开发需求文档 (PRD).pdf——原始产品需求文档

- docs/Mini-OpenClaw_README.pdf——原始设计参考文档

- docs/工业级智能体记忆系统开发实践.pdf——记忆系统设计参考文档

- docs/openclaw架构图.png / nanobot架构图.png——架构示意图

- nanobot/——nanobot 完整源码，作为实现参考（详见 [deepwiki.com/HKUDS/nanobot](https://deepwiki.com/HKUDS/nanobot)）

## 开源协议

本项目用于**教学与研究**。

> （注：文档部分内容可能由 AI 生成）