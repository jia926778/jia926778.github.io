---
title: VSCode快捷键
date: 2026-05-01 18:30:00
tags: [VSCode, 工具使用, 快捷键]
categories: 工具使用
toc: true
---

# VSCode快捷键学习手册


## 文档核心说明

本文档适用于 VSCode 初学者及希望提升编码效率的开发者，系统整理了 VSCode 全场景默认快捷键，以及从入门到进阶的自定义快捷键配置方案，帮助你快速掌握高效的编辑器操作技巧，适配不同的开发习惯。

## 一、基础快捷键大全

### 1\.1 通用快捷键（必记）

|功能|Windows/Linux|macOS|使用场景|
|---|---|---|---|
|打开命令面板（万能键）|`Ctrl\+Shift\+P` / `F1`|`Cmd\+Shift\+P` / `F1`|执行任何 VSCode 命令|
|快速打开文件|`Ctrl\+P`|`Cmd\+P`|大项目中快速定位文件|
|打开用户设置|`Ctrl\+,`|`Cmd\+,`|个性化配置编辑器|
|打开快捷键设置|`Ctrl\+K Ctrl\+S`|`Cmd\+K Cmd\+S`|查看和修改所有快捷键|
|新建窗口|`Ctrl\+Shift\+N`|`Cmd\+Shift\+N`|打开新的 VSCode 实例|
|关闭当前窗口|`Ctrl\+W`|`Cmd\+W`|关闭当前标签页或窗口|

### 1\.2 文件与视图操作

|功能|Windows/Linux|macOS|
|---|---|---|
|新建文件|`Ctrl\+N`|`Cmd\+N`|
|打开文件 / 文件夹|`Ctrl\+O` / `Ctrl\+K Ctrl\+O`|`Cmd\+O` / `Cmd\+K Cmd\+O`|
|保存文件|`Ctrl\+S`|`Cmd\+S`|
|保存所有文件|`Ctrl\+K S`|`Cmd\+K S`|
|另存为|`Ctrl\+Shift\+S`|`Cmd\+Shift\+S`|
|关闭当前文件|`Ctrl\+W`|`Cmd\+W`|
|关闭所有文件|`Ctrl\+K Ctrl\+W`|`Cmd\+K Cmd\+W`|
|切换标签页|`Ctrl\+Tab`|`Cmd\+Tab`|
|显示 / 隐藏侧边栏|`Ctrl\+B`|`Cmd\+B`|
|显示 / 隐藏终端|\\`Ctrl\+\\`\`|\\`Cmd\+\\`\`|
|显示 / 隐藏搜索面板|`Ctrl\+Shift\+F`|`Cmd\+Shift\+F`|
|显示 / 隐藏调试面板|`Ctrl\+Shift\+D`|`Cmd\+Shift\+D`|
|显示 / 隐藏扩展面板|`Ctrl\+Shift\+X`|`Cmd\+Shift\+X`|

### 1\.3 基础编辑快捷键

#### 1\.3\.1 行操作

|功能|Windows/Linux|macOS|
|---|---|---|
|移动当前行|`Alt\+↑/↓`|`Option\+↑/↓`|
|复制当前行|`Shift\+Alt\+↑/↓`|`Shift\+Option\+↑/↓`|
|删除当前行|`Ctrl\+Shift\+K`|`Cmd\+Shift\+K`|
|在下方插入新行|`Ctrl\+Enter`|`Cmd\+Enter`|
|在上方插入新行|`Ctrl\+Shift\+Enter`|`Cmd\+Shift\+Enter`|
|剪切整行（无选中时）|`Ctrl\+X`|`Cmd\+X`|
|复制整行（无选中时）|`Ctrl\+C`|`Cmd\+C`|
|缩进 / 取消缩进|`Ctrl\+\]` / `Ctrl\+\[`|`Cmd\+\]` / `Cmd\+\[`|

#### 1\.3\.2 光标与选择

|功能|Windows/Linux|macOS|
|---|---|---|
|选择当前行|`Ctrl\+L`|`Cmd\+L`|
|跳转到行首 / 行尾|`Home` / `End`|`Fn\+←` / `Fn\+→`|
|跳转到文件开头 / 结尾|`Ctrl\+Home` / `Ctrl\+End`|`Cmd\+↑` / `Cmd\+↓`|
|撤销光标操作|`Ctrl\+U`|`Cmd\+U`|
|跳转到匹配括号|`Ctrl\+Shift\+\\`|`Cmd\+Shift\+\\`|
|向上 / 下滚动一行|`Ctrl\+↑` / `Ctrl\+↓`|`Cmd\+↑` / `Cmd\+↓`|
|向上 / 下滚动一页|`Alt\+PageUp` / `Alt\+PageDown`|`Option\+PageUp` / `Option\+PageDown`|

#### 1\.3\.3 多光标编辑（效率神器）

|功能|Windows/Linux|macOS|
|---|---|---|
|添加多个光标|`Alt\+Click`|`Option\+Click`|
|向上 / 下添加光标|`Ctrl\+Alt\+↑/↓`|`Cmd\+Option\+↑/↓`|
|选中下一个匹配项|`Ctrl\+D`|`Cmd\+D`|
|跳过当前匹配项|`Ctrl\+K Ctrl\+D`|`Cmd\+K Cmd\+D`|
|选中所有匹配项|`Ctrl\+Shift\+L`|`Cmd\+Shift\+L`|
|在选中行末尾添加光标|`Shift\+Alt\+I`|`Shift\+Option\+I`|

#### 1\.3\.4 代码格式化与注释

|功能|Windows/Linux|macOS|
|---|---|---|
|格式化整个文档|`Shift\+Alt\+F`|`Shift\+Option\+F`|
|格式化选中代码|`Ctrl\+K Ctrl\+F`|`Cmd\+K Cmd\+F`|
|添加 / 移除行注释|`Ctrl\+/`|`Cmd\+/`|
|添加 / 移除块注释|`Shift\+Alt\+A`|`Shift\+Option\+A`|
|折叠 / 展开代码块|`Ctrl\+Shift\+\[` / `Ctrl\+Shift\+\]`|`Cmd\+Option\+\[` / `Cmd\+Option\+\]`|
|折叠 / 展开所有代码|`Ctrl\+K Ctrl\+0` / `Ctrl\+K Ctrl\+J`|`Cmd\+K Cmd\+0` / `Cmd\+K Cmd\+J`|
|修剪尾随空格|`Ctrl\+K Ctrl\+X`|`Cmd\+K Cmd\+X`|

### 1\.4 搜索与替换

|功能|Windows/Linux|macOS|
|---|---|---|
|查找|`Ctrl\+F`|`Cmd\+F`|
|替换|`Ctrl\+H`|`Cmd\+H`|
|全局搜索|`Ctrl\+Shift\+F`|`Cmd\+Shift\+F`|
|全局替换|`Ctrl\+Shift\+H`|`Cmd\+Shift\+H`|
|查找下一个 / 上一个|`F3` / `Shift\+F3`|`F3` / `Shift\+F3`|
|选中所有匹配项|`Alt\+Enter`|`Option\+Enter`|

### 1\.5 代码导航

|功能|Windows/Linux|macOS|
|---|---|---|
|跳转到定义|`F12`|`F12`|
|预览定义（不跳转）|`Alt\+F12`|`Option\+F12`|
|跳转到引用|`Shift\+F12`|`Shift\+F12`|
|跳转到行号|`Ctrl\+G`|`Cmd\+G`|
|跳转到文件内符号|`Ctrl\+Shift\+O`|`Cmd\+Shift\+O`|
|跳转到工作区符号|`Ctrl\+T`|`Cmd\+T`|
|返回上一个位置|`Alt\+←`|`Cmd\+←`|
|前进到下一个位置|`Alt\+→`|`Cmd\+→`|

### 1\.6 调试快捷键

|功能|Windows/Linux|macOS|
|---|---|---|
|开始 / 继续调试|`F5`|`F5`|
|停止调试|`Shift\+F5`|`Shift\+F5`|
|切换断点|`F9`|`F9`|
|单步跳过|`F10`|`F10`|
|单步进入|`F11`|`F11`|
|单步退出|`Shift\+F11`|`Shift\+F11`|
|显示悬停信息|`Ctrl\+K Ctrl\+I`|`Cmd\+K Cmd\+I`|

### 1\.7 终端快捷键

|功能|Windows/Linux|macOS|
|---|---|---|
|显示 / 隐藏终端|\\`Ctrl\+\\`\`|\\`Cmd\+\\`\`|
|新建终端|\\`Ctrl\+Shift\+\\`\`|\\`Cmd\+Shift\+\\`\`|
|复制选中内容|`Ctrl\+Shift\+C`|`Cmd\+Shift\+C`|
|粘贴|`Ctrl\+Shift\+V`|`Cmd\+Shift\+V`|
|向上 / 下滚动终端|`Ctrl\+↑` / `Ctrl\+↓`|`Cmd\+↑` / `Cmd\+↓`|
|向上 / 下滚动终端页面|`PageUp` / `PageDown`|`PageUp` / `PageDown`|

## 二、自定义快捷键进阶指南

### 2\.1 图形界面配置（新手推荐）

#### 2\.1\.1 打开快捷键编辑器

- **快捷键**：`Ctrl\+K Ctrl\+S` \(Windows/Linux\) / `Cmd\+K Cmd\+S` \(macOS\)

- **菜单方式**：文件 \&gt; 首选项 \&gt; 键盘快捷方式

- **命令面板方式**：`Ctrl\+Shift\+P` / `Cmd\+Shift\+P`，输入 \&\#34;Preferences: Open Keyboard Shortcuts\&\#34;

#### 2\.1\.2 修改快捷键步骤

1. 在搜索框中输入**功能关键词**（如 \&\#34;format\&\#34;、\&\#34;terminal\&\#34;）找到目标命令

2. 点击命令左侧的**铅笔图标**

3. 按下你想要设置的**新快捷键组合**（支持多键组合，如`Ctrl\+K Ctrl\+D`）

4. 按**Enter**确认保存

#### 2\.1\.3 其他常用操作

- **删除快捷键**：右键点击命令 \&gt; 移除键绑定

- **重置为默认**：右键点击命令 \&gt; 重置键绑定

- **查看冲突**：右键点击命令 \&gt; 显示相同键位绑定

- **复制命令名**：右键点击命令 \&gt; 复制命令 ID（用于 JSON 编辑）

### 2\.2 JSON 文件配置（高级灵活）

#### 2\.2\.1 打开配置文件

- 在快捷键编辑器中点击右上角的**打开文件图标**

- **命令面板方式**：`Ctrl\+Shift\+P` / `Cmd\+Shift\+P`，输入 \&\#34;Preferences: Open Keyboard Shortcuts \(JSON\)\&\#34;

#### 2\.2\.2 文件结构

每条快捷键规则是一个 JSON 对象，包含三个字段：

```json
[
  {
    "key": "快捷键组合",
    "command": "要执行的命令ID",
    "when": "触发条件（可选）"
  }
]
```

#### 2\.2\.3 按键组合写法规范

- 修饰键：`ctrl`、`shift`、`alt` \(Windows/Linux\) / `cmd`、`shift`、`option` \(macOS\)

- 普通键：直接写字母或数字，如`a`、`1`、`f5`

- 特殊键：`enter`、`tab`、`escape`、`space`、`backspace`、`delete`

- 多键组合：用空格分隔，如`ctrl\+k ctrl\+d`

#### 2\.2\.4 常用 when 条件（上下文感知）

`when`条件让你精确控制快捷键在什么场景下生效，是 VSCode 快捷键系统最强大的功能之一。

|条件|说明|
|---|---|
|`editorTextFocus`|焦点在文本编辑器中|
|`terminalFocus`|焦点在终端中|
|`textInputFocus`|焦点在任何可输入文本的控件中|
|`editorHasSelection`|编辑器中有选中的文本|
|`\!editorReadonly`|编辑器不是只读模式|
|`inDebugMode`|处于调试模式|
|`editorLangId == \&\#39;python\&\#39;`|当前文件是 Python 文件|
|`sideBarVisible`|侧边栏可见|

**示例**：

```json
// 仅在Python文件中启用重构提取变量
{
  "key": "ctrl+shift+r",
  "command": "python.refactor.extractVariable",
  "when": "editorTextFocus && editorLangId == 'python'"
},

// 终端有选中文本时才执行复制
{
  "key": "ctrl+c",
  "command": "workbench.action.terminal.copySelection",
  "when": "terminalFocus && terminalTextSelected"
}
```

### 2\.3 实用自定义快捷键示例

以下是广受开发者欢迎的自定义配置，可根据个人习惯调整：

```json
[
  // 将删除行改为Ctrl+D（原Ctrl+Shift+K）
  {
    "key": "ctrl+d",
    "command": "editor.action.deleteLines",
    "when": "editorTextFocus && !editorReadonly"
  },

  // 将添加下一个匹配项改为Ctrl+Shift+K（原Ctrl+D）
  {
    "key": "ctrl+shift+k",
    "command": "editor.action.addSelectionToNextFindMatch",
    "when": "editorFocus"
  },

  // 快速打开终端
  {
    "key": "ctrl+`",
    "command": "workbench.action.terminal.toggleTerminal"
  },

  // 快速格式化代码
  {
    "key": "ctrl+shift+f",
    "command": "editor.action.formatDocument",
    "when": "editorTextFocus && !editorReadonly"
  },

  // 快速注释多行
  {
    "key": "ctrl+shift+/",
    "command": "editor.action.blockComment",
    "when": "editorTextFocus && !editorReadonly"
  },

  // 快速保存所有文件
  {
    "key": "ctrl+shift+s",
    "command": "workbench.action.files.saveAll"
  }
]
```

### 2\.4 高级配置技巧

#### 2\.4\.1 查看所有默认快捷键

打开命令面板，输入 \&\#34;Preferences: Open Default Keyboard Shortcuts \(JSON\)\&\#34;，可查看所有内置命令的默认快捷键配置，是查找命令 ID 的最佳来源。

#### 2\.4\.2 导出与导入快捷键

- **导出**：打开`keybindings\.json`文件，复制所有内容保存到本地文件

- **导入**：在新设备上打开`keybindings\.json`文件，粘贴之前保存的内容

- **云端同步**：使用 VSCode 内置的**设置同步**功能，可同步包括快捷键在内的所有配置到 GitHub 或微软账户

#### 2\.4\.3 解决快捷键冲突

1. **VSCode 内部冲突**：

    - 打开快捷键编辑器，搜索冲突的快捷键组合，VSCode 会用黄色警告标记冲突项

    - 右键点击冲突项 \&gt; 显示相同键位绑定，查看所有冲突的命令

    - 修改其中一个快捷键或禁用不需要的绑定

2. **系统级冲突**：

    - 检查输入法、截图工具、系统快捷键是否占用了相同的组合键

    - 常见冲突：`Ctrl\+Space`（输入法切换）、`Ctrl\+Alt\+T`（Linux 终端）、`Cmd\+Space`（macOS Spotlight）

    - 解决方案：修改 VSCode 快捷键或系统快捷键

3. **扩展冲突**：

    - 某些扩展会注册自己的快捷键

    - 在快捷键编辑器中搜索 \&\#34;@source:extension\&\#34; 查看所有扩展注册的快捷键

    - 禁用不需要的扩展快捷键或修改为其他组合

### 2\.5 配置最佳实践

1. **保持一致性**：尽量遵循你习惯的编辑器快捷键，VSCode 提供了对应的键位映射扩展

2. **使用组合键**：优先使用`Ctrl\+Shift\+\*`或`Ctrl\+Alt\+\*`组合，避免与单键或简单组合冲突

3. **善用 when 条件**：让不同场景下的相同快捷键执行不同的操作

4. **定期备份**：将你的`keybindings\.json`文件备份到云存储或 Git 仓库

5. **从少量开始**：不要一次性修改太多快捷键，逐步适应

## 核心知识点速览

- **命令面板**`Ctrl\+Shift\+P`是 VSCode 的万能操作入口，可执行所有编辑器命令

- **多光标编辑**可大幅提升批量修改代码的效率，是提升编码速度的核心技巧

- 自定义快捷键支持**图形界面**（新手友好）和**JSON 文件**（高级灵活）两种配置方式

- `when`条件可实现上下文感知的快捷键触发，让同一组合键在不同场景执行不同操作

- 快捷键冲突可通过快捷键编辑器排查，分为 VSCode 内部、系统级、扩展三类冲突

- 支持**设置同步**功能，可将快捷键等配置云端同步到不同设备

- 格式化、注释、行操作是日常开发中最常用的基础编辑快捷键

- 代码导航快捷键可快速定位函数定义、引用，大幅提升代码阅读效率

- 终端专属快捷键可提升命令行操作效率，无需切换鼠标操作

- 自定义快捷键时优先使用组合键，避免与系统或基础操作的快捷键冲突
