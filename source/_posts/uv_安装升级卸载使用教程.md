---
title: uv使用一站式教程
date: 2026-04-13 14:30:00
tags: [UV, 工具使用, AI]
categories: 技术教程
toc: true
---

# uv 安装 / 升级 / 卸载 / 使用 专属详细教程

> 本文针对 UV 核心生命周期操作做了极致拆解，新手可全程跟着操作，老手可直接跳转对应模块查阅。



***

## 一、全平台安装教程

uv 支持多种安装方式，你可以根据自己的习惯选择，**优先推荐官方一键安装**，最省心。

### 1. 官方推荐一键安装（跨平台通用，90% 用户选这个）

这是官方最推荐的安装方式，自动适配你的系统，无需额外配置。

#### macOS / Linux 系统

打开终端，执行以下命令：



```
curl -LsSf https://astral.sh/uv/install.sh | sh
```

#### Windows 系统

以**普通用户权限**打开 PowerShell（无需管理员），执行以下命令：



```
irm https://astral.sh/uv/install.ps1 | iex
```

### 2. 系统包管理器安装（适合习惯包管理的用户）

如果你习惯用系统包管理器管理软件，可以选择对应命令：



| 系统 / 包管理器        | 安装命令                                                                                                   |
| ---------------- | ------------------------------------------------------------------------------------------------------ |
| macOS (Homebrew) | `brew install uv`                                                                                      |
| Windows (Winget) | `winget install astral-sh.uv`                                                                          |
| Windows (Scoop)  | `scoop install uv`                                                                                     |
| Debian/Ubuntu    | \`curl -fsSL [https://packages.astral.sh/deb/astral.asc](https://packages.astral.sh/deb/astral.asc)    |
| Fedora/RHEL      | `sudo dnf config-manager --add-repo https://packages.astral.sh/rpm/astral.repo && sudo dnf install uv` |

### 3. pip 安装（兼容所有已有 Python 环境）

如果你已经有 Python 环境，可直接用 pip 安装，和普通 Python 包一样：



```
pip install uv --upgrade
```

### 4. 手动二进制安装（适合自定义部署）

如果你想手动控制安装，可以直接从[官方 GitHub Release 页](https://github.com/astral-sh/uv/releases)下载对应系统的二进制文件，解压后把`uv`/`uv.exe`放到系统 PATH 目录下即可。

### 5. 安装验证

安装完成后，**关闭并重新打开终端**，执行以下命令，输出版本号即安装成功：



```
uv --version

# 示例输出：uv 0.5.10 (abcdef 2026-04-01)
```

### 6. 如何判断你的安装方式？

如果你忘了自己是怎么装的 UV，可通过以下命令判断：



```
# macOS/Linux 执行

which uv

# Windows PowerShell 执行

where uv
```

根据输出的路径，即可判断安装方式：



| 路径特征                                         | 安装方式                   |
| -------------------------------------------- | ---------------------- |
| `~/.local/bin/uv`                            | 官方一键安装                 |
| `/opt/homebrew/bin/uv` 或 `/usr/local/bin/uv` | Homebrew 安装            |
| `C:\Users\XXX\scoop\apps\uv\`                | Scoop 安装               |
| `C:\Program Files\uv\uv.exe`                 | Winget 安装              |
| 包含 `envs\` 或 `venv\` 路径                      | pip 安装在虚拟环境 / Conda 环境 |
| 自定义路径                                        | 手动二进制安装                |



***

## 二、升级 UV（重点：不同安装方式升级命令完全不同！）

⚠️ **警告：不要混用升级命令！** 比如 Homebrew 安装的不要用`uv self update`，否则会覆盖包管理器的文件，导致版本混乱！请根据上面的安装方式，选择对应的升级命令：

### 1. 官方一键安装的升级

如果你是用官方一键脚本装的，用 UV 自带的自更新命令，一键升级到最新版：



```
uv self update
```

### 2. 包管理器安装的升级

用你安装时用的包管理器来升级：



| 包管理器                | 升级命令                                     |
| ------------------- | ---------------------------------------- |
| Homebrew            | `brew upgrade uv`                        |
| Winget              | `winget upgrade astral-sh.uv`            |
| Scoop               | `scoop update uv`                        |
| APT (Debian/Ubuntu) | `sudo apt update && sudo apt upgrade uv` |
| DNF (Fedora/RHEL)   | `sudo dnf upgrade uv`                    |

### 3. pip 安装的升级

如果你是用 pip 装的，用 pip 升级即可：



```
pip install --upgrade uv
```

### 4. 手动安装的升级

手动下载最新的二进制文件，替换旧的文件即可。

### 升级注意事项



1. 升级 UV 不会影响你的项目、虚拟环境和依赖，锁文件`uv.lock`完全向后兼容，无需重新生成。

2. 如果升级后提示权限错误，**不要用 sudo 执行升级命令**，UV 是用户级工具，sudo 会导致权限混乱，删除旧的安装文件重新安装即可。



***

## 三、卸载 UV（彻底干净，不残留）

同样，根据你的安装方式，选择对应的卸载命令，卸载 UV**不会删除你的任何项目文件、虚拟环境或代码**，放心操作。

### 1. 官方一键安装的卸载

官方提供了专用的卸载脚本，一键彻底删除：



```
curl -LsSf https://astral.sh/uv/install.sh | sh -s -- --uninstall
```

执行后会自动删除 UV 的二进制、配置、缓存等所有文件。

### 2. 包管理器安装的卸载

用包管理器卸载：



| 包管理器                | 卸载命令                            |
| ------------------- | ------------------------------- |
| Homebrew            | `brew uninstall uv`             |
| Winget              | `winget uninstall astral-sh.uv` |
| Scoop               | `scoop uninstall uv`            |
| APT (Debian/Ubuntu) | `sudo apt remove uv`            |
| DNF (Fedora/RHEL)   | `sudo dnf remove uv`            |

### 3. pip 安装的卸载

用 pip 卸载即可：



```
pip uninstall uv
```

### 4. 手动安装的卸载

直接删除你下载的`uv`/`uv.exe`二进制文件即可。

### 5. 手动清理残留文件（如果卸载不干净）

如果卸载后还有残留，可手动删除以下目录（所有安装方式的残留都在这里）：



```
# macOS/Linux

rm -rf \~/.local/bin/uv

rm -rf \~/.local/share/uv

rm -rf \~/.config/uv

rm -rf \~/.cache/uv

# Windows（PowerShell）

Remove-Item -Recurse -Force \$env:USERPROFILE\\.local\bin\uv.exe

Remove-Item -Recurse -Force \$env:USERPROFILE\\.local\share\uv

Remove-Item -Recurse -Force \$env:APPDATA\uv

Remove-Item -Recurse -Force \$env:LOCALAPPDATA\uv\Cache
```

### 卸载注意事项



1. 卸载 UV 后，你之前用 UV 创建的虚拟环境、项目文件都还在，完全可以正常用，虚拟环境是标准 Python 格式，和 UV 无关。

2. 卸载不会删除你下载的 Python 版本（`uv python install`装的那些），如果要删，手动删`~/.local/share/uv/python`目录即可。



***

## 四、核心使用入门（装完就能用）

装完 UV 后，跟着以下步骤，5 分钟就能上手日常开发。

### 1. 必做：配置国内镜像源（解决下载慢）

国内用户必须配置，否则下载包会很慢，推荐全局配置，永久生效：



```
# 配置清华源（推荐）

uv config set pypi.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

### 2. 新项目从零开始完整流程



```
# 1. 创建并进入项目文件夹

mkdir my\_project && cd my\_project

# 2. 初始化UV项目（一键生成标准配置文件）

uv init

# 3. 锁定项目用的Python版本（比如用3.12）

uv python pin 3.12

# 4. 安装依赖（比如requests、pandas）

uv add requests pandas

# 5. 安装开发依赖（比如测试、格式化工具）

uv add --dev pytest black

# 6. 无需激活环境，直接运行代码！

uv run python main.py
```

### 3. 老项目（requirements.txt）迁移流程

如果你有老项目，用的是`requirements.txt`，1 分钟就能迁移到 UV：



```
# 1. 进入老项目文件夹

cd my\_old\_project

# 2. 初始化UV项目，不会覆盖你的现有文件

uv init --existing

# 3. 一键导入requirements.txt的所有依赖

uv add -r requirements.txt

# 4. 一键同步环境，自动创建虚拟环境、安装所有依赖

uv sync
```

搞定！现在你就有了标准的`pyproject.toml`和`uv.lock`，团队协作更方便。

### 4. 日常高频命令速查



| 场景                 | 命令                     |
| ------------------ | ---------------------- |
| 拉完代码同步环境           | `uv sync`              |
| 安装新依赖              | `uv add 包名`            |
| 运行代码 / 工具          | `uv run python xxx.py` |
| 升级所有依赖             | `uv lock --upgrade`    |
| 查看已安装包             | `uv pip list`          |
| 一键运行临时工具（比如 black） | `uvx black xxx.py`     |



***

## 五、常见问题排查

### 1. 升级后版本还是旧的？

原因：你系统里有多个 UV 版本，PATH 里旧的在前面。

解决：



* 卸载所有旧版本的 UV（用上面的卸载方法）

* 重新安装最新版

* 重启终端，重新加载 PATH

### 2. 卸载后还有 uv 命令？

原因：残留的旧二进制文件没删干净。

解决：执行上面的手动清理残留文件的命令，删除所有相关目录。

### 3. 安装 / 升级提示权限错误？

原因：你用了 sudo 执行命令，或者之前的文件权限乱了。

解决：



* **永远不要用 sudo 执行 uv 命令**，UV 是用户级工具

* 删除`~/.local/share/uv`目录，重新安装即可

### 4. Conda 环境里用 UV 有冲突？

避坑：不要在 Conda 的`base`环境里用 UV，Conda 会接管 Python 解释器，容易出冲突。建议用 UV 自己的 Python 版本管理，或者在 Conda 的虚拟环境里用。
