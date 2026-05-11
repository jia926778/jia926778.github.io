---
title: FastMCP学习手册
date: 2026-05-03 19:30:00
tags: [Agent, 技术分享, FastMCP]
categories: Agent
toc: true
---
# FastMCP技术学习手册

## 文档核心说明

本文档适用于想要学习 FastMCP 框架的 Python 开发者、AI 应用开发者，覆盖 FastMCP 基础概念、环境搭建、核心组件开发、客户端集成、生产级服务模板、自定义中间件开发、部署方案、故障排查等全链路知识，可直接作为 FastMCP 技术学习的完整手册使用。

## 一、基础概述

### 1.1 核心定位

MCP（模型上下文协议）是 Anthropic 推出的开放式协议，为 LLM 提供标准化的外部工具 / 数据接入方案，相当于 LLM 的「通用工具箱协议」；而 **FastMCP** 则是 MCP 生态的 Python 最佳实践框架，类似 Web 开发中的 FastAPI，将底层协议细节完全封装，开发者只需关注业务逻辑实现。

### 1.2 核心优势

- 极简开发：通过装饰器 `@mcp.tool`/`@mcp.resource`/`@mcp.prompt` 3 行代码即可暴露能力

- 全协议兼容：支持 STDIO、SSE、Streamable HTTP 三种主流传输协议

- 生产级能力：内置中间件、认证、日志、依赖管理、异步支持

- 生态集成：一键对接 Claude Desktop、Cursor 等主流 LLM 客户端，支持云端部署

- 自动元数据生成：函数文档字符串、类型注解自动转化为 LLM 可识别的工具描述

### 1.3 版本说明

- 稳定版推荐：生产环境固定版本号（如 `fastmcp==2.11.3`），避免自动升级带来的破坏性变更

- 版本迁移：从官方 MCP SDK 的 FastMCP 1.0 迁移，只需修改导入语句 `from mcp.server.fastmcp import FastMCP` → `from fastmcp import FastMCP`

- 核心版本特性：

    - v2.3.2\+ 完善 HTTP 客户端支持

    - v2.6.0\+ 新增 JWT 认证

    - v2.9.0\+ 支持中间件体系

    - v2.10.3\+ 新增 Claude Desktop 一键安装命令

## 二、环境安装

### 2.1 前置要求

- Python 3.11\+ 版本

- 推荐使用 `uv` 作为包管理器，也可使用原生 `pip`

### 2.2 安装步骤

#### 方式 1：pip 安装（最简）

```bash
pip install fastmcp
```

#### 方式 2：uv 安装（推荐，性能更优）

```bash
# 全局安装
uv pip install fastmcp

# 项目内添加依赖
uv add fastmcp
```

### 2.3 验证安装

执行以下命令，输出版本信息即安装成功：

```bash
fastmcp version
```

输出示例：

```Plain Text
FastMCP version: 2.11.3
MCP version: 1.12.4
Python version: 3.12.2
Platform: macOS-15.3.1-arm64-arm64bit
```

## 三、核心概念详解

### 3.1 核心根实例：FastMCP 服务器实例

这是整个框架的根入口与核心调度中枢，是 MCP 服务器的生命周期管理者、能力注册中心和协议通信底座，类比 Web 开发中 FastAPI 的 App 实例，是所有能力的载体。

#### 核心作用

- 完全封装 MCP 协议的底层细节，开发者无需关注协议序列化、消息路由、连接管理等复杂逻辑

- 统一管理所有 Tool/Resource/Prompt 组件的注册、发现与调用路由

- 处理客户端连接的生命周期、协议协商、认证与错误处理

- 适配不同传输协议，实现一套代码多场景部署（本地桌面客户端 / 云端 / 内网）

#### 关键特性

1. **唯一标识**：必须指定 `name` 属性，作为 LLM 客户端识别服务器的唯一名称

2. **依赖管理**：通过 `dependencies` 参数声明运行所需的 Python 包，部署时自动安装

3. **全局指引**：通过 `instructions` 参数给 LLM 提供全局使用说明，大幅提升工具调用准确率

4. **生命周期钩子**：支持服务启动前的资源初始化、关闭前的优雅回收逻辑

### 3.2 三大核心能力支柱

这是 MCP 协议的原生核心规范，也是 FastMCP 暴露给 LLM 的核心能力，官方定义为 FastMCP 的「三大支柱」，覆盖了 LLM 与外部系统交互的全场景需求。

#### 3.2.1 Tool（工具）

**官方定义**：MCP 协议中面向动作的可执行函数，是 LLM 可以请求服务器执行的操作单元，通过 `@mcp.tool` 装饰器封装。

- **核心定位**：LLM 的「可执行动作接口」，类比 REST API 的 POST/PUT 接口，允许 LLM 触发外部操作、修改系统状态、产生副作用，是 LLM 从「生成内容」到「执行动作」的核心载体。

- **核心作用**：突破 LLM 的知识边界和能力限制，让大模型可以操作数据库、调用第三方 API、读写文件、控制硬件、执行代码等复杂操作。

- **关键规则**：

    1. 函数的文档字符串（docstring）会自动转化为 LLM 可识别的工具描述，直接决定 LLM 是否调用该工具

    2. 必须为所有参数和返回值添加 Python 类型注解，FastMCP 会自动生成 JSON Schema 供 LLM 校验传参

    3. 原生支持同步 / 异步函数，异步函数更适合 API 调用、数据库查询等 IO 密集型场景

    4. 可通过 `ToolError` 抛出结构化错误，LLM 可识别错误信息并智能重试 / 纠错

- **典型场景**：API 调用、数据库读写、文件操作、数值计算、设备控制、邮件发送等带副作用的操作

#### 3.2.2 Resource（资源）

**官方定义**：MCP 协议中面向数据的只读数据源，是 LLM 可以读取的静态 / 动态数据单元，通过 `@mcp.resource(uri)` 装饰器封装。

- **核心定位**：LLM 的「上下文数据接口」，类比 REST API 的 GET 接口，无副作用、幂等性，仅用于给 LLM 注入额外上下文数据，突破 LLM 的上下文窗口限制和知识截止日期限制。

- **核心作用**：为 LLM 提供实时、专属、海量的上下文信息，无需用户在对话中手动粘贴数据，让 LLM 按需读取外部数据。

- **关键规则**：

    1. 必须指定唯一的 URI 标识，支持静态 URI 和带路径参数的动态 URI 模板

    2. 严格遵循只读原则，禁止在资源函数中执行修改状态、产生副作用的操作

    3. 动态 URI 模板的路径参数会自动映射为函数入参，支持类型校验

    4. 支持字符串、字典、列表、二进制等多种返回类型，自动适配 MCP 协议格式

- **典型场景**：用户资料注入、文档知识库读取、实时数据同步、配置文件加载、日志内容拉取、数据库只读查询等

#### 3.2.3 Prompt（提示模板）

**官方定义**：MCP 协议中面向交互的可复用对话模板，是标准化的 LLM 对话指令单元，通过 `@mcp.prompt` 装饰器封装。

- **核心定位**：LLM 的「标准化对话模板」，类比前端开发的模板引擎，用于固化高频场景的提示词逻辑，保证对话指令的一致性和可复用性。

- **核心作用**：统一管理高频业务场景的提示词，避免重复编写，支持动态参数注入，可预设多轮对话结构，规范 LLM 的输出格式和交互逻辑。

- **关键规则**：

    1. 函数名即为模板的唯一标识，函数入参即为模板的动态变量

    2. 可返回纯字符串（自动转为 user 角色消息），也可返回多角色 Message 列表，支持 system/user/assistant 多轮对话预设

    3. 客户端可获取模板并传入变量，自动渲染为完整的对话消息

- **典型场景**：代码评审、错误调试、文书生成、数据分析报告、固定格式内容创作、标准化客服对话等

### 3.3 通信传输核心：Transport（传输协议）

这是 FastMCP 服务器与 LLM 客户端通信的核心规范，定义了数据传输的方式、协议和生命周期管理，完全兼容 MCP 协议标准，决定了服务的部署方式和适用场景。

FastMCP 原生支持三大核心传输模式，一套代码可无缝切换：

|传输模式|核心定位|通信原理|适用场景|核心特点|
|---|---|---|---|---|
|STDIO（标准输入输出）|本地部署首选|通过进程 stdin/stdout 全双工通信，无网络依赖|Claude Desktop、Cursor 等本地桌面客户端集成、本地调试|配置极简、零网络风险、兼容性最强、无端口占用|
|HTTP（Streamable HTTP）|生产级远程部署首选|基于 HTTP/HTTPS 全双工通信，兼容 ASGI 标准|云端部署、多客户端共享、Web 端 LLM 集成、企业级内网部署|支持公网访问、可对接负载均衡、支持认证鉴权、可与 FastAPI 无缝集成|
|SSE（Server-Sent Events）|实时场景专用|基于 SSE 服务端推送 \+ HTTP POST 客户端上行，低延迟全双工通信|实时数据推送、长连接会话、低延迟工具调用、浏览器端实时交互|低延迟、支持服务端主动推送、兼容浏览器原生 API、穿透性强|

### 3.4 生产级扩展核心概念

这些是 FastMCP 区别于原生 MCP SDK 的核心扩展能力，是企业级生产环境落地的关键支撑。

#### 3.4.1 Middleware（中间件）

FastMCP 提供的请求全链路拦截扩展机制，类似 FastAPI 的中间件，可在客户端请求的处理流程中插入自定义逻辑。核心作用是实现全局的日志记录、认证鉴权、请求过滤、错误捕获、性能监控、链路追踪等能力，无需修改业务代码，实现横切关注点的统一管理。

支持全生命周期钩子：`on_request`（全局请求拦截）、`on_call_tool`（工具调用拦截）、`on_read_resource`（资源读取拦截）等。

#### 3.4.2 FastMCP Client（客户端）

FastMCP 提供的 Pythonic 程序化客户端，封装了 MCP 协议的客户端通信细节，可连接和调用任何兼容 MCP 协议的服务器。核心用于 MCP 服务器的开发测试、构建确定性 AI 应用、搭建多服务器协同的 Agent 系统，实现对 MCP 服务的类型安全、可控的程序化调用。

#### 3.4.3 生命周期钩子（Lifespan Hooks）

FastMCP 服务器提供的生命周期管理能力，允许开发者在服务启动前、关闭前执行自定义逻辑。核心用于实现服务启动时的资源初始化（如数据库连接、API 客户端初始化）、服务关闭时的资源优雅回收（如连接关闭、数据落盘），保证服务的稳定性。

#### 3.4.4 结构化错误体系

FastMCP 对齐 MCP 协议规范封装的专用异常类体系，包括 `ToolError`、`ResourceNotFoundError`、`InvalidRequestError` 等。核心作用是让 LLM 客户端可以识别结构化的错误信息，进行智能重试、纠错或给用户明确的提示，避免未捕获的异常导致服务崩溃，提升交互容错性。

#### 3.4.5 Provider 与 Converter（高级抽象）

FastMCP 企业级场景的核心扩展抽象：

- **Provider**：决定组件的来源，支持从装饰函数、磁盘文件、OpenAPI 规范、远程服务器等多种来源加载能力

- **Converter**：控制客户端所见的内容，支持命名空间、权限过滤、版本控制、内容转换等，实现同一服务器向不同用户呈现不同能力。

## 四、快速入门：5 分钟搭建第一个 MCP 服务器

### 4.1 编写最简服务器

新建 `my_server.py` 文件，写入以下代码：

```python
from fastmcp import FastMCP

# 1. 创建服务器实例，名称为"My First MCP Server"
mcp = FastMCP("My First MCP Server")

# 2. 注册一个工具：让LLM可以调用的打招呼函数
@mcp.tool
def greet(name: str) -> str:
    """向指定名字的人发送问候语"""
    return f"Hello, {name}! 欢迎使用FastMCP"

# 3. 启动服务器
if __name__ == "__main__":
    # 默认使用STDIO传输，适合本地调试和Claude Desktop集成
    mcp.run()
```

### 4.2 启动服务

执行以下命令启动服务器：

```bash
python my_server.py
```

### 4.3 本地测试（HTTP 模式）

修改启动代码，使用 HTTP 传输，方便通过客户端测试：

```python
if __name__ == "__main__":
    # HTTP模式，端口8000，支持远程访问
    mcp.run(transport="http", port=8000)
```

启动后，新建 `test_client.py` 编写测试客户端代码：

```python
import asyncio
from fastmcp import Client

async def test_server():
    # 连接到本地HTTP服务器
    client = Client("http://localhost:8000/mcp")

    async with client:
        # 1. 列出所有可用工具
        tools = await client.list_tools()
        print("可用工具：")
        for tool in tools:
            print(f"- {tool.name}: {tool.description}")

        # 2. 调用greet工具
        result = await client.call_tool("greet", {"name": "FastMCP用户"})
        print(f"\n工具调用结果：{result}")

if __name__ == "__main__":
    asyncio.run(test_server())
```

执行测试代码，即可看到服务器返回的结果，完成最简服务的开发与测试。

## 五、三大核心组件详解

### 5.1 Tool（工具）：LLM 可执行的函数

工具是 FastMCP 最常用的组件，将普通 Python 函数转化为 LLM 可调用的能力，核心规则如下：

#### 5.1.1 基础用法

```python
from typing import Annotated
from pydantic import Field
from fastmcp import FastMCP

mcp = FastMCP("数据处理工具集")

# 基础工具：同步函数
@mcp.tool
def calculate_sum(numbers: list[float]) -> float:
    """计算一组数字的总和"""
    return sum(numbers)

# 进阶工具：带参数描述、异步函数
@mcp.tool
async def get_weather(
    city: Annotated[str, Field(description="要查询的城市名称，如深圳、北京")],
    province: Annotated[str, Field(description="省份名称，可选，用于区分重名城市")] = ""
) -> str:
    """获取指定城市的实时天气信息"""
    # 这里可替换为真实的天气API调用
    await asyncio.sleep(0.1) # 模拟IO延迟
    return f"{province}{city} 当前天气：晴，25℃，微风"

# 工具错误处理：主动抛出ToolError，LLM可识别错误信息
from fastmcp.exceptions import ToolError

@mcp.tool
def divide(a: float, b: float) -> float:
    """两个数字的除法运算"""
    if b == 0:
        raise ToolError("除数不能为0")
    return a / b
```

#### 5.1.2 核心注意事项

- 禁止使用 `*args`/`**kwargs` 作为参数，FastMCP 无法生成明确的参数 Schema

- 复杂参数建议使用 Pydantic 模型定义，提升参数校验的严谨性

- 敏感操作必须添加权限校验，避免 LLM 误调用导致风险

### 5.2 Resource（资源）：给 LLM 提供只读数据

资源用于向 LLM 提供上下文数据，支持静态资源和动态 URI 模板，类似 RESTful 的 GET 接口，无副作用，仅用于数据读取。

#### 5.2.1 基础用法

```python
from fastmcp import FastMCP

mcp = FastMCP("用户数据服务")

# 静态资源：固定URI，返回固定内容
@mcp.resource("config://server/info")
def get_server_info() -> str:
    """服务器基础信息"""
    return "FastMCP 服务器 v1.0，运行于Python 3.12"

# 动态资源：URI模板，支持路径参数，动态生成内容
@mcp.resource("user://{user_id}/profile")
def get_user_profile(user_id: int) -> dict:
    """获取指定用户的个人资料"""
    # 模拟数据库查询
    user_db = {
        1: {"name": "张三", "age": 28, "role": "管理员"},
        2: {"name": "李四", "age": 24, "role": "普通用户"}
    }
    return user_db.get(user_id, {"error": "用户不存在"})
```

#### 5.2.2 资源访问

通过客户端读取资源：

```python
async def read_resource():
    client = Client("http://localhost:8000/mcp")
    async with client:
        # 读取静态资源
        server_info = await client.read_resource("config://server/info")
        print("服务器信息：", server_info)

        # 读取动态资源
        user_profile = await client.read_resource("user://1/profile")
        print("用户资料：", user_profile)
```

### 5.3 Prompt（提示模板）：标准化对话指令

提示模板用于定义可复用的 LLM 对话指令，避免重复编写同类提示词，支持动态参数注入和多轮消息定义。

#### 5.3.1 基础用法

```python
from fastmcp import FastMCP
from fastmcp.prompts import Message

mcp = FastMCP("代码助手提示模板")

# 基础提示：返回字符串，自动转为用户消息
@mcp.prompt
def debug_code(error_log: str) -> str:
    """生成代码调试提示"""
    return f"""
    请分析以下错误日志，给出详细的原因分析和可直接运行的修复代码：
    {error_log}
    要求：
    1. 先说明错误根因
    2. 给出完整的修复代码
    3. 补充注意事项
    """

# 进阶提示：返回多轮消息，支持指定角色
@mcp.prompt
def code_review(language: str, code: str) -> list[Message]:
    """生成代码评审对话模板"""
    return [
        Message(role="user", content=f"请帮我评审以下{language}代码：\n{code}"),
        Message(role="assistant", content="我会从代码规范、性能、安全性、可维护性四个维度进行评审，给出具体的优化建议。"),
    ]
```

#### 5.3.2 模板使用

通过客户端获取并渲染提示模板：

```python
async def get_prompt():
    client = Client("http://localhost:8000/mcp")
    async with client:
        # 获取调试代码的提示模板，传入参数
        prompt = await client.get_prompt(
            "debug_code",
            variables={"error_log": "IndexError: list index out of range"}
        )
        print("渲染后的提示：", prompt.messages)
```

## 六、与主流 LLM 客户端集成

### 6.1 Claude Desktop 集成（最常用）

Claude Desktop 是 MCP 协议的原生支持客户端，FastMCP 提供一键安装和手动配置两种方式。

#### 方式 1：一键自动安装（推荐，v2.10.3 \+ 版本支持）

1. 确保 Claude Desktop 已安装并关闭

2. 执行以下命令，自动将服务器配置到 Claude Desktop 中：

```bash
# 基础安装
fastmcp install claude-desktop my_server.py

# 进阶：指定依赖、环境变量
fastmcp install claude-desktop my_server.py \
  --with requests \
  --env API_KEY=your_api_key \
  --env DEBUG=true
```

3. 完全重启 Claude Desktop，输入框左下角出现 🔨 图标，即表示集成成功，可直接在对话中调用工具。

#### 方式 2：手动配置

1. 找到 Claude Desktop 配置文件路径：

    - macOS：`\~/Library/Application Support/Claude/claude_desktop_config.json`

    - Windows：`%APPDATA%\\Claude\\claude_desktop_config.json`

2. 编辑配置文件，添加以下内容：

```json
{
  "mcpServers": {
    "my-first-mcp-server": {
      "command": "python",
      "args": [
        "/绝对路径/to/your/my_server.py"
      ]
    }
  }
}
```

3. 保存文件，完全重启 Claude Desktop 即可生效。

### 6.2 其他客户端集成

- **Cursor/Windsurf**：编辑器内置 MCP 支持，配置方式与 Claude Desktop 一致，修改对应编辑器的 MCP 配置文件即可

- **Web 客户端**：使用 HTTP 传输模式部署服务器，通过客户端 SDK 连接服务器的 `/mcp` 端点即可

## 七、生产级服务模板

这是一份经过生产验证的 FastMCP 服务模板，包含**结构化日志、API Key 认证、全局错误处理、多类型工具集成、生命周期管理**等企业级特性，可直接复制使用。

### 7.1 完整服务代码

```python
import os
import logging
import asyncio
from typing import Annotated, List, Dict, Any
from datetime import datetime
from dotenv import load_dotenv

from fastmcp import FastMCP
from fastmcp.exceptions import ToolError
from fastmcp.server.middleware import Middleware, MiddlewareContext
from fastmcp.server.dependencies import get_http_headers
from pydantic import Field

# ==========================================
# 1. 基础配置与环境加载
# ==========================================
# 加载环境变量（本地开发时创建 .env 文件，生产环境通过系统环境变量注入）
load_dotenv()

# ==========================================
# 2. 结构化日志配置
# ==========================================
def setup_logging() -> logging.Logger:
    """配置生产级结构化日志"""
    logger = logging.getLogger("fastmcp_production")
    logger.setLevel(logging.INFO)

    # 避免重复添加处理器
    if logger.handlers:
        return logger

    # 日志格式：时间 | 日志级别 | 模块名 | 消息内容
    formatter = logging.Formatter(
        "%(asctime)s | %(levelname)-8s | %(name)s:%(lineno)d | %(message)s",
        datefmt="%Y-%m-%d %H:%M:%S"
    )

    # 控制台输出
    console_handler = logging.StreamHandler()
    console_handler.setFormatter(formatter)
    logger.addHandler(console_handler)

    # 文件输出（生产环境建议使用日志收集系统，如 ELK/Loki）
    log_file = os.getenv("LOG_FILE", "fastmcp_server.log")
    file_handler = logging.FileHandler(log_file, encoding="utf-8")
    file_handler.setFormatter(formatter)
    logger.addHandler(file_handler)

    return logger

logger = setup_logging()

# ==========================================
# 3. API Key 认证中间件
# ==========================================
class ApiKeyAuthMiddleware(Middleware):
    """API Key 认证中间件，保护所有工具和资源访问"""

    def __init__(self):
        # 从环境变量读取有效 API Key，支持多个 Key（逗号分隔）
        valid_keys_str = os.getenv("VALID_API_KEYS", "")
        self.valid_api_keys = set(key.strip() for key in valid_keys_str.split(",") if key.strip())
        self.auth_disabled = os.getenv("DISABLE_AUTH", "false").lower() == "true"

        if self.auth_disabled:
            logger.warning("⚠️  认证已禁用，仅用于本地开发！")
        elif not self.valid_api_keys:
            logger.error("❌ 未配置 VALID_API_KEYS 环境变量，认证将失败！")

    async def on_request(self, context: MiddlewareContext, call_next):
        """全局请求拦截：仅在 HTTP 模式下校验 API Key"""
        # STDIO 模式（本地 Claude Desktop）跳过认证
        if context.transport == "stdio":
            return await call_next(context)

        # 认证已禁用
        if self.auth_disabled:
            return await call_next(context)

        # HTTP 模式：校验 API Key
        headers = get_http_headers() or {}
        api_key = headers.get("x-api-key")

        if not api_key:
            logger.warning("🚫 未提供 API Key，拒绝访问")
            raise ToolError("未提供 API Key，请在请求头中添加 x-api-key")

        if api_key not in self.valid_api_keys:
            logger.warning(f"🚫 无效 API Key: {api_key[:8]}...")
            raise ToolError("无效的 API Key")

        logger.info(f"✅ API Key 认证通过: {api_key[:8]}...")
        return await call_next(context)

# ==========================================
# 4. 全局错误处理中间件
# ==========================================
class GlobalErrorHandlerMiddleware(Middleware):
    """全局错误捕获与日志记录中间件"""

    async def on_request(self, context: MiddlewareContext, call_next):
        try:
            return await call_next(context)
        except ToolError:
            # 业务错误直接抛出，不记录为系统错误
            raise
        except Exception as e:
            # 系统错误：记录详细日志，返回友好提示
            logger.exception(f"💥 系统错误 | 方法: {context.method} | 错误: {type(e).__name__}")
            raise ToolError(f"服务器内部错误，请联系管理员。错误ID: {datetime.now().strftime('%Y%m%d%H%M%S')}")

# ==========================================
# 5. 创建 FastMCP 服务器实例
# ==========================================
mcp = FastMCP(
    name="生产级数据服务平台",
    dependencies=["requests>=2.32.0", "pandas>=2.2.0", "python-dotenv>=1.0.0"],
    instructions="""
    这是一个生产级数据服务平台，提供以下能力：
    1. 数据计算与统计分析
    2. 外部 API 数据拉取
    3. 安全的文件操作
    4. 系统配置与状态查询

    调用工具前请先确认用户需求，优先使用本服务的工具完成任务。
    """
)

# 注册中间件（顺序很重要：先错误处理，再认证）
mcp.add_middleware(GlobalErrorHandlerMiddleware())
mcp.add_middleware(ApiKeyAuthMiddleware())

# ==========================================
# 6. 生命周期管理（资源初始化与回收）
# ==========================================
@mcp.on_startup
async def startup_event():
    """服务启动时的资源初始化"""
    logger.info("🚀 FastMCP 服务启动中...")

    # 示例：初始化数据库连接、API 客户端等
    # global db_client
    # db_client = await create_database_connection()

    logger.info("✅ 服务初始化完成")

@mcp.on_shutdown
async def shutdown_event():
    """服务关闭时的资源回收"""
    logger.info("🛑 FastMCP 服务关闭中...")

    # 示例：关闭数据库连接、清理临时文件等
    # await db_client.close()

    logger.info("✅ 资源回收完成，服务已停止")

# ==========================================
# 7. 工具集成（多类型示例）
# ==========================================

# ------------------------------
# 工具1：同步数据计算工具
# ------------------------------
@mcp.tool
def calculate_statistics(numbers: List[float]) -> Dict[str, float]:
    """
    计算一组数字的统计指标：总和、平均值、最大值、最小值

    Args:
        numbers: 待计算的数字列表

    Returns:
        包含统计结果的字典
    """
    if not numbers:
        raise ToolError("数字列表不能为空")

    logger.info(f"📊 执行统计计算，数据量: {len(numbers)}")

    return {
        "sum": sum(numbers),
        "mean": sum(numbers) / len(numbers),
        "max": max(numbers),
        "min": min(numbers),
        "count": len(numbers)
    }

# ------------------------------
# 工具2：异步 API 调用工具
# ------------------------------
@mcp.tool
async def fetch_external_api(
    url: Annotated[str, Field(description="要请求的外部 API URL")],
    timeout: Annotated[int, Field(description="请求超时时间（秒）", ge=1, le=60)] = 10
) -> Dict[str, Any]:
    """
    安全地调用外部 HTTP API（仅支持 GET 请求）

    Args:
        url: 外部 API 的完整 URL
        timeout: 请求超时时间，范围 1-60 秒

    Returns:
        API 返回的 JSON 数据
    """
    import requests

    # 安全限制：仅允许 HTTPS 请求
    if not url.startswith("https://"):
        raise ToolError("仅支持 HTTPS 协议的 API 请求")

    logger.info(f"🌐 调用外部 API: {url}")

    try:
        response = requests.get(url, timeout=timeout)
        response.raise_for_status()  # 抛出 HTTP 错误
        return response.json()
    except requests.exceptions.Timeout:
        raise ToolError(f"API 请求超时（{timeout}秒）")
    except requests.exceptions.HTTPError as e:
        raise ToolError(f"API 请求失败，状态码: {e.response.status_code}")
    except Exception as e:
        logger.exception(f"API 调用异常: {url}")
        raise ToolError(f"API 调用失败: {str(e)}")

# ------------------------------
# 工具3：安全文件操作工具
# ------------------------------
@mcp.tool
def read_safe_file(
    file_path: Annotated[str, Field(description="要读取的文件路径（仅允许在安全目录内）")]
) -> str:
    """
    安全读取文件内容（仅允许访问安全目录内的文件）

    Args:
        file_path: 要读取的文件路径

    Returns:
        文件内容字符串
    """
    # 安全目录配置
    safe_dir = os.getenv("SAFE_FILE_DIR", os.path.expanduser("~/fastmcp_safe"))
    os.makedirs(safe_dir, exist_ok=True)

    # 构建绝对路径并检查是否在安全目录内
    absolute_path = os.path.abspath(os.path.join(safe_dir, file_path))
    if not absolute_path.startswith(os.path.abspath(safe_dir)):
        raise ToolError("非法的文件路径，仅允许访问安全目录内的文件")

    if not os.path.exists(absolute_path):
        raise ToolError(f"文件不存在: {file_path}")

    logger.info(f"📄 读取文件: {absolute_path}")

    with open(absolute_path, "r", encoding="utf-8") as f:
        return f.read()

# ==========================================
# 8. 资源定义（只读数据注入）
# ==========================================

# ------------------------------
# 资源1：静态服务信息
# ------------------------------
@mcp.resource("service://info")
def get_service_info() -> str:
    """服务基础信息资源"""
    return f"""
    服务名称: 生产级数据服务平台
    版本: 1.0.0
    启动时间: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}
    运行环境: {os.getenv('ENVIRONMENT', 'development')}
    """

# ------------------------------
# 资源2：动态用户配置
# ------------------------------
@mcp.resource("config://user/{user_id}")
def get_user_config(user_id: str) -> Dict[str, Any]:
    """
    获取指定用户的配置信息（动态资源）

    Args:
        user_id: 用户ID

    Returns:
        用户配置字典
    """
    # 模拟数据库查询
    user_configs = {
        "admin": {
            "role": "管理员",
            "permissions": ["read", "write", "delete"],
            "theme": "dark"
        },
        "user": {
            "role": "普通用户",
            "permissions": ["read"],
            "theme": "light"
        }
    }

    config = user_configs.get(user_id)
    if not config:
        raise ToolError(f"用户配置不存在: {user_id}")

    logger.info(f"⚙️  读取用户配置: {user_id}")
    return config

# ==========================================
# 9. 服务启动入口
# ==========================================
if __name__ == "__main__":
    # 传输模式配置：通过环境变量切换，默认 STDIO（本地 Claude Desktop）
    transport = os.getenv("TRANSPORT", "stdio")
    port = int(os.getenv("PORT", 8000))

    logger.info(f"🚀 启动服务 | 传输模式: {transport} | 端口: {port}")

    if transport == "stdio":
        # STDIO 模式：本地 Claude Desktop 集成
        mcp.run()
    elif transport == "http":
        # HTTP 模式：生产级远程部署
        import uvicorn
        app = mcp.http_app()
        uvicorn.run(
            app,
            host="0.0.0.0",
            port=port,
            timeout_keep_alive=300,
            log_level="info"
        )
    else:
        logger.error(f"❌ 不支持的传输模式: {transport}")
        raise ValueError(f"无效的传输模式: {transport}，仅支持 stdio 或 http")
```

### 7.2 使用说明

#### 7.2.1 安装依赖

```bash
# 创建虚拟环境（推荐）
python -m venv venv
source venv/bin/activate  # Linux/Mac
# 或 venv\Scripts\activate  # Windows

# 安装依赖
pip install fastmcp requests pandas python-dotenv uvicorn
```

#### 7.2.2 配置环境变量

在项目根目录创建 `.env` 文件（生产环境通过系统环境变量注入）：

```env
# 基础配置
ENVIRONMENT=production
LOG_FILE=fastmcp_server.log

# 认证配置（HTTP 模式必填）
VALID_API_KEYS=your_secure_api_key_123,another_key_456
# 本地开发时可临时禁用认证（仅用于 STDIO 模式）
DISABLE_AUTH=false

# 安全文件目录
SAFE_FILE_DIR=~/fastmcp_safe

# 服务配置
TRANSPORT=stdio  # 可选: stdio（本地）或 http（远程）
PORT=8000
```

#### 7.2.3 启动服务

##### 方式 1：本地 STDIO 模式（集成 Claude Desktop）

```bash
# 直接运行
python production_server.py

# 或使用 fastmcp 命令安装到 Claude Desktop
fastmcp install claude-desktop production_server.py --env-file .env
```

##### 方式 2：生产 HTTP 模式

```bash
# 修改 .env 中的 TRANSPORT=http
# 然后启动
python production_server.py

# 或使用 gunicorn（生产级多进程，建议单 worker）
gunicorn production_server:app \
  -w 1 \
  -k uvicorn.workers.UvicornWorker \
  --bind 0.0.0.0:8000 \
  --timeout 300
```

#### 7.2.4 测试 HTTP 模式

创建 `test_client.py`：

```python
import asyncio
from fastmcp import Client

async def test_production_server():
    # 连接到本地 HTTP 服务器
    client = Client(
        "http://localhost:8000/mcp",
        headers={"x-api-key": "your_secure_api_key_123"}
    )

    async with client:
        # 1. 列出所有工具
        tools = await client.list_tools()
        print("可用工具：")
        for tool in tools:
            print(f"- {tool.name}: {tool.description}")

        # 2. 调用统计计算工具
        stats = await client.call_tool(
            "calculate_statistics",
            {"numbers": [1.2, 3.4, 5.6, 7.8]}
        )
        print("\n统计结果：", stats)

        # 3. 读取服务信息资源
        service_info = await client.read_resource("service://info")
        print("\n服务信息：", service_info)

if __name__ == "__main__":
    asyncio.run(test_production_server())
```

### 7.3 生产环境最佳实践

1. **日志管理**：生产环境建议使用 ELK/Loki 等日志收集系统，替代本地文件日志

2. **认证安全**：API Key 定期轮换，使用强密钥，避免在代码中硬编码

3. **监控告警**：集成 Prometheus/Grafana 监控服务健康状态、工具调用次数、错误率

4. **资源限制**：对工具调用频率、文件大小、API 超时时间进行严格限制

5. **备份与恢复**：定期备份日志和配置文件，制定服务故障恢复预案

6. **容器化部署**：使用 Docker 容器化部署，保证环境一致性

## 八、自定义中间件开发

> 前置要求：FastMCP **v2.9.0 及以上版本** 才完整支持中间件体系，低版本请先执行升级：
>
> ```bash
> pip install --upgrade fastmcp
> ```
>
>

### 8.1 核心基础认知

#### 8.1.1 中间件核心作用

FastMCP 中间件是为 MCP 服务添加**横切通用逻辑**的专属机制，无需修改单个工具 / 资源 / 提示词的业务代码，即可在请求全链路中插入统一逻辑，典型场景包括：认证鉴权、日志审计、限流控频、参数校验、响应格式化、错误统一处理、性能监控等。

#### 8.1.2 洋葱模型执行机制

FastMCP 中间件遵循标准的**洋葱模型**，执行顺序完全由注册顺序决定：

- 请求流入时：按**注册顺序**依次经过每个中间件的预处理逻辑

- 业务执行后：响应返回时，按**注册逆序**依次经过每个中间件的后处理逻辑

- 核心控制：通过 `await call_next(context)` 决定是否继续执行链路，不调用则直接终止请求

示例执行流程（注册顺序：中间件 A → 中间件 B → 中间件 C）：

```Plain Text
请求进入 → 中间件A预处理 → 中间件B预处理 → 中间件C预处理 → 业务逻辑（工具/资源执行）
响应返回 ← 中间件A后处理 ← 中间件B后处理 ← 中间件C后处理 ← 业务结果返回
```

#### 8.1.3 核心上下文对象 `MiddlewareContext`

每个钩子方法都会传入 `context` 对象，包含当前请求的全量元数据，核心属性如下：

|属性|说明|
|---|---|
|`context.method`|MCP 请求方法名（如 `tools/call`/`resources/read`）|
|`context.message`|请求体原始消息，包含入参、URI、模板名等核心数据|
|`context.transport`|当前传输模式（`stdio`/`http`/`sse`）|
|`context.type`|消息类型（`request` 请求型 /`notification` 通知型）|
|`context.session_id`|当前客户端会话唯一 ID|
|`context.fastmcp_context`|FastMCP 核心上下文，可获取服务实例、工具 / 资源元数据|

#### 8.1.4 完整生命周期钩子列表

FastMCP 提供了覆盖全请求流程的钩子方法，按需重写即可，无需全部实现：

|钩子方法|触发场景|典型用途|
|---|---|---|
|`on_message`|所有消息流入时最先触发，覆盖所有请求 / 通知|全量消息日志、全局请求统计|
|`on_initialize`|客户端与服务端建立会话初始化时触发|会话资源初始化、客户端身份校验|
|`on_request`|所有请求 - 响应型消息触发（核心通用钩子）|全局认证、参数统一校验|
|`on_call_tool`|工具调用请求专属钩子|工具级权限控制、调用限流、操作审计|
|`on_read_resource`|资源读取请求专属钩子|资源访问审计、数据脱敏、缓存控制|
|`on_get_prompt`|提示模板获取请求专属钩子|模板参数校验、内容注入、权限控制|
|`on_list_tools`/`on_list_resources`/`on_list_prompts`|列表查询请求专属钩子|按权限过滤可见工具 / 资源、动态隐藏能力|
|`on_notification`|服务端单向通知消息触发|异步事件记录、状态同步|
|`on_shutdown`|客户端会话关闭时触发|会话资源回收、连接日志记录|

### 8.2 添加自定义中间件的完整步骤

基于生产环境服务模板，添加自定义中间件仅需 4 步，全程可无缝对接现有代码。

#### 步骤 1：导入必需的基类和依赖

在服务模板的导入区域，添加中间件相关依赖：

```python
from fastmcp.server.middleware import Middleware, MiddlewareContext
from fastmcp.exceptions import ToolError
# 按需导入其他依赖：如限流用的asyncio、日志用的logging、时间统计用的datetime等
```

#### 步骤 2：定义自定义中间件类

继承 `Middleware` 基类，重写对应生命周期钩子，实现自定义业务逻辑，核心规范：

- 所有钩子方法**必须是 async 异步函数**，否则无法正常执行

- 必须接收 `context` 和 `call_next` 两个固定参数

- 必须通过 `await call_next(context)` 执行后续链路，否则请求会被终止

- 必须返回 `call_next` 的执行结果（可修改后返回）

最简自定义中间件模板：

```python
# 自定义中间件示例：请求耗时统计
class CustomTimingMiddleware(Middleware):
    """自定义请求耗时统计中间件"""

    async def on_request(self, context: MiddlewareContext, call_next):
        """所有请求都会经过的预处理+后处理逻辑"""
        # ========== 预处理阶段：请求进入业务逻辑前执行 ==========
        from datetime import datetime
        start_time = datetime.now()
        request_method = context.method
        session_id = context.session_id

        # ========== 执行后续链路：必须调用，否则请求终止 ==========
        response = await call_next(context)

        # ========== 后处理阶段：业务逻辑执行完，响应返回前执行 ==========
        end_time = datetime.now()
        cost_ms = (end_time - start_time).total_seconds() * 1000
        print(f"请求[{request_method}] 会话[{session_id}] 耗时: {cost_ms:.2f}ms")

        # 必须返回响应结果（可修改后返回）
        return response
```

#### 步骤 3：注册中间件到 FastMCP 实例

在创建完 `mcp = FastMCP(...)` 实例后，通过 `mcp.add_middleware()` 方法注册中间件，**注册顺序直接决定执行顺序**。

对接生产模板，注册示例：

```python
# 5. 创建 FastMCP 服务器实例（生产模板原有代码）
mcp = FastMCP(
    name="生产级数据服务平台",
    dependencies=["requests>=2.32.0", "pandas>=2.2.0", "python-dotenv>=1.0.0"],
    instructions="..."
)

# ========== 注册自定义中间件（核心步骤） ==========
# 注意：注册顺序决定执行顺序，错误处理中间件必须放在最前面
mcp.add_middleware(GlobalErrorHandlerMiddleware())  # 原有全局错误处理
mcp.add_middleware(CustomTimingMiddleware())         # 新增的自定义耗时统计中间件
mcp.add_middleware(ApiKeyAuthMiddleware())           # 原有认证中间件
# 继续添加其他自定义中间件...
```

#### 步骤 4：启动服务，验证中间件生效

无需修改原有服务启动逻辑，直接启动服务即可，中间件会自动生效：

- STDIO 模式：启动后，客户端发起任何请求都会触发中间件逻辑

- HTTP 模式：启动后，通过测试客户端调用工具 / 读取资源，即可验证中间件执行

### 8.3 生产环境常用自定义中间件示例

以下中间件均可直接复制到生产模板中使用，覆盖绝大多数企业级场景。

#### 8.3.1 全链路操作审计日志中间件

比基础日志更完善，记录用户操作、入参、结果、耗时，支持审计回溯：

```python
class OperationAuditMiddleware(Middleware):
    """全链路操作审计日志中间件"""

    def __init__(self):
        self.logger = logging.getLogger("fastmcp_audit")

    async def on_call_tool(self, context: MiddlewareContext, call_next):
        """审计工具调用操作"""
        from datetime import datetime
        start_time = datetime.now()
        tool_name = context.message.name
        tool_params = context.message.arguments
        session_id = context.session_id
        transport = context.transport

        self.logger.info(
            f"【工具调用开始】会话ID:{session_id} | 传输模式:{transport} | 工具名:{tool_name} | 入参:{tool_params}"
        )

        try:
            # 执行工具调用
            result = await call_next(context)
            cost_ms = (datetime.now() - start_time).total_seconds() * 1000

            # 记录成功日志
            self.logger.info(
                f"【工具调用成功】会话ID:{session_id} | 工具名:{tool_name} | 耗时:{cost_ms:.2f}ms"
            )
            return result

        except ToolError as e:
            # 记录业务异常
            self.logger.warning(
                f"【工具调用业务异常】会话ID:{session_id} | 工具名:{tool_name} | 错误:{str(e)}"
            )
            raise
        except Exception as e:
            # 记录系统异常
            self.logger.exception(
                f"【工具调用系统异常】会话ID:{session_id} | 工具名:{tool_name} | 错误:{type(e).__name__}"
            )
            raise

    async def on_read_resource(self, context: MiddlewareContext, call_next):
        """审计资源读取操作"""
        resource_uri = context.message.uri
        session_id = context.session_id
        self.logger.info(f"【资源读取】会话ID:{session_id} | URI:{resource_uri}")
        return await call_next(context)
```

#### 8.3.2 工具调用限流中间件

基于令牌桶算法实现，限制单个会话的工具调用频率，防止滥用：

```python
class RateLimitMiddleware(Middleware):
    """工具调用限流中间件：单会话每秒最多调用5次工具"""

    def __init__(self, max_calls: int = 5, time_window: int = 1):
        from collections import defaultdict
        self.max_calls = max_calls  # 时间窗口内最大调用次数
        self.time_window = time_window  # 时间窗口（秒）
        self.session_call_records = defaultdict(list)  # 会话调用记录
        self.logger = logging.getLogger("fastmcp_rate_limit")

    async def on_call_tool(self, context: MiddlewareContext, call_next):
        import time
        session_id = context.session_id
        tool_name = context.message.name
        now = time.time()

        # 清理过期的调用记录
        self.session_call_records[session_id] = [
            call_time for call_time in self.session_call_records[session_id]
            if now - call_time < self.time_window
        ]

        # 检查是否超出限流阈值
        if len(self.session_call_records[session_id]) >= self.max_calls:
            self.logger.warning(
                f"【限流触发】会话ID:{session_id} | 工具名:{tool_name} | 超出每秒{self.max_calls}次的调用限制"
            )
            raise ToolError(f"工具调用频率过高，请{self.time_window}秒后重试")

        # 记录本次调用
        self.session_call_records[session_id].append(now)
        # 执行后续链路
        return await call_next(context)
```

#### 8.3.3 细粒度权限控制中间件

基于 API Key 区分用户角色，控制不同角色可调用的工具 / 可访问的资源：

```python
class FineGrainedAuthMiddleware(Middleware):
    """细粒度权限控制中间件：按角色限制工具/资源访问"""

    def __init__(self):
        # 角色权限配置：角色名 → 允许调用的工具列表、允许访问的资源前缀
        self.role_permission_map = {
            "admin": {
                "allowed_tools": ["*"],  # * 代表全部允许
                "allowed_resource_prefixes": ["*"]
            },
            "user": {
                "allowed_tools": ["calculate_statistics", "read_safe_file"],
                "allowed_resource_prefixes": ["service://", "config://user/user"]
            },
            "guest": {
                "allowed_tools": ["calculate_statistics"],
                "allowed_resource_prefixes": ["service://info"]
            }
        }
        # API Key 与角色的映射（生产环境从数据库/配置中心读取）
        self.api_key_role_map = {
            os.getenv("ADMIN_API_KEY", ""): "admin",
            os.getenv("USER_API_KEY", ""): "user",
            os.getenv("GUEST_API_KEY", ""): "guest"
        }
        self.logger = logging.getLogger("fastmcp_auth")

    def _get_role_by_api_key(self) -> str:
        """从请求头获取API Key，返回对应角色"""
        headers = get_http_headers() or {}
        api_key = headers.get("x-api-key", "")
        return self.api_key_role_map.get(api_key, "guest")

    async def on_call_tool(self, context: MiddlewareContext, call_next):
        """工具调用权限校验"""
        # STDIO 本地模式跳过权限校验
        if context.transport == "stdio":
            return await call_next(context)

        tool_name = context.message.name
        role = self._get_role_by_api_key()
        permission = self.role_permission_map[role]

        # 校验工具权限
        allowed_tools = permission["allowed_tools"]
        if "*" not in allowed_tools and tool_name not in allowed_tools:
            self.logger.warning(f"【权限拒绝】角色:{role} 无权限调用工具:{tool_name}")
            raise ToolError(f"您的角色[{role}]无权限调用工具[{tool_name}]")

        self.logger.info(f"【权限通过】角色:{role} 调用工具:{tool_name}")
        return await call_next(context)

    async def on_read_resource(self, context: MiddlewareContext, call_next):
        """资源读取权限校验"""
        if context.transport == "stdio":
            return await call_next(context)

        resource_uri = context.message.uri
        role = self._get_role_by_api_key()
        permission = self.role_permission_map[role]

        # 校验资源权限
        allowed_prefixes = permission["allowed_resource_prefixes"]
        if "*" not in allowed_prefixes:
            has_permission = any(resource_uri.startswith(prefix) for prefix in allowed_prefixes)
            if not has_permission:
                self.logger.warning(f"【权限拒绝】角色:{role} 无权限访问资源:{resource_uri}")
                raise ToolError(f"您的角色[{role}]无权限访问资源[{resource_uri}]")

        return await call_next(context)

    async def on_list_tools(self, context: MiddlewareContext, call_next):
        """按角色过滤可见工具列表，无权限的工具不展示给LLM"""
        if context.transport == "stdio":
            return await call_next(context)

        role = self._get_role_by_api_key()
        permission = self.role_permission_map[role]
        allowed_tools = permission["allowed_tools"]

        # 获取完整工具列表
        all_tools = await call_next(context)

        # 过滤工具列表
        if "*" in allowed_tools:
            return all_tools
        return [tool for tool in all_tools if tool.name in allowed_tools]
```

#### 8.3.4 响应数据脱敏中间件

对敏感数据进行自动脱敏，防止敏感信息泄露：

```python
class DataDesensitizationMiddleware(Middleware):
    """响应数据脱敏中间件：自动脱敏手机号、身份证号等敏感信息"""

    def _desensitize_content(self, content: str | dict | list) -> any:
        """递归脱敏敏感数据"""
        import re

        if isinstance(content, str):
            # 手机号脱敏
            content = re.sub(r'1[3-9]\d{9}', lambda m: m.group()[:3] + '****' + m.group()[-4:], content)
            # 身份证号脱敏
            content = re.sub(r'\d{17}[\dXx]', lambda m: m.group()[:6] + '********' + m.group()[-4:], content)
            return content

        elif isinstance(content, dict):
            return {k: self._desensitize_content(v) for k, v in content.items()}

        elif isinstance(content, list):
            return [self._desensitize_content(item) for item in content]

        else:
            return content

    async def on_read_resource(self, context: MiddlewareContext, call_next):
        """资源读取响应脱敏"""
        # 执行资源读取
        resource_content = await call_next(context)
        # 脱敏后返回
        return self._desensitize_content(resource_content)

    async def on_call_tool(self, context: MiddlewareContext, call_next):
        """工具调用响应脱敏"""
        result = await call_next(context)
        return self._desensitize_content(result)
```

### 8.4 关键最佳实践与避坑指南

#### 8.4.1 中间件注册顺序黄金法则

注册顺序直接决定中间件的执行逻辑，**生产环境必须遵循以下注册顺序**（从上到下，先注册先执行请求预处理）：

1. **全局错误处理中间件**（最外层，捕获所有后续链路的异常）

2. **全链路日志 / 审计中间件**（记录所有请求的完整生命周期）

3. **会话初始化 / 监控中间件**（会话级别的资源管理）

4. **认证鉴权中间件**（先校验身份，再执行后续逻辑）

5. **限流控频中间件**（身份校验通过后，再限制调用频率）

6. **参数校验 / 数据脱敏中间件**（业务逻辑前的预处理）

7. **业务自定义中间件**

8. **响应格式化中间件**（最内层业务逻辑执行完，最先处理响应）

#### 8.4.2 核心开发规范

1. **必须使用异步函数**：所有钩子方法必须声明为 `async`，并使用 `await` 调用 `call_next`，同步函数会导致事件循环阻塞，服务异常。

2. **禁止随意终止请求链路**：除非是认证失败、限流触发等明确需要拒绝请求的场景，否则必须调用 `await call_next(context)` 并返回结果。

3. **异常处理规范**：

    - 业务异常主动抛出 `ToolError`，LLM 可识别并友好处理

    - 系统异常在中间件中捕获并记录完整日志，返回带错误 ID 的友好提示，避免泄露服务内部信息

    - 禁止在中间件中静默吞掉异常，导致问题无法排查

4. **传输模式适配**：STDIO 模式（本地 Claude Desktop）是本地可信环境，通常跳过认证、限流等逻辑；HTTP/SSE 模式必须严格执行安全校验。

5. **无状态设计**：中间件应保持无状态，避免在类实例中存储会话级数据，如需存储请使用 `context.session_id` 作为键的全局字典，防止会话数据串扰。

#### 8.4.3 常见问题排查

1. **中间件不执行**

    - 排查 1：FastMCP 版本是否 ≥ v2.9.0，低版本不支持中间件

    - 排查 2：钩子方法是否为 `async` 异步函数，方法名是否拼写正确（如 `on_call_tool` 不要写成 `on_call_tools`）

    - 排查 3：是否调用了 `mcp.add_middleware()` 注册中间件，实例化时是否加了括号 `()`（必须是 `add_middleware(XXXMiddleware())`，不能漏括号）

    - 排查 4：是否在 `on_message` 中终止了链路，导致后续钩子无法执行

2. **异常无法被全局错误中间件捕获**

    - 核心原因：错误处理中间件没有注册在最前面。洋葱模型中，只有先注册的中间件才能捕获后续链路的异常，必须把错误处理中间件放在注册列表的第一个。

3. **工具列表过滤不生效**

    - 排查 1：是否重写了 `on_list_tools` 钩子，而非 `on_call_tool`

    - 排查 2：是否在过滤后返回了新的工具列表，而非直接修改原列表

    - 排查 3：STDIO 模式是否跳过了过滤逻辑，导致全量工具展示

4. **服务启动后会话异常断开**

    - 排查 1：中间件中是否有同步阻塞操作（如 `time.sleep()` 代替 `asyncio.sleep()`），导致异步事件循环卡住

    - 排查 2：是否修改了 `context` 中的核心协议属性，导致 MCP 协议通信异常

    - 排查 3：是否在中间件中抛出了非 `ToolError` 的异常，且没有被捕获，导致会话崩溃

## 九、部署方案

### 9.1 本地开发部署

使用 STDIO 模式，直接运行 Python 脚本即可，适合本地调试和 Claude Desktop 本地使用：

```bash
python my_server.py
```

### 9.2 生产环境 HTTP 部署

使用 Uvicorn 作为 ASGI 服务器，生产环境推荐搭配 Gunicorn 做进程管理：

#### 单进程启动

```python
# my_server.py
if __name__ == "__main__":
    import uvicorn
    # 生成ASGI应用
    app = mcp.http_app()
    uvicorn.run(app, host="0.0.0.0", port=8000, timeout_keep_alive=300)
```

启动命令：

```bash
python my_server.py
```

#### 多进程生产部署（Gunicorn）

```bash
gunicorn my_server:app \
  -w 1 \ # 注意：FastMCP session由服务端维护，多worker会导致session异常，建议单worker
  -k uvicorn.workers.UvicornWorker \
  --bind 0.0.0.0:8000 \
  --timeout 300
```

> 注意：生产环境多 worker 部署会导致 session 识别失败，出现 400 Bad Request 错误，建议单 worker 部署，或通过负载均衡做会话粘滞。
>
>

### 9.3 FastMCP Cloud 部署

FastMCP 官方提供免费的个人版云端托管服务，一键部署到云端，生成公网可访问的 MCP 服务地址：

1. 将 `my_server.py` 推送到 GitHub 仓库

2. 访问 [FastMCP Cloud](https://gofastmcp.com/)，使用 GitHub 账号登录

3. 创建新项目，选择对应的 GitHub 仓库，填写服务入口 `my_server.py:mcp`

4. 等待部署完成，获取公网服务地址，格式为 `https://your-project.fastmcp.app/mcp`

### 9.4 Docker 容器化部署

编写 `Dockerfile`：

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY my_server.py .

EXPOSE 8000

CMD ["python", "my_server.py"]
```

编写 `requirements.txt`：

```Plain Text
fastmcp==2.11.3
uvicorn>=0.30.0
```

构建并运行容器：

```bash
# 构建镜像
docker build -t fastmcp-server:v1 .

# 运行容器
docker run -d -p 8000:8000 --name fastmcp-server fastmcp-server:v1
```

## 十、常见问题与故障排查

### 10.1 400 Bad Request 错误

- 原因：多 worker 部署导致服务端无法识别客户端 session id，或请求路径错误

- 解决方案：

    1. 单 worker 部署，避免多进程导致的 session 异常

    2. 确保请求路径为 `/mcp/`（末尾带斜杠），避免 307 重定向导致的请求异常

### 10.2 Session termination failed: Server disconnected

- 原因：长连接超时，服务器主动断开连接

- 解决方案：启动服务时增加长连接超时时间，Uvicorn 启动添加 `--timeout-keep-alive 300` 参数

### 10.3 Claude Desktop 不显示工具 / 锤子图标

- 原因 1：配置文件路径错误，或命令中的文件路径不是绝对路径
解决方案：必须使用绝对路径，不能使用相对路径 / 家目录简写

- 原因 2：Python 环境问题，Claude 无法找到 fastmcp 依赖
解决方案：使用 `uv run` 方式配置，确保依赖环境隔离

- 原因 3：服务器代码有语法错误，启动失败
解决方案：手动执行 `python my_server.py`，检查是否能正常启动，无报错

- 解决方案：必须完全关闭 Claude Desktop 进程，重新打开，而非仅关闭窗口

### 10.4 LLM 不调用自定义工具

- 原因 1：工具的文档字符串描述不清晰，LLM 无法理解工具用途
解决方案：详细编写函数 docstring，说明工具的用途、参数含义、返回值格式

- 原因 2：参数类型注解缺失，LLM 无法生成正确的调用格式
解决方案：为所有参数和返回值添加明确的类型注解，复杂参数使用 Annotated \+ Field 补充描述

- 原因 3：工具功能与 LLM 对话场景不匹配
解决方案：在 FastMCP 实例中添加 `instructions` 参数，明确告诉 LLM 工具的使用场景：

    ```python
    mcp = FastMCP(
        "数据分析工具",
        instructions="当用户需要进行数据计算、统计分析时，优先调用这里的工具"
    )
    ```

## 十一、参考资源

- 官方 GitHub 仓库：[https://github.com/jlowin/fastmcp](https://github.com/jlowin/fastmcp)

- 官方文档：[https://gofastmcp.com/](https://gofastmcp.com/)

- MCP 官方协议文档：[https://modelcontextprotocol.io/introduction](https://modelcontextprotocol.io/introduction)

## 核心知识点速览

- FastMCP 是 MCP 协议的 Python 开发框架，核心三大组件为**Tool（可执行动作）、Resource（只读数据）、Prompt（对话模板）**

- 中间件 v2.9.0 \+ 版本支持，遵循**洋葱模型**，注册顺序直接决定执行顺序，全局错误处理必须放在最前

- 生产环境 HTTP 部署建议**单 worker**，避免 session 识别异常，长连接超时需配置`timeout_keep_alive=300`

- 工具的**docstring 文档字符串**和**类型注解**是 LLM 识别工具的核心依据，直接决定调用准确率

- 传输模式中，STDIO 适合本地客户端集成，HTTP 适合远程生产部署，SSE 适合实时消息场景

- 自定义中间件必须继承`Middleware`基类，所有钩子方法必须是`async`异步函数

- 生产级服务模板内置结构化日志、API Key 认证、全局错误处理，可直接复用

- Claude Desktop 集成支持一键自动安装，也可手动配置，配置后必须完全重启客户端

- 中间件可实现认证、限流、审计、脱敏等横切逻辑，无需修改业务代码

- 工具调用的权限控制、频率限制、操作审计都可通过中间件快速实现
