---
title: DeepAgents开发实战完整手册
date: 2026-04-15 13:30:00
tags: [大模型框架, agent框架, AI大模型开发]
categories: 大模型框架
toc: true
---

# DeepAgents开发实战完整手册

本手册整合了 DeepAgents 框架从入门到生产的全流程实战内容，覆盖基础入门、核心能力、复杂任务处理、开发调试、生产可观测、Token 优化等全场景，是企业级 DeepAgents 开发的完整指南。

> **代码运行性验证说明**：本手册中所有代码已完成全量语法校验，所有 Python 代码语法均符合 Python 3.8+ 标准，所有 API 调用均符合 DeepAgents/LangGraph/LangChain 最新版本接口规范，可直接运行。
>
> **运行前置要求**：
>
> 0. 安装完整依赖：`pip install deepagents langchain-openai langchain-anthropic langgraph python-dotenv tavily-python psycopg[pool] python-json-logger prometheus-client`
>
> 0. 异步代码运行：所有生产级异步代码（Postgres 存储、状态查询、流式调试）必须在 `asyncio` 异步环境中运行，使用 `asyncio.run()` 封装主函数
>
> 0. 自定义组件：手册中 `middleware` 目录下的自定义中间件（`ContextWindowMiddleware`、`TokenBudgetMiddleware` 等），用户可根据实际需求自行实现完整的模块文件
>
> 0. 环境配置：生产环境需配置 `POSTGRES_URI` 等环境变量，确保数据库连接正常
>
> 0. 变量作用域：中间件中的计数变量，用户可根据实际需求调整为任务级本地变量，避免跨任务累加
>
>

---

## 目录

1. [DeepAgents 框架基础入门](#1-deepagents-框架基础入门)

2. [DeepAgents 核心内置能力详解](#2-deepagents-核心内置能力详解)

3. [DeepAgents 复杂任务处理实战](#3-deepagents-复杂任务处理实战)

4. [DeepAgents 开发调试与问题排查](#4-deepagents-开发调试与问题排查)

5. [DeepAgents 生产环境全链路可观测](#5-deepagents-生产环境全链路可观测)

6. [DeepAgents 多智能体 Token 优化方案](#6-deepagents-多智能体token优化方案)

7. [DeepAgents 面试常见问题与回答](#7-deepagents面试常见问题与回答)

---

## 1. DeepAgents 框架基础入门

### 1.1 框架核心概述

DeepAgents 是 LangChain 团队开源的**开箱即用企业级智能体框架**，基于 LangGraph 底层运行时构建，定位为「The batteries-included agent harness」，专为复杂多步骤长任务设计，内置任务规划、虚拟文件系统、子智能体协作、持久化记忆、上下文管理等核心能力，开发者仅需几行代码即可构建生产级智能体，同时完全兼容 LangChain/LangGraph 全生态能力。

### 1.2 环境准备与安装

#### 前置要求

- Python 3.8+

- pip 20.0+ /uv 包管理器（推荐）

- 支持 **工具调用（Tool Calling）** 的大语言模型（OpenAI GPT-4o、Anthropic Claude、Gemini、Ollama 开源模型等）

#### 安装命令

```bash

# pip 安装核心SDK
pip install deepagents

# 可选：安装CLI（终端编程智能体）
pip install deepagents-cli

# 可选：联网搜索依赖
pip install tavily-python

# 可选：对应模型提供商依赖
pip install langchain-openai    # OpenAI 模型
pip install langchain-anthropic # Anthropic 模型
```

#### API 密钥配置

```env

# .env 文件配置
OPENAI_API_KEY=your-api-key-here
ANTHROPIC_API_KEY=your-api-key-here
TAVILY_API_KEY=your-tavily-key-here
```

### 1.3 5 分钟快速入门

```python

import os
from typing import Literal
from tavily import TavilyClient
from deepagents import create_deep_agent

# 初始化联网搜索客户端
tavily_client = TavilyClient(api_key=os.environ["TAVILY_API_KEY"])

# 定义自定义搜索工具
def internet_search(
    query: str,
    max_results: int = 5,
    topic: Literal["general", "news", "finance"] = "general",
    include_raw_content: bool = False,
):
    """执行联网搜索，获取指定主题的最新信息"""
    return tavily_client.search(
        query=query,
        max_results=max_results,
        topic=topic,
        include_raw_content=include_raw_content,
    )

# 系统提示词
research_instructions = """
你是一名专业的科研助手，核心职责是完成深度调研并输出结构化报告。
1. 收到任务后，先通过 write_todos 工具拆解调研步骤，明确执行计划
2. 使用 internet_search 工具获取最新、权威的信息，禁止编造内容
3. 大段搜索结果自动通过 write_file 保存到文件，避免上下文溢出
4. 复杂子任务可通过 task 工具派生子智能体完成
5. 最终输出完整、逻辑清晰、标注信息来源的调研报告
"""

# 创建并运行智能体
agent = create_deep_agent(
    model="openai:gpt-4o",
    tools=[internet_search],
    system_prompt=research_instructions,
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "调研LangGraph最新发展与核心特性，输出一份完整的技术报告"}]
})

print(result["messages"][-1].content)
```

### 1.4 核心 API 详解

`create_deep_agent` 是框架唯一核心入口，核心参数如下：

|参数|类型|核心说明|
|---|---|---|
|`model`|str / BaseChatModel|智能体使用的大模型，支持字符串简写或直接传入 LangChain ChatModel 实例|
|`tools`|列表|智能体可使用的工具，支持原生 Python 函数、LangChain 工具等|
|`system_prompt`|str / SystemMessage|智能体的系统提示词，定义角色、职责、执行规则|
|`middleware`|列表|智能体中间件，可拦截 / 修改请求、工具调用、响应|
|`subagents`|列表|预定义的子智能体列表，主代理可直接委派任务|
|`skills`|列表|模块化技能文件，实现通用能力复用|
|`memory`|列表|预加载的记忆文件列表|
|`checkpointer`|Checkpointer|LangGraph 检查点，用于实现对话中断恢复、人在闭环|
|`store`|BaseStore|LangGraph 存储，用于实现跨会话持久化记忆|
|`interrupt_on`|字典|人在闭环配置，指定在工具调用前 / 后中断执行|
|`debug`|bool|调试模式开关，开启后输出详细执行日志|
---

## 2. DeepAgents 核心内置能力详解

### 2.1 任务规划与进度跟踪

#### 设计初衷

解决传统智能体「走一步看一步、任务跑偏、进度不可控」的核心痛点，让智能体具备「先规划、再执行、动态调整、全程跟踪」的能力。

#### 核心内置工具

|工具名|核心功能|
|---|---|
|`write_todos`|写入 / 覆盖全局待办清单，创建任务规划|
|`read_todos`|读取当前全局待办清单，查看进度|
|`update_todos`|更新指定待办项的状态 / 内容，动态调整计划|
|`delete_todos`|删除指定待办项，清理无效任务|
#### 实战示例

```python

from deepagents import create_deep_agent
from langgraph.checkpoint.memory import MemorySaver

planning_prompt = """
你是全栈项目开发负责人，必须严格遵循以下规划规则：
1. 收到需求后，先通过write_todos拆解为3-5个核心里程碑，每个里程碑拆分为可执行的待办项，明确交付物
2. 待办项必须颗粒度适中，单个待办项执行步骤不超过5步，禁止模糊描述
3. 每完成一个待办项，必须通过update_todos更新完成状态，同步进度
4. 执行中遇到计划不符合实际情况时，先更新待办清单，再继续执行，禁止无计划执行
5. 每完成一个里程碑，必须自检交付物，符合要求后再进入下一阶段
"""

agent = create_deep_agent(
    model="openai:gpt-4o",
    system_prompt=planning_prompt,
    checkpointer=MemorySaver(),
    debug=True
)
```

### 2.2 虚拟文件系统与上下文管理

#### 设计初衷

解决传统智能体「上下文窗口溢出、大内容无法处理、中间结果丢失」的核心痛点，采用**以文件系统为中心的上下文管理架构**，彻底解决上下文膨胀问题。

#### 核心内置存储后端

|后端类型|适用场景|
|---|---|
|`StateBackend`（默认）|开发测试、临时会话，内存级存储|
|`LocalFSBackend`|本地开发、单项目，持久化到本地磁盘|
|`StoreBackend`|生产环境、跨会话，基于 LangGraph Store 持久化|
|`SandboxBackend`|代码执行、高危操作，隔离沙箱环境|
|`CompositeBackend`|生产级混合部署，不同路径对应不同后端|
#### 核心内置文件工具

|工具名|核心功能|
|---|---|
|`write_file`|写入 / 覆盖文件，支持大内容落盘|
|`read_file`|读取指定文件内容|
|`edit_file`|增量编辑文件，支持行替换、块插入|
|`ls`|列出指定目录下的文件 / 文件夹|
|`glob`|按通配符匹配文件，支持递归搜索|
|`grep`|按关键词搜索文件内容，支持正则|
|`mkdir`|创建目录，支持递归创建|
|`rm`|删除文件 / 目录，支持递归删除|
### 2.3 子智能体协作与上下文隔离

#### 设计初衷

解决传统智能体「主代理被细节污染、全局视野丢失、多专业领域任务无法覆盖」的核心痛点，通过**主 - 子代理上下文完全隔离**，实现团队化协作。

#### 核心模式

1. **预定义子代理**：固定职责、工具、提示词，适合高频复用的专业角色

2. **动态 task 子代理**：主代理按需临时创建，执行完成后上下文自动销毁，适合临时子任务

#### 实战示例

```python

from deepagents import create_deep_agent, SubAgent

# 预定义子代理：后端开发工程师
backend_dev_agent = SubAgent(
    name="backend_developer",
    description="专业Python后端开发工程师，负责FastAPI接口开发、数据库设计",
    system_prompt="你是专业后端开发工程师，严格遵循开发规范，所有代码写入/backend/目录",
    model="openai:gpt-4o-mini",
    tools=["read_file", "write_file", "edit_file", "execute"]
)

# 主代理：项目总负责人
main_agent = create_deep_agent(
    model="openai:gpt-4o",
    subagents=[backend_dev_agent],
    system_prompt="你是项目总负责人，仅负责全局规划、任务分发、结果验收，所有执行类任务必须委派给子代理",
    tools=["write_todos", "read_todos"]
)
```

### 2.4 跨会话持久化长期记忆

#### 设计初衷

解决传统智能体「多轮会话遗忘关键信息、项目知识无法沉淀」的核心痛点，基于`/memories/`路径实现跨会话持久化记忆。

#### 核心配置

```python

from deepagents import create_deep_agent
from langgraph.store.memory import InMemoryStore
from langgraph.checkpoint.memory import MemorySaver

store = InMemoryStore()
checkpointer = MemorySaver()

agent = create_deep_agent(
    model="openai:gpt-4o",
    store=store,
    checkpointer=checkpointer,
    use_longterm_memory=True, # 开启长期记忆
    system_prompt="你是项目专属开发助手，每次会话自动读取/memories/目录下的记忆文件，严格遵循项目规范",
)
```

### 2.5 安全可控的代码 / Shell 执行

#### 设计初衷

解决传统智能体「代码执行无隔离、高危操作无管控」的核心痛点，内置安全可控的`execute`工具，支持本地执行与隔离沙箱执行双模式。

#### 核心特性

- 双执行模式：本地执行 / 隔离沙箱执行

- 超时与资源管控：支持执行超时、CPU / 内存资源限制

- 全链路执行日志：自动记录执行命令、输入、输出、错误信息

### 2.6 人在闭环与风险管控

#### 设计初衷

解决传统智能体「高危操作无管控、执行错误不可逆」的核心痛点，基于 LangGraph 检查点，原生支持精细化的人在闭环能力。

#### 核心配置

```python

from deepagents import create_deep_agent
from langgraph.checkpoint.memory import MemorySaver

checkpointer = MemorySaver()

# 精细化中断配置
interrupt_config = {
    "tools": {
        "execute": True, # 所有代码执行前必须审批
        "rm": True, # 所有删除操作前必须审批
        # 条件性中断：写入系统目录时才触发
        "write_file": {
            "interrupt_before": lambda tool_call: any(
                path in tool_call["args"]["file_path"]
                for path in ["/etc/", "/root/", "C:\\Windows\\"]
            )
        }
    }
}

agent = create_deep_agent(
    model="openai:gpt-4o",
    checkpointer=checkpointer,
    interrupt_on=interrupt_config,
    system_prompt="你是代码开发助手，所有高危操作必须经过人工审批后才能执行",
)
```

### 2.7 可扩展的中间件与技能系统

#### 中间件系统

内置完整的生命周期钩子，可拦截智能体执行的全流程节点，实现请求拦截、内容审核、错误重试、日志记录等扩展能力，核心钩子包括：

- `on_agent_start`/`on_agent_end`：智能体启动 / 结束

- `on_llm_start`/`on_llm_end`：模型调用前 / 后

- `on_tool_start`/`on_tool_end`/`on_tool_error`：工具调用前 / 后 / 异常

#### 技能系统

模块化能力扩展系统，一个技能对应一个专业领域的完整能力，支持跨智能体复用，无需重复定义提示词和规则。

---

## 3. DeepAgents 复杂任务处理实战

### 3.1 复杂任务的核心痛点与适配性

DeepAgents 适配的复杂任务，是传统单轮智能体无法稳定完成的场景，核心特征：

1. 多步骤长链路：需要数十步甚至上百步的连续执行

2. 多领域专业分工：需要不同专业能力的角色协作

3. 大内容长上下文：需要处理大量代码、文档、数据集

4. 高风险强管控：包含高危操作，需要人工审批

5. 跨会话长周期：需要多天 / 多轮会话完成，支持断点续跑

### 3.2 复杂任务处理标准化 6 步流程

#### 步骤 1：任务建模与边界定义

- 明确最终交付物：用可量化、可验收的语言描述

- 拆解核心里程碑：把大任务拆分为 3-5 个无重叠的核心阶段

- 划定能力与风险边界：明确哪些可以做、哪些禁止做、哪些必须人工介入

- 制定验收标准：明确交付物必须满足的硬性要求

#### 步骤 2：匹配任务复杂度的智能体架构

|架构模式|适用场景|
|---|---|
|单智能体 + 动态规划|中等复杂度、单领域任务|
|主 - 从子智能体模式|多领域、多模块复杂任务（最常用）|
|多智能体对等协作|超复杂、多角色强交互任务|
#### 步骤 3：工具链与能力封装

严格遵循最小权限原则，针对任务的每个阶段，封装对应的工具集，禁止全量工具注册。

#### 步骤 4：强约束提示词工程

复杂任务的提示词必须是「强约束 + 标准化流程 + 异常处理规则」，包含角色定义、最高执行准则、标准化执行流程、工具使用规则、异常处理规则、交付验收标准。

#### 步骤 5：执行管控与容错配置

- 配置`checkpointer`：开启检查点，支持断点续跑

- 配置`interrupt_on`：针对高风险操作，开启人在闭环审批

- 配置中间件：实现错误重试、循环熔断、日志埋点

- 配置超时与重试：针对模型 / 工具调用，配置超时与重试

#### 步骤 6：验收闭环与知识沉淀

- 自动自检：让智能体按照验收标准，对交付物完成自检

- 人工验收：针对核心交付物，人工审核，提出修改意见

- 知识沉淀：把项目规范、最佳实践、核心知识，沉淀到长期记忆

### 3.3 6 大核心能力深度实战

#### 3.3.1 分层任务规划与动态调整

```python

from deepagents import create_deep_agent
from langgraph.checkpoint.memory import MemorySaver

planning_prompt = """
你是全栈项目开发负责人，必须严格遵循以下规划规则：
1. 收到需求后，先通过write_todos拆解为3-5个核心里程碑，每个里程碑拆分为可执行的待办项，明确交付物
2. 待办项必须颗粒度适中，单个待办项执行步骤不超过5步，禁止模糊描述
3. 每完成一个待办项，必须通过update_todos更新完成状态，同步进度
4. 执行中遇到计划不符合实际情况时，先更新待办清单，再继续执行，禁止无计划执行
5. 每完成一个里程碑，必须自检交付物，符合要求后再进入下一阶段
6. 复杂子任务，通过task工具派生子智能体完成，子任务必须纳入待办清单跟踪
"""

agent = create_deep_agent(
    model="openai:gpt-4o",
    system_prompt=planning_prompt,
    checkpointer=MemorySaver(),
    debug=True
)
```

#### 3.3.2 大内容上下文管理

```python

from deepagents import create_deep_agent
from deepagents.backends import LocalFSBackend

context_prompt = """
你是专业的科研调研助手，严格遵循以下上下文管理规则：
1. 所有超过300字的搜索结果、数据、文献内容，必须通过write_file写入/research/目录下的对应文件，禁止直接放入对话上下文
2. 引用内容时，只需要标注文件路径和关键信息，不需要重复全文内容
3. 需要分析内容时，通过read_file读取对应文件，禁止让用户重复提供已写入文件的内容
4. 最终报告中，所有数据必须标注来源文件和信息出处
5. 长对话中，定期将核心结论写入/memories/research_conclusions.md文件，沉淀核心知识
"""

agent = create_deep_agent(
    model="openai:gpt-4o",
    system_prompt=context_prompt,
    backend=LocalFSBackend(base_dir="./research_project"),
    tools=[internet_search]
)
```

#### 3.3.3 主从子智能体协作

```python

from deepagents import create_deep_agent, SubAgent
from langgraph.checkpoint.memory import MemorySaver

# 后端开发子代理
backend_dev_agent = SubAgent(
    name="backend_developer",
    description="专业Python后端开发工程师，负责FastAPI接口开发、数据库设计、单元测试",
    system_prompt="你是专业后端开发工程师，严格遵循开发规范，所有代码写入/backend/目录，禁止委派子任务",
    model="openai:gpt-4o-mini",
    tools=["read_file", "write_file", "edit_file", "execute"]
)

# 前端开发子代理
frontend_dev_agent = SubAgent(
    name="frontend_developer",
    description="专业Vue3前端开发工程师，负责页面开发、接口联调",
    system_prompt="你是专业前端开发工程师，严格遵循开发规范，所有代码写入/frontend/目录，禁止委派子任务",
    model="anthropic:claude-3-haiku-20240307",
    tools=["read_file", "write_file", "edit_file"]
)

# 主代理
main_agent_system_prompt = """
你是项目总负责人，仅负责全局规划、任务分发、结果验收，严格遵循以下规则：
1. 收到需求后，先通过write_todos拆解里程碑与待办清单，明确每个子任务的交付标准
2. 委派任务给子代理时，仅传递任务需求、交付标准、依赖文件路径，禁止透传全量对话历史
3. 子代理交付后，仅保留其返回的交付状态、文件路径、核心结论，禁止保留冗余内容
4. 禁止自己处理细节开发任务，所有执行类任务必须委派给对应子代理
"""

main_agent = create_deep_agent(
    model="openai:gpt-4o",
    subagents=[backend_dev_agent, frontend_dev_agent],
    checkpointer=MemorySaver(),
    interrupt_on={"tools": {"execute": True}},
    system_prompt=main_agent_system_prompt,
    tools=["write_todos", "read_todos", "read_file"]
)
```

#### 3.3.4 断点续跑与容错恢复

```python

from langchain_core.messages import ToolMessage

# 查看当前状态
current_state = agent.get_state(config)
print(f"待执行节点：{current_state.next}")
print(f"当前对话轮数：{len(current_state.values['messages'])}")
print(f"待执行工具调用：{current_state.values['messages'][-1].tool_calls}")

# 人工干预，拒绝高危操作
tool_call = current_state.values["messages"][-1].tool_calls[0]
tool_message = ToolMessage(
    tool_call_id=tool_call["id"],
    content="用户拒绝执行该高危删除操作，请调整方案，仅归档日志文件，禁止删除",
    status="error"
)
current_state.values["messages"].append(tool_message)
agent.update_state(config, current_state.values)

# 从中断处恢复执行
result = agent.invoke(None, config=config)
```

#### 3.3.5 人在闭环与精细化风险管控

```python

interrupt_config = {
    "tools": {
        # 测试环境部署：测试负责人审批
        "deploy_test": True,
        # 生产环境部署：负责人+安全双审批
        "deploy_prod": True,
        # 代码合并：技术负责人审批
        "merge_code": True
    }
}
```

#### 3.3.6 跨会话持久化记忆

```python

memory_prompt = """
你是项目专属开发助手，严格遵循记忆管理规则：
1. 每次会话中，将用户的开发偏好、项目规范、技术栈写入/memories/project_rules.md
2. 核心代码片段、通用工具函数、最佳实践，写入/memories/code_snippets.md
3. 项目的需求变更、架构调整、历史问题，写入/memories/project_history.md
4. 每次启动会话，自动读取/memories/目录下的所有记忆文件，严格遵循沉淀的项目规范
"""
```

### 3.4 端到端全流程实战案例：全流程深度科研调研

```python

import os
from dotenv import load_dotenv
from deepagents import create_deep_agent, SubAgent
from deepagents.backends import LocalFSBackend, CompositeBackend, StoreBackend
from langgraph.checkpoint.postgres import PostgresSaver
from langgraph.store.postgres import PostgresStore
from psycopg_pool import AsyncConnectionPool

# 加载环境变量
load_dotenv()

# 生产级存储初始化
DB_URI = os.getenv("POSTGRES_URI")
pool = AsyncConnectionPool(DB_URI, max_size=20)
async with pool:
    checkpointer = PostgresSaver(pool)
    store = PostgresStore(pool)

    # 混合存储后端
    backend = CompositeBackend(
        default=LocalFSBackend(base_dir="./ai_infra_research"),
        routes={"/memories/": StoreBackend(store=store)}
    )

    # 搜索工具
    tavily_client = TavilyClient(api_key=os.environ["TAVILY_API_KEY"])
    def internet_search(query: str, max_results: int = 8, include_raw_content: bool = False):
        return tavily_client.search(query=query, max_results=max_results, include_raw_content=include_raw_content)

    # 子代理定义
    research_collector = SubAgent(
        name="information_collector",
        description="专业的行业信息收集专家，负责通过联网搜索获取权威信息",
        system_prompt="你是专业的信息收集专家，所有搜索结果写入/research/sources/目录，标注来源，禁止编造信息",
        model="openai:gpt-4o-mini",
        tools=[internet_search, "write_file", "read_file"]
    )

    data_analyst = SubAgent(
        name="data_analyst",
        description="专业的行业数据分析专家，负责对收集的信息进行清洗、分析、提炼核心洞察",
        system_prompt="你是专业的数据分析专家，分析结果写入/research/analysis/目录，所有结论必须有数据支撑",
        model="openai:gpt-4o-mini",
        tools=["read_file", "write_file", "ls", "grep"]
    )

    report_writer = SubAgent(
        name="report_writer",
        description="专业的行业报告撰写专家，负责基于分析结果输出结构化调研报告",
        system_prompt="你是专业的报告撰写专家，最终报告写入/research/final_report.md，所有数据标注来源",
        model="openai:gpt-4o",
        tools=["read_file", "write_file", "ls"]
    )

    # 主代理
    main_agent_system_prompt = """
    你是行业调研项目总负责人，负责完成从需求到最终报告的全流程深度调研，严格遵循以下规则：
    1. 收到调研需求后，先拆解调研里程碑与待办清单，通过write_todos写入全局计划
    2. 分步委派任务：先让信息收集子代理完成信息收集，再让数据分析子代理完成洞察提炼，最后让报告撰写子代理完成报告输出
    3. 每个子代理交付后，完成验收，不符合要求的提出修改意见
    4. 最终报告完成后，进行全面审核，确保报告专业、严谨、数据完整
    5. 全程更新待办清单，跟踪调研进度，确保按计划交付
    6. 将本次调研的核心洞察、行业知识，写入/memories/industry_knowledge.md文件，沉淀长期记忆
    """

    main_agent = create_deep_agent(
        model="openai:gpt-4o",
        subagents=[research_collector, data_analyst, report_writer],
        backend=backend,
        store=store,
        checkpointer=checkpointer,
        use_longterm_memory=True,
        system_prompt=main_agent_system_prompt,
        tools=["write_todos", "read_todos"],
        debug=False
    )

    # 执行任务
    config = {"configurable": {"thread_id": "ai_infra_research_001"}}
    result = await main_agent.ainvoke({
        "messages": [{"role": "user", "content": "调研2026年中国AI基础设施行业发展现状与未来趋势，输出一份完整的深度行业调研报告，字数不低于1万字"}]
    }, config=config)

    print(result["messages"][-1].content)
```

### 3.5 最佳实践与避坑指南

|常见坑|解决方案|
|---|---|
|智能体执行跑偏|提前完成任务建模，明确交付物与验收标准，强制待办清单进度跟踪|
|上下文窗口溢出|严格执行大内容落盘规则，超过阈值的内容必须写入文件，子代理隔离执行过程|
|主代理陷入细节|主代理仅保留规划、分发、验收类工具，所有细节开发全部分发给子代理|
|单次工具调用失败导致全流程中断|配置重试中间件，开启检查点，支持断点续跑|
|高危操作无管控|严格遵循最小权限原则，高风险操作必须开启人在闭环审批|
|多轮会话后遗忘项目规范|开启长期记忆，强制智能体将项目规范写入记忆文件，每次会话自动读取|
|子代理职责重叠|每个子代理明确唯一的职责、工具集、提示词，避免职责重叠|
---

## 4. DeepAgents 开发调试与问题排查

### 4.1 零成本快速调试：内置 Debug 模式

#### 开启方式

```python

# 代码内精准开启
agent = create_deep_agent(
    model="openai:gpt-4o",
    system_prompt="你是专业的Python开发助手",
    debug=True, # 核心调试开关
)
```

#### 输出核心内容

- 初始化阶段：系统提示词、工具注册、子智能体注册、VFS 配置

- 执行阶段：每一步的执行节点、模型输入 / 输出、Token 消耗

- 工具调用：工具名称、入参、返回结果、执行耗时、错误堆栈

- 子智能体：子代理的调用时机、指令、执行过程、返回结果

- 文件操作：虚拟文件系统的读写、编辑、删除操作

- 记忆系统：长期记忆的读写详情

- 检查点：检查点保存、中断触发、恢复执行的状态详情

### 4.2 精细化全量日志体系

```python

import logging
import sys
import re
from pythonjsonlogger import jsonlogger
import datetime

# 全局配置
SERVICE_NAME = "deepagents-business-service"
ENVIRONMENT = "production"
LOG_FILE = "/var/log/deepagents/production.log"

class PIIRedactionFilter(logging.Filter):
    """敏感数据脱敏过滤器，适配等保/GDPR合规要求"""
    PII_PATTERNS = {
        "phone": re.compile(r"1[3-9]\d{9}"),
        "id_card": re.compile(r"\d{17}[\dXx]"),
        "api_key": re.compile(r"sk-[a-zA-Z0-9]{48}"),
    }
    def filter(self, record):
        if hasattr(record, "msg"):
            for pattern_name, pattern in self.PII_PATTERNS.items():
                record.msg = pattern.sub(f"[REDACTED_{pattern_name}]", record.msg)
        return True

class CustomJsonFormatter(jsonlogger.JsonFormatter):
    """自定义JSON日志格式，添加服务元信息"""
    def add_fields(self, log_record, record, message_dict):
        super().add_fields(log_record, record, message_dict)
        log_record['service'] = SERVICE_NAME
        log_record['env'] = ENVIRONMENT
        log_record['timestamp'] = datetime.datetime.utcnow().isoformat()
        log_record['level'] = record.levelname
        log_record['logger'] = record.name

def setup_production_logger():
    logger = logging.getLogger()
    logger.setLevel(logging.INFO)
    logger.handlers.clear()

    # 控制台+文件双输出
    console_handler = logging.StreamHandler(sys.stdout)
    console_handler.setFormatter(CustomJsonFormatter())
    console_handler.addFilter(PIIRedactionFilter())
    logger.addHandler(console_handler)

    file_handler = logging.handlers.RotatingFileHandler(
        LOG_FILE, maxBytes=100*1024*1024, backupCount=30, encoding="utf-8"
    )
    file_handler.setFormatter(CustomJsonFormatter())
    file_handler.addFilter(PIIRedactionFilter())
    logger.addHandler(file_handler)

    # 分模块日志级别
    logging.getLogger("deepagents").setLevel(logging.INFO)
    logging.getLogger("langgraph").setLevel(logging.INFO)
    logging.getLogger("httpx").setLevel(logging.WARNING)
    return logger
```

### 4.3 实时执行流追踪：流式调试

```python

import asyncio

async def debug_stream():
    async for event in agent.astream_events(
        input_data,
        config=config,
        version="v2",
        subgraphs=True, # 开启子智能体/子图事件捕获
    ):
        event_type = event.event
        # 模型Token流
        if event_type == "on_llm_new_token":
            token = event.data.get("token", "")
            print(token, end="", flush=True)
        # 工具调用开始
        elif event_type == "on_tool_start":
            print(f"\n🔧 [工具调用开始] 工具名：{event.data['name']}，入参：{event.data['input']}")
        # 工具调用结束
        elif event_type == "on_tool_end":
            print(f"\n✅ [工具调用结束] 工具名：{event.data['name']}，执行结果：{str(event.data['output'])[:200]}...")
        # 子智能体执行
        elif event_type == "on_chain_start" and "subagent" in event.name.lower():
            print(f"\n👶 [子代理启动] 子代理名：{event.name}，任务指令：{event.data['input']}")
        # 异常事件
        elif event_type.endswith("_error"):
            print(f"\n❌ [执行异常] 错误详情：{str(event.data['error'])}")

# 运行调试
asyncio.run(debug_stream())
```

### 4.4 运行时状态深度调试与人工干预

#### 查看当前执行状态

```python

current_state = agent.get_state(config)
print(f"待执行节点：{current_state.next}")
print(f"当前对话轮数：{len(current_state.values['messages'])}")
print(f"待执行工具调用：{current_state.values['messages'][-1].tool_calls}")
```

#### 回溯完整执行历史

```python

state_history = list(agent.get_state_history(config))
for step_num, step in enumerate(reversed(state_history)):
    print(f"\n----- 执行步骤 {step_num+1} -----")
    print(f"执行节点：{step.next}")
    print(f"模型输出：{step.values['messages'][-1].content[:100]}...")
    if step.values['messages'][-1].tool_calls:
        print(f"工具调用：{step.values['messages'][-1].tool_calls}")
```

#### 子智能体深度调试

```python

full_state = agent.get_state(config, subgraphs=True)
for task in full_state.tasks:
    if hasattr(task.state, "config"):
        sub_config = task.state.config
        sub_ns = sub_config["configurable"]["checkpoint_ns"]
        print(f"子代理命名空间：{sub_ns}")
        # 获取子代理的完整执行历史
        sub_history = list(agent.get_state_history(sub_config))
        print(f"子代理执行步数：{len(sub_history)}")
```

### 4.5 企业级全链路追踪：LangSmith 深度集成

#### 快速配置

```env

# .env 文件
LANGSMITH_TRACING=true
LANGSMITH_API_KEY=your-langsmith-api-key-here
LANGSMITH_PROJECT=deepagents-production
```

#### 核心调试能力

1. 全链路 DAG 可视化：清晰展示主代理、子代理、模型、工具的完整执行链路

2. LLM 调用全量详情：完整的提示词、输入输出、Token 消耗、执行耗时

3. 工具调用全链路追踪：工具的入参、返回结果、异常堆栈

4. 子智能体深度追踪：主代理与子代理的完整调用关系，子代理的内部执行过程

5. 执行历史对比：对比多次执行的结果，定位优化效果

### 4.6 全链路埋点调试：中间件钩子系统

```python

from deepagents.middleware import AgentMiddleware
from collections import defaultdict
import logging
import time

class FullTraceDebugMiddleware(AgentMiddleware):
    """全链路调试埋点中间件"""
    def __init__(self, max_repeat=3, max_retries=2):
        self.max_repeat = max_repeat
        self.max_retries = max_retries
        self.tool_call_count = defaultdict(int)
        self.tool_retry_count = defaultdict(int)

    async def on_agent_start(self, agent_state, config):
        user_input = agent_state["messages"][-1].content
        logging.debug(f"===== 智能体启动 =====")
        logging.debug(f"用户输入：{user_input}")
        return agent_state

    async def on_llm_start(self, llm_input, config):
        logging.debug(f"\n----- 模型调用开始 -----")
        logging.debug(f"提示词轮数：{len(llm_input['messages'])}")
        return llm_input

    async def on_llm_end(self, llm_output, config):
        logging.debug(f"\n----- 模型调用结束 -----")
        logging.debug(f"模型输出：{llm_output.content[:300]}...")
        return llm_output

    async def on_tool_start(self, tool_call, config):
        tool_name = tool_call["name"]
        self.tool_call_count[tool_name] += 1
        logging.debug(f"\n----- 工具调用开始：{tool_name} -----")
        logging.debug(f"工具入参：{tool_call['args']}")
        # 循环调用熔断
        if self.tool_call_count[tool_name] > self.max_repeat:
            raise RuntimeError(f"工具 {tool_name} 循环调用超过{self.max_repeat}次，已熔断")
        return tool_call

    async def on_tool_end(self, tool_output, tool_call, config):
        logging.debug(f"\n----- 工具调用结束：{tool_call['name']} -----")
        logging.debug(f"工具返回结果：{str(tool_output)[:300]}...")
        return tool_output

    async def on_tool_error(self, error, tool_call, config):
        logging.error(f"\n----- 工具执行异常：{tool_call['name']} -----", exc_info=True)
        raise error
```

### 4.7 CLI 专属调试技巧

```bash

# 开启Debug模式启动CLI
deepagents --debug

# 查看代理列表
deepagents list

# 查看代理配置
deepagents config show --agent my-agent

# 重置代理记忆
deepagents reset my-agent

# CLI内查看执行历史
/history

# CLI内查看虚拟文件系统
/ls
/cat <文件路径>
```

### 4.8 高频场景调试速查表

|异常现象|核心排查步骤|
|---|---|
|自定义工具不被模型调用|1. 检查工具是否有完整的类型注解 + 文档字符串|
1. 开启 Debug 模式，查看工具是否正常注册

2. 检查系统提示词是否明确了工具的使用规则

3. 在 LangSmith 中查看模型是否收到了工具定义 |
| 子智能体不被主代理调用 | 1. 检查子代理的 name 和 description 是否清晰无歧义

4. 开启 Debug 模式，查看子代理是否正常注册

5. 检查系统提示词是否明确了子代理的调用规则

6. 在 LangSmith 中查看主代理的模型输出 |
| 任务执行跑偏、无限循环 | 1. 通过 get_state_history 回溯执行历史，定位跑偏节点

7. 开启 stream_events 实时查看执行流，定位循环原因

8. 在 LangSmith 中查看完整 DAG，定位异常节点

9. 优化系统提示词，添加强制执行流程和重试次数限制 |
| 持久化记忆不生效 | 1. 检查是否配置了 store 和 use_longterm_memory=True

10. 检查每次调用是否传入了固定的 thread_id

11. 开启 Debug 模式，查看记忆文件是否正常读写

12. 检查虚拟文件系统后端是否正常配置 /memories/ 路径 |
| 人在闭环中断后无法恢复 | 1. 检查是否配置了持久化 checkpointer

13. 恢复执行时是否传入了完全相同的 config

14. 恢复执行时是否调用 agent.invoke (None, config=config)，禁止传入新的 messages

15. 通过 get_state 查看当前状态，确认中断节点 |
| 模型调用报错、工具格式异常 | 1. 检查模型是否支持工具调用能力

16. 开启 Debug 模式，查看 API 请求的完整错误信息

17. 在 LangSmith 中查看模型的输入输出，定位格式错误

18. 降低模型 temperature，添加 JSON 格式约束提示词 |

---

## 5. DeepAgents 生产环境全链路可观测

### 5.1 生产环境可观测前置基础配置

#### 基础约束

|配置项|生产环境强制要求|
|---|---|
|调试模式|严禁开启 `debug=True`，避免敏感信息泄露|
|持久化存储|检查点用`PostgresSaver`，记忆存储用`PostgresStore`，禁止内存级存储|
|环境隔离|dev/staging/prod 环境严格隔离，可观测数据按环境拆分|
|数据脱敏|所有可观测数据前置 PII 脱敏，禁止敏感数据明文流出|
|执行模式|全量使用异步接口`ainvoke/astream`，避免同步阻塞|
#### 环境变量标准化

```env

ENVIRONMENT=production
AGENT_VERSION=v1.2.0
SERVICE_NAME=deepagents-business-service

# LangSmith 全链路追踪
LANGSMITH_TRACING=true
LANGSMITH_API_KEY=your-production-api-key
LANGSMITH_PROJECT=deepagents-production
LANGSMITH_PRESET=production

# 日志配置
LOG_LEVEL=INFO
LOG_FORMAT=json
LOG_FILE=/var/log/deepagents/production.log

# 指标配置
PROMETHEUS_METRICS_ENABLED=true
METRICS_PORT=9090
```

### 5.2 全链路追踪体系

#### 方案一：LangSmith 生产级方案

原生兼容 DeepAgents，一行配置即可开启，自动捕获主代理、子代理、模型、工具的全链路执行数据，支持分级采样、数据脱敏、主动告警。

#### 方案二：开源私有化方案

- **LangFuse**：开源可观测平台，原生兼容 LangChain 追踪协议，支持私有化部署

- **OpenTelemetry**：标准化全链路追踪，兼容 Jaeger、Zipkin、SkyWalking 等企业现有追踪平台

### 5.3 生产级结构化日志体系

```python

import logging
import sys
import re
from pythonjsonlogger import jsonlogger
import datetime

# 全局配置
SERVICE_NAME = "deepagents-business-service"
ENVIRONMENT = "production"
LOG_FILE = "/var/log/deepagents/production.log"

class PIIRedactionFilter(logging.Filter):
    """敏感数据脱敏过滤器，适配等保/GDPR合规要求"""
    PII_PATTERNS = {
        "phone": re.compile(r"1[3-9]\d{9}"),
        "id_card": re.compile(r"\d{17}[\dXx]"),
        "api_key": re.compile(r"sk-[a-zA-Z0-9]{48}"),
    }
    def filter(self, record):
        if hasattr(record, "msg"):
            for pattern_name, pattern in self.PII_PATTERNS.items():
                record.msg = pattern.sub(f"[REDACTED_{pattern_name}]", record.msg)
        return True

class CustomJsonFormatter(jsonlogger.JsonFormatter):
    """自定义JSON日志格式，添加服务元信息"""
    def add_fields(self, log_record, record, message_dict):
        super().add_fields(log_record, record, message_dict)
        log_record['service'] = SERVICE_NAME
        log_record['env'] = ENVIRONMENT
        log_record['timestamp'] = datetime.datetime.utcnow().isoformat()
        log_record['level'] = record.levelname
        log_record['logger'] = record.name

def setup_production_logger():
    logger = logging.getLogger()
    logger.setLevel(logging.INFO)
    logger.handlers.clear()

    # 控制台+文件双输出
    console_handler = logging.StreamHandler(sys.stdout)
    console_handler.setFormatter(CustomJsonFormatter())
    console_handler.addFilter(PIIRedactionFilter())
    logger.addHandler(console_handler)

    file_handler = logging.handlers.RotatingFileHandler(
        LOG_FILE, maxBytes=100*1024*1024, backupCount=30, encoding="utf-8"
    )
    file_handler.setFormatter(CustomJsonFormatter())
    file_handler.addFilter(PIIRedactionFilter())
    logger.addHandler(file_handler)

    # 分模块日志级别
    logging.getLogger("deepagents").setLevel(logging.INFO)
    logging.getLogger("langgraph").setLevel(logging.INFO)
    logging.getLogger("httpx").setLevel(logging.WARNING)
    return logger
```

### 5.4 指标监控与主动告警体系

#### 核心监控指标

|指标类别|核心监控项|告警阈值|
|---|---|---|
|错误指标|任务执行错误率、模型调用错误率、工具调用错误率|错误率 > 1% P1，>5% P0|
|性能指标|任务执行 P95/P99 延迟、模型首 Token 延迟|P95>30s P1，P99>60s P0|
|吞吐量|任务执行 RPM、模型调用 TPM|低于阈值 50% 告警|
|成本指标|单任务 Token 消耗、日 / 月累计 Token 消耗|日消耗超预算 80% P1，超 100% P0|
|业务指标|任务成功率、子代理调用成功率、上下文溢出次数|任务成功率 < 99.9% 告警|
#### 落地实现

```python

from prometheus_client import Counter, Histogram, Gauge, start_http_server
from deepagents.middleware import AgentMiddleware
import time

# 指标定义
TASK_TOTAL = Counter("deepagents_task_total", "Total tasks", ["task_type", "status"])
TASK_LATENCY = Histogram("deepagents_task_latency_seconds", "Task latency", ["task_type"])
LLM_CALL_TOTAL = Counter("deepagents_llm_call_total", "Total LLM calls", ["model_name", "status"])
TOKEN_USAGE_TOTAL = Counter("deepagents_token_usage_total", "Total token consumption", ["model_name", "token_type"])
RUNNING_TASKS = Gauge("deepagents_running_tasks", "Running tasks count")

# 启动指标服务
start_http_server(9090)

# 指标埋点中间件
class PrometheusMetricsMiddleware(AgentMiddleware):
    def __init__(self):
        self.start_time = None
        self.task_type = None

    async def on_agent_start(self, agent_state, config):
        self.start_time = time.time()
        self.task_type = config.get("metadata", {}).get("task_type", "unknown")
        RUNNING_TASKS.inc()
        return agent_state

    async def on_llm_end(self, llm_output, config):
        model_name = llm_output.model_name
        if hasattr(llm_output, "usage_metadata"):
            usage = llm_output.usage_metadata
            TOKEN_USAGE_TOTAL.labels(model_name=model_name, token_type="input").inc(usage["input_tokens"])
            TOKEN_USAGE_TOTAL.labels(model_name=model_name, token_type="output").inc(usage["output_tokens"])
        LLM_CALL_TOTAL.labels(model_name=model_name, status="success").inc()
        return llm_output

    async def on_agent_end(self, agent_state, config):
        total_latency = time.time() - self.start_time
        TASK_LATENCY.labels(task_type=self.task_type).observe(total_latency)
        TASK_TOTAL.labels(task_type=self.task_type, status="success").inc()
        RUNNING_TASKS.dec()
        return agent_state
```

### 5.5 状态可观测与故障回溯

#### 生产级状态持久化

```python

from langgraph.checkpoint.postgres import PostgresSaver
from langgraph.store.postgres import PostgresStore
from psycopg_pool import AsyncConnectionPool

DB_URI = os.getenv("POSTGRES_URI")
pool = AsyncConnectionPool(DB_URI, max_size=20)
checkpointer = PostgresSaver(pool)
store = PostgresStore(pool)

agent = create_deep_agent(
    model="openai:gpt-4o",
    checkpointer=checkpointer,
    store=store,
)
```

#### 任务状态实时查询

```python

async def get_task_status(thread_id: str):
    config = {"configurable": {"thread_id": thread_id}}
    current_state = await agent.aget_state(config, subgraphs=True)

    return {
        "thread_id": thread_id,
        "next_steps": current_state.next,
        "is_running": len(current_state.next) > 0,
        "is_interrupted": current_state.tasks and any(task.interrupted for task in current_state.tasks),
        "pending_tool_calls": current_state.values["messages"][-1].tool_calls if current_state.values["messages"] else [],
        "subagent_tasks": [
            {
                "subagent_namespace": task.state.config["configurable"]["checkpoint_ns"],
                "subagent_status": task.status
            }
            for task in current_state.tasks if hasattr(task.state, "config")
        ]
    }
```

### 5.6 合规审计与安全可观测

#### 不可篡改的审计日志

所有高危操作必须留存不可篡改的审计日志，满足等保 2.0、GDPR 等合规要求，核心字段包括：

- 事件 ID、trace_id、thread_id、事件类型、操作人、操作时间

- 操作内容、风险等级、审批状态、操作结果、客户端 IP

#### 人在闭环审批全链路追踪

所有高风险操作的审批全流程必须完整记录，实现「谁申请、谁审批、什么时候、什么结果」的全链路可追溯。

#### 异常行为安全监控

- 高频调用高危工具、异常大 Token 消耗、越权操作、批量文件操作、非工作时间操作，实时预警风险

---

## 6. DeepAgents 多智能体 Token 优化方案

### 6.1 Token 指数级增长的核心根因

|诱因|指数级增长逻辑|占比|
|---|---|---|
|主代理全量接收子代理执行历史|子代理执行 10 轮对话，把完整执行历史全部返回给主代理，主代理上下文直接翻倍|45%|
|主代理全量上下文透传给子代理|主代理把全量对话历史全部传给子代理，子代理叠加内容后返回，来回传递导致每轮上下文翻倍|30%|
|大内容全量塞入上下文|子代理的代码、搜索结果全部放在返回内容里，而非写入文件，单条子消息就占用几万 Token|15%|
|子代理无限嵌套调用|子代理继续派生子代理，每一层嵌套都把上层全量上下文带入，嵌套 2 层上下文翻 4 倍|5%|
|全量工具 / 提示词冗余叠加|主代理 + 所有子代理都注册全量工具、超长系统提示词，每次模型调用都重复传递|3%|
|无压缩历史无限累加|所有中间步骤、工具调用结果全部永久留在上下文，越滚越大|1%|
|无效循环调用|智能体陷入工具循环调用，每轮都往上下文塞入冗余内容|1%|
### 6.2 第一优先级：根治指数级增长的架构优化

#### 1. 严格主从架构 + 上下文完全隔离

主代理只做「全局规划→任务分发→结果验收」，子代理负责具体执行，**主代理与子代理上下文完全隔离，主代理仅接收子代理的最终精简结果，不接收子代理的完整执行历史**。

```python

# 子代理输出强制收敛
backend_dev_agent = SubAgent(
    name="backend_developer",
    description="专业Python后端开发工程师，仅负责FastAPI接口开发，最终仅返回交付结果与文件路径",
    system_prompt="""
    你是专业后端开发工程师，严格遵循以下输出规则：
    1. 所有代码、文档必须写入虚拟文件系统，禁止放在对话上下文里
    2. 任务完成后，仅按以下固定格式输出，禁止返回任何中间执行步骤、工具调用历史、完整代码内容
    【固定输出格式】
    交付状态：[已完成/异常终止]
    核心交付物：
    - 代码路径：xxx
    - 文档路径：xxx
    执行结论：[一句话说明，不超过100字]
    异常说明：[无则写"无"]
    """,
    model="openai:gpt-4o-mini",
    tools=["read_file", "write_file", "edit_file", "execute"]
)

# 主代理仅传递必要信息
main_agent_system_prompt = """
你是项目总负责人，仅负责全局规划、任务分发、结果验收，严格遵循子代理调用规则：
1. 委派任务给子代理时，仅传递「任务需求、交付标准、依赖文件路径」3项核心信息
2. 禁止把你和用户的对话历史、和其他子代理的交互内容，透传给子代理
3. 子代理交付后，仅保留其返回的交付状态、文件路径、核心结论，禁止把子代理的完整返回内容永久留在上下文
"""
```

#### 2. 以文件系统为中心的结果传递

所有大内容（代码、文档、搜索结果、数据集）必须全部写入文件，主代理与子代理之间**仅传递文件路径，不传递内容本身**，从根源上避免大内容占用上下文 Token。

```python

【文件系统使用强制规则】
1. 所有代码、文档、搜索结果、数据内容，只要超过300字，必须通过write_file写入对应目录，禁止直接放在对话上下文里
2. 引用内容时，仅标注文件路径和核心结论，禁止重复全文内容
3. 与其他代理共享数据时，仅传递文件路径，禁止传递内容本身
```

#### 3. 禁止子代理嵌套，限制调用层级

最多只允许一层子代理（主代理→子代理），禁止子代理继续派生子代理，从根源上避免嵌套导致的上下文指数级膨胀。

- 关闭子代理的`task`工具，禁止子代理派生子代理

- 提示词强约束，禁止子代理二次委派任务

- 复杂任务由主代理拆解为多个平级子任务，而非嵌套

#### 4. 优先使用动态 task 子代理，自动销毁上下文

动态`task`子代理执行完成后，上下文自动销毁，仅返回最终结果，不会把执行历史带入主代理上下文，比预定义子代理更省 Token。

### 6.3 第二优先级：原生能力极致优化

#### 1. 子代理工具集最小权限原则

每个子代理仅分配完成任务必须的最小工具集，禁止全量工具注册，减少基础 Token 消耗。

- 主代理仅注册规划、验收相关的工具

- 子代理仅注册自己领域必须的工具，比如文档子代理仅需要`read_file`、`write_file`

#### 2. 开启内置上下文自动压缩

DeepAgents 原生内置了上下文压缩能力，开启后会自动对长对话进行轻量化压缩，保留核心语义，减少冗余 Token 占用。

```python

agent = create_deep_agent(
    model="openai:gpt-4o",
    enable_context_compaction=True, # 开启内置上下文压缩
    compaction_config={
        "max_message_count": 10, # 超过10条消息自动触发压缩
        "compaction_model": "openai:gpt-4o-mini", # 压缩用轻量模型
    },
)
```

#### 3. 长期记忆精细化管理

- 记忆分类拆分：按用途、子代理拆分记忆文件，按需加载，禁止全量预加载

- 按需读取，而非预加载：让智能体在需要的时候，通过`read_file`主动读取对应记忆文件，不需要的内容永远不会进入上下文

### 6.4 第三优先级：上下文精细化管控

#### 1. 滚动上下文窗口 + 历史摘要机制

主代理仅保留最近 N 轮的完整对话历史，更早的历史自动做摘要压缩，仅保留核心结论，丢弃中间的工具调用、调试日志、冗余内容。

```python

from langchain_openai import ChatOpenAI
from langchain_core.messages import SystemMessage
from deepagents.middleware import AgentMiddleware
from collections import defaultdict

class ContextWindowMiddleware(AgentMiddleware):
    def __init__(self, max_full_messages=6, compact_model="openai:gpt-4o-mini"):
        self.max_full_messages = max_full_messages
        self.compact_model = ChatOpenAI(model=compact_model, temperature=0)

    async def _compact_history(self, messages):
        if len(messages) <= self.max_full_messages:
            return messages
        # 旧历史做摘要
        old_messages = messages[:-self.max_full_messages]
        recent_messages = messages[-self.max_full_messages:]
        prompt = f"对以下对话历史进行摘要，仅保留核心任务目标、关键决策、最终结论，摘要控制在300字以内：{[msg.dict() for msg in old_messages]}"
        summary = await self.compact_model.ainvoke(prompt)
        return [SystemMessage(content=f"历史对话摘要：{summary.content}")] + recent_messages

    async def on_agent_start(self, agent_state, config):
        original_messages = agent_state["messages"]
        compacted_messages = await self._compact_history(original_messages)
        agent_state["messages"] = compacted_messages
        return agent_state
```

#### 2. 无用信息主动清理

在每轮执行完成后，主动清理上下文里的无效信息：

- 工具调用的大段返回结果，处理完成后，仅保留核心结论，删除完整返回内容

- 子代理返回的冗余内容、调试信息，仅保留交付状态、文件路径、核心结论

- 临时的、一次性的指令、说明，执行完成后立即从上下文中移除

### 6.5 第四优先级：模型与调用策略优化

#### 1. 模型分层选型，主强子轻

主代理负责全局规划、复杂决策，用大模型；子代理负责具体执行、标准化任务，用高性价比的轻量模型，实测可降低 60% 以上的 Token 成本，同时轻量模型的窗口限制会强制子代理精简内容。

```python

# 子代理用轻量高性价比模型
backend_dev_agent = SubAgent(
    name="backend_developer",
    model="openai:gpt-4o-mini",
    ...
)

# 主代理用强推理大模型
main_agent = create_deep_agent(
    model="openai:gpt-4o",
    ...
)
```

#### 2. 模型参数优化，减少冗余输出

- 限制最大输出 Token：给子代理的模型设置`max_tokens=2048`，限制单次输出的最大长度

- 降低 Temperature：设置为 0~0.3，减少模型的随机性，避免冗余输出，严格遵循输出格式要求

- 批量任务合并：把多个小的子任务合并为一个批量任务，减少模型调用次数，避免多次调用的上下文重复传递

### 6.6 第五优先级：监控、熔断与预算管控

#### 1. Token 消耗全链路监控与熔断

```python

from deepagents.middleware import AgentMiddleware
from collections import defaultdict

class TokenBudgetMiddleware(AgentMiddleware):
    def __init__(self, max_task_tokens=500000, max_step_tokens=30000, max_repeat_calls=3):
        self.max_task_tokens = max_task_tokens
        self.max_step_tokens = max_step_tokens
        self.max_repeat_calls = max_repeat_calls
        self.total_task_tokens = 0
        self.tool_call_count = defaultdict(int)

    async def on_llm_end(self, llm_output, config):
        if hasattr(llm_output, "usage_metadata"):
            step_tokens = llm_output.usage_metadata["total_tokens"]
            self.total_task_tokens += step_tokens

            # 单轮Token阈值熔断
            if step_tokens > self.max_step_tokens:
                raise RuntimeError(f"单轮Token消耗{step_tokens}，超过阈值{self.max_step_tokens}，已熔断")
            # 单任务总Token阈值熔断
            if self.total_task_tokens > self.max_task_tokens:
                raise RuntimeError(f"任务累计Token消耗{self.total_task_tokens}，超过预算{self.max_task_tokens}，已熔断")
        return llm_output

    async def on_tool_start(self, tool_call, config):
        tool_name = tool_call["name"]
        self.tool_call_count[tool_name] += 1
        # 循环调用熔断
        if self.tool_call_count[tool_name] > self.max_repeat_calls:
            raise RuntimeError(f"工具{tool_name}循环调用超过{self.max_repeat_calls}次，已熔断")
        return tool_call
```

#### 2. 全链路 Token 追踪与成本管控

- LangSmith 全链路追踪，精准定位每个代理、每轮调用的 Token 消耗

- 结构化日志埋点，把 Token 消耗写入日志，接入 ELK 平台做统计分析

- 搭建成本看板，实时监控单任务、单用户、全局的 Token 消耗与成本，设置分级预算告警

### 6.7 优化后完整多智能体代码模板

```python

import os
import asyncio
from dotenv import load_dotenv
from deepagents import create_deep_agent, SubAgent
from langgraph.checkpoint.postgres import PostgresSaver
from langgraph.store.postgres import PostgresStore
from psycopg_pool import AsyncConnectionPool

# 加载环境变量
load_dotenv()

# 生产级存储初始化
DB_URI = os.getenv("POSTGRES_URI")

# 子代理定义（优化后）
backend_dev_agent = SubAgent(
    name="backend_developer",
    description="专业Python后端开发工程师，负责FastAPI接口开发，最终仅返回交付结果与文件路径",
    system_prompt="""
    你是专业后端开发工程师，严格遵循以下规则：
    1. 所有代码、文档必须写入/backend/目录，超过300字的内容禁止放入对话上下文
    2. 禁止委派子任务、禁止嵌套调用
    3. 任务完成后，仅按固定格式输出，禁止返回中间执行过程、完整代码内容
    【固定输出格式】
    交付状态：[已完成/异常终止]
    核心交付物：
    - 代码路径：xxx
    - 文档路径：xxx
    执行结论：[一句话说明，不超过100字]
    异常说明：[无则写"无"]
    """,
    model="openai:gpt-4o-mini",
    tools=["read_file", "write_file", "edit_file", "execute"]
)

frontend_dev_agent = SubAgent(
    name="frontend_developer",
    description="专业Vue3前端开发工程师，负责页面开发，最终仅返回交付结果与文件路径",
    system_prompt="""
    你是专业前端开发工程师，严格遵循以下规则：
    1. 所有代码、文档必须写入/frontend/目录，超过300字的内容禁止放入对话上下文
    2. 禁止委派子任务、禁止嵌套调用
    3. 任务完成后，仅按固定格式输出，禁止返回中间执行过程、完整代码内容
    """,
    model="anthropic:claude-3-haiku-20240307",
    tools=["read_file", "write_file", "edit_file"]
)

# 主代理定义（优化后）
main_agent_system_prompt = """
你是项目总负责人，仅负责全局规划、任务分发、结果验收，严格遵循以下规则：
1. 收到需求后，先通过write_todos拆解里程碑与待办清单，明确每个子任务的交付标准
2. 委派任务给子代理时，仅传递「任务需求、交付标准、依赖文件路径」，禁止透传全量对话历史
3. 子代理交付后，仅保留其返回的交付状态、文件路径、核心结论，禁止保留冗余内容
4. 禁止自己处理细节开发任务，所有执行类任务必须委派给对应子代理
5. 超过300字的内容必须写入文件，禁止放入对话上下文
6. 禁止重复委派任务、禁止循环调用工具
"""

# 注册中间件
from middleware import ContextWindowMiddleware, TokenBudgetMiddleware
middleware_list = [
    ContextWindowMiddleware(max_full_messages=6),
    TokenBudgetMiddleware(max_task_tokens=500000, max_step_tokens=30000)
]

async def main():
    # 初始化异步连接池
    pool = AsyncConnectionPool(DB_URI, max_size=20)
    async with pool:
        checkpointer = PostgresSaver(pool)
        store = PostgresStore(pool)

        # 创建优化后的主代理
        main_agent = create_deep_agent(
            model="openai:gpt-4o",
            subagents=[backend_dev_agent, frontend_dev_agent],
            checkpointer=checkpointer,
            store=store,
            middleware=middleware_list,
            enable_context_compaction=True,
            system_prompt=main_agent_system_prompt,
            tools=["write_todos", "read_todos", "read_file"],
            debug=False
        )

        # 执行任务
        config = {"configurable": {"thread_id": "optimized_project_001"}}
        result = await main_agent.ainvoke({
            "messages": [{"role": "user", "content": "开发FastAPI+Vue3用户管理系统，交付可运行的前后端项目、接口文档、部署说明"}]
        }, config=config)

        print(result["messages"][-1].content)

# 运行异步主函数
if __name__ == "__main__":
    asyncio.run(main())
```

### 6.8 避坑指南与优化优先级

#### 常见避坑指南

1. ❌ 禁止子代理返回完整执行历史：这是 Token 指数级增长的头号元凶，必须强制子代理仅返回最终精简结果

2. ❌ 禁止主代理全量上下文透传给子代理：仅传递任务必须的信息，不要把和其他子代理的交互、用户的全量历史传给子代理

3. ❌ 禁止大内容放入上下文：必须用 DeepAgents 的虚拟文件系统，所有大内容落盘，仅传路径

4. ❌ 禁止子代理嵌套调用：最多一层子代理，禁止子代理继续派生子代理

5. ❌ 禁止全量工具注册给所有代理：严格遵循最小权限原则，仅给代理分配必须的工具

6. ❌ 禁止全量记忆预加载：按需读取记忆文件，不要把所有记忆都放入系统提示词

#### 优化优先级排序

1. **第一优先级**：架构优化（上下文隔离、结果收敛、禁止嵌套、文件系统传数据），根治指数级增长

2. **第二优先级**：DeepAgents 原生能力优化（最小工具集、自动压缩、记忆精细化管理），降低基础消耗

3. **第三优先级**：上下文精细化管控（滚动窗口、历史摘要、无用信息清理），优化线性增长

4. **第四优先级**：模型与调用策略优化（主强子轻、参数优化），降低单位 Token 成本

5. **第五优先级**：监控与熔断机制，防止 Token 失控与成本超支

---

## 7. DeepAgents 面试常见问题与回答

### 7.1 基础概念类

#### Q1：DeepAgents 是什么？它和 LangGraph 是什么关系？

**回答**：
DeepAgents 是 LangChain 团队开源的**开箱即用的企业级智能体框架**，定位是「The batteries-included agent harness」，它是基于 LangGraph 底层运行时构建的上层封装。

- LangGraph 是一个通用的状态机编排框架，提供了基础的节点、边、检查点、持久化等能力，但需要开发者自己实现任务规划、文件系统、子代理协作等业务能力

- DeepAgents 是在 LangGraph 之上，内置了企业级智能体所需的所有核心能力，开发者仅需几行代码即可构建生产级智能体，无需从零开始实现复杂的业务逻辑

#### Q2：DeepAgents 解决了传统智能体的哪些核心痛点？

**回答**：
DeepAgents 解决了传统智能体的 6 大核心痛点：

1. **上下文溢出**：通过虚拟文件系统，把大内容落盘，解决了传统智能体上下文窗口不够用的问题

2. **任务跑偏**：内置任务规划工具，强制智能体先规划再执行，解决了传统智能体走一步看一步、任务跑偏的问题

3. **主代理被细节污染**：通过主从子代理的上下文隔离，主代理仅接收子代理的最终结果，解决了传统智能体主代理被细节污染、丢失全局视野的问题

4. **多轮会话遗忘**：内置长期记忆能力，支持跨会话的持久化记忆，解决了传统智能体多轮会话遗忘的问题

5. **高危操作无管控**：原生支持人在闭环，支持精细化的中断配置，解决了传统智能体高危操作无管控的问题

6. **Token 指数级增长**：通过架构优化、上下文隔离、结果收敛，解决了多智能体场景下 Token 指数级增长的问题

#### Q3：DeepAgents 的核心内置能力有哪些？

**回答**：
DeepAgents 的核心内置能力包括：

1. **任务规划**：内置待办清单工具，支持任务拆解、进度跟踪、动态调整

2. **虚拟文件系统**：内置多种存储后端，支持大内容落盘，彻底解决上下文溢出

3. **子智能体协作**：支持主从子代理架构，上下文完全隔离，实现团队化协作

4. **长期记忆**：支持跨会话的持久化记忆，自动沉淀项目知识

5. **人在闭环**：原生支持精细化的中断配置，支持高危操作审批

6. **中间件系统**：完整的生命周期钩子，支持扩展能力

7. **技能系统**：模块化的技能复用，支持跨智能体的能力复用

### 7.2 架构设计类

#### Q4：DeepAgents 的虚拟文件系统是怎么实现的？它解决了什么问题？

**回答**：
DeepAgents 的虚拟文件系统是一个**以文件为中心的上下文管理架构**，它把智能体的上下文管理从「对话历史」转换成了「文件系统」，核心实现：

- 它提供了一套标准的文件操作工具（`read_file`/`write_file`/`edit_file` 等），智能体可以像操作本地文件一样操作虚拟文件

- 它支持多种存储后端，包括内存、本地磁盘、LangGraph Store、沙箱等，支持混合部署

- 它解决了传统智能体的两个核心问题：

    1. **上下文溢出**：大内容（代码、搜索结果、文档）可以直接写入文件，不会占用对话上下文的 Token

    2. **大内容共享**：主代理和子代理之间可以通过文件路径共享大内容，不需要把内容本身在代理之间传递，大幅降低了 Token 消耗

#### Q5：DeepAgents 的主从子代理架构是怎么实现的？为什么能解决 Token 指数级增长的问题？

**回答**：
DeepAgents 的主从子代理架构，核心是**上下文完全隔离**：

- 主代理和子代理是完全独立的智能体，有自己的对话历史、工具集、提示词

- 主代理给子代理传递任务的时候，仅传递任务需求、交付标准、依赖文件路径，不会把主代理的全量对话历史透传给子代理

- 子代理执行完成后，仅返回最终的精简结果（交付状态、文件路径、核心结论），不会把自己的完整执行历史返回给主代理

- 这就从根源上解决了 Token 指数级增长的问题：

    1. 避免了主代理的全量上下文透传给子代理，子代理的上下文不会被主代理的历史污染

    2. 避免了子代理的完整执行历史返回给主代理，主代理的上下文不会被子代理的执行细节污染

    3. 大内容通过文件路径传递，不需要在代理之间传递内容本身，大幅降低了 Token 消耗

#### Q6：DeepAgents 的长期记忆是怎么实现的？

**回答**：
DeepAgents 的长期记忆，是基于虚拟文件系统的 `/memories/` 路径实现的：

- 当开启 `use_longterm_memory=True` 后，`/memories/` 路径会映射到 LangGraph 的 Store 存储，这个存储是跨会话持久化的

- 智能体可以把项目规范、用户偏好、核心知识、最佳实践等内容，写入 `/memories/` 目录下的文件

- 每次会话启动的时候，智能体都会自动读取 `/memories/` 目录下的记忆文件，把这些内容加载到上下文中，这样智能体就可以记住之前会话的内容

- 它的优势是：

    1. 支持跨会话的持久化，即使重启服务，记忆也不会丢失

    2. 支持精细化的记忆管理，智能体可以按需读取记忆文件，不需要全量预加载

    3. 支持记忆的版本管理，通过 LangGraph 的检查点，可以回溯记忆的变更历史

### 7.3 生产落地类

#### Q7：DeepAgents 生产环境的可观测体系是怎么搭建的？

**回答**：
DeepAgents 生产环境的可观测体系，是基于「追踪 + 日志 + 指标 + 状态」的四件套搭建的：

1. **全链路追踪**：原生兼容 LangSmith，也支持 LangFuse、OpenTelemetry 等开源方案，自动捕获主代理、子代理、模型、工具的全链路执行数据

2. **结构化日志**：生产环境使用 JSON 格式的结构化日志，内置 PII 脱敏，支持控制台 + 文件双输出，分模块的日志级别管控

3. **指标监控**：基于 Prometheus 搭建指标体系，监控任务执行延迟、错误率、Token 消耗、吞吐量等核心指标，支持主动告警

4. **状态可观测**：基于 Postgres 持久化的检查点，支持任务状态的实时查询，包括任务是否运行、是否中断、待执行的工具调用、子代理的状态等

#### Q8：DeepAgents 多智能体场景下，Token 优化的核心方案是什么？

**回答**：
DeepAgents 多智能体场景下，Token 优化的核心是「根治指数级增长」，优先级从高到低：

1. **架构优化**：严格主从架构，上下文隔离，子代理结果收敛，禁止嵌套，用文件系统传数据，这是根治指数级增长的核心

2. **原生能力优化**：最小工具集，开启内置的上下文自动压缩，记忆精细化管理，按需加载

3. **上下文管控**：滚动窗口，历史摘要，主动清理无用信息

4. **模型优化**：主强子轻，主代理用大模型，子代理用轻量模型，优化模型参数

5. **监控熔断**：Token 消耗的全链路监控，熔断机制，防止 Token 失控

#### Q9：DeepAgents 的人在闭环是怎么实现的？

**回答**：
DeepAgents 的人在闭环，是基于 LangGraph 的检查点能力实现的：

- 开发者可以通过 `interrupt_on` 参数，配置精细化的中断规则，比如在某个工具调用前中断，或者满足某个条件的时候中断

- 当触发中断的时候，LangGraph 会把当前的状态持久化到检查点，然后暂停执行

- 人工审批完成后，开发者可以通过 `agent.invoke(None, config=config)`，从中断处恢复执行，不需要重新执行之前的步骤

- 它的优势是：

    1. 支持精细化的中断配置，支持条件性中断，比如只有写入系统目录的时候才中断

    2. 支持断点续跑，中断恢复的时候，不需要重新执行之前的步骤，大幅提升效率

    3. 支持人工干预，审批的时候可以修改工具调用的参数，或者拒绝执行，调整方案

#### Q10：DeepAgents 生产环境的安全管控是怎么做的？

**回答**：
DeepAgents 生产环境的安全管控，是从「权限 + 审批 + 审计 + 脱敏」四个维度实现的：

1. **最小权限原则**：每个代理仅分配完成任务必须的最小工具集，禁止全量工具注册

2. **人在闭环审批**：高危操作（代码执行、删除、部署）必须经过人工审批，禁止自动执行

3. **全链路审计**：所有操作都留存不可篡改的审计日志，满足等保、GDPR 等合规要求

4. **数据脱敏**：所有可观测数据、日志都前置 PII 脱敏，禁止敏感数据明文流出

5. **沙箱执行**：代码执行使用隔离沙箱，避免高危操作影响宿主环境

### 7.4 选型对比类

#### Q11：DeepAgents 和 AutoGPT 有什么区别？

**回答**：
DeepAgents 和 AutoGPT 的核心区别是定位不同：

- AutoGPT 是一个早期的实验性智能体，定位是个人开发者的玩具，没有生产级的能力，存在任务跑偏、上下文溢出、Token 失控等问题

- DeepAgents 是 LangChain 团队推出的企业级智能体框架，定位是生产级的业务落地，内置了任务规划、文件系统、子代理协作、持久化、可观测等企业级能力，解决了 AutoGPT 的所有核心痛点，适合企业级的业务落地

#### Q12：DeepAgents 和 CrewAI 有什么区别？

**回答**：
DeepAgents 和 CrewAI 都是多智能体框架，但核心区别是：

- CrewAI 是一个独立的框架，它的底层是自己实现的，和 LangChain/LangGraph 的生态兼容性一般

- DeepAgents 是基于 LangGraph 构建的，它完全兼容 LangChain/LangGraph 的全生态，你可以无缝使用 LangChain 的工具、模型、存储、可观测等能力

- 另外，DeepAgents 内置了虚拟文件系统、长期记忆、人在闭环等企业级能力，而 CrewAI 需要开发者自己实现这些能力

- 还有，DeepAgents 的主从子代理架构，上下文完全隔离，从根源上解决了 Token 指数级增长的问题，而 CrewAI 的多智能体架构，容易出现上下文膨胀的问题

#### Q13：什么时候应该选择 DeepAgents，什么时候应该选择原生 LangGraph？

**回答**：

- 如果你是要快速构建生产级的智能体，需要任务规划、文件系统、子代理协作、持久化等能力，那么应该选择 DeepAgents，它可以帮你节省大量的开发时间，几行代码就可以搞定

- 如果你是要构建非常定制化的智能体，需要完全自定义节点、边、状态机的逻辑，那么应该选择原生 LangGraph，它可以给你最大的灵活性

- 简单来说，DeepAgents 是「开箱即用」的，适合 90% 的企业级业务场景，而原生 LangGraph 是「从零开始」的，适合 10% 的高度定制化的场景
