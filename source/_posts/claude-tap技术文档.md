---
title: claude-tap技术文档
date: 2026-05-17 19:30:00
tags: [Claude Code, 技术分享, claude-tap, AI 编程, cc代理分析工具]
categories: 工具使用
toc: true
---
# claude-tap技术文档

## 目录

- [1. 概述](#1-概述)

- [2. 技术架构](#2-技术架构)

    - [2.1 整体架构](#21-整体架构)

    - [2.2 工作原理](#22-工作原理)

    - [2.3 代理模式](#23-代理模式)

    - [2.4 流量处理机制](#24-流量处理机制)

- [3. 核心技术实现](#3-核心技术实现)

    - [3.1 安全脱敏机制](#31-安全脱敏机制)

    - [3.2 低开销流式转发](#32-低开销流式转发)

    - [3.3 自包含查看器](#33-自包含查看器)

    - [3.4 实时同步机制](#34-实时同步机制)

- [4. 项目代码结构](#4-项目代码结构)

- [5. 安装部署](#5-安装部署)

    - [5.1 系统要求](#51-系统要求)

    - [5.2 安装方式](#52-安装方式)

    - [5.3 升级方法](#53-升级方法)

- [6. 详细使用指南](#6-详细使用指南)

    - [6.1 通用命令格式](#61-通用命令格式)

    - [6.2 支持的客户端](#62-支持的客户端)

- [7. 高级功能](#7-高级功能)

- [8. CLI 完整参数参考](#8-cli-完整参数参考)

- [9. 技术特性对比](#9-技术特性对比)

- [10. 常见问题](#10-常见问题)


---

## 1. 概述

claude-tap 是一个开源的 AI 代理流量分析工具，专门用于拦截并查看各类终端 AI 助手的 API 流量，打破 AI 编程助手的 "黑箱" 状态。

### 核心价值

- **透明化**：完整查看系统提示词、上下文管理、工具调用细节

- **可观测**：精确追踪 Token 用量、缓存命中、API 调用时序

- **可调试**：支持实时查看、历史回溯、流量导出

- **兼容性**：支持 8+ 主流 AI 客户端，无需修改客户端代码

### 支持的客户端

|客户端|类型|默认代理模式|
|---|---|---|
|Claude Code|Anthropic 官方 CLI|反向代理|
|Codex CLI|OpenAI 终端助手|反向代理|
|Kimi CLI|月之暗面终端助手|反向代理|
|Gemini CLI|Google 终端助手|正向代理|
|OpenCode|多 Provider 终端助手|正向代理|
|Pi|多 Provider Coding Agent|正向代理|
|Hermes Agent|多 Provider Python Agent|正向代理|
|Cursor CLI|Cursor 编辑器 CLI|正向代理|

---

## 2. 技术架构

### 2.1 整体架构

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=OTI2OTNkZDlkMWFiMWQ5NjlkMmUwMjY5OGNmYWNjZTlfMzI5YTRhOWUxMGY2NTljMjc2NTAyY2JiNGFkMGI2ZjZfSUQ6NzY0MDgyMTc2OTUyNzU3NzU1M18xNzc5MDE3NTY1OjE3NzkxMDM5NjVfVjM)

claude-tap 采用分层架构设计，从上到下分为：

1. **客户端层**：各类 AI 终端助手（Claude Code、Codex 等）

2. **代理层**：反向 / 正向代理服务器，负责流量拦截与转发

3. **记录层**：流量记录、脱敏处理、Trace 存储

4. **查看层**：HTML 查看器、实时 Dashboard、导出功能

5. **上游层**：AI 服务提供商 API 端点

### 2.2 工作原理

claude-tap 采用 "中间人代理"（Man-in-the-Middle Proxy）技术，完整工作流程如下：

```Plain Text
1. 启动阶段
   ├── 启动代理服务器（反向/正向）
   ├── 生成临时 CA 证书（用于正向代理 HTTPS 拦截）
   └── 启动子进程运行目标 AI 客户端

2. 流量拦截
   ├── 客户端请求被重定向到本地代理
   ├── 代理解析请求内容，进行脱敏处理
   ├── 记录完整的请求元数据和 payload

3. 流量转发
   ├── 将请求转发到上游 API 服务器
   ├── 实时接收上游响应，流式转发回客户端
   ├── 同步记录响应内容和流数据

4. 数据存储
   ├── 将请求-响应对写入 JSONL Trace 文件
   ├── 实时推送到 Live Viewer（如果启用）

5. 后处理
   ├── 客户端退出后，生成自包含 HTML 查看器
   ├── 自动清理过期 Trace 文件
   └── 自动打开查看器（默认）
```

### 2.3 代理模式

claude-tap 支持两种代理模式，根据不同客户端的特性自动选择：

#### 反向代理模式（Reverse Proxy）

**适用场景**：支持自定义 Base URL 的客户端
**工作机制**：

- 代理服务器监听本地端口

- 修改客户端的 Base URL 指向本地代理

- 客户端所有请求都发送到代理，代理转发到上游

- 无需证书，因为客户端主动信任代理

**技术优势**：

- 零证书配置

- 低 overhead，性能最优

- 完全透明，客户端无感知

**支持客户端**：Claude Code、Codex CLI、Kimi CLI

#### 正向代理模式（Forward Proxy）

**适用场景**：不支持 Base URL 的多 Provider 客户端
**工作机制**：

- 代理服务器作为 HTTPS 正向代理

- 注入 `HTTPS_PROXY` 环境变量到客户端进程

- 生成临时 CA 证书，注入到客户端信任链

- 拦截所有 HTTPS 流量，无需修改客户端配置

**技术优势**：

- 支持任意上游域名

- 可捕获客户端所有 Provider 的流量

- 无需客户端支持自定义 Base URL

**支持客户端**：Gemini CLI、OpenCode、Pi、Hermes、Cursor CLI

### 2.4 流量处理机制

#### SSE 流式转发

针对 AI 服务常用的 Server-Sent Events 流式响应，claude-tap 实现了零拷贝的流式转发：

- 逐 chunk 转发，不等待完整响应

- 实时记录流数据，支持断点续传

- 保持原有的流控和背压机制

- 增加延迟 < 1ms，几乎无性能损耗

#### WebSocket 支持

对于部分客户端使用的 WebSocket 长连接：

- 完整支持 WebSocket 帧转发

- 支持双向流量记录

- 自动处理帧分片和重组

- 兼容 OAuth 认证的 WebSocket 连接

---

## 3. 核心技术实现

### 3.1 安全脱敏机制

为了保护用户隐私，claude-tap 实现了自动脱敏机制：

```python
# 自动脱敏的 Header 列表
REDACTED_HEADERS = {
    "authorization",
    "proxy-authorization",
    "x-api-key",
    "x-auth-token",
    "cookie",
    "set-cookie",
}
```

**脱敏规则**：

- 所有敏感认证 Header 自动替换为 `[REDACTED]`

- 不会记录任何 API Key、Token、Cookie 等敏感信息

- 脱敏是在记录阶段进行，转发时使用原始值

- 支持自定义脱敏规则扩展

### 3.2 低开销流式转发

claude-tap 使用 Python AsyncIO 实现异步非阻塞 IO：

- 全异步架构，单线程可处理数千并发连接

- 流式处理，无需缓存完整请求 / 响应

- 内存占用极低，单 Trace 仅占用几 MB 内存

- 支持 GB 级大流量的处理

性能测试数据：

- 平均转发延迟：< 0.5ms

- CPU 占用：< 5%（正常负载）

- 内存占用：~20MB（基础运行）

- 并发连接：支持 1000+ 并发

### 3.3 自包含查看器

查看器是一个完全独立的 HTML 文件，**零外部依赖**：

- 所有功能内置，无需网络连接即可查看

- 包含完整的 CSS、JS、数据，单文件即可运行

- 支持暗色 / 亮色主题切换

- 响应式设计，支持移动端

**前端技术栈**：

- 原生 Vanilla JS，无框架依赖

- Tailwind CSS 内置编译

- 支持 i18n 国际化（8 种语言）

- 支持键盘导航、搜索、过滤

### 3.4 实时同步机制

Live 模式下，claude-tap 使用 SSE（Server-Sent Events）实现实时同步：

- 代理服务器作为 SSE 服务端

- 浏览器查看器作为客户端订阅更新

- 新的 Trace 数据实时推送到浏览器

- 支持断线重连和增量同步

- 无需 WebSocket，兼容性更好

---

## 4. 项目代码结构

```Plain Text
claude-tap/
├── claude_tap/                 # 核心代码
│   ├── __init__.py
│   ├── __main__.py            # 入口点
│   ├── cli.py                 # CLI 解析
│   ├── proxy/
│   │   ├── __init__.py
│   │   ├── reverse_proxy.py   # 反向代理实现
│   │   ├── forward_proxy.py   # 正向代理实现
│   │   └── ca.py              # 临时 CA 证书生成
│   ├── clients/               # 客户端适配层
│   │   ├── __init__.py
│   │   ├── claude.py
│   │   ├── codex.py
│   │   ├── gemini.py
│   │   ├── kimi.py
│   │   ├── opencode.py
│   │   ├── pi.py
│   │   ├── hermes.py
│   │   └── cursor.py
│   ├── recorder/              # 记录层
│   │   ├── __init__.py
│   │   ├── trace.py           # Trace 管理
│   │   └── redaction.py       # 脱敏处理
│   ├── viewer/                # 查看器生成
│   │   ├── __init__.py
│   │   ├── generator.py       # HTML 生成
│   │   └── live.py            # Live 模式
│   └── utils/                 # 工具函数
├── viewer/                     # 查看器前端资源
│   └── index.html             # 查看器模板
├── tests/                      # 测试用例
├── docs/                       # 文档
├── pyproject.toml             # 项目配置
├── README.md                  # 英文 README
└── README_zh.md               # 中文 README
```

**语言分布**：

- Python: 76.7%（后端代理、CLI、记录层）

- HTML: 21.4%（前端查看器）

- Shell: 1.9%（脚本、安装工具）

---

## 5. 安装部署

### 5.1 系统要求

- Python 3.11 或更高版本

- 目标 AI 客户端已安装

- 操作系统：Linux /macOS/ Windows（WSL 推荐）

- 网络：可访问 AI 服务 API

### 5.2 安装方式

#### 推荐：使用 uv（最快最可靠）

```bash
# 安装 uv 包管理器
curl -LsSf https://astral.sh/uv/install.sh | sh

# 安装 claude-tap
uv tool install claude-tap
```

#### 使用 pip

```bash
pip install claude-tap
```

### 5.3 升级方法

```bash
# 使用 uv
uv tool upgrade claude-tap

# 使用 pip
pip install --upgrade claude-tap

# 内置升级命令
claude-tap update
```

---

## 6. 详细使用指南

### 6.1 通用命令格式

```bash
claude-tap [--tap-* 选项] -- [客户端参数]
```

所有非 `--tap-*` 开头的参数都会透传给所选的 AI 客户端。

### 6.2 支持的客户端

#### 1. Claude Code（默认）

```bash
# 基本使用
claude-tap

# 实时查看模式
claude-tap --tap-live

# 指定模型 + 自动批准工具调用
claude-tap --tap-live -- --model claude-sonnet-4-6 --dangerously-skip-permissions

# 搭配 DeepSeek API
export ANTHROPIC_AUTH_TOKEN="<你的 DeepSeek API key>"
claude-tap \
  --tap-proxy-mode reverse \
  --tap-target https://api.deepseek.com/anthropic \
  -- --permission-mode bypassPermissions
```

#### 2. Codex CLI

```bash
# OAuth 用户（ChatGPT Plus/Pro/Team）
claude-tap --tap-client codex

# API Key 用户
claude-tap --tap-client codex

# 全自动模式 + 实时查看
claude-tap --tap-client codex --tap-live -- --full-auto
```

#### 3. Kimi CLI

```bash
# 基本使用
claude-tap --tap-client kimi

# 开启思考过程
claude-tap --tap-client kimi -- --thinking

# 使用 Open Platform API
claude-tap --tap-client kimi --tap-target https://api.moonshot.ai/v1
```

#### 4. Gemini CLI

```bash
# 基本使用
claude-tap --tap-client gemini -- -p "hello"

# 实时查看
claude-tap --tap-client gemini --tap-live -- -p "hello"
```

#### 5. OpenCode

```bash
# 正向代理模式（捕获所有 Provider）
claude-tap --tap-client opencode

# 实时查看
claude-tap --tap-client opencode --tap-live
```

#### 6. Pi

```bash
# 使用 OpenAI Codex OAuth
claude-tap --tap-client pi -- --model openai-codex/gpt-5.3-codex-spark -p "hello"

# 实时查看
claude-tap --tap-client pi --tap-live -- --model openai-codex/gpt-5.3-codex-spark -p "hello"
```

#### 7. Hermes Agent

```bash
# 交互式 TUI 模式
claude-tap --tap-client hermes --tap-live

# Gateway 模式（捕获 Slack/Telegram 触发的调用）
claude-tap --tap-client hermes -- gateway start
```

#### 8. Cursor CLI

```bash
# 基本使用
claude-tap --tap-client cursor -- -p --trust --model auto "hello"

# 继续上次对话
claude-tap --tap-client cursor -- -p --trust --model auto --continue "continue"
```

---

## 7. 高级功能

### 7.1 独立 Dashboard

浏览历史 Trace，不启动客户端：

```bash
claude-tap dashboard

# 自定义端口和目录
claude-tap dashboard --tap-output-dir ./my-traces --tap-live-port 3000
```

### 7.2 导出功能

从 JSONL Trace 生成 HTML 查看器：

```bash
claude-tap export trace.jsonl -o trace.html
```

### 7.3 纯代理模式

仅启动代理，手动连接：

```bash
# 启动代理
claude-tap --tap-no-launch --tap-port 8080

# 另一个终端连接
ANTHROPIC_BASE_URL=http://127.0.0.1:8080 claude
```

### 7.4 自定义配置

```bash
# 自定义 Trace 目录
claude-tap --tap-output-dir ./my-traces

# 限制最大 Trace 数量
claude-tap --tap-max-traces 10

# 禁用自动打开查看器
claude-tap --tap-no-open

# 固定代理端口
claude-tap --tap-port 8080
```

---

## 8. CLI 完整参数参考

|参数|说明|默认值|
|---|---|---|
|`--tap-client CLIENT`|启动的客户端|`claude`|
|`--tap-target URL`|上游 API 地址|自动检测|
|`--tap-live`|启动实时查看器|关闭|
|`--tap-live-port PORT`|实时查看器端口|自动分配|
|`--tap-no-open`|退出后不自动打开 HTML|关闭|
|`--tap-output-dir DIR`|Trace 输出目录|`./.traces`|
|`--tap-port PORT`|代理端口|自动分配|
|`--tap-host HOST`|绑定地址|`127.0.0.1`|
|`--tap-no-launch`|仅启动代理，不启动客户端|关闭|
|`--tap-max-traces N`|最大保留 Trace 数量|50|
|`--tap-no-update-check`|禁用更新检查|关闭|
|`--tap-proxy-mode MODE`|代理模式: reverse/forward|自动选择|

---

## 9. 技术特性对比

|特性|claude-tap|其他代理工具|
|---|---|---|
|支持多客户端|✅ 8+ 客户端|❌ 通常只支持单个|
|自动脱敏|✅ 内置自动脱敏|❌ 需手动配置|
|自包含查看器|✅ 单文件 HTML，零依赖|❌ 需外部服务|
|实时查看|✅ SSE 实时同步|❌ 仅事后查看|
|低开销|✅ <0.5ms 延迟|❌ 通常更高|
|正向 / 反向代理|✅ 自动切换|❌ 通常只支持一种|
|WebSocket 支持|✅ 完整支持|❌ 部分不支持|
|SSE 流式支持|✅ 零拷贝转发|❌ 部分缓存完整响应|
|国际化|✅ 8 种语言|❌ 通常仅英文|

---

## 10. 常见问题

### Q: 我的 API Key 会被记录吗？

A: 不会。claude-tap 会自动脱敏所有认证 Header，不会在 Trace 文件中记录任何敏感的认证信息。

### Q: 会影响客户端性能吗？

A: 影响极小。claude-tap 采用异步流式转发，平均增加延迟 <0.5ms，几乎无法感知。

### Q: 支持 Windows 吗？

A: 推荐使用 WSL2。原生 Windows 部分功能可能有限制，正向代理的证书注入在 Windows 上需要额外配置。

### Q: Trace 文件存在哪里？

A: 默认在当前目录的 `./.traces/` 下，按日期分组，每个会话一个独立的 Trace 文件。

### Q: 可以在生产环境使用吗？

A: claude-tap 主要用于开发调试。生产环境使用请确保数据安全，不要泄露敏感信息。

---

**最后更新**: 2026-05-17