---
title: Gitignore 使用手册
date: 2026-05-11 13:30:00
tags: [.gitignore, 工具使用, git]
categories: 工具使用
toc: true
---
# Gitignore 使用手册


## 文档核心说明

本文档适用于 Git 版本控制系统的开发者，涵盖从入门到进阶的完整 .gitignore 技术知识，包括：

- .gitignore 基础概念与作用范围

- 完整的语法规则与模式匹配

- Python/Java 项目专用配置（支持 VSCode/PyCharm/IDEA）

- 高级使用技巧与最佳实践

- 常见问题排查与调试方法

## 一、基础概念

### 1.1 什么是 .gitignore

`.gitignore` 是 Git 版本控制系统中的**纯文本配置文件**，用于指定 Git 应该忽略的文件和目录。当 Git 检测到该文件时，会自动跳过匹配规则的文件，不会将它们纳入版本控制。

#### 主要作用

- 防止敏感数据（密码、API 密钥、私钥）被意外提交

- 排除编译生成的文件（`.class`、`.o`、`.exe`、`.pyc`）

- 忽略依赖目录（`node_modules/`、`venv/`）

- 过滤操作系统和编辑器生成的临时文件

- 保持仓库整洁，只保留源代码和必要文件

### 1.2 忽略机制的三个级别

Git 提供了三种不同作用范围的忽略配置，适用于不同场景：

|级别|文件位置|作用范围|是否提交到仓库|适用场景|
|---|---|---|---|---|
|**项目级**|项目根目录下的`.gitignore`|所有克隆该仓库的用户|✅ 是|所有开发者都需要忽略的文件（构建产物、依赖）|
|**本地级**|`.git/info/exclude`|仅当前用户的当前仓库|❌ 否|个人临时文件、本地测试脚本|
|**全局级**|`~/.gitignore_global`（可自定义）|当前用户的所有 Git 仓库|❌ 否|操作系统文件、个人编辑器配置|

#### 全局忽略配置方法

```bash
# 创建全局忽略文件
touch ~/.gitignore_global

# 配置 Git 使用该文件
git config --global core.excludesfile ~/.gitignore_global
```

## 二、语法规则

### 2.1 基础语法

- **注释**：以`#`开头的行

- **空行**：被忽略，可用于分隔不同规则组

- **转义字符**：`\`用于转义特殊字符（如`*`匹配字面量`*`）

### 2.2 核心模式匹配

|符号|含义|示例|
|---|---|---|
|`*`|匹配任意数量字符（不包括`/`）|`*.log` 匹配所有.log 文件|
|`**`|匹配任意数量的目录（递归）|`**/temp/` 匹配任意位置的 temp 目录|
|`?`|匹配单个任意字符|`file?.txt` 匹配 file1.txt、fileA.txt|
|`[]`|匹配方括号内任意一个字符|`file[0-9].txt` 匹配 file0.txt 到 file9.txt|
|`!`|否定模式（不忽略）|`!important.log` 不忽略 important.log|
|`/`（开头）|锚定到项目根目录|`/build/` 仅匹配根目录下的 build 目录|
|`/`（结尾）|仅匹配目录|`logs/` 仅匹配 logs 目录，不匹配 logs 文件|

### 2.3 常用模式示例

```gitignore
# 忽略单个文件
.env
secrets.txt

# 忽略所有特定扩展名的文件
*.pyc
*.class
*.o
*.exe

# 忽略整个目录
node_modules/
__pycache__/
build/
dist/

# 忽略特定目录下的特定文件
docs/*.pdf

# 忽略所有子目录中的特定文件
**/*.tmp

# 否定模式：忽略所有.txt但保留README.txt
*.txt
!README.txt

# 保留空目录（Git默认不跟踪空目录）
!logs/.gitkeep
```

## 三、官方模板库

GitHub 官方维护了一个全面的`.gitignore`模板库，包含了几乎所有主流编程语言、框架、工具和操作系统的最佳实践规则。

**仓库地址**：[https://github.com/github/gitignore](https://github.com/github/gitignore)

### 模板分类

- **根目录**：主流语言和框架模板（Python、Java、Node.js、C\+\+ 等）

- **Global/**：全局模板（操作系统、编辑器、工具）

- **community/**：社区贡献的特殊领域模板

### 常用模板推荐

- **Python**：Python.gitignore（包含虚拟环境、缓存、测试文件等）

- **Node.js**：Node.gitignore（包含 node_modules、npm 日志等）

- **Java**：Java.gitignore（包含.class 文件、Maven/Gradle 构建产物）

- **C\+\+**：C\+\+.gitignore（包含.o、.obj、可执行文件等）

- **Unity**：Unity.gitignore（包含 Library、Temp、Obj 等 Unity 工作目录）

## 四、项目专用配置

### 4.1 Python 项目专用（VSCode \+ PyCharm 双支持）

适用于所有 Python 项目：Django/Flask/FastAPI/ 数据分析 / 机器学习等

```gitignore
##############################################################################
# Python 项目 .gitignore (VSCode + PyCharm)
##############################################################################

# ==============================================
# 第一部分：操作系统自动生成的文件（所有项目通用）
# 这些文件是操作系统为了管理文件系统而创建的，与项目代码无关
# ==============================================

# Windows 系统文件
Thumbs.db               # Windows 缩略图缓存文件
ehthumbs.db             # Windows 媒体缩略图缓存
Desktop.ini             # Windows 文件夹自定义配置
$RECYCLE.BIN/           # Windows 回收站文件夹
*.lnk                   # Windows 快捷方式

# macOS 系统文件
.DS_Store               # macOS 文件夹视图配置（最常见的误提交文件）
._*                     # macOS 资源分支文件（在非苹果文件系统上生成）
.AppleDouble           # macOS 双分支文件格式
.LSOverride            # macOS 文件夹覆盖配置
.Trashes               # macOS 回收站
.Spotlight-V100        # macOS  Spotlight 搜索索引
.TemporaryItems        # macOS 临时文件

# Linux 系统文件
*~                      # 文本编辑器备份文件
*.swp                   # Vim 交换文件
*.swo                   # Vim 交换文件（第二个）
*.bak                   # 通用备份文件
*.tmp                   # 通用临时文件
.#*                     # Emacs 自动保存文件
.Trash-*                # Linux 回收站
.directory              # KDE 桌面配置文件

# ==============================================
# 第二部分：敏感信息与临时文件（绝对不能提交！）
# 这些文件包含密码、API密钥、本地配置等敏感信息
# ==============================================

# 环境变量与密钥文件
.env                    # 通用环境变量文件（最常用）
.env.local              # 本地环境变量（覆盖.env）
.env.development.local  # 开发环境本地变量
.env.test.local         # 测试环境本地变量
.env.production.local   # 生产环境本地变量
.env.*.local            # 所有环境的本地变量
secrets.*               # 所有以secrets开头的密钥文件
*.key                   # 私钥文件
*.pem                   # PEM格式证书/密钥
config.ini              # INI格式配置文件
config.yaml             # YAML格式配置文件
config.json             # JSON格式配置文件

# 日志文件
*.log                   # 所有.log后缀的日志文件
*.log.*                 # 分割后的日志文件（如app.log.1）
logs/                   # logs目录下的所有文件
log/                    # log目录下的所有文件
nohup.out               # nohup命令生成的输出文件

# 临时与缓存文件
*.tmp                   # 临时文件
*.temp                  # 临时文件
*.cache                 # 缓存文件
*.pid                   # 进程ID文件
*.lock                  # 锁文件
tmp/                    # 临时目录
temp/                   # 临时目录
cache/                  # 缓存目录

# ==============================================
# 第三部分：Python 语言核心忽略规则
# 这些是Python运行、编译、打包过程中自动生成的文件
# ==============================================

# 编译字节码文件（Python解释器自动生成）
*.py[cod]               # 匹配.pyc、.pyo、.pyd文件
*$py.class              # Jython编译的class文件
*.so                    # C扩展编译的共享库文件
.Python                 # 虚拟环境中的Python符号链接
__pycache__/            # Python 3.2+字节码缓存目录

# 打包与分发产物
build/                  # setuptools构建目录
dist/                   # 打包后的发布目录（包含wheel和tar.gz）
*.egg-info/             # 包元信息目录
.installed.cfg          # 安装记录文件
*.egg                   # 旧格式的egg包
MANIFEST                # 打包清单文件
wheels/                 # wheel包存放目录
eggs/                   # egg包存放目录
.eggs/                  # 开发模式下的egg目录
lib/                    # 构建生成的库目录
lib64/                  # 64位系统构建生成的库目录
parts/                  # zc.buildout生成的目录
sdist/                  # 源码包构建目录
var/                    # 变量数据目录

# 虚拟环境（每个开发者本地创建，不需要提交）
venv/                   # 最常用的虚拟环境目录名
env/                    # 另一个常用的虚拟环境目录名
.venv/                  # 隐藏的虚拟环境目录（推荐）
.env/                   # 隐藏的环境目录
ENV/                    # 大写的虚拟环境目录
venv.bak/               # 备份的虚拟环境
.venv.bak/              # 备份的隐藏虚拟环境
pyvenv.cfg              # 虚拟环境配置文件
.python-version         # pyenv版本文件
pip-log.txt             # pip安装日志

# 测试与覆盖率报告
htmlcov/                # HTML格式的覆盖率报告
.tox/                   # tox测试工具目录
.nox/                   # nox测试工具目录
.coverage               # coverage工具生成的覆盖率数据
.coverage.*             # 分割的覆盖率数据
coverage.xml            # XML格式的覆盖率报告
.pytest_cache/          # pytest缓存目录
cover/                  # 旧版coverage报告目录
nosetests.xml           # nose测试报告
junit.xml               # JUnit格式的测试报告
.hypothesis/            # hypothesis测试工具缓存

# 代码质量工具生成的文件
.mypy_cache/            # mypy类型检查缓存
.ruff_cache/            # ruff代码检查缓存
.pytype/                # pytype类型检查缓存
.pylint.d               # pylint缓存目录
.flake8                 # flake8配置文件（可选提交）
.isort.cfg              # isort配置文件（可选提交）
.pre-commit-config.yaml # pre-commit配置文件（可选提交）

# Jupyter Notebook 相关
.ipynb_checkpoints      # Notebook检查点目录
.ipython                # IPython配置目录
.profile_default/       # IPython默认配置
*.ipynb.meta            # Notebook元数据文件

# ==============================================
# 第四部分：常用Python框架特定规则
# ==============================================

# Django 框架
local_settings.py       # 本地配置文件（覆盖settings.py）
db.sqlite3              # SQLite数据库文件
db.sqlite3-journal      # SQLite数据库日志
media/                  # 用户上传的媒体文件
static/                 # 静态文件目录（可选提交）
staticfiles/            # collectstatic生成的静态文件

# Flask 框架
instance/               # Flask实例目录
.webassets-cache        # Flask-Assets缓存

# FastAPI 框架
# FastAPI没有特殊的忽略规则，上面的通用规则已经覆盖

# 数据科学与机器学习
*.csv                   # CSV数据文件
*.tsv                   # TSV数据文件
*.xlsx                  # Excel文件
*.parquet               # Parquet数据文件
*.feather               # Feather数据文件
*.h5                    # HDF5数据文件
*.hdf5                  # HDF5数据文件
*.pkl                   # Pickle序列化文件
*.pickle                # Pickle序列化文件
*.joblib                # Joblib序列化文件
*.npz                   # NumPy压缩数组
*.npy                   # NumPy数组
data/                   # 数据目录
dataset/                # 数据集目录
models/                 # 训练好的模型目录
checkpoints/            # 训练检查点目录
results/                # 实验结果目录
outputs/                # 输出文件目录

# ==============================================
# 第五部分：开发工具忽略规则
# ==============================================

# VSCode 编辑器
.vscode/                # VSCode项目配置目录
# 可选：如果团队需要统一配置，取消下面的注释
# !.vscode/settings.json    # 共享的编辑器设置
# !.vscode/launch.json      # 共享的调试配置
# !.vscode/tasks.json       # 共享的任务配置
# !.vscode/extensions.json  # 推荐的扩展列表

*.code-workspace        # VSCode工作区文件
.history/               # VSCode历史记录插件
.vscode-test/           # VSCode扩展测试目录

# PyCharm (JetBrains 系列)
.idea/                  # JetBrains IDE项目配置目录
*.iml                   # 模块配置文件
*.ipr                   # 旧版项目文件
*.iws                   # 旧版工作区文件
out/                    # 编译输出目录
production/             # 生产模式编译输出
classes/                # 类文件目录
artifacts/              # 构建产物目录

# 可选：如果团队需要统一配置，取消下面的注释
# !.idea/runConfigurations/   # 共享的运行配置
# !.idea/codeStyles/          # 共享的代码风格
# !.idea/inspectionProfiles/  # 共享的代码检查配置

# ==============================================
# 第六部分：其他常用工具
# ==============================================

# Git 相关
*.patch                 # Git补丁文件
*.diff                  # Git差异文件
*.orig                  # 合并冲突时的原始文件
*.rej                   # 合并冲突时的拒绝文件

# Docker 相关
.dockerignore           # Docker忽略文件
docker-compose.override.yml  # 本地覆盖的docker-compose配置
docker-compose.*.yml    # 其他环境的docker-compose配置
!docker-compose.yml     # 保留基础docker-compose配置
!docker-compose.base.yml # 保留基础docker-compose配置
volumes/                # Docker数据卷挂载目录

# 包管理器（如果项目中用到前端）
node_modules/           # Node.js依赖目录
package-lock.json       # npm锁定文件（可选提交）
yarn.lock               # Yarn锁定文件（可选提交）
pnpm-lock.yaml          # pnpm锁定文件（可选提交）
```

### 4.2 Java 项目专用（VSCode \+ IntelliJ IDEA 双支持）

适用于所有 Java 项目：Spring Boot/Maven/Gradle/ 微服务等

```gitignore
##############################################################################
# Java 项目 .gitignore (VSCode + IntelliJ IDEA)
##############################################################################

# ==============================================
# 第一部分：操作系统自动生成的文件（与Python项目完全相同）
# ==============================================

# Windows 系统文件
Thumbs.db
ehthumbs.db
Desktop.ini
$RECYCLE.BIN/
*.lnk

# macOS 系统文件
.DS_Store
._*
.AppleDouble
.LSOverride
.Trashes
.Spotlight-V100
.TemporaryItems

# Linux 系统文件
*~
*.swp
*.swo
*.bak
*.tmp
.#*
.Trash-*
.directory

# ==============================================
# 第二部分：敏感信息与临时文件（绝对不能提交！）
# ==============================================

# 环境变量与密钥文件
.env
.env.local
.env.development.local
.env.test.local
.env.production.local
.env.*.local
secrets.*
*.key
*.pem
*.jks                   # Java密钥库文件
*.keystore              # Java密钥库
*.truststore            # Java信任库
config.ini
config.yaml
config.json

# 日志文件
*.log
*.log.*
logs/
log/
nohup.out

# 临时与缓存文件
*.tmp
*.temp
*.cache
*.pid
*.lock
tmp/
temp/
cache/

# ==============================================
# 第三部分：Java 语言核心忽略规则
# ==============================================

# 编译产物
*.class                 # Java字节码文件
*.jar                   # JAR包文件
*.war                   # WAR包文件
*.ear                   # EAR包文件
*.nar                   # NAR包文件
*.zip                   # 压缩包
*.tar.gz                # tar.gz压缩包
*.rar                   # rar压缩包
*.7z                    # 7z压缩包

# 构建目录
target/                 # Maven默认构建输出目录（最重要）
build/                  # Gradle默认构建输出目录
bin/                    # 旧版IDE的二进制目录
out/                    # IDEA默认编译输出目录
gen/                    # 生成的源代码目录
generated/              # 通用生成代码目录
generated-sources/      # Maven生成的源代码
generated-test-sources/ # Maven生成的测试源代码

# ==============================================
# 第四部分：构建工具特定规则
# ==============================================

# Maven 构建工具
.mvn/                       # Maven wrapper目录
!/.mvn/wrapper/maven-wrapper.jar        # 保留Maven wrapper JAR
!/.mvn/wrapper/maven-wrapper.properties # 保留Maven wrapper配置
dependency-reduced-pom.xml  # 依赖精简后的pom文件
buildNumber.properties      # 构建号属性文件
.mvn/timing.properties      # Maven计时属性

# Gradle 构建工具
.gradle/                    # Gradle缓存目录
!gradle/wrapper/gradle-wrapper.jar          # 保留Gradle wrapper JAR
!gradle/wrapper/gradle-wrapper.properties   # 保留Gradle wrapper配置
.gradletasknamecache        # Gradle任务名缓存

# ==============================================
# 第五部分：常用Java框架特定规则
# ==============================================

# Spring Boot 框架
application-*.properties  # 特定环境的配置文件
application-*.yml         # 特定环境的YAML配置
application-*.yaml        # 特定环境的YAML配置
bootstrap-*.properties    # Spring Cloud引导配置
bootstrap-*.yml           # Spring Cloud引导YAML
bootstrap-*.yaml          # Spring Cloud引导YAML

# Hibernate 框架
hibernate.properties    # Hibernate配置文件
hibernate.cfg.xml       # Hibernate XML配置

# MyBatis 框架
mybatis-config.xml      # MyBatis主配置文件

# Tomcat 服务器
tomcat.ports            # Tomcat端口配置
tomcat-users.xml        # Tomcat用户配置
server.xml              # Tomcat服务器配置
context.xml             # Tomcat上下文配置

# ==============================================
# 第六部分：开发工具忽略规则
# ==============================================

# VSCode 编辑器
.vscode/
# 可选：如果团队需要统一配置，取消下面的注释
# !.vscode/settings.json
# !.vscode/launch.json
# !.vscode/tasks.json
# !.vscode/extensions.json

*.code-workspace
.history/
.vscode-test/

# IntelliJ IDEA (JetBrains 系列)
.idea/
*.iml
*.ipr
*.iws
out/
production/
classes/
artifacts/

# 可选：如果团队需要统一配置，取消下面的注释
# !.idea/runConfigurations/
# !.idea/codeStyles/
# !.idea/inspectionProfiles/

# IDEA 特定文件
.idea_modules/
atlassian-ide-plugin.xml
com_crashlytics_export_strings.xml
crashlytics.properties
crashlytics-build.properties
fabric.properties
lint.xml

# ==============================================
# 第七部分：其他常用工具
# ==============================================

# Git 相关
*.patch
*.diff
*.orig
*.rej

# Docker 相关
.dockerignore
docker-compose.override.yml
docker-compose.*.yml
!docker-compose.yml
!docker-compose.base.yml
volumes/

# CI/CD 相关
.github/
!.github/workflows/     # 保留GitHub Actions工作流
!.github/actions/       # 保留GitHub Actions自定义动作
.gitlab-ci.yml          # GitLab CI配置
.travis.yml             # Travis CI配置
.circleci/              # CircleCI配置
.jenkinsfile            # Jenkins配置
```

## 五、高级技巧与最佳实践

### 5.1 保留空目录

Git 不会跟踪空目录。如果需要保留目录结构，可以在空目录中创建一个名为`.gitkeep`的空文件：

```gitignore
# 忽略logs目录下的所有文件，但保留目录本身
logs/*
!logs/.gitkeep
```

### 5.2 复杂目录排除

```gitignore
# 排除根目录下所有内容，但保留src目录和README.md
/*
!/src
!/README.md

# 排除src目录下的所有内容，但保留src/main
/src/*
!/src/main
```

### 5.3 最佳实践

- **尽早创建**：在项目初始化时就创建`.gitignore`文件

- **使用官方模板**：优先使用 GitHub 官方模板，避免遗漏常见文件

- **分类组织**：用注释将规则按类别分组（如依赖、构建产物、编辑器文件）

- **避免过度宽泛**：不要使用过于宽泛的规则（如`*.txt`），可能会误删重要文件

- **区分级别**：项目级只放所有开发者都需要忽略的文件，个人文件放在全局或本地忽略中

## 六、常见问题与故障排除

### 6.1 文件已经被跟踪，如何忽略？

如果文件已经被 Git 跟踪，添加到`.gitignore`后不会自动停止跟踪。需要先从 Git 索引中移除：

```bash
# 从索引中移除文件（保留本地文件）
git rm --cached filename

# 从索引中移除整个目录
git rm -r --cached directory/

# 提交更改
git commit -m "Stop tracking ignored files"
```

### 6.2 检查文件是否被忽略

使用`git check-ignore`命令调试忽略规则：

```bash
# 检查单个文件
git check-ignore -v filename

# 检查多个文件
git check-ignore -v *.log
```

`-v`参数会显示是哪个规则导致文件被忽略。

### 6.3 否定模式不生效

如果否定模式不生效，通常是因为父目录已经被忽略。Git 不会重新检查已经被忽略的目录中的文件：

```bash
# 错误写法
logs/
!logs/important.log  # 不生效，因为logs目录已被忽略

# 正确写法
logs/*
!logs/important.log
```

## 七、实用工具

- **[gitignore.io]\(gitignore.io\)**：在线生成自定义`.gitignore`文件，支持选择操作系统、编程语言和 IDE

- **GitHub 模板选择器**：在 GitHub 创建新仓库时，可以直接选择对应的`.gitignore`模板

- **VSCode 插件**：如`.gitignore Generator`，可以在编辑器中快速生成模板

## 核心知识点速览

- `.gitignore` 是 Git 中用于指定忽略文件的纯文本配置文件，可防止敏感数据和临时文件被提交

- Git 忽略机制分为项目级、本地级、全局级三个级别，分别适用于不同场景

- 支持`*`、`**`、`?`、`!`等模式匹配，可实现灵活的文件过滤

- 已被跟踪的文件添加到`.gitignore`后不会自动生效，需要用`git rm --cached`手动移除

- 使用`git check-ignore -v`可以调试忽略规则，查看文件被哪条规则忽略

- 否定模式不生效通常是因为父目录已被忽略，需要先排除子文件再否定

- Python 项目需忽略虚拟环境、字节码缓存、数据模型文件等

- Java 项目需忽略编译产物、构建目录、密钥库文件等

- 可通过`.gitkeep`文件保留 Git 不跟踪的空目录

- 优先使用 GitHub 官方模板，避免遗漏常见的忽略规则
