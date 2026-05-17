---
title: Claude Code 学习手册
date: 2026-05-17 22:30:00
tags: [Claude Code, cc, AI开发工具, AI coding, Claude]
categories: 工具使用
toc: true
---
# Claude Code 学习手册

## 文档核心说明

本文档适用于 Claude Code 的入门到进阶用户，整合了从基础安装到高级配置的完整知识，覆盖：

- 基础安装与认证配置

- 层级架构与工作原理

- 核心使用方法与快捷键

- 扩展生态（Skills、MCP 服务器）

- 高级配置与性能优化

- 版本升级与问题修复

## 一、基础入门

### 1.1 什么是 Claude Code

Claude Code 是 Anthropic 推出的**代理式编码系统**，能够自主理解整个代码库、跨多个文件进行修改、运行命令、执行测试，并最终交付可提交的代码。

### 1.2 系统要求

- **操作系统**：macOS 13.0+、Windows 10 1809+、Ubuntu 20.04+、Debian 10+、Alpine Linux 3.19+

- **硬件**：4GB+ RAM，x64 或 ARM64 处理器

- **网络**：互联网连接

- **账户**：需要 Claude Pro、Max、Team、Enterprise 或 Console 账户（免费计划不包含 Claude Code）

### 1.3 下载安装

#### 原生安装（自动更新，推荐）

**macOS, Linux, WSL:**

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

**Windows PowerShell:**

```powershell
irm https://claude.ai/install.ps1 | iex
```

**Windows CMD:**

```cmd
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

#### 包管理器安装

**Homebrew (macOS):**

```bash
# 稳定版（约一周更新一次）
brew install --cask claude-code

# 最新版（实时更新）
brew install --cask claude-code@latest
```

**WinGet (Windows):**

```powershell
winget install Anthropic.ClaudeCode
```

**Linux 包管理器:**

```bash
# Debian/Ubuntu
sudo install -d -m 0755 /etc/apt/keyrings
sudo curl -fsSL https://downloads.claude.ai/keys/claude-code.asc -o /etc/apt/keyrings/claude-code.asc
echo "deb [signed-by=/etc/apt/keyrings/claude-code.asc] https://downloads.claude.ai/claude-code/apt/stable stable main" | sudo tee /etc/apt/sources.list.d/claude-code.list
sudo apt update && sudo apt install claude-code

# Fedora/RHEL
sudo tee /etc/yum.repos.d/claude-code.repo <<'EOF'
[claude-code]
name=Claude Code
baseurl=https://downloads.claude.ai/claude-code/rpm/stable
enabled=1
gpgcheck=1
gpgkey=https://downloads.claude.ai/keys/claude-code.asc
EOF
sudo dnf install claude-code
```

#### npm 安装

```bash
npm install -g @anthropic-ai/claude-code
```

### 1.4 验证安装

```bash
# 查看版本
claude --version

# 全面检查安装和配置
claude doctor
```

### 1.5 首次认证

安装完成后，在终端中运行：

```bash
cd your-project-directory
claude
```

系统会自动打开浏览器，引导您完成 OAuth 认证流程。

## 二、层级架构

### 2.1 三层功能架构（用户视角）

这是最常用的功能划分，将系统分为三个功能层：

```Plain Text
┌─────────────────────────────────────────────────────────┐
│                    EXTENSION LAYER                      │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐    │
│  │   MCP   │  │  Hooks  │  │ Skills  │  │ Plugins │    │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘    │
│  外部工具集成、确定性自动化、领域专业知识、打包扩展    │
├─────────────────────────────────────────────────────────┤
│                    DELEGATION LAYER                     │
│  ┌─────────────────────────────────────────────────┐    │
│  │              Subagents (最多10个并行)            │    │
│  │   探索型 | 规划型 | 通用型 | 自定义              │    │
│  └─────────────────────────────────────────────────┘    │
│  隔离上下文专注工作，仅返回摘要给主代理                │
├─────────────────────────────────────────────────────────┤
│                      CORE LAYER                         │
│  ┌─────────────────────────────────────────────────┐    │
│  │              主代理循环 + 共享上下文             │    │
│  │   200K tokens(标准) | 1M tokens(Opus 4.6)       │    │
│  └─────────────────────────────────────────────────┘    │
│  主对话、任务协调、最终决策                            │
└─────────────────────────────────────────────────────────┘
```

#### 核心层（Core Layer）

- 主代理循环和共享上下文窗口

- 8 个核心内置工具

- 上下文压缩管道

- 标准 200K tokens 上下文，Opus 4.6 支持 1M tokens

#### 委托层（Delegation Layer）

- 子代理系统，最多支持 10 个并行子代理

- 每个子代理拥有独立的干净上下文窗口

- 完成任务后仅返回摘要给主代理，避免主上下文膨胀

#### 扩展层（Extension Layer）

- Hooks：事件驱动的脚本

- Skills：封装可复用的专业知识

- Plugins：打包可分发的扩展包

- MCP 服务器：连接外部工具和数据源

### 2.2 五层系统架构（技术视角）

基于源代码分析的技术实现划分：

```Plain Text
┌─────────────────────────────────────────────────────────┐
│                    SURFACE LAYER                        │
│  交互式CLI | 无头CLI | Agent SDK | IDE集成 | 桌面应用  │
│  负责用户输入和结果展示                                │
├─────────────────────────────────────────────────────────┤
│                      CORE LAYER                         │
│  代理循环 | 五层上下文压缩管道 | 状态机                │
│  核心决策和迭代逻辑                                    │
├─────────────────────────────────────────────────────────┤
│                  SAFETY/ACTION LAYER                   │
│  权限系统 | Auto模式分类器 | Hook管道 | 工具池 | 沙箱  │
│  安全控制和行动执行                                    │
├─────────────────────────────────────────────────────────┤
│                     STATE LAYER                        │
│  上下文组装 | 运行时状态 | 会话持久化 | CLAUDE.md | 记忆│
│  数据管理和状态维护                                    │
├─────────────────────────────────────────────────────────┤
│                    BACKEND LAYER                       │
│  执行后端 | 外部资源访问 | API通信 | MCP客户端         │
│  底层基础设施和外部交互                                │
└─────────────────────────────────────────────────────────┘
```

#### 表面层（Surface Layer）

- 交互式 CLI、无头 CLI、Agent SDK

- IDE 集成、桌面应用、浏览器扩展

- 负责用户交互和结果展示

#### 核心层（Core Layer）

- 核心代理循环

- 五层上下文压缩管道

- 会话状态管理

#### 安全与行动层（Safety/Action Layer）

- 权限系统、Auto 模式分类器

- Hook 管道、工具池

- Shell 沙箱、子代理生成器

#### 状态层（State Layer）

- 上下文组装、运行时状态

- 会话持久化、[CLAUDE.md](CLAUDE.md) 系统

- 自动记忆、Sidechain 转录

#### 后端层（Backend Layer）

- 工具实现、Shell 执行器

- MCP 客户端、API 适配器

- 远程执行支持、LSP 集成

### 2.3 核心子系统

#### 五层上下文压缩管道

每次模型调用前自动运行，解决上下文窗口有限的问题：

1. **预算减少**：计算可用 token 预算，预留系统空间

2. **剪切**：剪切过长的工具输出，保留开头结尾

3. **微压缩**：删除冗余字符，压缩格式

4. **上下文折叠**：合并相关消息为摘要

5. **自动压缩**：使用 Claude 模型生成历史摘要

#### 七层安全防御体系

深度防御策略，工具请求必须通过所有七层：

1. 工具预过滤

2. 拒绝优先规则评估

3. 权限模式约束

4. Auto 模式分类器

5. Shell 沙箱检查

6. 会话恢复检查

7. PreToolUse Hook 拦截

### 2.4 完整架构流程图

#### 端到端执行流程

```mermaid
flowchart TD
    A[用户输入] --> B[表面层 Surface Layer]
    B --> C[状态层 State Layer]
    C --> D[核心层 Core Layer]
    D --> E{有工具调用吗?}
    E -->|否| F[返回结果给用户]
    E -->|是| G[安全与行动层 Safety/Action Layer]
    G --> H{安全检查通过?}
    H -->|否| I[返回拒绝原因]
    I --> D
    H -->|是| J[执行工具]
    J --> K{工具类型?}
    K -->|内置工具| L[直接执行]
    K -->|MCP工具| M[调用MCP服务器]
    K -->|Task工具| N[生成子代理]
    L --> O[工具执行结果]
    M --> O
    N --> O
    O --> P[状态层: 更新会话状态]
    P --> D```

## 三、核心使用

### 3.1 基本启动方式

​```bash
# 在当前项目目录启动
claude

# 直接执行任务
claude "修复auth.test.ts中失败的测试"

# 管道输入
tail -200 app.log | claude -p "分析这些日志中的错误"
```

### 3.2 内置工具集

|工具|功能|
|---|---|
|`Bash`|执行 shell 命令（最强大的通用工具）|
|`Read`|读取文件内容|
|`Edit`|对文件进行精确编辑|
|`Write`|创建新文件或覆盖现有文件|
|`Grep`|使用正则表达式搜索文件内容（基于 ripgrep）|
|`Glob`|根据模式匹配查找文件|
|`Task`|生成子代理执行独立任务|
|`TodoWrite`|跟踪任务进度|

### 3.3 常用斜杠命令

- `/help` - 查看帮助

- `/init` - 初始化项目，自动生成 [CLAUDE.md](CLAUDE.md) 文件

- `/model` - 切换使用的模型（Opus, Sonnet, Haiku）

- `/cost` - 查看当前会话的费用

- `/memory` - 管理自动记忆

- `/review` - 发起代码审查

- `/mcp` - 管理 MCP 服务器连接

- `/sandbox` - 启用沙箱模式（Linux/WSL2）

- `/loop` - 定时重复执行任务

- `/batch` - 批量处理文件

### 3.4 快捷键

- `Ctrl+C` - 中断当前操作

- `Ctrl+O` - 查看 Claude 的详细思考过程

- `Shift+Tab` - 快速切换权限模式

- `双击Esc` - 撤销上一次文件修改

- `Shift+Enter` - 多行输入

## 四、工作原理

### 4.1 核心：代理循环

Claude Code 的核心是一个极其简单的`while`循环：

```typescript
while (claude_response.has_tool_call) {
    result = execute_tool(tool_call);
    claude_response = send_to_claude(result);
}
return claude_response.text;
```

循环分为三个阶段，不断迭代直到任务完成：

1. **收集上下文**：扫描项目、读取文件、加载配置

2. **采取行动**：分析上下文，规划执行路径，调用工具

3. **验证结果**：检查行动效果，决定下一步操作

### 4.2 上下文管理

Claude Code 采用**代理式搜索**，不使用传统 RAG：

- 直接在本地机器上操作实时代码库

- 像人类工程师一样遍历文件系统、读取文件、搜索

- 不需要预先构建和维护代码库索引

- 始终使用最新的代码状态

### 4.3 安全机制

Claude Code 采用**深度防御**策略，多层安全机制相互配合：

1. 权限系统：分级控制工具执行权限

2. Auto 模式分类器：AI 驱动的风险评估

3. 输入层防护：扫描提示注入尝试

4. Shell 沙箱：隔离执行命令

5. 可逆性：所有文件修改都可以一键撤销

6. 审计日志：完整记录所有操作

## 五、扩展生态

### 5.1 常用 Skills 推荐

#### 官方必备 Skills

|Skill 名称|核心功能|安装方法|
|---|---|---|
|**frontend-design**|生成非 "AI 风格" 的高质量 UI，支持 50 种设计风格|预装|
|**mcp-builder**|交互式生成生产级 MCP 服务器|预装|
|**skill-creator**|对话式创建自定义 Skills|预装|
|**batch**|跨代码库并行执行大规模变更|预装|
|**everything-claude-code**|完整工程化技能体系|`/plugin marketplace add anthropics/everything-claude-code`|

#### 通用开发 Skills

- **planning-with-files**：Manus 风格的持久化任务规划系统

- **code-refactoring**：结构化重构，遵循 SOLID 原则

- **conventional-commits**：自动生成符合规范的提交信息

- **test-harness**：自动生成全面的测试套件

- **debugger-agent**：专业的调试助手

#### 前端开发 Skills

- **react-best-practices**：Vercel 官方推荐的 React 最佳实践

- **vue3-guidelines**：Vue 3 和 Composition API 指南

- **tailwind-expert**：Tailwind CSS 高级技巧

- **web-quality-skills**：Web 质量检查，覆盖性能、可访问性、SEO

#### 后端与数据 Skills

- **api-design**：RESTful API 和 GraphQL 设计最佳实践

- **sql-optimizer**：SQL 查询优化专家

- **pandas-pro**：Pandas 高级数据处理

- **dbt-transformation-patterns**：官方 DBT 转换模式

#### DevOps 与基础设施 Skills

- **hashicorp-agent-skills**：Terraform、Vault 等官方技能

- **docker-expert**：Docker 容器化和镜像优化

- **kubernetes-guide**：Kubernetes 部署和管理

- **github-actions**：GitHub Actions 工作流创建

#### 文档与协作 Skills

- **readme-generator**：自动生成完整的 [README.md](README.md)

- **changelog-generator**：从 Git 历史生成更新日志

- **pr-description**：自动生成 Pull Request 描述

- **code-reviewer**：结构化的代码审查

#### 安全与质量 Skills

- **security-scan**：基于 OWASP 的漏洞扫描

- **trail-of-bits-security**：运行 CodeQL 和 Semgrep 分析

- **secrets-detector**：检测硬编码密钥

- **dependency-audit**：检查依赖漏洞

### 5.2 常用 MCP 服务器推荐

#### 官方必备 MCP

|MCP 名称|核心功能|安装命令|
|---|---|---|
|**Filesystem**|安全的文件系统访问|`claude mcp add filesystem`|
|**Git**|完整的 Git 操作|`claude mcp add git`|
|**GitHub**|管理仓库、PR、Issues|`claude mcp add --transport http github https://api.githubcopilot.com/mcp/`|
|**PostgreSQL**|连接和查询 PostgreSQL 数据库|`claude mcp add postgres`|
|**Brave Search**|隐私优先的网页搜索|`claude mcp add brave-search`|
|**Fetch**|获取网页内容并转换为 Markdown|`claude mcp add fetch`|
|**Time**|获取当前时间，时区转换|`claude mcp add time`|
|**Sequential Thinking**|多步骤推理辅助|`claude mcp add sequential-thinking`|

#### 开发工具类 MCP

- **Semgrep**：静态代码分析，漏洞检测

- **Prisma**：管理 Prisma 数据库模式和迁移

- **Docker**：管理 Docker 容器和镜像

- **Figma Dev Mode**：从 Figma 设计文件生成代码

- **Magic MCP**：生成高质量 UI 组件

#### 数据库类 MCP

- **MySQL/MariaDB**：完整的 MySQL 支持

- **SQLite**：本地数据库操作

- **MongoDB**：NoSQL 数据库查询

- **Redis**：缓存和键值存储操作

- **Supabase**：完整的 Supabase 平台集成

#### 云服务与 DevOps 类 MCP

- **AWS**：完整的 AWS 服务集成

- **Azure**：Microsoft Azure 服务

- **Vercel**：一键部署到 Vercel 平台

- **Kubernetes**：管理 Kubernetes 集群

- **Terraform**：基础设施即代码

- **Sentry**：错误跟踪和性能监控

#### 生产力与协作类 MCP

- **Slack**：发送消息，读取频道

- **Linear**：问题跟踪和项目管理

- **Notion**：读取和写入 Notion 页面

- **Jira**：Atlassian Jira 问题跟踪

- **Fireflies**：会议转录和分析

#### 知识与搜索类 MCP

- **Context7**：获取最新的库文档和代码示例

    ```bash
    claude mcp add --transport http context7 https://mcp.context7.com/mcp
    ```

- **Playwright**：浏览器自动化，测试，网页抓取

- **Memory MCP**：跨会话持久化记忆

### 5.3 常用 CLI 工具

#### 官方核心 CLI 命令

```bash
# 项目管理
claude init              # 初始化项目，生成CLAUDE.md
claude config            # 查看和修改配置
claude status            # 查看当前会话状态
claude doctor            # 全面检查安装和配置
claude update            # 更新Claude Code到最新版本

# 代码操作
claude "任务描述"        # 直接执行任务
claude -p "提示词"       # 非交互式模式
claude review            # 代码审查
claude refactor          # 重构代码
claude test              # 运行测试

# MCP管理
claude mcp add           # 添加MCP服务器
claude mcp list          # 列出已安装的MCP服务器
claude mcp remove        # 移除MCP服务器
claude mcp restart       # 重启MCP服务器

# 会话管理
claude history           # 查看会话历史
claude resume            # 恢复上次会话
claude fork              # 分叉当前会话
claude export            # 导出会话历史
```

#### 社区 CLI 工具

- **claude-code-router**：基于任务复杂度自动切换模型，优化成本

- **claude-code-action**：在 GitHub Actions 中运行 Claude Code

- **mcptools**：MCP 服务器管理工具

## 六、高级配置

### 6.1 配置文件优先级

- **项目级配置**：`./.claude/settings.json`（优先级最高，仅当前项目生效）

- **全局根配置**：`~/.claude.json`（所有项目生效）

### 6.2 全局配置文件（~/.claude.json）

#### 极简必备配置

```json
{
  "hasCompletedOnboarding": true,
  "autoUpdates": true,
  "autoUpdatesChannel": "stable",

  "env": {
        "CLAUDE_CODE_ATTRIBUTION_HEADER": "0",
        "CLAUDE_CODE_ENABLE_TELEMETRY": "0",
        "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1"
  },

  "permissions": {
    "defaultMode": "auto"
  }
}
```

#### 完整版全能配置

```json
{
  "hasCompletedOnboarding": true,
  "theme": "dark",
  "verbose": false,
  "autoUpdates": true,
  "autoUpdatesChannel": "stable",

  "cacheEnabled": true,
  "contextCompression": true,
  "parallelTasksCount": 3,

  "env": {
    "CLAUDE_CODE_ATTRIBUTION_HEADER": "0",
    "CLAUDE_CODE_ENABLE_TELEMETRY": "0",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1"
  },

  "permissions": {
    "defaultMode": "auto",
    "autoMode": {
      "enabled": true,
      "environment": "development"
    }
  }
}
```

### 6.3 核心配置项详解

#### 跳过首次引导

```json
"hasCompletedOnboarding": true
```

**作用**：永久跳过 Claude 启动引导教程，直接进入工作界面。

#### CCH 问题修复

**CCH（Claude Code Attribution Header）**：默认的随机归因头导致缓存失效，表现为卡顿、慢、费 token。

```json
"CLAUDE_CODE_ATTRIBUTION_HEADER": "0"
```

**注意**：必须是字符串`"0"`，不能写数字`0`。

#### 全局代理配置

```json
"HTTP_PROXY": "http://127.0.0.1:7890",
"HTTPS_PROXY": "http://127.0.0.1:7890"
```

**作用**：永久设置全局代理，无需每次终端手动设置。

#### 权限模式配置

|模式|说明|
|---|---|
|`auto`|AI 自动判断是否执行（推荐，平衡安全 + 效率）|
|`acceptEdits`|自动执行文件修改|
|`default`|每次操作手动确认|

#### 其他优化配置

- `autoUpdates`: 自动更新 Claude

- `theme: dark`: 深色模式

- `cacheEnabled: true`: 开启本地缓存

- `parallelTasksCount: 3`: 最大并行子任务数

- `CLAUDE_CODE_ENABLE_TELEMETRY": "0"`: 关闭遥测

### 6.4 版本升级指南

#### Windows CMD

```cmd
:: 设置代理
set HTTPS_PROXY=http://127.0.0.1:7890
set HTTP_PROXY=http://127.0.0.1:7890
:: 升级
claude update
```

#### Windows PowerShell

```powershell
# 设置代理
$env:HTTPS_PROXY = "http://127.0.0.1:7890"
$env:HTTP_PROXY = "http://127.0.0.1:7890"
# 升级
claude update
```

#### macOS / Linux

```bash
# 设置代理
export HTTPS_PROXY=http://127.0.0.1:7890
export HTTP_PROXY=http://127.0.0.1:7890
# 升级
claude update
```

#### 强制升级

```bash
claude update --force
```

#### 包管理器升级

```bash
# macOS Homebrew
brew upgrade --cask claude-code

# Windows Winget
winget upgrade Anthropic.ClaudeCode
```

### 6.5 生效方法

1. 修改配置文件后**重启 Claude 终端会话**

2. 验证配置加载：

```bash
claude config list
```

## 七、性能优化

1. **少而精**：不要安装太多 Skills 和 MCP 服务器，建议保持 10 个以内核心扩展

2. **项目级配置**：项目特定的配置放在项目目录，避免全局污染

3. **定期更新**：定期更新 Claude 和扩展，获取最新优化

4. **缓存优化**：开启 CCH 修复，提升缓存命中率到 95% 以上

5. **权限模式**：使用`auto`模式，平衡安全与效率

## 核心知识点速览

- Claude Code 核心是代理循环，通过工具调用自主完成开发任务

- 配置`"CLAUDE_CODE_ATTRIBUTION_HEADER": "0"`可修复 CCH 缓存问题，大幅提升速度

- 权限模式推荐使用`auto`，平衡安全与效率

- 扩展生态包含 Skills、MCP 服务器，可大幅扩展 Claude 能力

- 项目级配置优先级高于全局配置，可实现项目隔离

- 全局配置可永久写入代理，无需每次终端手动设置

- 子代理系统可解决上下文有限问题，支持最多 10 个并行任务

- 七层安全防御体系保障工具执行安全

- 五层上下文压缩管道高效利用有限的上下文窗口

- 升级时可通过代理配置解决网络访问问题
