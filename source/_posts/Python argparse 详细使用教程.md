---
title: argparse 使用教程
date: 2026-04-24 12:30:00
tags: [argparse, Python, 模型微调]
categories: 模型微调
toc: true
---
# Python argparse 详细使用教程

## 一、核心概念

`argparse` 是 Python 标准库中用于解析命令行参数的模块，核心概念如下：

- **ArgumentParser**：参数解析器对象，负责整个解析流程的控制。

- **add\_argument\(\)**：向解析器添加位置参数或可选参数。

- **parse\_args\(\)**：解析命令行输入，返回一个 `Namespace` 对象，包含所有参数值。

---

## 二、基本使用流程

### 2\.1 最小化示例

```python
# 1. 导入模块
import argparse

# 2. 创建解析器对象
parser = argparse.ArgumentParser(description="一个简单的命令行工具")

# 3. 添加参数
parser.add_argument("name", help="输入你的名字")

# 4. 解析参数
args = parser.parse_args()

# 5. 使用参数
print(f"你好, {args.name}!")
```

### 2\.2 运行方式

```bash
# 查看帮助
python script.py -h

# 正常运行
python script.py 张三
# 输出：你好, 张三!
```

---

## 三、位置参数与可选参数

### 3\.1 位置参数

按顺序传递的参数，无需前缀：

```python
parser.add_argument("a", type=int, help="第一个数字")
parser.add_argument("b", type=int, help="第二个数字")
args = parser.parse_args()
print(f"和: {args.a + args.b}")
```

### 3\.2 可选参数

以 `-` 或 `--` 开头的参数，顺序不限：

```python
# 短选项（-v）和长选项（--verbose）可同时定义
parser.add_argument("-v", "--verbose", action="store_true", help="显示详细信息")
parser.add_argument("-o", "--output", help="输出文件路径")
```

---

## 四、参数类型与验证

### 4\.1 内置类型

通过 `type` 参数指定参数类型，自动进行类型转换：

```python
parser.add_argument("num", type=int, help="整数")
parser.add_argument("price", type=float, help="浮点数")
parser.add_argument("file", type=open, help="文件对象（自动打开）")
```

### 4\.2 选项限制（choices）

限制参数只能从指定列表中选择：

```python
parser.add_argument("mode", choices=["read", "write", "append"], help="操作模式")
```

### 4\.3 自定义类型验证

通过函数实现复杂验证逻辑：

```python
def valid_port(port_str):
    port = int(port_str)
    if not (1 <= port <= 65535):
        raise argparse.ArgumentTypeError("端口必须在 1-65535 之间")
    return port

parser.add_argument("-p", "--port", type=valid_port, help="端口号")
```

---

## 五、参数动作（action）

### 5\.1 store（默认）

存储参数值（最常用）：

```python
parser.add_argument("--name", action="store", help="存储名字")
```

### 5\.2 store\_true / store\_false

存储布尔值（无需传参）：

```python
parser.add_argument("--verbose", action="store_true", help="开启详细模式")
parser.add_argument("--quiet", action="store_false", help="关闭详细模式")
```

### 5\.3 store\_const

存储预设常量：

```python
parser.add_argument("--level", action="store_const", const=10, help="设置等级为10")
```

### 5\.4 append

收集多个值到列表：

```python
parser.add_argument("--file", action="append", help="添加多个文件")
# 运行：python script.py --file a.txt --file b.txt
# args.file = ["a.txt", "b.txt"]
```

### 5\.5 count

计数参数出现次数：

```python
parser.add_argument("-v", "--verbose", action="count", help="详细程度（-v, -vv）")
# 运行：python script.py -vv
# args.verbose = 2
```

### 5\.6 help /version（内置动作）

- `-h` / `--help`：自动生成帮助信息（无需手动添加）。

- `--version`：显示版本信息：

    ```python
    parser.add_argument("--version", action="version", version="%(prog)s 1.0")
    ```

---

## 六、子命令（subparsers）

用于实现类似 `git clone` / `git commit` 的分层命令结构：

```python
# 创建主解析器
parser = argparse.ArgumentParser(description="版本控制工具")
subparsers = parser.add_subparsers(title="子命令", dest="subcommand")

# 创建 clone 子解析器
clone_parser = subparsers.add_parser("clone", help="克隆仓库")
clone_parser.add_argument("url", help="仓库地址")

# 创建 commit 子解析器
commit_parser = subparsers.add_parser("commit", help="提交更改")
commit_parser.add_argument("-m", "--message", required=True, help="提交信息")

# 解析并使用
args = parser.parse_args()
if args.subcommand == "clone":
    print(f"克隆仓库: {args.url}")
elif args.subcommand == "commit":
    print(f"提交信息: {args.message}")
```

---

## 七、互斥参数组

限制多个参数只能选择一个：

```python
group = parser.add_mutually_exclusive_group()
group.add_argument("-v", "--verbose", action="store_true", help="详细模式")
group.add_argument("-q", "--quiet", action="store_true", help="安静模式")
```

---

## 八、默认值与必需参数

### 8\.1 默认值（default）

```python
parser.add_argument("--port", type=int, default=8080, help="端口号（默认8080）")
```

### 8\.2 必需参数（required=True）

```python
parser.add_argument("--output", required=True, help="输出文件路径（必需）")
```

---

## 九、帮助信息定制

### 9\.1 解析器描述与结语

```python
parser = argparse.ArgumentParser(
    description="这是工具的描述信息",
    epilog="这是帮助信息的结语"
)
```

### 9\.2 参数帮助文本

```python
parser.add_argument("--name", help="用户的名字（支持中文）")
```

### 9\.3 修改参数显示名（metavar）

改变帮助信息中参数的显示名称（不影响代码中属性名）：

```python
parser.add_argument("--input", metavar="FILE", help="输入文件")
# 帮助信息显示：--input FILE
```

### 9\.4 修改属性名（dest）

改变 `Namespace` 中参数的属性名：

```python
parser.add_argument("--n", dest="name", help="名字")
# 使用：args.name
```

---

## 十、高级用法

### 10\.1 从文件读取参数

通过 `fromfile_prefix_chars` 允许从文件读取参数：

```python
parser = argparse.ArgumentParser(fromfile_prefix_chars="@")
# 创建 args.txt 文件，内容：--name 张三 --port 8080
# 运行：python script.py @args.txt
```

### 10\.2 父解析器（parents）

复用参数定义：

```python
# 父解析器（不解析参数）
parent_parser = argparse.ArgumentParser(add_help=False)
parent_parser.add_argument("--verbose", action="store_true", help="详细模式")

# 子解析器继承父解析器
child_parser = argparse.ArgumentParser(parents=[parent_parser])
child_parser.add_argument("--name", help="名字")
```

### 10\.3 允许参数缩写（allow\_abbrev）

默认开启，允许长选项缩写（如 `--ver` 代替 `--verbose`）：

```python
parser = argparse.ArgumentParser(allow_abbrev=True)
```

---

## 十一、常见问题与最佳实践

### 11\.1 常见问题

1. **参数顺序**：位置参数必须按顺序传递，可选参数顺序不限。

2. **布尔参数**：推荐使用 `store_true` / `store_false`，避免传递 `True` / `False` 字符串。

3. **参数冲突**：使用互斥组解决参数冲突问题。

### 11\.2 最佳实践

1. **始终添加 help 文本**：方便用户理解参数用途。

2. **使用子命令组织复杂工具**：提高可读性。

3. **验证参数类型**：避免运行时错误。

4. **提供默认值**：提升用户体验。

---

## 核心要点总结

1. **核心流程**：导入模块 → 创建解析器 → 添加参数 → 解析参数 → 使用参数。

2. **参数类型**：位置参数（按顺序）、可选参数（带前缀）。

3. **常用动作**：`store_true`（布尔值）、`append`（列表）、`count`（计数）。

4. **高级功能**：子命令（分层结构）、互斥组（参数互斥）、父解析器（参数复用）。

5. **最佳实践**：添加 help 文本、验证参数类型、提供默认值。
