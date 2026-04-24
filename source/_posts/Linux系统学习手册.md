---
title: Linux系统学习手册
date: 2026-03-24 20:30:00
tags: [Linux, 技术分享, Linux系统]
categories: Linux
toc: true
---
# Linux系统学习手册
## 文档核心说明
本文档适用于Linux零基础入门学习者、后端开发人员、运维工程师，内容覆盖从Linux基础认知到企业级运维的全链路核心知识，所有操作均适配Ubuntu 24.04、Rocky Linux 9等主流发行版，核心知识点、操作步骤、避坑要点均经过实操验证，可直接作为Linux学习与日常运维的参考手册。

## 一、Linux基础认知与发行版选型
### 1.1 Linux核心定义与特性
Linux是1991年林纳斯·托瓦兹开发的**开源免费操作系统内核**，基于其衍生出数百种发行版，核心特性如下：
- 开源免费：内核源码完全公开，无商业授权成本，是全球服务器首选系统
- 多用户多任务：支持多用户同时登录，并行运行多个进程，资源调度效率极高
- **一切皆文件**：硬件、进程、网络、配置等所有内容均以文件形式存在，是Linux最核心的设计理念
- 高稳定高安全：服务器可连续运行数年不重启，社区与企业级安全更新及时

### 1.2 主流发行版对比与选型指南
Linux发行版分为桌面端与服务器端两大阵营，主流选型对比如下：

| 发行版 | 包管理器 | 核心定位 | 适合人群 | 核心优势 |
|--------|----------|----------|----------|----------|
| Ubuntu 24.04 LTS | APT | 桌面/服务器通用 | 新手、开发者、云原生场景 | 文档最全、社区最活跃、硬件兼容好、Docker/K8s生态最强 |
| Linux Mint | APT | 桌面端 | Windows迁移用户、纯新手 | 界面符合Windows使用习惯、开箱即用、稳定无强制Snap |
| Debian 12 | APT | 服务器/轻量场景 | 极致稳定需求、小内存VPS | 资源占用极低、纯净开源、稳定性拉满，适合长期运行的服务 |
| Rocky Linux/AlmaLinux | DNF | 企业级服务器 | 企业生产环境、数据库服务 | 100%兼容RHEL、免费开源、企业级稳定、SELinux安全体系完善 |
| Fedora 44 | DNF | 桌面/开发 | 技术爱好者、前沿开发者 | 集成最新开源技术、软件更新快、RedHat上游版本 |
| Arch Linux | Pacman | 极客定制 | 高级用户、DIY爱好者 | 滚动更新、软件永远最新、AUR仓库覆盖海量软件、文档极其完善 |

> 新手选型建议：桌面端首选Linux Mint/Ubuntu；服务器学习首选Ubuntu Server；企业生产首选Rocky Linux/AlmaLinux。

### 1.3 系统安装方式（3种零门槛方案）
#### 1.3.1 虚拟机安装（新手首选，零风险）
1. 下载虚拟机软件：VMware Workstation Player（免费）或 VirtualBox（开源免费）
2. 下载对应发行版的ISO镜像文件
3. 新建虚拟机，选择ISO镜像，分配≥2核CPU、≥2G内存、≥20G磁盘
4. 启动虚拟机，跟随安装向导设置语言、时区、用户名与密码
5. 安装完成重启后即可进入系统

#### 1.3.2 WSL安装（Windows用户专属，一键部署）
Windows 10/11 内置Linux子系统，无需虚拟机，步骤如下：
1. 以管理员身份打开PowerShell，执行命令：
```bash
wsl --install
```
2. 系统自动启用WSL功能，默认安装Ubuntu发行版，等待下载完成
3. 重启电脑，打开Ubuntu终端，设置用户名和密码即可使用
4. 进阶：通过 `wsl --list --online` 查看可安装发行版，`wsl --install -d 发行版名称` 安装指定系统

#### 1.3.3 云服务器/物理机安装
- 云服务器：阿里云/腾讯云/AWS等平台，购买时选择对应Linux镜像，服务商自动完成安装，通过SSH远程登录即可
- 物理机：使用Rufus工具制作启动U盘，写入ISO镜像，电脑从U盘启动，跟随向导安装，**安装前务必备份磁盘数据**

## 二、Linux核心基础：文件系统与目录结构
### 2.1 核心设计理念：一切皆文件
Linux中，所有内容均以文件形式管理：普通文件、目录、硬件设备、进程、网络套接字等，所有操作均基于文件读写完成。

### 2.2 标准目录结构详解
Linux采用单根树状目录结构，根目录为 `/`，所有目录均从根目录衍生，核心目录功能如下：

| 目录 | 核心功能 | 关键说明 |
|------|----------|----------|
| `/` | 根目录 | 整个文件系统的起点，所有目录的父目录 |
| `/home` | 普通用户家目录 | 每个普通用户拥有 `/home/用户名` 专属目录，简称 `~`，存放用户个人数据，用户拥有完全权限 |
| `/root` | 超级管理员root家目录 | 仅root用户可访问，权限最高 |
| `/etc` | 系统配置文件目录 | 所有系统、服务的配置文件均存放于此，修改前建议备份 |
| `/bin` `/sbin` | 系统命令二进制文件 | `/bin` 存放普通用户可执行的基础命令；`/sbin` 存放root管理员的系统管理命令 |
| `/usr` | 用户程序与资源目录 | 安装的软件、共享库、文档、源码等均存放于此，类似Windows的Program Files |
| `/var` | 动态数据目录 | 存放运行时不断变化的文件，如系统日志、服务日志、数据库文件、缓存文件 |
| `/dev` | 设备文件目录 | 存放硬件设备文件，如磁盘 `/dev/sda`、终端 `/dev/tty` 等 |
| `/proc` | 虚拟文件系统 | 存放系统与进程的实时运行数据，不占用磁盘空间，如CPU信息 `/proc/cpuinfo` |
| `/tmp` | 临时文件目录 | 所有用户均可创建临时文件，系统重启后自动清空 |

### 2.3 路径规范
- **绝对路径**：从根目录 `/` 开始的完整路径，唯一确定文件位置，例如 `/home/user/test.txt`
- **相对路径**：从当前工作目录开始的路径，`.` 代表当前目录，`..` 代表上一级目录，例如 `./docs`、`../test`
- 核心确认命令：`pwd`，用于查看当前工作目录的绝对路径

## 三、Linux核心常用命令大全
Linux命令通用格式：`命令 [选项] [参数]`，短选项（`-l`）可合并（`ls -lh`），所有命令均可通过 `命令 --help` 或 `man 命令` 查看完整帮助文档。

### 3.1 基础终端操作命令
| 命令 | 核心功能 | 实操示例 | 避坑提示 |
|------|----------|----------|----------|
| `cd` | 切换工作目录 | `cd /etc` 切换到etc目录；`cd ~` 回到家目录；`cd ..` 返回上一级；`cd -` 回到上一次所在目录 | 路径不存在会报错，可用Tab键自动补全路径 |
| `ls` | 列出目录内容 | `ls` 列出当前目录文件；`ls -l` 详细列表；`ls -a` 显示隐藏文件；`ls -lh` 人性化显示文件大小 | 禁止使用 `ls /*` 查看根目录，避免终端卡死 |
| `pwd` | 显示当前工作目录 | `pwd` 直接执行，输出当前目录的绝对路径 | 无 |
| `clear` | 清空终端屏幕 | `clear` 直接执行，快捷键 `Ctrl+L` 效果相同 | 仅清空屏幕，不删除历史命令 |
| `history` | 查看命令历史 | `history` 列出所有执行过的命令；`!100` 执行历史中第100条命令 | 敏感密码不要直接写在命令中，避免被history记录 |
| `shutdown`/`reboot` | 关机/重启 | `shutdown -h now` 立即关机；`shutdown -h 10` 10分钟后关机；`reboot` 立即重启 | 远程服务器关机后无法手动开机，慎用 |
| `man` | 查看命令帮助文档 | `man ls` 查看ls命令的完整手册，按q退出 | 无 |
| `whoami` | 查看当前登录用户名 | `whoami` 直接执行，输出当前用户名 | 无 |
| `exit` | 退出当前终端/登录 | `exit` 直接执行 | 无 |

### 3.2 文件与目录操作命令
| 命令 | 核心功能 | 实操示例 | 避坑提示 |
|------|----------|----------|----------|
| `mkdir` | 创建目录 | `mkdir test` 创建test目录；`mkdir -p a/b/c` 递归创建多级目录 | 目录已存在会报错，加 `-p` 可避免报错 |
| `touch` | 创建空文件/修改文件时间 | `touch test.txt` 创建空文件；`touch a.txt b.txt` 批量创建文件 | 不会覆盖已有文件内容，仅修改访问时间 |
| `cp` | 复制文件/目录 | `cp test.txt /home/` 复制文件到指定目录；`cp -r test_dir /home/` 递归复制目录及所有内容 | 目标路径已存在同名文件会直接覆盖，加 `-i` 可开启覆盖提示 |
| `mv` | 移动/重命名文件 | `mv old.txt new.txt` 重命名；`mv test.txt /tmp/` 移动文件到tmp目录 | 目标路径同名文件会被覆盖，加 `-i` 开启提示 |
| `rm` | 删除文件/目录 | `rm test.txt` 删除文件；`rm -r test_dir` 删除目录；`rm -rf test_dir` 强制递归删除目录 | ⚠️ 高危命令！严禁执行 `rm -rf /`、`rm -rf *` 等命令，删除后数据无法恢复，新手建议加 `-i` 确认后删除 |
| `ln` | 创建链接 | `ln -s 源文件 软链接名` 创建软链接（类似Windows快捷方式）；`ln 源文件 硬链接名` 创建硬链接 | 软链接源文件删除后失效，硬链接不受影响 |
| `find` | 查找文件/目录 | `find /home -name "test.txt"` 按名称精准查找；`find / -name "*.log"` 查找所有.log后缀的文件；`find /tmp -type f -mtime +7` 查找tmp目录下7天前的文件 | 无 |
| `which` | 查找命令的可执行文件路径 | `which ls` 查找ls命令的路径；`which python3` 查找python3的安装路径 | 无 |

### 3.3 文件查看与编辑命令
#### 3.3.1 文件查看命令
| 命令 | 核心功能 | 实操示例 | 适用场景 |
|------|----------|----------|----------|
| `cat` | 查看文件全部内容 | `cat test.txt` 查看文件；`cat -n test.txt` 显示行号 | 小文件查看，大文件会刷屏 |
| `less` | 分页查看文件 | `less test.txt` 打开文件，空格翻页、上下键滚动、`/关键词` 搜索、`q` 退出 | 大文件查看，日志排查，支持搜索，最常用 |
| `head` | 查看文件头部内容 | `head test.txt` 默认看前10行；`head -n 20 test.txt` 看前20行 | 配置文件预览 |
| `tail` | 查看文件尾部内容 | `tail test.txt` 默认看最后10行；`tail -n 30 test.txt` 看最后30行；`tail -f test.log` 实时刷新文件内容 | 日志实时监控，服务启动故障排查必备 |
| `grep` | 文本搜索过滤 | `grep "error" test.log` 筛选包含error的行；`grep -i "error" test.log` 忽略大小写；`grep -n "error" test.log` 显示行号；`grep -v "error" test.log` 排除包含error的行 | 日志排查、文本筛选，运维核心命令 |

#### 3.3.2 Vim编辑器核心操作
Vim是Linux系统默认的标配文本编辑器，核心操作分为3种模式，基础操作如下：
1. 打开/创建文件：`vim 文件名`，进入**命令模式**
2. 进入编辑模式：按 `i` 键，左下角出现 `-- INSERT --`，即可输入修改文本
3. 保存/退出：编辑完成后，按 `ESC` 键回到命令模式，输入以下指令回车执行：
   - `:w` 保存文件，不退出
   - `:wq` 保存并退出
   - `:q!` 不保存，强制退出
> 进阶技巧：命令模式下，`/关键词` 可搜索内容，`dd` 删除整行，`yy` 复制整行，`p` 粘贴内容

### 3.4 压缩与解压命令
Linux最常用的压缩格式为 `.tar.gz`、`.zip`，核心命令如下：
| 命令 | 核心功能 | 实操示例 |
|------|----------|----------|
| `tar` | 打包/解压tar.gz/tar.bz2文件 | `tar -zcvf test.tar.gz 目标文件/目录` 打包并压缩为tar.gz；`tar -zxvf test.tar.gz` 解压tar.gz；`tar -zxvf test.tar.gz -C /opt` 解压到指定/opt目录 |
| `zip`/`unzip` | 处理zip格式 | `zip test.zip 目标文件` 压缩为zip；`zip -r test.zip 目录` 递归压缩目录；`unzip test.zip` 解压zip；`unzip test.zip -d /opt` 解压到/opt目录 |
> 参数说明：`-z` 用gzip压缩，`-c` 创建打包文件，`-v` 显示过程，`-x` 解压，`-f` 指定文件名（必须放在最后）

### 3.5 核心进阶能力：管道符与重定向
- **管道符 `|`**：将前一个命令的输出，作为后一个命令的输入，实现命令组合，示例：`ps aux | grep nginx`（筛选nginx相关进程）
- **重定向**：将命令输出写入文件，而非打印到屏幕
  - `>` 覆盖写入，示例：`echo "hello" > test.txt`
  - `>>` 追加写入，示例：`echo "new" >> test.txt`

## 四、用户与权限管理
Linux是多用户操作系统，权限体系是系统安全的核心，所有文件都有「所有者、所属组、其他用户」三类权限。

### 4.1 用户分类与管理
#### 4.1.1 Linux用户分类
- **root超级管理员**：UID=0，拥有系统最高权限，可执行任何操作，日常操作不建议使用
- **系统用户**：UID=1-999，系统服务运行使用的用户，默认无法登录，权限受限，保障服务安全
- **普通用户**：UID≥1000，日常操作用户，仅在家目录有完全权限，系统级操作需要sudo授权

#### 4.1.2 核心用户管理命令（均需root权限执行）
| 命令 | 核心功能 | 实操示例 |
|------|----------|----------|
| `useradd` | 创建用户 | `useradd testuser` 创建普通用户；`useradd -m testuser` 创建用户并自动生成家目录 |
| `passwd` | 设置/修改用户密码 | `passwd testuser` 给指定用户设置密码；`passwd` 修改当前用户密码 |
| `userdel` | 删除用户 | `userdel testuser` 删除用户；`userdel -r testuser` 删除用户并同时删除家目录 |
| `usermod` | 修改用户属性 | `usermod -G sudo testuser` 将用户加入sudo组；`usermod -s /bin/bash testuser` 修改用户默认Shell |
| `su` | 切换用户 | `su testuser` 切换到普通用户；`su - root` 切换到root用户并加载root环境变量 |
| `id` | 查看用户信息 | `id testuser` 查看用户的UID、GID、所属组 |

### 4.2 用户组管理
用户组用于批量管理用户权限，一个用户可加入多个组，一个组可包含多个用户，核心命令如下：
| 命令 | 核心功能 | 实操示例 |
|------|----------|----------|
| `groupadd` | 创建用户组 | `groupadd devgroup` 创建开发组 |
| `groupdel` | 删除用户组 | `groupdel devgroup` 删除用户组 |
| `gpasswd` | 管理组成员 | `gpasswd -a testuser devgroup` 将用户加入组；`gpasswd -d testuser devgroup` 将用户从组中移除 |

### 4.3 文件权限体系
#### 4.3.1 权限基础说明
每个文件/目录都有三类权限，分别对应「所有者user(u)、所属组group(g)、其他用户other(o)」，每类权限包含3种基础权限：

| 权限 | 字符 | 数字 | 文件作用 | 目录作用 |
|------|------|------|----------|----------|
| 读 | r | 4 | 可查看文件内容 | 可列出目录内的文件列表 |
| 写 | w | 2 | 可修改、删除文件内容 | 可在目录内创建、删除、重命名文件 |
| 执行 | x | 1 | 可执行文件（脚本、程序） | 可进入目录（cd命令） |

权限查看示例：`ls -l` 输出 `-rwxr-xr-- 1 root devgroup 1024 Apr 22 10:00 test.sh`
- 第1位：`-` 代表普通文件，`d` 代表目录，`l` 代表软链接
- 第2-4位 `rwx`：所有者root的权限（读写执行，数字7）
- 第5-7位 `r-x`：所属组devgroup的权限（读执行，数字5）
- 第8-10位 `r--`：其他用户的权限（只读，数字4）
- 数字权限：将三类权限的数字拼接，示例中为 `754`

#### 4.3.2 权限修改命令 `chmod`
核心用法：
1. 数字方式修改（最常用）：
```bash
chmod 755 test.sh  # 所有者读写执行，组和其他用户读执行
chmod 644 test.txt  # 所有者读写，组和其他用户只读
chmod -R 755 test_dir  # 递归修改目录及所有子文件的权限
```
2. 符号方式修改：
```bash
chmod u+x test.sh  # 给所有者添加执行权限
chmod g-w test.txt  # 给所属组移除写权限
chmod o=r test.sh  # 其他用户仅设置读权限
chmod a+x test.sh  # 给所有用户添加执行权限
```

#### 4.3.3 所有者修改命令 `chown`
修改文件的所有者和所属组，需root权限执行：
```bash
chown testuser:devgroup test.txt  # 同时修改所有者为testuser，所属组为devgroup
chown -R testuser:devgroup test_dir  # 递归修改目录所有者
chown testuser test.txt  # 仅修改所有者
chown :devgroup test.txt  # 仅修改所属组
```

### 4.4 sudo权限配置
普通用户无法执行系统级操作，通过sudo可临时获得root权限执行命令，无需切换到root用户，更安全。
1. 基础用法：`sudo 命令`，示例：`sudo apt update`，输入当前用户密码即可执行
2. 配置方式：
   - 方式1（简单）：将用户加入sudo组（Ubuntu/Debian）或wheel组（Rocky/CentOS）
     ```bash
     # Ubuntu/Debian
     usermod -G sudo testuser
     # Rocky/CentOS
     usermod -G wheel testuser
     ```
   - 方式2（精细化）：使用`visudo`命令编辑sudoers配置文件（自动检查语法，避免配置错误）
     ```bash
     visudo
     # 给testuser免密sudo权限
     testuser ALL=(ALL) NOPASSWD: ALL
     # 限制仅能执行特定命令
     testuser ALL=(ALL) /usr/bin/apt, /usr/bin/systemctl
     ```

## 五、软件包管理
Linux安装软件主要有3种方式：**包管理器安装（首选）、源码编译安装、二进制包安装**，不同发行版的包管理器不同。

### 5.1 Debian/Ubuntu 系列：APT 包管理器（.deb格式）
核心命令如下（均需sudo权限）：
| 命令 | 核心功能 | 实操示例 |
|------|----------|----------|
| `apt update` | 更新软件源缓存 | `sudo apt update` 同步最新的软件包信息，安装软件前必执行 |
| `apt install` | 安装软件 | `sudo apt install nginx` 安装nginx；`sudo apt install nginx python3 -y` 批量安装，-y自动确认 |
| `apt remove` | 卸载软件 | `sudo apt remove nginx` 卸载软件，保留配置文件 |
| `apt purge` | 彻底卸载软件 | `sudo apt purge nginx` 卸载软件并删除所有配置文件 |
| `apt upgrade` | 升级所有可更新的软件 | `sudo apt update && sudo apt upgrade -y` 更新缓存并升级所有软件 |
| `apt search` | 搜索软件包 | `apt search python3` 搜索包含python3的软件包 |
| `apt show` | 查看软件包详情 | `apt show nginx` 查看nginx的版本、描述、依赖等信息 |
| `dpkg -i` | 安装本地deb包 | `sudo dpkg -i test.deb` 安装本地下载的deb安装包 |

### 5.2 RHEL/Rocky/AlmaLinux 系列：DNF/YUM 包管理器（.rpm格式）
Rocky/AlmaLinux 9默认使用DNF，兼容YUM命令，核心命令如下（均需sudo权限）：
| 命令 | 核心功能 | 实操示例 |
|------|----------|----------|
| `dnf makecache` | 更新软件源缓存 | `sudo dnf makecache` 同步最新软件包信息 |
| `dnf install` | 安装软件 | `sudo dnf install nginx -y` 安装nginx，-y自动确认 |
| `dnf remove` | 卸载软件 | `sudo dnf remove nginx` 卸载软件 |
| `dnf update` | 升级软件 | `sudo dnf update -y` 升级所有可更新的软件包 |
| `dnf search` | 搜索软件 | `dnf search python3` 搜索软件包 |
| `dnf info` | 查看软件详情 | `dnf info nginx` 查看软件包详细信息 |
| `rpm -ivh` | 安装本地rpm包 | `sudo rpm -ivh test.rpm` 安装本地rpm包 |

### 5.3 源码编译安装软件
通用步骤如下（以nginx为例）：
1. 安装编译依赖环境
```bash
# Ubuntu/Debian
sudo apt install build-essential gcc make wget -y
# Rocky/CentOS
sudo dnf install gcc make wget -y
```
2. 下载源码包，解压
```bash
wget http://nginx.org/download/nginx-1.26.0.tar.gz
tar -zxvf nginx-1.26.0.tar.gz
cd nginx-1.26.0
```
3. 配置编译参数，指定安装路径
```bash
./configure --prefix=/usr/local/nginx
```
4. 编译与安装
```bash
make  # 编译
sudo make install  # 安装到指定目录
```
5. 配置环境变量
```bash
echo "export PATH=$PATH:/usr/local/nginx/sbin" >> ~/.bashrc
source ~/.bashrc
```

## 六、进程与服务管理
### 6.1 进程查看与监控
进程是正在运行的程序实例，核心管理命令如下：
| 命令 | 核心功能 | 实操示例 |
|------|----------|----------|
| `ps` | 查看进程快照 | `ps aux` 查看系统所有进程的详细信息；`ps aux | grep nginx` 筛选nginx相关进程 |
| `top` | 实时监控进程与系统资源 | `top` 直接执行，实时显示CPU、内存使用率，进程排名，按q退出；按P按CPU排序，按M按内存排序 |
| `htop` | 增强版top | `htop` 界面更美观，支持鼠标操作，需先通过包管理器安装 |
| `pstree` | 以树状结构显示进程父子关系 | `pstree` 查看进程树；`pstree -p` 显示进程PID |

### 6.2 进程终止与优先级调整
| 命令 | 核心功能 | 实操示例 | 避坑提示 |
|------|----------|----------|----------|
| `kill` | 终止指定PID的进程 | `kill 1234` 正常终止PID为1234的进程；`kill -9 1234` 强制终止进程（进程无响应时使用） | 不要随意kill系统进程，会导致系统崩溃 |
| `killall` | 按进程名终止所有进程 | `killall nginx` 终止所有nginx进程；`killall -9 python3` 强制终止所有python3进程 | 慎用，会终止所有同名进程，避免误杀 |
| `pkill` | 按名称/条件终止进程 | `pkill nginx` 终止nginx进程；`pkill -u testuser python3` 终止testuser用户的所有python3进程 | 无 |
| `nice`/`renice` | 调整进程优先级 | `nice -n -10 ./test.sh` 以优先级-10启动进程；`renice -5 1234` 修改PID1234进程的优先级为-5 | 优先级范围-20（最高）到19（最低），普通用户只能调大优先级，root可调小 |

### 6.3 systemd 服务管理（主流系统标配）
目前所有主流Linux发行版均使用systemd作为系统初始化系统，通过 `systemctl` 命令管理服务。

#### 6.3.1 核心服务管理命令
| 命令 | 核心功能 | 实操示例 |
|------|----------|----------|
| `systemctl status` | 查看服务详细状态 | `sudo systemctl status nginx` 查看nginx服务的运行状态、日志、开机自启状态 |
| `systemctl start` | 启动服务 | `sudo systemctl start nginx` 启动nginx服务 |
| `systemctl stop` | 停止服务 | `sudo systemctl stop nginx` 停止nginx服务 |
| `systemctl restart` | 重启服务 | `sudo systemctl restart nginx` 重启服务，配置修改后执行 |
| `systemctl reload` | 重载服务配置 | `sudo systemctl reload nginx` 平滑重载配置，不中断服务（仅支持热重载的服务） |
| `systemctl enable` | 设置服务开机自启 | `sudo systemctl enable nginx` 设置nginx开机自启；`sudo systemctl enable --now nginx` 开启自启并立即启动服务 |
| `systemctl disable` | 取消服务开机自启 | `sudo systemctl disable nginx` 取消开机自启 |
| `systemctl daemon-reload` | 重载systemd配置 | 修改自定义.service文件后，必须执行此命令让配置生效 |

#### 6.3.2 自定义systemd服务标准模板
可将自定义脚本/程序配置为systemd服务，实现开机自启、故障自动重启，标准模板如下：
1. 创建服务文件，路径为 `/etc/systemd/system/myapp.service`
```ini
[Unit]
Description=我的自定义应用服务
After=network.target
Wants=network.target

[Service]
Type=simple
ExecStart=/usr/bin/python3 /opt/myapp/main.py
WorkingDirectory=/opt/myapp
Restart=on-failure
RestartSec=5s
User=testuser
Group=testuser

[Install]
WantedBy=multi-user.target
```
2. 重载配置，启动服务，设置开机自启
```bash
sudo systemctl daemon-reload
sudo systemctl start myapp
sudo systemctl enable myapp
```
3. 查看服务状态与日志
```bash
sudo systemctl status myapp
journalctl -u myapp -f
```

## 七、网络配置与远程管理
### 7.1 网络基础配置
Linux主流使用 `ip` 命令管理网络，替代老旧的 `ifconfig` 命令，核心操作如下：
| 命令 | 核心功能 | 实操示例 |
|------|----------|----------|
| `ip addr` | 查看网卡与IP地址信息 | `ip addr` 查看所有网卡信息；`ip addr show eth0` 查看指定网卡eth0的IP、MAC地址 |
| `ip link set` | 启停网卡 | `sudo ip link set eth0 up` 启动网卡；`sudo ip link set eth0 down` 关闭网卡 |
| `ip route` | 查看/配置路由表 | `ip route` 查看路由表，默认网关；`sudo ip route add default via 192.168.1.1` 设置默认网关 |
| `hostname` | 查看/修改主机名 | `hostname` 查看当前主机名；`sudo hostnamectl set-hostname linux-server` 永久修改主机名 |

DNS配置：永久修改DNS服务器，编辑 `/etc/systemd/resolved.conf`（systemd系统）或 `/etc/resolv.conf` 文件，添加：
```
nameserver 223.5.5.5
nameserver 8.8.8.8
```

### 7.2 防火墙配置
#### 7.2.1 UFW 防火墙（Ubuntu/Debian默认）
| 命令 | 核心功能 | 实操示例 |
|------|----------|----------|
| `ufw status` | 查看防火墙状态 | `sudo ufw status verbose` 查看详细状态、已开放端口 |
| `ufw enable` | 启用防火墙 | `sudo ufw enable` 开启防火墙，开机自启 |
| `ufw disable` | 关闭防火墙 | `sudo ufw disable` 关闭防火墙 |
| `ufw allow` | 放行端口/服务 | `sudo ufw allow 22/tcp` 放行22端口SSH；`sudo ufw allow 80/tcp` 放行80端口HTTP |
| `ufw deny` | 拒绝端口/服务 | `sudo ufw deny 3306/tcp` 拒绝3306端口外网访问 |
| `ufw delete` | 删除规则 | `sudo ufw delete allow 22/tcp` 删除放行22端口的规则 |
| `ufw default` | 设置默认规则 | `sudo ufw default deny incoming` 默认拒绝所有入站流量；`sudo ufw default allow outgoing` 默认允许所有出站流量 |

#### 7.2.2 Firewalld 防火墙（Rocky/CentOS默认）
| 命令 | 核心功能 | 实操示例 |
|------|----------|----------|
| `firewall-cmd --state` | 查看防火墙状态 | `firewall-cmd --state` 输出running/stopped |
| `firewall-cmd --list-all` | 查看所有规则 | `firewall-cmd --list-all` 查看已开放端口、服务、规则 |
| `firewall-cmd --add-port` | 临时放行端口 | `sudo firewall-cmd --add-port=22/tcp` 临时放行22端口，重启后失效 |
| `firewall-cmd --add-port --permanent` | 永久放行端口 | `sudo firewall-cmd --add-port=80/tcp --permanent` 永久放行80端口 |
| `firewall-cmd --remove-port` | 删除端口规则 | `sudo firewall-cmd --remove-port=3306/tcp --permanent` 删除永久规则 |
| `firewall-cmd --reload` | 重载规则 | 修改永久规则后，必须执行此命令生效 |

### 7.3 SSH 远程登录与安全配置
SSH是Linux远程管理的核心工具，默认端口22，通过加密传输。

#### 7.3.1 基础远程登录
1. 服务端安装SSH服务（多数系统默认已安装）
```bash
# Ubuntu/Debian
sudo apt install openssh-server -y
sudo systemctl enable --now sshd
# Rocky/CentOS
sudo dnf install openssh-server -y
sudo systemctl enable --now sshd
```
2. 客户端远程登录
```bash
# 标准格式：ssh 用户名@服务器IP地址
ssh testuser@192.168.1.100
# 指定端口登录
ssh -p 2222 testuser@192.168.1.100
```

#### 7.3.2 SSH 密钥登录（免密+更安全）
步骤如下：
1. 客户端生成密钥对
```bash
ssh-keygen
# 一路回车，默认生成在 ~/.ssh/ 目录，id_rsa为私钥（妥善保管，切勿泄露），id_rsa.pub为公钥
```
2. 将公钥上传到服务器
```bash
ssh-copy-id testuser@192.168.1.100
# 手动方式：将客户端id_rsa.pub的内容，追加到服务器 ~/.ssh/authorized_keys 文件中
```
3. 测试免密登录，直接执行 `ssh testuser@192.168.1.100` 即可登录

#### 7.3.3 SSH 安全加固
编辑SSH配置文件 `/etc/ssh/sshd_config`，修改以下配置：
```ini
# 修改默认SSH端口，避免被批量扫描
Port 2222
# 禁止root用户直接远程登录
PermitRootLogin no
# 禁用密码登录，仅允许密钥登录（配置好密钥后再开启）
PasswordAuthentication no
# 关闭空密码登录
PermitEmptyPasswords no
# 登录超时时间，60秒无操作自动断开
LoginGraceTime 60
# 最大尝试登录次数
MaxAuthTries 3
```
修改完成后，重启sshd服务生效：
```bash
sudo systemctl restart sshd
```
> 注意：修改端口后，必须在防火墙中放行新的端口，否则会导致无法远程登录。

### 7.4 常用网络诊断命令
| 命令 | 核心功能 | 实操示例 |
|------|----------|----------|
| `ping` | 测试网络连通性 | `ping baidu.com` 测试与百度的连通性；`ping 192.168.1.1` 测试与网关的连通性，按Ctrl+C停止 |
| `telnet` | 测试端口是否开放 | `telnet 192.168.1.100 22` 测试服务器22端口是否可访问 |
| `nc` | 网络工具，端口测试/扫描 | `nc -zv 192.168.1.100 80` 测试80端口是否开放 |
| `ss` | 查看端口监听与网络连接 | `ss -tulpn` 查看系统所有正在监听的端口 |
| `traceroute` | 路由跟踪，排查网络链路 | `traceroute baidu.com` 跟踪访问百度的网络节点 |
| `curl` | 网络请求工具，测试接口/网页 | `curl baidu.com` 访问百度网页；`curl -I https://baidu.com` 查看请求响应头；`curl -O https://example.com/file.tar.gz` 下载文件 |
| `wget` | 文件下载工具 | `wget https://example.com/file.tar.gz` 下载文件；`wget -c https://example.com/file.tar.gz` 断点续传下载 |

## 八、Shell脚本编程入门
Shell是Linux的命令解释器，默认使用Bash，Shell脚本可将多个命令组合，实现自动化任务。

### 8.1 脚本基础规范
1. 脚本文件后缀为 `.sh`，第一行必须指定解释器：`#!/bin/bash`
2. 脚本需要添加执行权限：`chmod +x test.sh`
3. 执行脚本：`./test.sh`（当前目录）或 `/opt/test.sh`（绝对路径）
4. 注释：单行注释用 `#`，多行注释用 `: ' 注释内容 '`

### 8.2 核心语法
#### 8.2.1 变量
- 定义变量：`变量名=值`，等号两边不能有空格，变量名建议大写
- 调用变量：`$变量名` 或 `${变量名}`
- 示例：
```bash
#!/bin/bash
NAME="Linux"
VERSION="24.04"
echo "Hello $NAME $VERSION"
# 只读变量
readonly PI=3.14159
# 删除变量
unset NAME
```
- 内置特殊变量：
  - `$0`：脚本本身的文件名
  - `$1-$n`：脚本的第1到第n个输入参数
  - `$#`：输入参数的个数
  - `$?`：上一条命令的执行结果，0为成功，非0为失败
  - `$$`：当前脚本的进程PID

#### 8.2.2 条件判断
语法格式：
```bash
if [ 条件表达式 ]; then
    执行语句
elif [ 条件表达式 ]; then
    执行语句
else
    执行语句
fi
```
常用条件判断：
- 数值判断：`-eq` 等于、`-ne` 不等于、`-gt` 大于、`-lt` 小于、`-ge` 大于等于、`-le` 小于等于
- 文件判断：`-f` 文件是否存在、`-d` 目录是否存在、`-x` 文件是否有执行权限
- 字符串判断：`==` 等于、`!=` 不等于、`-z` 字符串是否为空

示例：
```bash
#!/bin/bash
FILE="/etc/nginx/nginx.conf"
if [ -f $FILE ]; then
    echo "文件存在"
else
    echo "文件不存在"
fi

NUM=10
if [ $NUM -gt 5 ]; then
    echo "数值大于5"
fi
```

#### 8.2.3 循环语句
1. for循环
```bash
#!/bin/bash
for i in {1..10}; do
    echo "数字：$i"
done

for file in *.txt; do
    echo "文件：$file"
done
```
2. while循环
```bash
#!/bin/bash
i=1
while [ $i -le 5 ]; do
    echo "i=$i"
    i=$((i+1))
done
```

#### 8.2.4 函数
```bash
#!/bin/bash
function add() {
    a=$1
    b=$2
    echo $((a+b))
}

result=$(add 10 20)
echo "计算结果：$result"
```

### 8.3 实用脚本示例
#### 示例1：自动备份日志文件
```bash
#!/bin/bash
LOG_DIR="/var/log/nginx"
BACKUP_DIR="/backup/nginx"
DATE=$(date +%Y%m%d)
BACKUP_FILE="nginx_log_$DATE.tar.gz"

mkdir -p $BACKUP_DIR
tar -zcvf $BACKUP_DIR/$BACKUP_FILE $LOG_DIR

if [ $? -eq 0 ]; then
    echo "备份成功，文件：$BACKUP_DIR/$BACKUP_FILE"
else
    echo "备份失败"
    exit 1
fi

find $BACKUP_DIR -name "nginx_log_*.tar.gz" -mtime +7 -delete
```

#### 示例2：系统资源监控脚本
```bash
#!/bin/bash
CPU_USAGE=$(top -bn1 | grep Cpu | awk '{print 100 - $8}')
MEM_USAGE=$(free | grep Mem | awk '{print $3/$2 * 100.0}')
DISK_USAGE=$(df -h / | grep / | awk '{print $5}' | sed 's/%//g')

echo "===== 系统资源监控 ====="
echo "CPU使用率：$CPU_USAGE%"
echo "内存使用率：$MEM_USAGE%"
echo "根目录磁盘使用率：$DISK_USAGE%"

if [ $DISK_USAGE -gt 90 ]; then
    echo "警告：磁盘使用率超过90%！"
fi
```

## 九、系统运维与监控
### 9.1 磁盘与存储管理
| 命令 | 核心功能 | 实操示例 |
|------|----------|----------|
| `df -h` | 查看磁盘分区空间使用情况 | `df -h` 人性化显示所有分区的总容量、已用、可用、使用率；`df -i` 查看inode使用情况 |
| `du -h` | 查看文件/目录的磁盘占用 | `du -sh /home` 查看home目录总大小；`du -sh * | sort -hr` 查看当前目录下所有文件/目录的大小并排序 |
| `fdisk`/`parted` | 磁盘分区工具 | `sudo fdisk -l` 查看所有磁盘与分区信息；`sudo fdisk /dev/sdb` 对磁盘sdb进行分区 |
| `mkfs` | 格式化分区 | `sudo mkfs.ext4 /dev/sdb1` 格式化为ext4文件系统；`sudo mkfs.xfs /dev/sdb1` 格式化为xfs文件系统 |
| `mount`/`umount` | 挂载/卸载分区 | `sudo mount /dev/sdb1 /data` 将分区挂载到/data目录；`sudo umount /dev/sdb1` 卸载分区 |
| `blkid` | 查看分区UUID | `blkid` 查看所有分区的UUID，用于配置开机自动挂载 |

开机自动挂载：编辑 `/etc/fstab` 文件，添加以下内容，重启后自动挂载：
```
UUID=xxxxxx-xxxx-xxxx-xxxx-xxxxxx /data ext4 defaults 0 0
```

### 9.2 系统资源监控
| 命令 | 核心功能 | 实操示例 |
|------|----------|----------|
| `free -h` | 查看内存使用情况 | `free -h` 人性化显示总内存、已用、空闲、缓存、交换分区使用情况 |
| `vmstat` | 系统资源统计，CPU、内存、IO | `vmstat 1 5` 每秒统计1次，共5次，查看CPU上下文切换、IO等待 |
| `iostat` | 磁盘IO监控 | `iostat -x 1` 实时查看磁盘读写速度、IO使用率，排查磁盘瓶颈 |
| `mpstat` | CPU核心监控 | `mpstat -P ALL 1` 实时查看每个CPU核心的使用率 |

### 9.3 系统日志管理
Linux系统日志默认存放在 `/var/log/` 目录，核心日志文件：
- `/var/log/syslog`：Ubuntu/Debian系统全局日志
- `/var/log/messages`：Rocky/CentOS系统全局日志
- `/var/log/auth.log`：Ubuntu/Debian认证登录日志，排查SSH登录失败、sudo操作
- `/var/log/secure`：Rocky/CentOS认证登录日志
- `/var/log/dmesg`：内核启动日志，排查硬件驱动问题
- `/var/log/cron`：定时任务执行日志

日志查看核心工具：
1. `tail -f /var/log/syslog` 实时监控系统日志
2. `journalctl`：systemd统一日志管理工具，核心用法：
   - `journalctl -f` 实时查看系统所有日志
   - `journalctl -u nginx.service` 查看指定服务的日志
   - `journalctl --since "2026-04-22" --until "2026-04-23"` 查看指定时间段的日志
   - `journalctl -p err` 查看错误级别的日志
   - `journalctl -k` 查看内核日志

### 9.4 定时任务 crontab
crontab是Linux的定时任务工具，用于周期性执行脚本/命令。

#### 9.4.1 核心命令
| 命令 | 核心功能 |
|------|----------|
| `crontab -e` | 编辑当前用户的定时任务 |
| `crontab -l` | 列出当前用户的所有定时任务 |
| `crontab -r` | 删除当前用户的所有定时任务 |

#### 9.4.2 时间格式
```
# 分  时  日  月  周  要执行的命令
  *   *   *   *   *   /bin/bash /opt/backup.sh
```
- 分：0-59
- 时：0-23
- 日：1-31
- 月：1-12
- 周：0-7（0和7都代表周日）
- 特殊符号：
  - `*`：代表每一个单位，`* * * * *` 代表每分钟执行一次
  - `*/n`：每隔n个单位，`*/5 * * * *` 每隔5分钟执行一次
  - `n-m`：范围，`0 9-18 * * *` 每天9点到18点，整点执行
  - `n,m`：多个值，`0 8,12,18 * * *` 每天8点、12点、18点执行

#### 9.4.3 常用示例
```bash
# 每天凌晨2点执行备份脚本
0 2 * * * /bin/bash /opt/backup.sh
# 每隔10分钟执行一次监控脚本
*/10 * * * * /bin/bash /opt/monitor.sh
# 每周一到周五的18点执行
0 18 * * 1-5 /usr/bin/echo "下班啦"
# 每月1号凌晨3点清理日志
0 3 1 * * /usr/bin/find /var/log -name "*.log" -mtime +30 -delete
```
> 注意：定时任务中的命令必须写**绝对路径**，否则会执行失败。

### 9.5 系统备份与恢复
常用备份工具：
1. `tar`：文件级备份，适合备份配置文件、脚本、数据文件
2. `rsync`：增量备份工具，适合本地/远程数据同步备份，核心用法：
```bash
# 本地增量备份
rsync -av /home/ /backup/home/
# 远程同步备份到服务器
rsync -av /home/ testuser@192.168.1.100:/backup/home/
# 增量备份，删除目标端不存在的文件
rsync -av --delete /home/ /backup/home/
```
3. `dd`：磁盘级备份，整盘克隆，核心用法：
```bash
# 整盘备份，将sda磁盘克隆到sdb磁盘
sudo dd if=/dev/sda of=/dev/sdb bs=4M status=progress
# 备份磁盘为镜像文件
sudo dd if=/dev/sda of=/backup/disk.img bs=4M status=progress
```

## 十、系统安全加固基础
1. **最小化安装**：服务器仅安装需要的软件包，关闭不需要的服务，减少攻击面
2. **用户权限管控**：
   - 禁止root用户直接远程登录，日常使用普通用户，通过sudo执行管理操作
   - 给用户分配最小必要权限，避免过度授权
   - 定期清理无用的用户和用户组
3. **SSH安全加固**：修改默认端口、禁用密码登录、禁用root登录
4. **防火墙配置**：默认拒绝所有入站流量，仅放行必要的端口
5. **SELinux/AppArmor**：启用强制访问控制，限制服务的权限，不要随意关闭
6. **系统更新与补丁**：定期更新系统和软件包，修复安全漏洞
```bash
# Ubuntu/Debian
sudo apt update && sudo apt upgrade -y
# Rocky/CentOS
sudo dnf update -y
```
7. **密码安全策略**：设置强密码，定期更换密码，配置密码复杂度策略
8. **日志审计**：启用日志服务，定期审计登录日志、sudo操作日志、服务日志
9. **禁用不必要的服务和端口**：关闭不需要的服务，如Telnet、FTP、RPC等
10. **防暴力破解**：安装fail2ban工具，自动封禁多次SSH登录失败的IP
```bash
# Ubuntu/Debian安装fail2ban
sudo apt install fail2ban -y
sudo systemctl enable --now fail2ban
```

## 十一、常见问题排查与解决方案
### 11.1 无法SSH远程登录服务器
排查步骤：
1. 检查网络连通性：`ping 服务器IP`，不通则检查服务器网络、防火墙、云服务器安全组
2. 检查端口是否可访问：`telnet 服务器IP 端口` 或 `nc -zv 服务器IP 端口`，不通则检查防火墙是否放行端口、SSH服务是否启动
3. 检查SSH服务状态：服务器本地执行 `sudo systemctl status sshd`，查看服务是否运行，配置文件是否有错误
4. 检查认证日志：查看 `/var/log/auth.log` 或 `/var/log/secure`，定位具体失败原因
5. 检查SELinux/AppArmor：是否限制了SSH服务，临时关闭测试

### 11.2 磁盘空间满，无法写入文件
排查步骤：
1. 查看磁盘整体使用率：`df -h`，定位满的分区
2. 定位大文件/大目录：`du -sh * | sort -hr` 从根目录开始逐级排查，删除无用的大文件
3. 清理日志文件：`echo > /var/log/syslog` 清空大日志文件，不要直接rm，避免服务不释放句柄
4. 检查inode是否耗尽：`df -i`，inode满了即使有磁盘空间也无法写入文件，删除大量小文件
5. 检查已删除但未释放的文件：`lsof | grep deleted`，找到占用的进程，重启进程释放空间

### 11.3 服务启动失败
排查步骤：
1. 查看服务状态：`sudo systemctl status 服务名`，查看报错信息
2. 查看服务日志：`journalctl -u 服务名 -f`，查看详细的启动报错日志
3. 检查配置文件：执行配置检查命令，例如 `nginx -t`、`sshd -t`，排查语法错误
4. 检查端口占用：用 `ss -tulpn | grep 端口号` 查看端口是否被占用，停止占用进程或修改服务端口
5. 检查权限：服务运行的用户没有配置文件、数据目录的权限，修改对应权限
6. 检查依赖：服务依赖的其他服务未启动、依赖的库文件缺失，安装对应依赖

### 11.4 系统内存不足，OOM杀死进程
排查步骤：
1. 查看内存使用情况：`free -h`，查看内存、交换分区使用情况
2. 定位占用内存高的进程：`top`（按M排序）或 `htop`，找到内存占用高的进程
3. 查看OOM日志：`dmesg | grep -i oom`，查看被系统杀死的进程
4. 临时解决：重启占用内存高的服务，释放内存；增加交换分区swap
5. 永久解决：优化程序内存占用，增加物理内存，限制服务的内存使用上限

### 11.5 系统CPU使用率过高
排查步骤：
1. 查看CPU整体使用率：`top` 或 `htop`，按P排序，定位CPU占用高的进程
2. 区分用户态CPU和内核态CPU，`us` 是用户态，`sy` 是内核态，`wa` 是IO等待
3. 若用户态CPU高：分析对应进程的程序，是否有死循环、计算密集型任务，优化程序
4. 若内核态CPU高：排查系统调用、上下文切换频繁，用 `vmstat 1` 查看上下文切换次数
5. 若IO等待`wa`高：排查磁盘IO瓶颈，用 `iostat -x 1` 查看磁盘IO使用率，优化磁盘读写

## 核心知识点速览
- Linux核心设计理念是**一切皆文件**，所有硬件、进程、配置等均以文件形式管理
- 日常99%的场景仅需掌握40个左右的核心命令，无需死记硬背上千个命令，通过`man`/`--help`可随时查询用法
- Linux权限体系分为所有者、所属组、其他用户三类，通过`chmod`修改权限、`chown`修改所有者，是系统安全的核心
- 主流Linux发行版均使用systemd管理服务，通过`systemctl`命令完成服务的启停、状态查看、开机自启配置
- SSH是Linux远程管理的核心工具，推荐使用密钥登录替代密码登录，并修改默认端口、禁用root远程登录提升安全性
- Shell脚本的核心价值是实现自动化运维，通过变量、条件判断、循环、函数，可将重复操作标准化、自动化
- crontab是Linux定时任务的标配工具，时间格式分为「分时日月周」5个字段，命令必须使用绝对路径避免执行失败
- 系统故障排查的核心逻辑是：先定位问题范围，再通过日志、监控命令逐层深入，找到根因后针对性解决
- Linux学习的核心是**实操**，脱离场景的背诵毫无意义，每学一个知识点必须亲手实操验证，踩坑排错是最快的成长路径
