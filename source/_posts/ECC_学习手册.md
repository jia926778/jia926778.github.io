---
title: Everything Claude Code (ECC) 学习手册
date: 2026-05-16 21:30:00
tags: [Claude Code plugin, 技术分享, ECC, Everything Claude Code, AI 编程, cc插件]
categories: 工具使用
toc: true
---
# Everything Claude Code (ECC) 学习手册

## 文档核心说明

本文档适用于希望系统学习 Everything Claude Code (ECC) 技术的开发者，涵盖从入门到进阶的完整知识体系。

**覆盖核心知识模块**：

* 基础入门：项目概述、前提条件、三种官方安装方式、配置优化、常见问题

* 核心架构：代理、技能、规则、钩子、MCP 集成的完整定义与实现

* 基础使用：75+ 常用命令、快捷键、实用技巧、标准工作流示例

* 工作流编排：编排原语、执行引擎、高级特性、Hermes Operator v2.0

* 底层原理：Claude Code 扩展机制、代理委派、上下文管理

* 设计模式：架构模式、结构模式、行为模式、AI 原生设计模式

* 工具对比：与原生 Claude Code、Cursor、Codex、OpenCode、GitHub Copilot 的对比

* 自定义开发：代理开发、技能开发、工作流定义、测试调试

* 最佳实践：Token 优化、安全注意事项、适用场景、局限性

***

## 一、基础入门

### 1.1 项目概述

Everything Claude Code（简称 **ECC**）是由 Anthropic 黑客松冠军 Affaan Mustafa 创建的**生产级 AI 代理性能优化系统**，最初诞生于 2025 年 9 月的 Anthropic x Forum Ventures 黑客松，作者团队仅用 8 小时就使用这套配置完全构建了 zenith.chat 并夺冠。

它**不是一个独立的 AI 编程工具**，而是为 Claude Code、Cursor、Codex、OpenCode 等主流 AI 编程工具量身定制的 "高级改装套件"，将个人级的 AI 编程助手体验提升为**团队级、可积累、可治理的 AI 开发基础设施**。

截至 2026 年 5 月，ECC 已迭代至 **v2.0.0-rc.1** 版本，在 GitHub 上获得了超过 140k stars 和 21k forks，是目前最受欢迎的 AI 编程增强工具。

**核心价值**：

* 将通用 AI 助手转化为专业的开发专家

* 统一团队开发规范和最佳实践

* 自动化重复的开发流程，提升效率

* 可积累的技能库，持续提升团队能力

* 跨工具支持，一套配置适配所有主流 AI 编程环境

### 1.2 前提条件

在安装 ECC 之前，请确保你的环境满足以下要求：

* Claude Code CLI **v2.1.0+** 或 Claude Code 桌面版

* Anthropic API Key（从 [console.anthropic.com](https://console.anthropic.com) 获取）

* Node.js **18+**

* Git

* 至少 1GB 可用磁盘空间（用于存储配置和缓存）

### 1.3 安装与配置

ECC 提供三种官方安装方式，**严禁叠加使用多种安装方式**，这是最常见的安装错误原因。

#### 方式一：插件安装（官方推荐）

这是最稳定、最推荐的安装方式：

```
# 1. 添加 ECC 插件市场

/plugin marketplace add https://github.com/affaan-m/everything-claude-code

# 2. 安装 ECC 插件

/plugin install ecc@ecc
```

**重要：插件无法自动分发规则**，需要手动复制你需要的规则：

```
# 克隆仓库

git clone https://github.com/affaan-m/everything-claude-code.git

cd everything-claude-code

# 复制通用规则和你使用的语言规则（不要复制所有规则）

mkdir -p ~/.claude/rules/ecc

cp -r rules/common ~/.claude/rules/ecc/

cp -r rules/typescript ~/.claude/rules/ecc/  # 根据你的技术栈选择

cp -r rules/python ~/.claude/rules/ecc/
```

**验证安装**：

```
/ecc:plan "测试安装是否成功"
```

#### 方式二：脚本安装（手动控制）

适用于需要完全控制安装内容的用户：

```
# macOS/Linux

./install.sh --profile standard --target claude

# Windows PowerShell

.\install.ps1 --profile standard --target claude

# 或使用 npx

npx ecc-install --profile standard --target claude
```

**可用配置文件**：

* `minimal`：仅核心规则（适合低资源环境）

* `standard`：核心规则 + 基础技能 + 代理（推荐大多数用户）

* `full`：全量安装（包含所有实验性功能）

#### 方式三：完全手动安装

适用于需要深度定制的用户：

```
# 1. 克隆仓库

git clone https://github.com/affaan-m/everything-claude-code.git

cd everything-claude-code

# 2. 安装依赖

npm install

# 3. 复制代理

cp agents/*.md ~/.claude/agents/

# 4. 复制规则

mkdir -p ~/.claude/rules/ecc

cp -r rules/common ~/.claude/rules/ecc/

cp -r rules/typescript ~/.claude/rules/ecc/

# 5. 复制技能

mkdir -p ~/.claude/skills/ecc

cp -r .agents/skills/* ~/.claude/skills/ecc/

# 6. 安装钩子运行时

./install.sh --target claude --modules hooks-runtime
```

#### 配置优化（推荐）

在 `~/.claude/settings.json` 中添加以下配置，优化 token 成本和自动行为：

```
{

  "autoCompact": true,

  "compactThreshold": 10000,

  "maxTokensPerRequest": 8192,

  "defaultModel": "claude-3-5-sonnet-20240620",

  "autoApprove": {

    "readFiles": true,

    "writeFiles": true,

    "runCommands": false,

    "installPackages": false

  },

  "env": {

    "MAX_THINKING_TOKENS": "10000",

    "CLAUDE_AUTOCOMPACT_PCT_OVERRIDE": "50"

  }

}
```

#### 常见安装问题

**问题 1：权限被拒绝**

```
# 解决方案：修复目录权限

sudo chown -R $USER ~/.claude/
```

**问题 2：Node.js 版本过低**

```
# 解决方案：升级 Node.js

nvm install 18

nvm use 18
```

**问题 3：Claude Code 版本不兼容**

```
# 解决方案：升级 Claude Code

npm update -g @anthropic-ai/claude-code
```

**问题 4：网络连接失败**

```
# 解决方案：使用代理

export https_proxy=http://127.0.0.1:7890

curl -fsSL https://raw.githubusercontent.com/affaan-m/everything-claude-code/main/scripts/install.sh | bash
```

**问题 5：重复钩子错误**

* 不要在 `.claude-plugin/plugin.json` 中添加 `"hooks"` 字段

* Claude Code v2.1+ 会自动加载插件的 `hooks/hooks.json`

* 手动删除 `~/.claude/settings.json` 中重复的 hooks 配置

**问题 6：卸载与重置**

```
# 1. 卸载插件

/plugin uninstall ecc@ecc

# 2. 删除 ECC 规则

rm -rf ~/.claude/rules/ecc/

# 3. 完全卸载（脚本安装）

node scripts/uninstall.js
```

***

## 二、核心架构

ECC 采用分层模块化设计，包含 6 大核心组件，形成了完整的 AI 开发工作流闭环：

| 模块                 | 数量   | 核心作用                                         | 技术实现            |
| ------------------ | ---- | -------------------------------------------- | --------------- |
| **Agents（专业代理）**   | 60+  | 针对不同任务的专业化子智能体，实现任务委派与并行处理                   | 上下文内角色切换        |
| **Skills（技能）**     | 228+ | 可重用的工作流模板和领域知识，覆盖主流技术栈                       | 提示模板 + YAML 元数据 |
| **Commands（斜杠命令）** | 75+  | 一键触发完整开发流程的快捷指令                              | 自定义命令 + 提示注入    |
| **Rules（规则）**      | 34+  | 强制执行的开发最佳实践，包含通用规则和 12 种语言专项规则               | 系统提示注入          |
| **Hooks（钩子）**      | 8 类  | 基于事件触发的自动化流程，实现开发过程的无人值守                     | 事件驱动脚本          |
| **MCP 配置**         | 14+  | 预配置 GitHub/Vercel/Supabase 等 20+ 服务的模块化能力提供者 | MCP 协议          |

### 2.1 代理系统

代理是 ECC 的核心执行单元，每个代理本质上是一个带有角色定义、工具集和模型配置的提示模板。

**核心代理列表**：

* **架构师 (architect)**：系统设计、可扩展性分析、技术选型决策

* **规划师 (planner)**：任务分解、开发计划生成、风险评估

* **TDD 导师 (tdd-guide)**：测试驱动开发全流程指导

* **代码审查员 (code-reviewer)**：多维度代码质量检查

* **安全审计员 (security-reviewer)**：漏洞扫描与风险评估

* **构建错误解决器 (build-error-resolver)**：自动定位和修复构建失败

* **数据库专家 (database-reviewer)**：数据库设计、查询优化、迁移指导

* **DevOps 专家 (devops-expert)**：CI/CD 配置、容器化、部署指导

* **语言专家**：TypeScript/Python/Go/Java/Rust/PHP 等 12 种语言的深度专项专家

**代理定义格式**：

```
---
name: code-reviewer
description: Reviews code for quality, security, and maintainability
tools: ["Read", "Grep", "Glob", "Bash"]
model: claude-3-5-sonnet-20240620
timeout: 300
---

你是一位资深代码审查员，必须从以下维度审查代码：
1. 代码可读性和命名规范
2. 代码可维护性和架构一致性
3. 性能优化和资源使用
4. 安全漏洞和错误处理
5. 测试覆盖率和测试质量
你必须为每个问题提供具体的修复建议，并给出严重程度评分。
```

### 2.2 技能系统

技能是 ECC 的 "知识库"，包含经过实战验证的工作流和最佳实践，是 ECC 的主要工作流表面。

**技能分类**：

* 开发工作流：TDD、敏捷开发、CI/CD 配置、Git 分支管理

* 技术栈知识：React/Vue/Angular、Django/FastAPI、Spring Boot、Node.js

* 工具使用：Docker/Kubernetes、AWS/GCP/Azure、数据库优化

* 项目管理：需求分析、任务分解、进度跟踪、文档生成

* 领域知识：机器学习、数据工程、移动开发、安全审计

**技能定义格式**：

```
---
name: tdd-workflow
description: Test-Driven Development workflow
agents: ["tdd-guide"]
phases:
  - name: write-tests
    description: Write failing tests
  - name: implement
    description: Implement minimal code to pass tests
  - name: refactor
    description: Refactor code while keeping tests passing
  - name: verify
    description: Verify test coverage
---

# TDD 工作流

当执行此技能时，你必须严格按照以下步骤进行：
1. 理解需求，明确功能边界
2. 编写第一个最简单的测试用例
3. 运行测试，确认测试失败（RED）
4. 编写最少的代码使测试通过（GREEN）
5. 重构代码，提高可读性和可维护性
6. 重复步骤 2-5 直到所有需求都被覆盖
7. 生成测试覆盖率报告，确保覆盖率≥80%
```

### 2.3 规则系统

规则是 ECC 的 "纪律系统"，通过系统提示注入强制 Claude 在所有任务中遵循最佳实践。

**规则分类**：

* **通用规则**：代码注释规范、错误处理原则、命名约定、提交信息格式

* **语言专项规则**：针对每种语言的编码风格、性能优化、安全最佳实践

* **项目规则**：可自定义团队专属的开发规范和项目要求

**规则执行机制**：

* 规则在代码生成前注入，而不是生成后检查

* 强制规则必须严格遵守，违反的代码会被自动拒绝

* 推荐规则强烈建议遵守，除非有特殊理由

### 2.4 钩子系统

钩子是 ECC 的 "自动化引擎"，基于事件触发自动执行预定义的操作。

**支持的事件类型**：

* `sessionStart`：会话开始时

* `sessionEnd`：会话结束时

* `beforeToolUse`：调用工具前

* `afterToolUse`：调用工具后

* `beforeFileWrite`：写入文件前

* `afterFileWrite`：写入文件后

* `beforeCommit`：提交代码前

* `afterCommit`：提交代码后

**常用钩子示例**：

```
{
  "hooks": {
    "afterFileWrite": [
      {
        "command": "prettier --write {{filePath}}",
        "description": "自动格式化代码",
        "include": ["*.ts", "*.tsx", "*.js", "*.jsx"]
      }
    ],
    "beforeCommit": [
      {
        "command": "npm run lint",
        "description": "运行代码检查",
        "failOnError": true
      }
    ]
  }
}
```

### 2.5 MCP 集成

ECC 预配置了 14+ 常用服务的 MCP（Model Context Protocol）服务器，允许 Claude 直接与这些服务交互。

**预配置的 MCP 服务**：

* GitHub：创建 Issue、PR、管理仓库

* Vercel：部署应用、查看部署状态

* Supabase：数据库操作、认证配置

* Exa：网络搜索、研究

* Playwright：端到端测试

* Context7：文档查询

**MCP 配置格式**：

```
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "YOUR_GITHUB_TOKEN"
      }
    }
  }
}
```

***

## 三、基础使用

### 3.1 基础使用流程

1. 进入项目目录：`cd your-project`

2. 启动 Claude Code：`claude`

3. 描述任务：`/ecc:plan "为这个项目添加用户认证功能"`

4. 审查计划：确认 Claude 生成的开发计划

5. 执行任务：Claude 会自动调用对应的代理和技能完成任务

6. 代码审查：`/code-review` 审查生成的代码

7. 测试验证：`/test` 运行测试用例

### 3.2 完整命令速查表

#### 核心管理命令

| 命令                        | 功能               | 常用示例                                |
| ------------------------- | ---------------- | ----------------------------------- |
| `/model <model-name>`     | 切换当前使用的模型        | `/model claude-3-5-sonnet-20240620` |
| `/clear`                  | 清空当前对话上下文        | `/clear`                            |
| `/compact`                | 手动压缩上下文          | `/compact`                          |
| `/cost`                   | 查看本次会话的 token 花费 | `/cost`                             |
| `/fork`                   | 分支当前对话           | `/fork`                             |
| `/checkpoints`            | 列出所有文件级撤销点       | `/checkpoints`                      |
| `/rewind <checkpoint-id>` | 恢复到指定的撤销点        | `/rewind 3`                         |
| `/save <name>`            | 保存当前会话为模板        | `/save auth-workflow`               |
| `/load <name>`            | 加载已保存的会话模板       | `/load auth-workflow`               |

#### 项目开发命令

| 命令                   | 功能          | 常用示例                                                         |
| -------------------- | ----------- | ------------------------------------------------------------ |
| `/ecc:plan <task>`   | 生成详细的开发计划   | `/ecc:plan "添加用户注册功能"`                                       |
| `/execute <plan-id>` | 执行指定的开发计划   | `/execute 1`                                                 |
| `/tdd <feature>`     | 启动 TDD 工作流  | `/tdd "实现密码重置功能"`                                            |
| `/refactor <scope>`  | 重构指定范围的代码   | `/refactor src/auth/`                                        |
| `/debug <error>`     | 自动分析并修复错误   | `/debug "TypeError: Cannot read property 'id' of undefined"` |
| `/build-fix`         | 自动定位并修复构建失败 | `/build-fix`                                                 |
| `/dep-fix`           | 自动解决依赖冲突    | `/dep-fix`                                                   |

#### 代码质量与安全命令

| 命令               | 功能          | 常用示例                           |
| ---------------- | ----------- | ------------------------------ |
| `/code-review`   | 执行全面的代码审查   | `/code-review src/components/` |
| `/security-scan` | 执行安全漏洞扫描    | `/security-scan --full`        |
| `/lint`          | 运行代码风格检查并修复 | `/lint`                        |
| `/format`        | 自动格式化所有代码   | `/format`                      |
| `/test`          | 运行所有测试用例    | `/test tests/auth/`            |
| `/test-coverage` | 生成测试覆盖率报告   | `/test-coverage`               |
| `/perf-analyze`  | 分析代码性能瓶颈    | `/perf-analyze src/api/`       |

#### 文档与协作命令

| 命令                | 功能                 | 常用示例                       |
| ----------------- | ------------------ | -------------------------- |
| `/docs`           | 为代码自动生成 API 文档     | `/docs src/models/User.ts` |
| `/readme`         | 生成或更新项目 README     | `/readme`                  |
| `/changelog`      | 生成变更日志             | `/changelog v1.2.0`        |
| `/pr-description` | 生成 Pull Request 描述 | `/pr-description`          |
| `/commit`         | 自动生成规范的提交信息        | `/commit`                  |

#### 部署与运维命令

| 命令                     | 功能                 | 常用示例                                   |
| ---------------------- | ------------------ | -------------------------------------- |
| `/deploy <platform>`   | 部署到指定平台            | `/deploy vercel`                       |
| `/ci-setup <provider>` | 配置 CI/CD 流水线       | `/ci-setup github-actions`             |
| `/dockerize`           | 自动生成 Docker 配置     | `/dockerize`                           |
| `/k8s-deploy`          | 生成 Kubernetes 部署配置 | `/k8s-deploy`                          |
| `/db-migrate <name>`   | 生成并执行数据库迁移         | `/db-migrate "add-user-avatar-column"` |

#### 代理调用命令

| 命令                            | 功能           | 常用示例                                   |
| ----------------------------- | ------------ | -------------------------------------- |
| `/architect <question>`       | 咨询系统架构师      | `/architect "设计一个高并发消息系统"`             |
| `/security-expert <question>` | 咨询安全专家       | `/security-expert "如何防止 SQL 注入"`       |
| `/db-expert <question>`       | 咨询数据库专家      | `/db-expert "优化这个慢查询"`                 |
| `/devops-expert <question>`   | 咨询 DevOps 专家 | `/devops-expert "配置 Kubernetes 自动扩缩容"` |
| `/frontend-expert <question>` | 咨询前端专家       | `/frontend-expert "优化 React 应用性能"`     |

#### 工作流编排命令

| 命令                               | 功能       | 常用示例                              |
| -------------------------------- | -------- | --------------------------------- |
| `/orchestrate <workflow> <task>` | 启动预定义工作流 | `/orchestrate feature "实现用户登录功能"` |
| `/multi-plan <task>`             | 多模型协同规划  | `/multi-plan "设计微服务架构"`           |
| `/multi-execute <plan>`          | 多模型协同执行  | `/multi-execute 1`                |
| `/quality-gate`                  | 执行质量门控检查 | `/quality-gate`                   |
| `/loop-start`                    | 启动自主循环执行 | `/loop-start`                     |

#### 插件管理命令

| 命令                              | 功能       | 常用示例                                                                         |
| ------------------------------- | -------- | ---------------------------------------------------------------------------- |
| `/plugin list`                  | 列出已安装的插件 | `/plugin list`                                                               |
| `/plugin install <plugin>`      | 安装插件     | `/plugin install ecc@ecc`                                                    |
| `/plugin update <plugin>`       | 更新插件     | `/plugin update ecc@ecc`                                                     |
| `/plugin uninstall <plugin>`    | 卸载插件     | `/plugin uninstall ecc@ecc`                                                  |
| `/plugin marketplace add <url>` | 添加插件市场   | `/plugin marketplace add https://github.com/affaan-m/everything-claude-code` |

### 3.3 实用快捷键

| 快捷键         | 功能                  |
| ----------- | ------------------- |
| `Tab`       | 切换思维链 (CoT) 显示 / 隐藏 |
| `Esc Esc`   | 立即中断当前任务            |
| `Shift+Tab` | 切换自动编辑模式            |
| `Ctrl+U`    | 删除整行输入              |
| `Ctrl+C`    | 取消当前输入或生成           |
| `Ctrl+D`    | 退出 Claude Code      |
| `Ctrl+R`    | 搜索历史命令              |
| `Ctrl+L`    | 清屏                  |

### 3.4 实用技巧

1. **Token 节省**：使用 `mgrep` 替代系统 `grep`，可减少约 50% 的 token 消耗

```
alias grep="mgrep"
```

1. **快速执行**：在命令后加 `--auto` 可自动批准所有安全操作

```
/tdd "实现登录功能" --auto
```

1. **指定模型**：在任何命令后加 `--model <model-name>` 临时使用特定模型

```
/security-scan --model claude-3-opus-20240229
```

1. **仅生成计划**：使用 `--dry-run` 参数只生成计划不执行

```
/refactor src/ --dry-run
```

1. **更新 ECC**：定期更新到最新版本获取新功能

```
/plugin update ecc@ecc
```

### 3.5 常用工作流示例

#### 新功能开发工作流

```
# 1. 生成开发计划
/ecc:plan "添加用户认证功能，支持邮箱密码和 OAuth2 登录"

# 2. 按照 TDD 模式实现功能
/tdd "实现邮箱密码登录功能"

# 3. 审查代码质量
/code-review

# 4. 安全审计
/security-scan

# 5. 运行测试
/test

# 6. 生成文档
/docs
```

#### Bug 修复工作流

```
# 1. 编写失败的测试用例
/tdd "修复用户登录时的空指针异常"

# 2. 实现修复代码
# 3. 验证测试通过
# 4. 代码审查
/code-review

# 5. 提交代码
/commit
```

#### 代码重构工作流

```
# 1. 生成重构计划
/ecc:plan "将用户模块重构为微服务架构"

# 2. 执行重构
/refactor src/user/

# 3. 运行测试确保功能不变
/test

# 4. 代码审查
/code-review
```

#### 生产部署准备工作流

```
# 1. 安全审计
/security-scan --full

# 2. 运行所有测试
/test

# 3. 生成测试覆盖率报告
/test-coverage

# 4. 生成变更日志
/changelog v1.2.0

# 5. 部署到生产环境
/deploy vercel
```

***

## 四、工作流编排

### 4.1 编排机制概述

ECC 的工作流编排**不是**基于传统的工作流引擎，而是一个**纯提示工程驱动的编排层**，完全构建在 Claude Code 官方提供的扩展机制之上。

它的核心技术哲学是：**不重新发明轮子，而是将 Claude 本身的推理能力转化为一个可编程的工作流执行引擎**。

### 4.2 核心编排原语

ECC 的工作流编排系统由 5 个核心原语构成：

#### 1. 代理 (Agents)

工作流的基本执行单元，负责完成特定类型的任务。每个代理可以指定独立的模型、工具集和超时时间。

#### 2. 技能 (Skills)

可重用的工作流模板，包含经过实战验证的分步工作流描述。技能可以被代理调用，也可以直接作为独立的工作流执行。

#### 3. 命令 (Commands)

用户与 ECC 工作流系统交互的入口。每个命令对应一个工作流定义，用户通过输入斜杠命令来启动对应的工作流。

#### 4. 状态 (State)

ECC 使用 **SQLite 数据库** 来存储工作流的状态信息，包括工作流实例 ID、当前执行阶段、已完成的步骤、代理间的交接文档等。

#### 5. 钩子 (Hooks)

事件驱动的自动化引擎，允许在工作流执行的特定事件点自动执行预定义的操作。

### 4.3 工作流执行引擎

ECC 的工作流执行引擎完全运行在 Claude 的推理过程中，整个执行流程分为 6 个阶段：

1. **任务接收与分析**：解析命令参数，确定工作流类型和任务描述

2. **工作流实例化**：创建工作流实例，分配唯一 ID，初始化执行阶段

3. **代理调用与上下文传递**：将当前工作流状态和交接文档注入到代理的上下文中

4. **工具调用与结果处理**：验证工具调用权限，执行工具调用，将结果返回给代理

5. **阶段转换与质量门控**：检查代理输出是否符合质量标准，生成交接文档，启动下一个代理

6. **结果整合与报告生成**：整合所有代理的输出，生成结构化的最终报告

### 4.4 高级编排特性

#### 1. 条件分支与循环

ECC 支持基于中间结果的条件分支和循环逻辑：

```
## 条件分支
如果测试覆盖率≥80%：
  → 进入代码审查阶段
否则：
  → 返回 TDD 阶段，补充测试用例

## 修复循环
当存在未解决的代码审查问题时：
1. 让开发者修复问题
2. 重新运行代码审查
3. 重复直到所有问题都解决
```

#### 2. 嵌套工作流

一个工作流可以作为另一个工作流的一个阶段：

```
## 微服务开发工作流
1. 架构设计阶段：调用 `/orchestrate architect`
2. 服务实现阶段：为每个服务调用 `/orchestrate feature`
3. 集成测试阶段：调用 `/orchestrate integration-test`
4. 部署阶段：调用 `/orchestrate deploy`
```

#### 3. 多模型协同编排

ECC 支持多模型协同开发，不同的任务可以分配给最适合的模型：

```
## 多模型协同工作流
1. **研究阶段**：Claude 3.5 Sonnet（通用推理）
2. **后端开发**：OpenAI Codex（后端代码生成）
3. **前端开发**：Google Gemini（UI/UX 设计）
4. **代码审查**：Claude 3 Opus（深度审查）
5. **安全审计**：Claude 3 Opus（安全分析）
```

#### 4. 分布式并行执行

对于大型项目，ECC 支持基于 **tmux** 和 **git worktree** 的分布式并行执行：

* 为每个代理创建独立的 git 工作树

* 在独立的 tmux 窗格中启动代理会话

* 主编排器监控所有代理的执行状态

* 自动合并代理的修改并解决冲突

#### 5. 人工介入点

ECC 支持在工作流的关键节点设置人工介入点：

```
## 人工介入点：架构设计审查
在开始实现之前，请用户审查架构设计方案：
- [ ] 批准架构设计
- [ ] 请求修改
- [ ] 取消工作流
用户批准后，工作流将继续执行。
```

### 4.5 状态管理与错误处理

#### SQLite 状态存储

ECC 使用 SQLite 数据库作为状态存储，包含以下表：

* `workflows`：工作流实例信息

* `phases`：工作流阶段信息

* `agents`：代理执行记录

* `handoffs`：代理间交接文档

* `tool_calls`：工具调用历史

* `user_interactions`：用户交互记录

#### 会话持久化与恢复

ECC 会自动保存会话状态，当用户重新打开 Claude Code 时，会自动检测到未完成的工作流并恢复执行。

#### 错误检测与自动重试

* **工具调用错误**：自动重试最多 3 次，重试失败则暂停工作流

* **模型调用错误**：自动切换到备用模型，或请求用户干预

* **超时错误**：延长超时时间或终止任务

* **验证错误**：要求代理重新生成输出

#### 回滚机制

ECC 支持工作流的回滚操作，可以将项目恢复到任意之前的状态：

```
/rewind <workflow-id> <phase-id>
```

### 4.6 Hermes Operator（v2.0 新特性）

ECC v2.0.0-rc.1 引入了 **Hermes Operator**，这是一个全新的工作流编排控制平面，提供了以下高级功能：

1. **跨会话 / 跨工作树编排**：管理多个 Claude Code 会话和 git 工作树，实现真正的分布式并行执行

2. **可视化仪表板**：基于 Tkinter 的桌面应用，提供工作流状态监控、代理执行进度查看、token 消耗统计等功能

3. **持续学习与技能进化**：自动从成功的工作流中提取可重用的模式，转化为新的技能

4. **状态快照**：将本地状态存储导出为可移植的交接文档

5. **Rust 控制平面**：用 Rust 重写核心编排逻辑，提供更好的性能和可靠性

**启动 Hermes 仪表板**：

```
npm run dashboard
# 或
python3 ./ecc_dashboard.py
```

***

## 五、底层原理

### 5.1 ECC 的本质

ECC**不是**一个独立的 AI 编程工具，也不是 Claude Code 的 fork 版本，而是一个**纯配置驱动的增强层**。它完全基于 Claude Code 官方提供的扩展机制构建，没有修改 Claude Code 的任何二进制代码。

ECC 的安装过程本质上就是将它的配置文件复制到 Claude Code 的配置目录中。整个过程没有编译、没有二进制文件，完全是文本文件的复制。

### 5.2 Claude Code 原生扩展机制

ECC 的所有功能都构建在 Claude Code 提供的三个核心扩展点之上：

#### 1. 系统提示注入

Claude Code 在启动时会自动读取 `~/.claude/rules/` 目录下的所有 `.md` 文件，并将它们的内容**追加到系统提示的末尾**。这是 ECC 最核心的注入点。

**系统提示分层结构**：

1. Claude Code 原生系统提示（不可修改）

2. ECC 基础规则层（通用行为规范）

3. 代理定义层（60+ 专业代理的角色描述）

4. 技能定义层（228+ 技能的工作流描述）

5. 命令定义层（75+ 斜杠命令的实现）

6. 语言专项规则层（12 种语言的编码规范）

7. 项目自定义规则层（项目级配置）

#### 2. 自定义斜杠命令

Claude Code 允许用户在 `~/.claude/settings.json` 中定义自定义斜杠命令。当用户输入一个斜杠命令时，Claude Code 会自动将其替换为预定义的提示文本。

#### 3. 钩子系统

Claude Code 提供了基于事件的钩子机制，允许在特定事件发生时自动执行预定义的操作。

### 5.3 代理委派系统的实现原理

ECC 的代理委派完全在**单个 Claude 会话内部**完成，没有外部 API 调用，也没有多个模型实例。

**代理委派流程**：

```
用户输入任务
    ↓
主代理分析任务类型
    ↓
匹配最适合的专业代理
    ↓
在当前上下文中注入该代理的角色提示
    ↓
告诉 Claude："现在你是 XX 代理，请按照 XX 规则执行任务"
    ↓
Claude 切换角色并执行任务
    ↓
任务完成后，主代理切换回原角色
    ↓
主代理整合结果并返回给用户
```

**关键创新**：上下文内角色切换技术，零延迟、低开销、保留完整上下文。

### 5.4 上下文管理与优化技术

#### 智能上下文压缩

ECC 实现了一个基于语义的上下文压缩算法：

* 保留所有的系统规则和代理定义

* 保留最近的 10 轮对话

* 压缩较早的对话，只保留关键信息

* 完全删除无关的信息（如调试输出、错误信息）

#### Token 优化策略

* 使用 `mgrep` 替代系统 `grep`，只返回最相关的上下文

* 自动截断长文件，只读取与当前任务相关的部分

* 使用缩写和简洁的表达方式

* 避免重复信息

#### MCP 上下文管理

* 每个 MCP 工具描述都会消耗 token，过多的 MCP 会将 200k 上下文减少到～70k

* 建议保持不超过 10 个 MCP 服务器和 80 个工具处于激活状态

* 使用 `/mcp` 命令禁用不需要的 MCP 服务器

***

## 六、ECC 设计模式

### 6.1 核心设计哲学

1. **纯配置驱动**：不修改 Claude Code 的任何二进制代码

2. **LLM 作为执行引擎**：将 Claude 本身的推理能力转化为可编程的工作流执行器

3. **质量内置而非事后检查**：通过前置约束和自动化流程确保输出质量

### 6.2 架构模式

#### 1. 四层扩展架构

* **Hooks 层**：安全验证、自动化、事件响应

* **Skills 层**：可重用知识、工作流模板

* **Agents 层**：角色专业化、任务执行

* **Commands 层**：用户交互入口、工作流触发器

#### 2. 纯配置驱动架构

所有功能都通过文本文件定义，没有编译步骤，没有二进制文件。

#### 3. 控制平面 / 数据平面分离架构（v2.0）

* **控制平面**：Hermes Operator（Rust 编写），负责工作流编排、状态管理、资源调度

* **数据平面**：多个 Claude Code 会话，负责实际的代码生成和任务执行

#### 4. 插件化架构

允许用户将一组相关的代理、技能、命令和钩子打包成一个插件，方便分发和安装。

### 6.3 结构模式

#### 1. 代理模式

每个专业代理都是 Claude 通用能力的一个 "代理"，专注于特定类型的任务。

#### 2. 组合模式

简单的技能和代理可以组合成更复杂的工作流。

#### 3. 装饰器模式

通过规则和钩子来 "装饰"Claude Code 的基础功能，在不修改核心代码的情况下增强其能力。

#### 4. 外观模式

提供统一的斜杠命令接口，隐藏了内部复杂的编排逻辑。

#### 5. 享元模式

共享系统提示和技能定义，优化 token 消耗。

### 6.4 行为模式

#### 1. 策略模式

根据任务类型动态选择最适合的代理、模型和技能。

#### 2. 观察者模式

钩子系统是观察者模式的典型实现，当特定事件发生时，所有注册的钩子都会被通知。

#### 3. 状态模式

工作流执行引擎是一个有限状态机，工作流在不同的状态之间转换。

#### 4. 命令模式

斜杠命令系统将用户的请求封装为一个命令对象，包含执行该请求所需的所有信息。

#### 5. 模板方法模式

技能系统定义了工作流的骨架，将具体步骤的实现延迟到代理。

#### 6. 责任链模式

规则系统是责任链模式的实现，多个规则依次检查代码，只有通过所有规则的代码才会被接受。

#### 7. 中介者模式

主代理作为中介者，协调多个子代理之间的交互，子代理之间不直接通信。

### 6.5 AI 原生设计模式

这些是 ECC 针对 LLM 的特性设计的独特模式：

#### 1. 上下文内角色切换模式

在单个 Claude 会话内部实现多代理协作，不需要创建新的会话或 API 调用。

#### 2. 纯提示工程编排模式

工作流编排完全通过提示工程实现，没有传统的控制流代码。

#### 3. 交接文档协议模式

定义标准化的交接文档格式，确保代理之间能够清晰、准确地传递上下文信息。

#### 4. 质量门控模式

在工作流的每个阶段都设置质量门控，只有满足质量标准的输出才能进入下一阶段。

#### 5. 增量验证循环模式

每完成一个小步骤就进行验证，而不是等到整个任务完成后再验证。

#### 6. Trinity 模式

复杂任务处理分为三个阶段：探索→规划→执行。

#### 7. Git Worktree 隔离模式

使用 git worktree 为每个任务创建独立的工作环境，避免不同任务之间的代码干扰。

#### 8. 持续学习模式

通过钩子系统观察用户的操作，自动学习用户的偏好和编码风格，并转化为新的技能。

***

## 七、工具对比与选型

### 7.1 ECC 增强 Claude Code vs 原生 Claude Code

| 特性       | 原生 Claude Code | ECC 增强 Claude Code |
| -------- | -------------- | ------------------ |
| 代理数量     | 1 个通用代理        | 60+ 专业代理           |
| 内置命令     | 10+ 基础命令       | 75+ 开发专用命令         |
| 代码规则     | 无强制规则          | 34+ 最佳实践规则         |
| 自动化      | 无              | 8 类事件钩子            |
| MCP 配置   | 需手动配置          | 预配置 14+ 服务         |
| 上下文管理    | 基础             | 智能压缩 + 持久化         |
| 代码质量     | 不稳定            | 一致且符合最佳实践          |
| Token 效率 | 一般             | 高（节省 5.5x）         |

### 7.2 ECC 增强 Claude Code vs Cursor

| 特性       | ECC 增强 Claude Code | Cursor           |
| -------- | ------------------ | ---------------- |
| 运行环境     | 终端 / 桌面应用          | VS Code fork 编辑器 |
| 核心哲学     | 任务委派与自动化           | 结对编程与辅助          |
| 模型支持     | 仅 Anthropic 模型     | 多模型支持            |
| 代码质量     | 更高（67% 盲测胜率）       | 良好               |
| Token 效率 | 高                  | 一般               |
| IDE 集成   | 一般                 | 深度集成             |
| 学习曲线     | 较陡                 | 平缓               |
| 适用场景     | 复杂多文件任务、自动化流程      | 行级编辑、快速原型        |

### 7.3 ECC 与其他 AI 代理框架的对比

| 特性     | ECC             | LangGraph    | CrewAI    | Claude Agent Teams |
| ------ | --------------- | ------------ | --------- | ------------------ |
| 架构     | 提示工程驱动          | 图状状态机        | 角色驱动      | 原生多代理              |
| 运行环境   | Claude Code CLI | Python 应用    | Python 应用 | Claude API         |
| 状态管理   | SQLite          | 内置状态存储       | 简单状态      | 内置状态               |
| 工具集成   | MCP 协议          | LangChain 工具 | CrewAI 工具 | MCP 协议             |
| 代码生成能力 | 极强              | 一般           | 一般        | 强                  |
| 开箱即用   | 极高              | 低            | 中等        | 中等                 |
| 可定制性   | 高               | 极高           | 高         | 中等                 |

### 7.4 跨工具支持对比

ECC 是目前唯一支持所有主流 AI 编程工具的增强框架：

| 工具             | 代理支持 | 命令支持    | 技能支持  | 钩子支持   | 规则支持  | MCP 支持 |
| -------------- | ---- | ------- | ----- | ------ | ----- | ------ |
| Claude Code    | ✅ 60 | ✅ 75    | ✅ 228 | ✅ 8 类  | ✅ 34  | ✅ 14   |
| Cursor IDE     | ✅ 48 | ✅ 共享    | ✅ 共享  | ✅ 15 类 | ✅ 34  | ✅ 共享   |
| Codex CLI      | ✅ 共享 | ❌ 指令式   | ✅ 32  | ❌      | ❌ 指令式 | ✅ 7    |
| OpenCode       | ✅ 12 | ✅ 35    | ✅ 37  | ✅ 11 类 | ✅ 13  | ✅ 完整   |
| GitHub Copilot | ❌    | ❌ 6 个提示 | ❌     | ❌      | ✅ 1   | ❌      |

***

## 八、自定义开发

### 8.1 ECC 文件结构

```
everything-claude-code/
├── agents/           # 60 个专业代理定义
├── skills/           # 228 个工作流技能
├── commands/         # 75 个斜杠命令
├── rules/            # 34 个编码规则
├── hooks/            # 钩子配置和脚本
├── mcp-configs/      # MCP 服务器配置
├── scripts/          # 安装和工具脚本
├── tests/            # 测试套件
├── .claude-plugin/   # 插件元数据
└── ecc_dashboard.py  # Hermes 仪表板
```

### 8.2 自定义代理开发

1. 在 `agents/` 目录下创建一个新的 Markdown 文件

2. 添加 YAML 前端元数据，定义代理的名称、描述、工具集和模型

3. 编写代理的角色描述和行为规范

4. 保存文件，Claude Code 会自动加载

**示例：自定义代理**

```
---
name: my-custom-agent
description: My custom agent for specific tasks
tools: ["Read", "Write", "Bash"]
model: claude-3-5-sonnet-20240620
---

你是我的自定义代理，专门处理以下任务：
1. 任务一
2. 任务二
3. 任务三

你必须遵循以下规则：
- 规则一
- 规则二
- 规则三
```

### 8.3 自定义技能开发

1. 在 `skills/` 目录下创建一个新的目录

2. 在该目录下创建 `SKILL.md` 文件

3. 添加 YAML 前端元数据，定义技能的名称、描述和使用的代理

4. 编写技能的工作流步骤

5. 保存文件，Claude Code 会自动加载

**示例：自定义技能**

```
---
name: my-custom-skill
description: My custom workflow skill
agents: ["my-custom-agent"]
phases:
  - name: phase1
    description: Phase 1 description
  - name: phase2
    description: Phase 2 description
---

# 我的自定义技能
当执行此技能时，你必须严格按照以下步骤进行：
1. 步骤一
2. 步骤二
3. 步骤三
```

### 8.4 自定义命令开发

1. 在 `commands/` 目录下创建一个新的 Markdown 文件

2. 添加 YAML 前端元数据，定义命令的名称、描述和参数

3. 编写命令被触发时发送给 Claude 的提示文本

4. 保存文件，Claude Code 会自动加载

**示例：自定义命令**

```
---
name: /my-command
description: My custom command
arguments:
  - name: arg1
    type: string
    required: true
    description: Argument 1 description
---

# 我的自定义命令
请执行以下任务：{{arg1}}
你必须遵循以下规则：
- 规则一
- 规则二
```

### 8.5 测试与调试

1. **测试单个代理**：

```
/invoke-agent my-custom-agent "测试任务"
```

1. **测试工作流**：

```
/orchestrate custom "测试任务" --dry-run
```

1. **查看工作流日志**：

```
/workflow-log <workflow-id>
```

1. **运行测试套件**：

```
node tests/run-all.js
```

***

## 九、最佳实践

### 9.1 适用场景

ECC 最适合以下场景：

* **中大型项目开发**：需要多人协作、代码质量要求高的项目

* **复杂多文件任务**：系统重构、微服务开发、CI/CD 配置

* **自动化流程**：测试自动化、部署自动化、文档生成

* **团队开发**：需要统一代码规范和开发流程的团队

* **技术学习**：通过观察专业代理的工作方式学习最佳实践

### 9.2 不适用场景

ECC 不适合以下场景：

* **简单的单行代码修改**：使用原生 Claude Code 或 Cursor 更高效

* **需要深度 IDE 集成的任务**：如调试、断点设置

* **离线环境**：需要连接 Anthropic API

* **预算非常有限的项目**：ECC 会增加 API 调用次数

### 9.3 Token 优化最佳实践

1. **模型选择**：

* 默认使用 Claude 3.5 Sonnet，处理 80%+ 的任务

* 仅在复杂架构、深度推理时使用 Claude 3 Opus

* 简单任务可以使用 Claude 3 Haiku

1. **上下文管理**：

* 使用 `/clear` 在不相关任务之间重置上下文

* 使用 `/compact` 在逻辑断点处压缩上下文

* 禁用不需要的 MCP 服务器

1. **配置优化**：

```
{
  "env": {
    "MAX_THINKING_TOKENS": "10000",
    "CLAUDE_AUTOCOMPACT_PCT_OVERRIDE": "50",
    "CLAUDE_CODE_SUBAGENT_MODEL": "haiku"
  }
}
```

### 9.4 安全最佳实践

1. **权限管理**：

* 谨慎开启 `autoApprove` 中的 `runCommands` 和 `installPackages` 选项

* 永远不要在生产环境中开启完全自动批准

* 使用钩子拦截危险操作

1. **安全审计**：

* 定期运行 `/security-scan` 检查代码漏洞

* 使用 AgentShield 扫描 ECC 配置本身的漏洞

* 审查所有自动生成的代码

1. **密钥管理**：

* 永远不要将 API 密钥提交到代码仓库

* 使用环境变量存储敏感信息

* 定期轮换 API 密钥

### 9.5 局限性

1. **提示工程的天花板**：所有功能都基于提示工程，存在固有的不稳定性

2. **可观测性不足**：缺乏完善的监控和调试工具

3. **token 开销较大**：大量的规则和代理定义会增加每次请求的 token 消耗

4. **大规模团队协作支持有限**：目前主要面向个人开发者和小团队

5. **依赖 Claude Code 的能力**：ECC 无法实现 Claude Code 本身不支持的功能

### 9.6 未来发展方向

1. **ECC 2.0 Rust 控制平面**：用 Rust 重写核心编排逻辑，提供更好的性能和可靠性

2. **云原生部署**：支持在服务器上部署 ECC 控制平面，实现团队共享和远程执行

3. **AI 驱动的工作流优化**：使用机器学习自动优化工作流的执行顺序和代理分配

4. **更丰富的集成**：与更多的开发工具和服务集成，如 Jira、Slack、GitHub Actions 等

***

## 核心知识点速览

* ECC 是一个纯配置驱动的 AI 编程增强框架，不是独立工具，而是 Claude Code 等工具的 "高级改装套件"

* ECC 包含 6 大核心组件：60+ 代理、228+ 技能、75+ 命令、34+ 规则、8 类钩子、14+ MCP 配置

* ECC 的核心创新是**上下文内角色切换**技术，在单个会话内实现多代理协作，零延迟、低开销

* ECC 支持三种安装方式，**严禁叠加使用**，推荐使用插件安装

* ECC 的工作流编排完全基于提示工程，没有传统的控制流代码

* ECC v2.0 引入了 Hermes Operator，提供跨会话编排、可视化仪表板和持续学习功能

* ECC 支持所有主流 AI 编程工具，包括 Claude Code、Cursor、Codex、OpenCode 和 GitHub Copilot

* 合理使用 ECC 可以将开发效率提升 5-10 倍，同时显著提高代码质量

* Token 优化是使用 ECC 的关键，默认使用 Sonnet 模型，仅在必要时使用 Opus

* ECC 最适合中大型项目和复杂多文件任务，简单任务使用原生工具更高效
