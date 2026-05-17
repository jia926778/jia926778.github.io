---
title: 飞书CLI技术文档
date: 2026-05-17 20:20:00
tags: [飞书cli, 技术分享, cli, AI 编程]
categories: 工具使用
toc: true
---
# 飞书CLI技术文档


## 目录

1. [简介](#简介)

2. [快速安装](#快速安装)

3. [基础配置](#基础配置)

4. [核心命令速查](#核心命令速查)

5. [进阶使用技巧](#进阶使用技巧)

6. [AI Agent 集成](#ai-agent-集成)

7. [故障排除](#故障排除)

8. [相关资源](#相关资源)

## 简介

飞书 CLI 是字节跳动飞书开放平台于 2026 年 3 月 28 日正式开源的官方命令行工具，采用 Go 语言开发，核心定位是 "Built for humans and AI Agents"。它既为人类提供终端高效办公体验，也专为 AI 智能体设计，让 AI 能够直接 "动手操作" 飞书全业务流程。

### 核心特性

- 官方出品，API 稳定可靠，持续更新迭代

- AI 原生设计，内置 24 个结构化 AI Agent Skills

- 支持 12 个核心业务域，200 + 精选高频命令

- 三层命令体系：快捷命令、API 命令、Raw API

- 凭证本地加密存储，支持 dry-run 预览模式

- 多身份支持，可切换个人用户和机器人身份

## 快速安装

### 推荐一键安装（包含 AI 技能）

```bash
npx @larksuite/cli@latest install
```

### 国内镜像安装

```bash
# 安装CLI
npm install -g @larksuite/cli --registry=https://registry.npmmirror.com

# 安装AI Agent技能
npx skills add larksuite/cli -y -g
```

### 版本管理

```bash
# 查看当前版本
lark-cli version

# 更新到最新版本
lark-cli update
```

## 基础配置

### 身份认证

```bash
# 扫码登录飞书账号
lark-cli auth login

# 查看当前登录状态
lark-cli auth status

# 查看所有已登录账号
lark-cli auth list

# 切换登录账号
lark-cli auth switch <account>

# 退出当前账号
lark-cli auth logout
```

### 通用参数

- `--help`: 查看任意命令帮助

- `--dry-run`: 预览操作不执行

- `--json`: 输出 JSON 格式（适合脚本调用）

- `--verbose`: 显示详细日志信息

## 核心命令速查

### 消息与群组

```bash
# 列出最近的聊天列表
lark-cli chat list

# 搜索聊天
lark-cli chat search "关键词"

# 发送文本消息
lark-cli message send --chat <chat_id> --text "消息内容"

# 发送文件
lark-cli message send --chat <chat_id> --file ./report.pdf

# 发送图片
lark-cli message send --chat <chat_id> --image ./screenshot.png

# 查看最近20条消息
lark-cli message list --chat <chat_id> --limit 20

# 回复指定消息
lark-cli message reply <message_id> --text "回复内容"

# 创建群聊
lark-cli group create --name "群名称" --members "user1@example.com,user2@example.com"

# 添加群成员
lark-cli group add-member --chat <chat_id> --members "user3@example.com"
```

### 云文档

```bash
# 创建空白文档
lark-cli doc create --title "文档标题"

# 导入本地Markdown为飞书文档
lark-cli doc import --input ./README.md --title "导入的文档"

# 导出飞书文档为Markdown
lark-cli doc export <doc_id> -o ./output.md

# 导出文档并下载图片到本地
lark-cli doc export <doc_id> -o ./output.md --download-images

# 获取文档内容
lark-cli doc get <doc_id>

# 更新文档内容
lark-cli doc update <doc_id> --content "# 新标题\n\n新内容"

# 在文档末尾追加内容
lark-cli doc append <doc_id> --content "\n\n追加的内容"

# 删除文档（移至回收站）
lark-cli doc delete <doc_id>

# 分享文档给用户（编辑权限）
lark-cli doc share <doc_id> --user "user@example.com" --permission "edit"
```

### 电子表格

```bash
# 创建空白电子表格
lark-cli sheet create --title "表格标题"

# 导入CSV到电子表格
lark-cli sheet import --input ./data.csv --sheet <spreadsheet_id>

# 导出工作表为CSV
lark-cli sheet export <spreadsheet_id> --sheet "Sheet1" -o ./data.csv

# 获取指定范围单元格数据
lark-cli sheet cell get <spreadsheet_id> --range "Sheet1!A1:C5"

# 设置单个单元格值
lark-cli sheet cell set <spreadsheet_id> --range "Sheet1!A1" --value "标题"

# 批量设置单元格值
lark-cli sheet cell batch-set <spreadsheet_id> --range "Sheet1!A2:B3" --values '[["a","b"],["c","d"]]'
```

### 多维表格

```bash
# 创建空白多维表格
lark-cli bitable create --title "多维表格标题"

# 列出多维表格中的所有数据表
lark-cli bitable table list <app_token>

# 列出数据表中的所有记录
lark-cli bitable record list <app_token> <table_id>

# 创建新记录
lark-cli bitable record create <app_token> <table_id> --fields '{"名称":"记录1","状态":"进行中"}'

# 更新记录
lark-cli bitable record update <app_token> <table_id> <record_id> --fields '{"状态":"已完成"}'

# 删除记录
lark-cli bitable record delete <app_token> <table_id> <record_id>
```

### 日历与会议

```bash
# 查看今日日程（快捷命令）
lark-cli calendar +agenda

# 查看今日到明天的日程
lark-cli calendar events list --start-time today --end-time tomorrow

# 创建会议
lark-cli calendar event create --title "会议标题" --start-time "2026-05-18T10:00:00+08:00" --end-time "2026-05-18T11:00:00+08:00" --attendees "user1@example.com,user2@example.com"

# 查看最近10个已结束的会议
lark-cli meeting list --status ended --limit 10

# 获取会议逐字稿
lark-cli meeting transcript <meeting_id>

# 获取会议纪要
lark-cli meeting minutes <meeting_id>
```

### 任务管理

```bash
# 列出所有未完成的任务
lark-cli task list --status incomplete

# 创建任务并分配给他人
lark-cli task create --title "任务标题" --assignee "user@example.com" --due-date "2026-05-20"

# 将任务标记为已完成
lark-cli task update <task_id> --status completed

# 给任务添加评论
lark-cli task comment <task_id> --text "任务评论"

# 删除任务
lark-cli task delete <task_id>
```

### 云空间

```bash
# 列出云空间根目录文件
lark-cli drive list

# 上传文件到指定文件夹
lark-cli drive upload --file ./document.pdf --folder <folder_id>

# 下载文件到本地
lark-cli drive download <file_token> -o ./downloaded.pdf

# 创建新文件夹
lark-cli drive mkdir --name "新文件夹" --parent <parent_folder_id>

# 删除文件（移至回收站）
lark-cli drive delete <file_token>
```

### 通讯录

```bash
# 搜索用户
lark-cli contact user search "张三"

# 获取用户详细信息
lark-cli contact user get <user_id>

# 列出所有部门
lark-cli contact department list

# 列出部门成员
lark-cli contact department members <department_id>
```

## 进阶使用技巧

### 脚本集成

所有命令都支持`--json`输出，方便在 Shell 脚本或 Python 中解析：

```bash
# 获取JSON格式的聊天列表
lark-cli chat list --json > chats.json

# 使用jq解析JSON
lark-cli chat list --json | jq '.[] | {name: .name, id: .chat_id}'
```

### 批量操作

结合 xargs 可以实现批量处理：

```bash
# 批量添加群成员
cat users.txt | xargs -I {} lark-cli group add-member --chat <chat_id> --members {}

# 批量导出文档
cat doc_ids.txt | xargs -I {} lark-cli doc export {} -o {}.md
```

### 定时任务

结合 crontab 可以实现自动化操作：

```bash
# 每天早上9点发送日报提醒
0 9 * * * lark-cli message send --chat <chat_id> --text "请大家提交昨日日报"

# 每周五晚上6点自动备份重要文档
0 18 * * 5 lark-cli doc export <doc_id> -o /backup/weekly/$(date +%Y%m%d).md
```

## AI Agent 集成

飞书 CLI 原生支持 MCP 协议，可直接在所有主流 AI 编辑器中使用：

- Claude Code

- Cursor

- GitHub Copilot

- Windsurf

- Trae

### AI 快捷命令

直接输入即可，无需复杂参数：

- `+agenda`: 查看今日日程

- `+today`: 查看今日所有待办和会议

- `+tomorrow`: 查看明日安排

- `+meetings`: 查看本周所有会议

- `+tasks`: 查看所有未完成任务

- `+summary`: 生成今日工作摘要

- `+weekly`: 生成本周工作周报

## 故障排除

### 常见问题

1. **登录失败**

    - 确保网络连接正常

    - 尝试使用`lark-cli auth logout`清除缓存后重新登录

    - 检查是否有多个飞书账号冲突

2. **权限不足**

    - 确认你在飞书中拥有对应资源的操作权限

    - 机器人身份需要管理员在开放平台配置相应权限

3. **资源 ID 错误**

    - 资源 ID 可在飞书网页版 URL 中获取

    - 文档 ID 在`/d/`和`/`之间

    - 聊天 ID 在`/im/chat/`之后

## 相关资源

- [GitHub 开源仓库](https://github.com/larksuite/cli)

- [官方网站](https://www.feishu.cn/feishu-cli)

- [官方文档](https://open.feishu.cn/document/mcp_open_tools/feishu-cli-let-ai-actually-do-your-work-in-feishu)

- [NPM 包](https://www.npmjs.com/package/@larksuite/cli)
