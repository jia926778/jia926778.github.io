---
title: conda使用教程
date: 2026-04-14 14:30:00
tags: [conda, 工具使用, python, AI]
categories: 技术教程
toc: true
---

# conda安装升级使用全教程


## 一、Miniconda 简介

Miniconda 是 Anaconda 的极简精简版，仅包含 `conda` 包管理器、Python 及其核心依赖，体积小、启动快、无冗余预装包，是 Python 环境管理和包管理的首选工具。

- 核心优势：**环境隔离**（彻底解决 Python 版本 / 包冲突）、跨平台兼容、一键式包管理，支持 Windows、macOS、Linux 全平台。

- 与 Anaconda 区别：Anaconda 预装了数百个科学计算包，体积超 3GB；Miniconda 仅保留核心能力，体积不足 100MB，所有包按需安装。

## 二、安装前准备

1. 系统要求：Windows 10+/macOS 10.15+/Linux 主流发行版，仅支持 64 位系统。

2. 关键注意事项：

    - 安装路径**严禁包含中文、空格、特殊字符**，避免后续出现未知报错。

    - 优先选择「仅为当前用户安装」，无需管理员权限，稳定性更高。

    - 无需提前安装 Python，Miniconda 会自带对应版本的 Python 解释器。

    - 安装前关闭杀毒软件 / 安全管家，避免拦截安装程序写入系统配置。

## 三、分系统详细安装步骤

### 3.1 Windows 系统安装

1. 下载安装包

    - 官方下载页：[https://docs.anaconda.net.cn/miniconda/install/](https://docs.anaconda.net.cn/miniconda/install/)

    - 最新版直接下载：Miniconda3-latest-Windows-x86_64.exe

    - 历史版本 / 其他架构归档：[https://repo.anaconda.com/miniconda](https://repo.anaconda.com/miniconda)

2. 图形化安装流程

    - 双击 exe 安装包，点击「Next」，点击「I Agree」接受许可协议。

    - 安装类型：选择「Just Me」（仅当前用户，新手强推）；如需给设备所有用户使用，选「All Users」（需管理员权限）。

    - 选择安装路径：默认路径为`C:\Users\用户名\miniconda3`，可自定义到 D 盘等非系统盘，确保路径无中文和空格。

    - 高级选项设置：

        - 新手推荐：勾选「Add Miniconda3 to my PATH environment variable」（添加到系统环境变量，官方默认不推荐，但可避免手动配置环境变量的麻烦）。

        - 必选：勾选「Register Miniconda3 as my default Python 3.xx」。

        - 其余选项保持默认，点击「Install」等待安装完成。

    - 安装结束后，点击「Next」，取消所有勾选，点击「Finish」完成安装。

3. PowerShell 适配（可选）
打开「开始菜单 - Anaconda Prompt」，执行`conda init powershell`，重启 PowerShell 即可正常使用 conda 命令。

### 3.2 macOS 系统安装

1. 下载安装包

    - 芯片区分：Intel 芯片选 x86_64 版本，Apple M 系列芯片选 Apple Silicon 版本。

    - 终端一键下载：

        ```bash

        curl https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-$(uname -m).sh -o ~/Downloads/miniconda.sh
        ```

2. 终端安装流程

    - 赋予安装包执行权限：

        ```bash

        chmod +x ~/Downloads/miniconda.sh
        ```

    - 执行安装脚本：

        ```bash

        bash ~/Downloads/miniconda.sh
        ```

    - 交互式操作步骤：

        1. 按回车查看许可协议，拉到文末输入`yes`接受协议。

        2. 确认安装路径，默认`~/miniconda3`，可自定义路径，回车确认。

        3. 询问是否执行`conda init`时，输入`yes`（关键步骤，否则终端无法识别 conda 命令）。

    - 安装完成后，重启终端，配置自动生效。

### 3.3 Linux 系统安装

1. 下载安装包

    - x86_64 架构终端一键下载：

        ```bash

        curl https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -o ~/Downloads/miniconda.sh
        ```

    - ARM 架构：将链接中的`x86_64`替换为`aarch64`即可。

2. 终端安装流程

    - 赋予执行权限：

        ```bash

        chmod +x ~/Downloads/miniconda.sh
        ```

    - 执行安装脚本：

        ```bash

        bash ~/Downloads/miniconda.sh
        ```

    - 交互式操作：和 macOS 完全一致，接受协议、确认安装路径、`conda init`选`yes`。

    - 生效方式：重启终端，或执行`source ~/.bashrc`（bash 终端）/`source ~/.zshrc`（zsh 终端）立即生效。

## 四、安装成功验证

打开终端（Windows 用 Anaconda Prompt/cmd/PowerShell，macOS/Linux 用系统终端），执行以下命令，有正常版本 / 信息输出即安装成功：

```bash

# 查看conda版本，验证命令是否生效
conda --version

# 查看conda详细信息，包括安装路径、镜像源等
conda info

# 查看已安装的基础包
conda list
```

若提示`conda: 未找到命令`/`conda不是内部或外部命令`，参考文末常见问题解决方案。

## 五、国内镜像源配置（必做，解决下载慢 / 超时）

conda 默认使用国外官方源，国内访问速度极慢、频繁超时，必须更换为国内镜像源，推荐清华大学 TUNA 镜像源，以下提供两种配置方式。

### 5.1 一键命令行配置（新手推荐）

打开终端，依次执行以下命令，自动写入配置文件，无需手动修改：

```bash

# 清除原有镜像配置（可选，避免新旧配置冲突）
conda config --remove-key channels

# 添加清华镜像核心源
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/bioconda/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/

# 下载时显示包的来源地址，便于排查问题
conda config --set show_channel_urls yes

# 关闭默认defaults源，强制使用镜像源，避免回退到国外源
conda config --set default_channels_override https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/ https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r/
```

### 5.2 手动修改配置文件

1. 找到配置文件路径：

    - Windows：`C:\Users\你的用户名.condarc`

    - macOS/Linux：`~/.condarc`

2. 用文本编辑器打开文件，清空原有内容，粘贴以下配置并保存：

    ```yaml

    channels:
      - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/
      - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
      - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
      - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r/
      - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/
      - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/bioconda/
    show_channel_urls: yes
    default_channels_override:
      - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
      - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r/
    ```

3. 重启终端，配置即可生效。

### 5.3 配置验证

执行`conda config --show channels`，查看输出的镜像源是否为上述配置的清华源，确认无误即配置成功。

## 六、Conda 升级操作

### 6.1 升级 Conda 自身（核心）

**必须在 base 环境中执行**，这是官方唯一推荐的升级方式，**严禁使用 pip 升级 conda**，否则会导致环境损坏。

1. 激活 base 环境：

    ```bash

    conda activate base
    ```

2. 执行升级命令：

    ```bash

    # 稳定版升级（日常使用推荐）
    conda update -n base conda

    # 强制重装升级到最新版（版本异常时使用）
    conda update -n base conda --force-reinstall
    ```

3. 升级完成后，执行`conda --version`即可查看新版本号。

### 6.2 升级 Python 版本

⚠️ 注意：Python 大版本升级可能导致部分包不兼容，**推荐新建环境而非直接升级现有环境的 Python 版本**。

1. 升级 base 环境的 Python：

    ```bash

    # 升级到指定大版本，例如3.12
    conda install python=3.12
    ```

2. 升级指定虚拟环境的 Python：

    ```bash

    # 先激活目标环境
    conda activate 环境名
    # 升级到指定Python版本
    conda install python=3.11
    ```

### 6.3 升级环境内的所有包

```bash

# 升级当前激活环境的所有包
conda update --all

# 升级指定环境的所有包，无需提前激活
conda update -n 环境名 --all
```

## 七、Conda 核心使用教程

Conda 的核心能力是**环境管理**和**包管理**，也是日常使用最频繁的功能，以下是全场景常用命令。

### 7.1 环境管理（核心）

环境隔离是 Conda 最大的优势，为每个项目创建独立环境，彻底解决不同项目间的版本冲突问题。

#### 7.1.1 查看所有环境

```bash

conda env list
# 等价命令
conda info --envs
```

输出结果中，带`*`的是当前激活的环境，默认初始环境为 base。

#### 7.1.2 创建虚拟环境

```bash

# 基础创建（指定环境名+Python版本，必选参数）
conda create -n 环境名 python=3.11
# 示例：创建名为ai_project的环境，指定Python3.10版本
conda create -n ai_project python=3.10

# 创建环境时同时预装多个包
conda create -n ai_project python=3.10 numpy pandas pytorch

# 从yml配置文件创建环境（团队协作/环境迁移推荐）
conda env create -f environment.yml
```

⚠️ 新手必看：**不要在 base 环境中安装项目相关包**，base 环境仅用于管理 conda 本身，每个项目单独创建专属环境。

#### 7.1.3 激活 / 切换环境

```bash

# 激活指定环境
conda activate 环境名
# 示例：激活ai_project环境
conda activate ai_project
```

激活成功后，终端前缀会显示`(环境名)`，此时所有安装 / 卸载 / 更新操作，都仅作用于当前激活的环境。

```bash

# 无需先退出，直接切换到另一个环境
conda activate 另一个环境名
```

#### 7.1.4 退出环境

```bash

# 退出当前环境，回到base环境
conda deactivate
```

#### 7.1.5 克隆环境

```bash

# 克隆现有环境，生成全新的独立环境
conda create -n 新环境名 --clone 原环境名
# 示例：把ai_project克隆为备份环境ai_project_backup
conda create -n ai_project_backup --clone ai_project
```

#### 7.1.6 导出 / 导入环境（团队协作 / 迁移必备）

```bash

# 导出当前激活环境的完整配置到yml文件
conda env export > environment.yml

# 仅导出用户手动安装的包，去除系统相关依赖（跨平台迁移推荐）
conda env export --from-history > environment.yml
```

```bash

# 从yml配置文件导入，一键创建完全一致的环境
conda env create -f environment.yml
```

#### 7.1.7 删除环境

```bash

# 彻底删除指定环境及环境内所有包（谨慎操作！）
conda remove -n 环境名 --all
# 等价命令
conda env remove -n 环境名
```

### 7.2 包管理

用于在指定环境中安装、卸载、更新 Python 包，**优先使用 conda 安装，conda 源中没有的包，再用 pip 安装**。
⚠️ 关键原则：**先激活目标环境，再执行包管理操作**，避免包安装到错误的环境中。

#### 7.2.1 安装包

```bash

# 安装最新版的包
conda install 包名
# 示例：安装numpy
conda install numpy

# 安装指定版本的包
conda install numpy=1.26.0

# 同时安装多个包
conda install numpy pandas matplotlib

# 从指定渠道安装（例如conda-forge社区源）
conda install -c conda-forge 包名

# 不激活环境，直接给指定环境安装包
conda install -n 环境名 包名
```

#### 7.2.2 查看已安装的包

```bash

# 查看当前激活环境的所有已安装包
conda list

# 查看指定环境的所有已安装包
conda list -n 环境名

# 搜索云端可安装的包，查看所有可用版本
conda search 包名
```

#### 7.2.3 更新包

```bash

# 更新当前环境的指定包
conda update 包名

# 更新指定环境的指定包
conda update -n 环境名 包名

# 更新当前环境的所有包
conda update --all
```

#### 7.2.4 卸载包

```bash

# 卸载当前环境的指定包
conda remove 包名

# 卸载指定环境的指定包
conda remove -n 环境名 包名

# 同时卸载多个包
conda remove numpy pandas
```

### 7.3 常用维护命令

```bash

# 清理conda缓存、无用安装包和孤立包，释放磁盘空间
conda clean -y --all

# 查看conda所有配置
conda config --show

# 重置conda镜像源配置为默认状态
conda config --remove-key channels
```

## 八、常见问题与解决方案

### 问题 1：终端提示`conda: 未找到命令`/`conda不是内部或外部命令`

**原因**：环境变量未配置，或 conda init 未执行。
**解决方案**：

1. Windows：

    - 方法 1：直接使用「开始菜单」里的「Anaconda Prompt」，自带完整环境配置，无需额外设置。

    - 方法 2：手动添加环境变量：将 Miniconda 安装目录下的`Scripts`、`bin`、`Library/bin`三个文件夹路径，添加到系统 PATH 环境变量，重启终端即可。

2. macOS/Linux：

    - 执行`~/miniconda3/bin/conda init`（默认安装路径），重启终端。

    - 若使用 zsh 终端，额外执行`~/miniconda3/bin/conda init zsh && source ~/.zshrc`。

### 问题 2：下载包时速度极慢、超时、报错 CondaHTTPError

**原因**：使用了国外官方源，国内网络访问受限。
**解决方案**：

1. 按照教程第五部分配置国内清华镜像源。

2. 配置后执行`conda clean -i`清除索引缓存，重新执行安装命令。

3. 检查系统代理 / VPN，关闭代理，或配置镜像源地址绕过代理。

### 问题 3：Windows PowerShell 无法激活环境，执行 conda activate 报错

**原因**：PowerShell 执行策略限制，或未完成 conda 初始化。
**解决方案**：

1. 以管理员身份打开 PowerShell。

2. 执行`Set-ExecutionPolicy RemoteSigned`，输入`Y`确认修改执行策略。

3. 打开 Anaconda Prompt，执行`conda init powershell`。

4. 重启 PowerShell，即可正常使用 conda 激活命令。

### 问题 4：安装包时报错`Solving environment: failed`（包依赖冲突）

**原因**：包版本依赖不兼容，或安装渠道不匹配。
**解决方案**：

1. 优先使用 conda-forge 渠道安装：`conda install -c conda-forge 包名`。

2. 新建干净的虚拟环境，重新安装所需包，避免环境内包过多导致的依赖冲突。

3. 明确指定包的版本号，缩小 conda 的依赖匹配范围，避免冲突。

### 问题 5：macOS/Linux 重启终端后，conda 命令失效

**原因**：conda init 未写入对应 shell 的配置文件。
**解决方案**：

1. 执行`echo $SHELL`查看当前使用的终端类型。

2. 若为 bash 终端：执行`~/miniconda3/bin/conda init bash && source ~/.bashrc`。

3. 若为 zsh 终端：执行`~/miniconda3/bin/conda init zsh && source ~/.zshrc`。

4. 重启终端即可永久生效。

## 九、最佳实践

1. **严格环境隔离**：一个项目对应一个独立虚拟环境，绝不把所有包都装在 base 环境，避免环境污染和版本冲突。

2. **环境版本锁定**：项目交付 / 团队协作时，用`conda env export --from-history > environment.yml`导出环境配置，确保所有人的运行环境完全一致。

3. **安装渠道规范**：优先使用 main、conda-forge 等官方 / 社区正规渠道，避免使用未知第三方渠道，降低安全风险。

4. **定期环境维护**：定期执行`conda clean -y --all`清理缓存，释放磁盘空间；base 环境定期更新 conda 版本，获取最新功能和安全修复。

5. **conda 与 pip 配合规范**：优先用 conda 安装包，conda 源中没有的包，再激活对应环境后用 pip 安装；严禁在 base 环境用 pip 安装大量包，避免破坏 conda 依赖体系。

---
