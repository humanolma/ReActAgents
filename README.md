# ReAct Agent 服务系统

一个基于 LangGraph + FastAPI + Celery 的生产级 ReAct Agent 服务系统，支持多用户多会话、Human-in-the-Loop 人工审批、异步任务执行和会话状态恢复。

## 核心特性

- **ReAct Agent**：基于 LangGraph `create_react_agent` 实现 Thought → Action → Observation 循环
- **Human-in-the-Loop**：敏感工具调用前触发人工审批，支持 accept / reject / edit / response 四种响应
- **异步任务**：Celery 任务队列解耦 API 与 Agent 执行，接口立即返回 `task_id`，客户端轮询状态
- **状态恢复**：PostgreSQL 持久化 Agent 状态，Redis 管理会话/任务索引，支持客户端断线恢复
- **多模型支持**：OpenAI / 阿里通义千问 / OneAPI / Ollama 一键切换
- **MCP 工具生态**：集成高德地图 MCP Server，可扩展更多工具

## 技术栈

- **后端框架**：FastAPI + Uvicorn
- **Agent 框架**：LangGraph + LangChain
- **异步任务**：Celery + Redis
- **数据存储**：PostgreSQL（Agent 记忆持久化）+ Redis（会话状态）
- **部署**：Docker Compose

## 快速开始

### 1. 克隆项目

```bash
git clone https://github.com/yourusername/ReActAgentHILApiMultiSessionTask.git
cd ReActAgentHILApiMultiSessionTask
```

### 2. 创建虚拟环境

```bash
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 3. 配置环境变量

```bash
cp .env.example .env
# 编辑 .env 文件，填入你的 API Key
```

### 4. 启动数据库

```bash
cd docker/postgresql && docker-compose up -d
cd ../redis && docker-compose up -d
```

### 5. 启动服务

```bash
# 终端 1：启动 Celery Worker
celery -A 01_backendServer.celery_app worker --loglevel=info

# 终端 2：启动 FastAPI 服务
python 01_backendServer.py

# 终端 3：启动前端 Demo
python 02_frontendServer.py
```

## 项目结构

```
.
├── 01_backendServer.py      # FastAPI 后端服务入口
├── 02_frontendServer.py     # Rich CLI 前端 Demo
├── utils/
│   ├── config.py            # 全局配置
│   ├── models.py            # Pydantic 数据模型
│   ├── llms.py              # 多模型配置
│   ├── tools.py             # 工具定义 + HITL 包装
│   ├── tasks.py             # Celery 任务 + Agent 执行逻辑
│   ├── redis.py             # Redis 会话管理
│   └── models.py            # 请求/响应模型
├── docker/
│   ├── postgresql/          # PostgreSQL Docker Compose
│   └── redis/               # Redis Docker Compose
├── requirements.txt         # Python 依赖
├── .env.example             # 环境变量示例
└── README.md                # 项目说明
```

## 关键流程

```
用户输入问题
    ↓
FastAPI 接收请求 → Celery 异步执行 → 返回 task_id
    ↓
Agent 思考 → 需要调工具 → interrupt() 中断
    ↓
前端展示审批信息 → 用户决策
    ↓
Command(resume=...) 恢复 → Agent 继续执行 → 生成回答
```

## 注意事项

- 生产环境请使用 JWT 认证替代前端传入的 `user_id`
- 建议配置 Celery `acks_late=True` 和 `task_time_limit` 提升任务可靠性
- MCP Server 需要配置有效的高德地图 API Key


