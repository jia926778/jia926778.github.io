---
title: Windows系统Docker使用教程
date: 2026-04-16 21:30:00
tags: [Docker, 技术分享, AI大模型]
categories: 工具使用
toc: true
---
# Windows系统Docker使用教程

本文档整合了 Windows 系统下 Docker Desktop 从安装、配置、使用到问题排查的全流程内容，覆盖 WSL 2 后端全场景，适配 Windows 10/11 64 位系统。

---

## 目录

1. \[Docker Desktop 安装教程\]\(\#1\-docker\-desktop\-安装教程\)

2. \[Docker Desktop 升级教程\]\(\#2\-docker\-desktop\-升级教程\)

3. \[Docker Desktop 卸载教程\]\(\#3\-docker\-desktop\-卸载教程\)

4. \[Docker 核心使用教程\]\(\#4\-docker\-核心使用教程\)

5. \[Docker 地址修改完整指南\]\(\#5\-docker\-地址修改完整指南\)

6. \[查看 Docker 运行后端类型\]\(\#6\-查看docker运行后端类型\)

7. \[Docker Compose YAML 配置详解\]\(\#7\-docker\-compose\-yaml\-配置详解\)

8. \[Docker 常见问题排查手册\]\(\#8\-docker\-常见问题排查手册\)

---

## 1\. Docker Desktop 安装教程

### 1\.1 前置准备与系统要求

#### 系统与硬件最低要求

|项目|最低要求|推荐配置|
|---|---|---|
|操作系统|Windows 10 64 位 22H2（内部版本 19045\+）/ Windows 11 64 位 22H2\+|Windows 11 专业版 23H2\+|
|CPU|支持 SLAT 二级地址转换、Intel VT\-x/AMD\-V 虚拟化的 64 位处理器|4 核 8 线程及以上|
|内存|4GB 系统内存|8GB\+ 内存|
|存储|10GB 可用空间|SSD 20GB\+ 可用空间|
|核心依赖|启用 WSL 2 或 Hyper\-V|WSL 2 最新内核|

#### 关键前置检查与配置

##### 步骤 1：确认硬件虚拟化已开启

1. 按下 `Ctrl\+Shift\+Esc` 打开任务管理器 → 切换到「性能」→「CPU」

2. 查看「虚拟化」是否显示**已启用**

3. 若显示禁用：重启电脑，按 Del/F2/F10（依主板 / 品牌而定）进入 BIOS，开启`Intel Virtual Technology \(VT\-x\)`或`AMD\-V`，保存重启

##### 步骤 2：安装并启用 WSL 2

以**管理员身份**打开 PowerShell / 终端，执行以下命令：

```powershell
# 一键安装WSL 2（自动启用相关Windows功能、下载内核、设置默认版本）
wsl --install

# 若已安装旧版WSL，执行更新并设置默认版本
wsl --update
wsl --set-default-version 2

# 验证WSL版本，输出VERSION为2即为正常
wsl -l -v
```

执行完成后**重启电脑**，确保 WSL 2 组件生效。

### 1\.2 详细安装教程

#### 方式 1：图形化界面安装（推荐新手）

1. **下载安装包**
访问[Docker 官方下载页](https://www.docker.com/products/docker-desktop/)，点击`Download for Windows`，获取最新版安装包`Docker Desktop Installer\.exe`。

2. **执行安装**
双击安装包，在安装向导中：

    - 必选：勾选`Use WSL 2 instead of Hyper\-V`（默认勾选，家庭版系统仅支持此选项）

    - 可选：勾选`Add shortcut to desktop`创建桌面快捷方式

    - 点击`OK`，等待安装完成（约 2\-5 分钟）

    - 安装完成后，点击`Close and restart`按需重启电脑。

3. **首次启动与协议确认**
从开始菜单 / 桌面打开 Docker Desktop，首次启动会弹出服务协议，勾选接受协议后点击`Accept`；等待底部状态栏鲸鱼图标从`Starting`变为`Running`，即启动成功。

4. **安装验证**
打开 PowerShell/CMD，执行以下命令，输出版本号即安装成功：

    ```powershell
    # 查看Docker版本
    docker --version
    docker compose version

    # 运行测试容器，正常拉取并输出hello world即环境完全正常
    docker run hello-world
    ```

#### 方式 2：命令行静默安装（适合批量部署）

以管理员身份打开 PowerShell，执行以下命令：

```powershell
# 下载安装包（可替换为最新版本链接）
Invoke-WebRequest "https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe" -OutFile "$env:TEMP\DockerInstaller.exe"

# 静默安装，默认WSL 2后端
Start-Process "$env:TEMP\DockerInstaller.exe" -Wait -ArgumentList "install --quiet --backend=wsl-2"
```

安装完成后，重启终端即可使用 docker 命令。

---

## 2\. Docker Desktop 升级教程

### 2\.1 重要前置：升级前数据备份

升级不会默认删除镜像、容器、卷数据，但为避免异常，建议关键数据提前备份：

```powershell
# 1. 备份本地镜像到tar包
docker image save -o my-images.tar 镜像名1:标签 镜像名2:标签

# 2. 备份命名卷数据（以mysql卷为例）
docker run --rm -v mysql_data:/source -v D:\backup:/dest alpine tar -czvf /dest/mysql_data.tar.gz -C /source .

# 3. 导出容器配置（可选）
docker inspect 容器名 > container-config.json
```

恢复时可通过`docker image load \-i my\-images\.tar`还原镜像。

### 2\.2 升级方式

#### 方式 1：自动升级（推荐）

Docker Desktop 默认开启自动更新检测，操作步骤：

1. 打开 Docker Desktop，右上角若出现蓝色更新提示，点击进入更新页面

2. 点击`Download update`，等待后台下载更新包

3. 下载完成后，点击`Apply and restart`，Docker 会自动完成升级并重启

4. 重启后，执行`docker \-\-version`验证版本是否更新成功。

可通过「设置」→「Software updates」自定义更新策略：

- `Automatically check for updates`：开启更新检测（默认开启）

- `Always download updates`：后台自动下载更新包

- `Automatically update components`：自动更新 Compose、CLI 等组件（无需完整重启）

#### 方式 2：手动覆盖升级

适用于自动升级失败、需跨大版本升级的场景：

1. 关闭正在运行的 Docker Desktop（右键托盘图标→Quit Docker Desktop）

2. 从 Docker 官网下载最新版安装包

3. 双击安装包，执行安装（无需卸载旧版本，安装程序会自动覆盖升级）

4. 安装完成后启动 Docker，所有原有镜像、容器、配置均会保留。

---

## 3\. Docker Desktop 卸载教程

### 3\.1 标准卸载（保留数据）

适用于临时卸载、后续可能重装的场景：

1. 完全关闭 Docker Desktop（右键托盘图标→Quit Docker Desktop）

2. 打开 Windows「设置」→「应用」→「已安装的应用」

3. 在列表中找到`Docker Desktop`，点击右侧三个点→「卸载」

4. 跟随卸载向导，点击确认，等待卸载完成。

也可通过命令行执行标准卸载：

```powershell
# 方式1：通过安装包卸载
Start-Process 'C:\Program Files\Docker\Docker\Docker Desktop Installer.exe' -Wait uninstall

# 方式2：通过winget卸载
winget uninstall Docker.DockerDesktop
```

### 3\.2 彻底卸载（清除所有残留）

适用于重装报错、环境异常、完全清理的场景，步骤如下：

1. 执行上述**标准卸载**步骤，完成主程序卸载

2. 清理 WSL 2 相关分发（会删除所有镜像、容器、卷数据，不可逆）

    ```powershell
    # 停止所有WSL实例
    wsl --shutdown

    # 注销Docker专用WSL分发
    wsl --unregister docker-desktop
    wsl --unregister docker-desktop-data

    # 验证是否注销成功，无相关名称即完成
    wsl -l -v
    ```

3. 删除残留文件目录
以管理员身份打开 PowerShell，执行以下命令删除所有残留目录：

    ```powershell
    # 删除程序安装目录
    rm -rf "C:\Program Files\Docker"
    rm -rf "C:\ProgramData\Docker"
    rm -rf "C:\ProgramData\DockerDesktop"

    # 删除用户缓存与配置
    rm -rf "$env:LOCALAPPDATA\Docker"
    rm -rf "$env:APPDATA\Docker"
    rm -rf "$env:APPDATA\Docker Desktop"
    rm -rf "$env:USERPROFILE\.docker"
    ```

4. 清理环境变量与注册表（高级操作，谨慎执行）

    - 环境变量：右键「此电脑」→「属性」→「高级系统设置」→「环境变量」，删除系统 / 用户变量中所有包含`DOCKER`的变量项。

    - 注册表：按下`Win\+R`输入`regedit`打开注册表编辑器，删除以下路径（操作前建议备份注册表）：

        ```Plain Text
        HKEY_LOCAL_MACHINE\SOFTWARE\Docker Inc.
        HKEY_CURRENT_USER\SOFTWARE\Docker Inc.
        HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Docker
        ```

5. 重启电脑，完成彻底卸载。

---

## 4\. Docker 核心使用教程

### 4\.1 国内镜像源配置（必做）

Docker 官方镜像源在国内访问受限，需配置国内加速源：

1. 打开 Docker Desktop → 右上角「设置」→ 左侧「Docker Engine」

2. 在右侧 JSON 配置中，添加`registry\-mirrors`字段，完整配置示例：

    ```json
    {
      "builder": {
        "gc": {
          "defaultKeepStorage": "20GB",
          "enabled": true
        }
      },
      "experimental": false,
      "registry-mirrors": [
        "https://docker.m.daocloud.io",
        "https://docker.1ms.run",
        "https://mirror.ccs.tencentyun.com",
        "https://docker.mirrors.ustc.edu.cn",
        "https://docker.xuanyuan.me",
        "https://docker.xiaogenban1993.com"
      ]
    }
    ```

3. 点击右下角「Apply \&amp; Restart」，等待 Docker 重启生效

4. 验证配置是否成功，执行以下命令，输出中包含配置的镜像地址即生效：

    ```powershell
    docker info | findstr "Registry Mirrors"
    ```

### 4\.2 核心概念速记

- **镜像（Image）**：容器的 “安装包 / 模板”，只读，包含程序运行所需的代码、环境、依赖，例如 nginx、mysql 镜像。

- **容器（Container）**：镜像运行后的实例，独立的运行环境，可启动、停止、删除，每个容器相互隔离。

- **仓库（Registry）**：存储和分发镜像的服务，例如 Docker Hub、国内镜像仓库。

- **Docker Compose**：用于定义和运行多容器应用的工具，通过 yaml 文件配置所有服务，一键启动 / 停止。

### 4\.3 镜像管理高频命令

```powershell
# 1. 搜索镜像（也可直接在Docker Hub网页搜索）
docker search nginx

# 2. 拉取镜像（不指定标签默认拉取latest最新版）
docker pull nginx:latest
docker pull mysql:8.0

# 3. 查看本地所有镜像
docker images

# 4. 删除本地镜像（镜像被容器占用时需先删除容器）
docker rmi nginx:latest
docker rmi 镜像ID

# 5. 清理无用镜像（释放磁盘空间）
docker image prune -a
```

### 4\.4 容器管理高频命令

#### 核心运行命令详解

```powershell
# 基础格式：docker run [选项] 镜像名 [命令]
# 示例：运行nginx容器，端口映射、后台运行、自定义名称
docker run -d --name my-nginx -p 8080:80 nginx:latest
```

常用参数说明：

- `\-d`：后台守护进程运行容器

- `\-\-name 容器名`：给容器自定义名称，便于管理

- `\-p 主机端口:容器端口`：端口映射，将主机端口绑定到容器端口，外部可通过主机端口访问容器服务

- `\-v 主机路径:容器路径`：数据卷挂载，实现数据持久化（容器删除数据不丢失）

- `\-it`：交互式运行，进入容器终端，通常搭配`/bin/bash`使用

- `\-\-rm`：容器退出后自动删除，常用于临时测试

#### 高频容器操作命令

```powershell
# 1. 查看运行中的容器
docker ps
# 查看所有容器（包括已停止的）
docker ps -a

# 2. 启动/停止/重启容器
docker start my-nginx
docker stop my-nginx
docker restart my-nginx

# 3. 进入运行中的容器（推荐exec，退出后容器不会停止）
docker exec -it my-nginx /bin/bash

# 4. 查看容器日志（排查问题必备）
docker logs my-nginx
# 实时查看日志
docker logs -f --tail 100 my-nginx

# 5. 主机与容器之间文件复制
# 容器文件复制到主机
docker cp my-nginx:/etc/nginx/nginx.conf D:\docker\nginx.conf
# 主机文件复制到容器
docker cp D:\docker\nginx.conf my-nginx:/etc/nginx/nginx.conf

# 6. 删除容器（必须先停止容器，或加-f强制删除）
docker rm my-nginx
docker rm -f 容器ID
# 批量删除所有已停止的容器
docker container prune

# 7. 查看容器资源占用
docker stats my-nginx
```

### 仓库管理操作(Registry)

```powershell
# 登录镜像仓库（如 Docker Hub、阿里云 ACR）
docker login registry.cn-hangzhou.aliyuncs.com
# 登出
docker logout registry.cn-hangzhou.aliyuncs.com

# 给本地镜像打标签（格式：仓库地址/命名空间/仓库名:标签）
docker tag 本地镜像ID/镜像名:标签 registry.cn-hangzhou.aliyuncs.com/命名空间/仓库名:标签

# 推送镜像到仓库
docker push registry.cn-hangzhou.aliyuncs.com/命名空间/仓库名:标签

# 从仓库拉取镜像
docker pull registry.cn-hangzhou.aliyuncs.com/命名空间/仓库名:标签
```

### 数据卷（Volume）

```powershell
# 创建数据卷
docker volume create 数据卷名

# 查看所有数据卷
docker volume ls

# 查看数据卷详细信息
docker volume inspect 数据卷名

# 删除未使用的数据卷
docker volume prune

# 启动容器时挂载数据卷（推荐方式，数据持久化）
docker run -d -v 数据卷名:容器内路径 --name 容器名 镜像名:标签
```

### 网络（Network）

```powershell
# 查看所有网络
docker network ls

# 创建自定义网络（推荐用于容器间通信）
docker network create 网络名

# 查看网络详细信息
docker network inspect 网络名

# 启动容器时加入自定义网络
docker run -d --network 网络名 --name 容器名 镜像名:标签

# 将运行中的容器加入网络
docker network connect 网络名 容器ID/容器名

# 清理未使用的网络
docker network prune
```

### 4\.5 数据持久化（核心必备）

容器删除后，容器内产生的数据会全部丢失，需通过**数据卷挂载**实现数据持久化，两种常用方式：

#### 方式 1：命名卷挂载（推荐，Docker 统一管理）

```powershell
# 示例：运行mysql容器，将数据库数据持久化到命名卷mysql_data
docker run -d --name my-mysql \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -v mysql_data:/var/lib/mysql \
  mysql:8.0
```

命名卷优势：无需关心主机路径，Docker 自动管理，支持跨容器共享数据，备份 / 迁移更便捷。

#### 方式 2：绑定挂载（主机路径挂载，适合配置文件修改）

```powershell
# 示例：运行nginx容器，挂载主机配置文件和网页目录
docker run -d --name my-nginx \
  -p 8080:80 \
  -v D:\docker\nginx\nginx.conf:/etc/nginx/nginx.conf \
  -v D:\docker\nginx\html:/usr/share/nginx/html \
  nginx:latest
```

绑定挂载优势：可直接在主机上修改文件，容器内实时生效，适合开发调试场景。

---

## 5\. Docker 地址修改完整指南

Docker 的**镜像拉取下载源地址**（从哪里下载镜像）和**本地镜像 / 容器 / 数据的存储路径**（下载的文件存在本地哪里），**都完全支持自定义修改**。

### 5\.1 修改镜像文件下载地址（镜像拉取源 / 加速源）

这个配置修改的是拉取 Docker 镜像时的上游仓库服务器地址，核心解决官方源国内访问慢、拉取超时 / 失败的问题，配置一次永久生效。

#### 详细设置步骤

1. 打开 Docker Desktop，点击右上角齿轮图标进入**设置 \(Settings\)**；

2. 左侧菜单选择**Docker Engine**，进入 Docker 守护进程配置页；

3. 在右侧的 JSON 编辑框中，找到或新增`registry\-mirrors`字段，填入国内可用的镜像加速地址，完整配置示例如下（2026 年实测可用）：

    ```json
    {
      "builder": {
        "gc": {
          "defaultKeepStorage": "20GB",
          "enabled": true
        }
      },
      "experimental": false,
      "registry-mirrors": [
        "https://docker.m.daocloud.io",
        "https://docker.1ms.run",
        "https://mirror.ccs.tencentyun.com",
        "https://docker.mirrors.ustc.edu.cn",
        "https://docker.xuanyuan.me",
        "https://docker.xiaogenban1993.com"
      ],
      "dns": ["223.5.5.5", "8.8.8.8"]
    }
    ```

4. 点击右下角**Apply \&amp; Restart**，等待 Docker 自动重启，配置即可生效；

5. 验证配置是否生效：以管理员身份打开 PowerShell，执行以下命令，输出中包含你配置的镜像地址即为成功。

    ```powershell
    docker info | findstr "Registry Mirrors"
    ```

### 5\.2 修改 Docker 本地默认存储路径

Docker 默认将镜像、容器、数据卷、日志等所有数据存放在系统 C 盘，长期使用会占用大量 C 盘空间，可通过以下方法迁移到其他磁盘，适配不同 Docker 运行后端。

#### 核心说明

Windows 系统 Docker Desktop 主流使用**WSL 2 后端**（官方推荐，家庭 / 专业版通用），其数据默认存放在 WSL 的虚拟磁盘文件中，路径为：
`C:\\Users\\你的用户名\\AppData\\Local\\Docker\\wsl\\data\\ext4\.vhdx`
所有镜像、容器、卷数据都在这个虚拟磁盘里，迁移核心是对`docker\-desktop\-data`这个 WSL 发行版进行导出 \+ 重新导入到新盘符。

#### 方案 1：WSL 2 后端 已安装环境迁移（推荐，保留所有数据）

##### 前置准备

1. 备份关键数据（避免迁移异常导致数据丢失）：

    ```powershell
    # 备份本地镜像到tar包（可选，重要镜像建议备份）
    docker save -o D:\Docker_Backup\my-images.tar 镜像名1:标签 镜像名2:标签
    # 查看所有WSL发行版，确认存在docker-desktop和docker-desktop-data
    wsl -l -v
    ```

2. 完全关闭 Docker Desktop：右键任务栏托盘 Docker 图标，选择**Quit Docker Desktop**，等待图标完全消失，确保进程全部退出；

3. 提前在目标磁盘创建文件夹，例如`D:\\Docker\\wsl\\data`（用于存放新的虚拟磁盘）、`D:\\Docker\_Backup`（用于存放临时备份包）。

##### 完整迁移步骤

以管理员身份打开 PowerShell，依次执行以下命令：

1. 关闭所有正在运行的 WSL 实例

    ```powershell
    wsl --shutdown
    ```

2. 导出 docker\-desktop\-data 数据到备份 tar 包（保留所有镜像、容器、卷数据）

    ```powershell
    # 路径替换为你创建的备份文件夹路径
    wsl --export docker-desktop-data D:\Docker_Backup\docker-desktop-data.tar
    ```

3. 注销原 C 盘的 docker\-desktop\-data 发行版（释放 C 盘空间）

    ```powershell
    wsl --unregister docker-desktop-data
    ```

4. 重新导入 docker\-desktop\-data 到新的磁盘路径

    ```powershell
    # 第一个路径：新的存储目录，第二个路径：上一步导出的tar包路径
    wsl --import docker-desktop-data D:\Docker\wsl\data D:\Docker_Backup\docker-desktop-data.tar --version 2
    ```

5. 导入完成后，重新启动 Docker Desktop，等待服务启动完成；

6. 验证迁移是否成功：

    - 执行`docker images`，查看原有镜像是否正常显示；

    - 执行`docker run hello\-world`，测试容器运行正常；

    - 查看新路径`D:\\Docker\\wsl\\data`下是否生成`ext4\.vhdx`虚拟磁盘文件；

7. 确认迁移完全成功后，可删除`D:\\Docker\_Backup\\docker\-desktop\-data\.tar`临时备份包，释放磁盘空间。

#### 方案 2：WSL 2 后端 全新安装时直接指定存储路径

适合还未安装 Docker Desktop 的用户，一步到位指定存储路径，无需后续迁移：

1. 提前完成 WSL 2 安装配置，在目标磁盘创建安装目录，例如`D:\\Program Files\\Docker`、`D:\\Program Files\\Docker\\data`；

2. 从 Docker 官网下载最新安装包`Docker Desktop Installer\.exe`，放到指定目录；

3. 以管理员身份打开 PowerShell，导航到安装包所在目录，执行以下静默安装命令，直接指定程序安装路径和数据存储路径：

    ```powershell
    # 替换为你的实际路径，双反斜杠转义
    start /w "" "Docker Desktop Installer.exe" install -accept-license `
    --installation-dir="D:\\Program Files\\Docker" `
    --wsl-default-data-root="D:\\Program Files\\Docker\\data"
    ```

4. 等待安装完成，启动 Docker 后，数据会默认存放在你指定的路径，无需额外配置。

#### 方案 3：Hyper\-V 后端 图形化修改路径（Windows 专业 / 企业版）

如果你的 Docker 使用 Hyper\-V 虚拟机后端，可直接通过图形界面修改：

1. 打开 Docker Desktop，进入**设置 \(Settings\)**；

2. 左侧菜单选择**Resources** → **Advanced**；

3. 找到**Disk image location**选项，点击**Browse**，选择目标磁盘的文件夹（例如`D:\\Docker\\Hyper\-V`）；

4. 点击**Apply \&amp; Restart**，Docker 会自动将虚拟磁盘迁移到新路径，等待重启完成即可生效。

#### 方案 4：Windows 容器模式 修改存储路径

如果你运行的是 Windows 原生容器，可通过修改 daemon 配置指定存储路径：

1. 打开 Docker Desktop，右键托盘图标，选择**Switch to Windows containers**，切换到 Windows 容器模式；

2. 进入**设置 \(Settings\)** → **Docker Engine**；

3. 在 JSON 配置中添加`data\-root`字段，指定新的存储路径，示例如下：

    ```json
    {
      "experimental": false,
      "data-root": "D:\\Docker\\windows-containers"
    }
    ```

    注意：路径必须用双反斜杠转义，且提前手动创建好目标文件夹；

4. 点击**Apply \&amp; Restart**，重启 Docker 生效；

5. 验证：执行`docker info \| findstr \&\#34;Docker Root Dir\&\#34;`，输出路径与你配置的一致即为成功。

---

## 6\. 查看 Docker 运行后端类型

Windows 系统下 Docker Desktop 主要支持 **WSL 2**（官方推荐，性能最优）和 **Hyper\-V**（仅 Windows 专业 / 企业版可选）两种后端，可通过以下 3 种方法快速确认当前使用的后端类型。

### 方法 1：通过 Docker Desktop 图形界面查看（最直观）

1. 打开 Docker Desktop，点击右上角齿轮图标进入 **设置 \(Settings\)**；

2. 在左侧菜单选择 **General**（通用）；

3. 查看右侧面板中的 **Use the WSL 2 based engine** 选项：

    - ✅ 若该选项**已勾选**，且下方显示`WSL 2 is enabled`，则当前后端为 **WSL 2**；

    - ❌ 若该选项**未勾选**，则当前后端为 **Hyper\-V**（仅 Windows 专业 / 企业版可见此状态）。

### 方法 2：通过`docker info`命令查看（最准确）

以管理员身份打开 PowerShell/CMD，执行以下命令：

```powershell
docker info
```

在输出结果中查找以下关键信息：

- **WSL 2 后端**：会在输出中看到 `OSType: linux`，且在`Kernel Version`或`Operating System`字段中包含 `WSL 2` 字样；

- **Hyper\-V 后端**：会在输出中看到 `OSType: linux`，且在`Operating System`字段中包含 `Docker Desktop on Windows \(Hyper\-V\)` 字样。

### 方法 3：通过 WSL 命令辅助验证（WSL 2 专属）

以管理员身份打开 PowerShell，执行以下命令：

```powershell
wsl -l -v
```

- 若输出列表中包含 `docker\-desktop` 和 `docker\-desktop\-data` 两个 WSL 发行版，且`VERSION`列显示为`2`，则当前 Docker 后端为 **WSL 2**；

- 若输出中无 Docker 相关的 WSL 发行版，或提示`未找到WSL发行版`，则当前后端为 **Hyper\-V**。

### 补充说明

- **Windows 家庭版**：仅支持 WSL 2 后端，无法使用 Hyper\-V，因此默认且只能是 WSL 2；

- **Windows 专业 / 企业版**：可在 Docker Desktop 设置的`General`中手动切换后端，切换后需重启 Docker 生效。

---

## 7\. Docker Compose YAML 配置详解

Docker Compose 是用于定义和运行多容器应用的工具，通过 `docker\-compose\.yml` YAML 文件配置所有服务，一键启动 / 停止。

### 7\.1 YAML 文件结构

一个标准的 `docker\-compose\.yml` 文件通常包含以下顶层配置：

```yaml
version: '3.8'  # Compose 文件格式版本（推荐 3.8，兼容 Docker Engine 19.03+）

services:       # 核心：定义所有服务（容器）
  服务名1:
    配置项...
  服务名2:
    配置项...

volumes:        # 可选：定义命名卷（用于数据持久化）
  卷名:

networks:       # 可选：定义自定义网络（用于服务间通信）
  网络名:
```

### 7\.2 Services 层核心配置项（最常用）

`services` 是 YAML 文件的核心，每个服务对应一个容器，下面是每个服务的高频配置项：

#### 1\. `image`：指定镜像（必选，二选一）

直接使用已有的镜像，无需构建。

```yaml
services:
  nginx:
    image: nginx:latest  # 格式：镜像名:标签
```

#### 2\. `build`：构建镜像（必选，二选一）

从 Dockerfile 构建自定义镜像，适合需要定制化的场景。

```yaml
services:
  my-app:
    build: ./app  # Dockerfile 所在目录的路径
    # 或更详细的配置：
    build:
      context: ./app  # 构建上下文路径
      dockerfile: Dockerfile.prod  # 指定 Dockerfile 文件名
      args:  # 传递构建参数（ARG）
        - NODE_ENV=production
```

#### 3\. `ports`：端口映射（高频）

将主机端口绑定到容器端口，外部可通过主机端口访问容器服务。

```yaml
services:
  nginx:
    image: nginx:latest
    ports:
      - "8080:80"  # 格式：主机端口:容器端口
      - "3306:3306"  # 可映射多个端口
```

#### 4\. `volumes`：数据卷挂载（核心，数据持久化）

将主机目录 / 命名卷挂载到容器，实现数据持久化（容器删除数据不丢失）。

```yaml
services:
  mysql:
    image: mysql:8.0
    volumes:
      # 方式1：命名卷挂载（推荐，Docker 统一管理）
      - mysql_data:/var/lib/mysql
      # 方式2：主机路径绑定挂载（适合开发调试，Windows 路径用正斜杠）
      - D:/docker/mysql/conf:/etc/mysql/conf.d
      # 方式3：只读挂载（:ro 后缀，容器内无法修改）
      - D:/docker/nginx/html:/usr/share/nginx/html:ro

# 顶层声明命名卷（方式1必须声明）
volumes:
  mysql_data:
```

#### 5\. `environment`：环境变量（高频）

传递环境变量到容器，常用于配置数据库密码、服务端口等。

```yaml
services:
  mysql:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=123456  # 格式：变量名=值
      - MYSQL_DATABASE=my_db
      - MYSQL_USER=user
      # 或使用键值对格式（更清晰）
      MYSQL_PASSWORD: password
```

#### 6\. `depends\_on`：服务依赖关系

定义服务启动顺序，确保依赖的服务先启动（但不等待依赖服务 “完全就绪”，仅保证启动顺序）。

```yaml
services:
  web:
    image: my-web-app
    depends_on:
      - db  # web 服务会在 db 服务启动后再启动
      - redis
  db:
    image: mysql:8.0
  redis:
    image: redis:latest
```

#### 7\. `command`：覆盖容器默认启动命令

替换镜像内置的启动命令，适合临时调整容器行为。

```yaml
services:
  nginx:
    image: nginx:latest
    command: nginx -g 'daemon off;'  # 覆盖默认启动命令
    # 或使用列表格式（避免 shell 转义问题）
    command: ["nginx", "-g", "daemon off;"]
```

#### 8\. `restart`：重启策略（生产环境必选）

定义容器退出后的重启行为，保证服务高可用。

```yaml
services:
  nginx:
    image: nginx:latest
    restart: always  # 可选值：
    # - no：不重启（默认）
    # - always：无论退出原因都重启（生产环境推荐）
    # - on-failure：仅在异常退出（非0状态码）时重启
    # - unless-stopped：除非手动停止，否则一直重启
```

#### 9\. `networks`：自定义网络

将服务加入指定网络，实现服务间通信（同一网络内的服务可通过服务名互相访问）。

```yaml
services:
  web:
    image: my-web-app
    networks:
      - app-network  # 加入自定义网络
  db:
    image: mysql:8.0
    networks:
      - app-network  # 同一网络内，web 可通过 "db" 作为主机名访问数据库

# 顶层声明自定义网络
networks:
  app-network:
    driver: bridge  # 默认驱动，单主机网络
```

#### 10\. `container\_name`：自定义容器名

给容器指定固定名称（默认会自动生成名称），便于管理。

```yaml
services:
  nginx:
    image: nginx:latest
    container_name: my-nginx  # 自定义容器名
```

### 7\.3 完整实战示例：LNMP 环境

下面是一个包含 Nginx、MySQL、PHP 的完整 `docker\-compose\.yml` 文件，整合了上述所有核心配置：

```yaml
version: '3.8'

services:
  # Nginx 服务
  nginx:
    image: nginx:latest
    container_name: lnmp-nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx/html:/usr/share/nginx/html
      - ./nginx/conf:/etc/nginx/conf.d
      - ./nginx/logs:/var/log/nginx
    depends_on:
      - php
    networks:
      - lnmp-network
    restart: unless-stopped

  # PHP 服务
  php:
    image: php:8.2-fpm
    container_name: lnmp-php
    volumes:
      - ./nginx/html:/var/www/html
    networks:
      - lnmp-network
    restart: unless-stopped

  # MySQL 服务
  mysql:
    image: mysql:8.0
    container_name: lnmp-mysql
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: lnmp_db
    volumes:
      - mysql_data:/var/lib/mysql
      - ./mysql/conf:/etc/mysql/conf.d
    networks:
      - lnmp-network
    restart: unless-stopped

# 顶层声明
volumes:
  mysql_data:

networks:
  lnmp-network:
    driver: bridge
```

### 7\.4 配套 Docker Compose 常用命令

编写好 YAML 文件后，使用以下命令管理服务：

```powershell
# 1. 启动所有服务（后台运行）
docker compose up -d

# 2. 停止并删除所有服务、网络（保留数据卷）
docker compose down

# 3. 停止并删除所有服务、网络、数据卷（数据会丢失！）
docker compose down -v

# 4. 查看服务运行状态
docker compose ps

# 5. 查看服务日志（实时查看）
docker compose logs -f --tail 100

# 6. 进入指定服务的容器
docker compose exec nginx /bin/bash

# 7. 重启指定服务
docker compose restart nginx

# 8. 重新构建并启动服务（修改 Dockerfile 后使用）
docker compose up -d --build
```

### 7\.5 关键注意事项

1. **缩进敏感**：YAML 文件严格使用空格缩进（建议 2 个空格），禁止使用 Tab；

2. **路径写法**：Windows 系统下主机路径用正斜杠（如 `D:/docker/data`），或双反斜杠转义（`D:\\\\docker\\\\data`）；

3. **服务名通信**：同一网络内的服务可通过服务名互相访问（如 web 服务连接数据库时，主机名填 `db` 即可）；

4. **版本兼容**：确保 `version` 字段与你的 Docker Engine 版本兼容（推荐 `3\.8`，覆盖绝大多数场景）。

---

## 8\. Docker 常见问题排查手册

### 【通用前置排查 4 步走】

无论遇到任何问题，先执行以下步骤，可解决 70% 的偶发异常：

1. **权限与基础校验**：以**管理员身份**运行 PowerShell / 终端 / Docker Desktop；任务管理器→性能→CPU，确认「虚拟化」为**已启用**，未开启则进入 BIOS 开启 Intel VT\-x/AMD\-V。

2. **WSL 2 核心修复**：

    ```powershell
    # 更新WSL 2内核
    wsl --update
    # 设置WSL 2为默认版本
    wsl --set-default-version 2
    # 验证WSL状态，VERSION为2即为正常
    wsl -l -v
    ```

3. **Windows 功能校验**：打开「控制面板→程序→启用或关闭 Windows 功能」，确保勾选：

    - 适用于 Linux 的 Windows 子系统

    - 虚拟机平台

    - 家庭版系统**禁止勾选 Hyper\-V**（仅专业 / 企业版支持，勾选会直接导致启动失败）

4. **基础重启修复**：关闭 Docker Desktop→执行`wsl \-\-shutdown`→重启电脑→重新启动 Docker，解决异常退出导致的临时故障。

### 8\.1 Docker Desktop 启动 / 安装失败（最高频）

#### 问题 1：启动报错 `WSL 2 installation is incomplete\.`

**现象**：打开 Docker 弹出该报错，引擎无法启动
**核心根因**：WSL 2 内核未安装 / 版本过低、WSL 分发损坏、未设置 WSL 2 为默认版本
**解决方案**：

1. 管理员 PowerShell 执行`wsl \-\-update`升级内核，完成后重启电脑；

2. 执行`wsl \-\-set\-default\-version 2`设置 WSL 2 为默认版本；

3. 若仍报错，执行`wsl \-\-install`重新安装 WSL 2，重启后重试；

4. 终极修复（会清空原有镜像 / 容器，提前备份）：

    ```powershell
    wsl --shutdown
    wsl --unregister docker-desktop
    wsl --unregister docker-desktop-data
    ```

    重启 Docker Desktop，会自动重建 WSL 分发，恢复正常启动。

#### 问题 2：启动报错 `Hardware assisted virtualization must be enabled in the BIOS`

**现象**：启动报错，提示虚拟化未开启
**核心根因**：BIOS 未开启 CPU 虚拟化、Windows 虚拟机平台功能未启用、第三方虚拟机软件冲突
**解决方案**：

1. 任务管理器确认虚拟化状态，未开启则重启电脑进入 BIOS，开启`Intel Virtual Technology \(VT\-x\)`/`AMD\-V`；

2. 确认已勾选「虚拟机平台」Windows 功能，重启电脑生效；

3. 若同时安装 VMware/VirtualBox，需开启「Windows Hypervisor Platform」功能，避免虚拟化冲突；

4. 企业设备确认组策略未禁用虚拟化功能。

#### 问题 3：启动一直卡在`Starting`状态，长时间无响应

**现象**：托盘图标一直显示 Starting，超过 5 分钟未进入 Running 状态
**核心根因**：WSL 2 实例异常、配置文件损坏、默认端口被占用、安全软件拦截、旧版本残留
**解决方案**：

1. 关闭 Docker，执行`wsl \-\-shutdown`，等待 10 秒后以管理员身份重新启动 Docker；
