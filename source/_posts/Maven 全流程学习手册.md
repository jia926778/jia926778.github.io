---
title: Maven 学习手册
date: 2026-04-26 19:30:00
tags: [java, 技术分享, Maven]
categories: java
toc: true
---
# Maven 全流程学习手册

## 文档核心说明

本文档适用于 Java 开发入门者、需要快速搭建 Maven 开发环境的开发者，整合了 Maven 环境准备、下载安装、系统配置、核心配置、项目使用、IDE 集成的完整步骤，所有操作均为生产环境验证的稳定方案，新手可直接按步骤执行。

## 一、前置准备：JDK 环境

Maven 基于 Java 开发，必须先配置 JDK 环境，版本兼容规则如下：

|Maven 版本|最低 JDK 要求|推荐生产版本|
|---|---|---|
|3.9.x 稳定版|JDK 8+|JDK 8 / JDK 17 / JDK 21（LTS 版本）|
|4.0.x 预览版|JDK 17+|不推荐生产环境使用|

### 验证 JDK 环境

打开终端 / 命令提示符，执行以下命令，均正常输出版本信息即为配置成功：

```bash
# 验证Java运行环境
java -version
# 验证Java编译环境（必须配置JAVA_HOME才会生效）
javac -version
```

若命令报错，需先完成 JDK 安装与`JAVA_HOME`环境变量配置，再进行后续步骤。

## 二、Maven 下载与安装

### 1. 官方下载

- 官方下载地址：[https://maven.apache.org/download.cgi](https://maven.apache.org/download.cgi)

- 生产环境推荐：**最新稳定版 apache-maven-3.9.15**

- 安装包选择规则：

    |    系统|    推荐安装包|    说明|
    |---|---|---|
    |    Windows| apache-maven-3.9.15-bin.zip |    二进制免安装版，解压即可用|
    |    macOS/Linux| apache-maven-3.9.15-bin.tar.gz |    二进制压缩包，解压即可用|

> ❗ **避坑提醒**：不要下载`src`源码包，必须下载带`bin`的二进制包；解压路径**严禁包含中文、空格、特殊字符**，推荐路径：
>
> - Windows：`D:\dev\apache-maven-3.9.15`
>
> - macOS/Linux：`/usr/local/apache-maven-3.9.15` 或 `\~/opt/apache-maven-3.9.15`
>
>

### 2. 解压安装

- Windows：右键 zip 包，选择「解压到当前文件夹」，放到上述推荐路径即可

- macOS/Linux：终端执行解压命令

    ```bash
    # 对应下载的tar.gz包路径
    tar -zxvf apache-maven-3.9.15-bin.tar.gz
    # 移动到目标目录（示例）
    sudo mv apache-maven-3.9.15 /usr/local/
    ```

## 三、系统环境变量配置

环境变量的作用是让系统在任意目录都能识别`mvn`命令，统一管理 Maven 路径，方便版本切换。

### 核心环境变量说明

|变量名|作用|是否必须|示例|
|---|---|---|---|
|**`MAVEN_HOME`**|Maven 安装根目录（bin 的上一级）|✅ 推荐|`D:\dev\apache-maven-3.9.15`|
|**`PATH`**|系统查找可执行文件的路径|✅ 必须|`%MAVEN_HOME%\bin`（Win）/ `$MAVEN_HOME/bin`（macOS/Linux）|
|**`MAVEN_OPTS`**|JVM 启动参数（内存、编码等）|❌ 可选|`-Xms512m -Xmx1g -Dfile.encoding=UTF-8`|

### 1. Windows 系统配置

1. 右键「此电脑」→「属性」→「高级系统设置」→「环境变量」

2. 在**系统变量**区域，点击「新建」：

    - 变量名：`MAVEN_HOME`

    - 变量值：Maven 解压根目录（如`D:\dev\apache-maven-3.9.15`）

3. 找到系统变量中的`Path`，双击编辑，点击「新建」，添加：

    ```Plain Text
    %MAVEN_HOME%\bin
    ```

4. 可选：新建系统变量`MAVEN_OPTS`，值为`-Xms512m -Xmx1g -Dfile.encoding=UTF-8`

5. 依次点击所有窗口的「确定」，配置完成。

### 2. macOS/Linux 系统配置

1. 打开终端，先确认当前使用的 shell：

    ```bash
    echo $SHELL
    # 输出/bin/bash → 编辑~/.bash_profile
    # 输出/bin/zsh → 编辑~/.zshrc（macOS 10.15+默认）
    ```

2. 执行编辑命令（以 zsh 为例）：

    ```bash
    vim ~/.zshrc
    ```

3. 按`i`进入编辑模式，在文件末尾添加以下内容：

    ```bash
    # Maven 环境变量
    export MAVEN_HOME=/usr/local/apache-maven-3.9.15
    export PATH=$MAVEN_HOME/bin:$PATH
    # 可选：JVM参数
    export MAVEN_OPTS="-Xms512m -Xmx1g -Dfile.encoding=UTF-8"
    ```

4. 按`Esc`，输入`:wq`保存退出，执行以下命令让配置立即生效：

    ```bash
    source ~/.zshrc
    # bash用户执行 source ~/.bash_profile
    ```

### 3. 安装验证

**必须打开新的终端 / CMD 窗口**（旧窗口无法读取新配置的环境变量），执行命令：

```bash
mvn -v
```

成功输出包含 Maven 版本、Java 版本、系统信息的内容，即为安装配置成功。

## 四、Maven 核心配置（settings.xml）

`settings.xml`是 Maven 的核心配置文件，用于控制本地仓库、镜像源、全局编译规则等，分为 2 个级别：

- **全局配置**：Maven 安装目录下的`conf/settings.xml`，对所有用户生效

- **用户级配置**：用户目录下的`.m2/settings.xml`，仅对当前用户生效，**优先级更高**，重装 Maven 不会丢失，推荐使用。

> 若用户目录下没有`.m2`文件夹，手动创建即可；没有`settings.xml`，直接新建该文件，复制下方完整配置模板。
>
>

### 完整配置模板（开箱即用）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.2.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.2.0 http://maven.apache.org/xsd/settings-1.2.0.xsd">

  <!-- 1. 本地仓库配置：自定义依赖包存放路径，默认在C盘用户目录下的.m2/repository -->
  <localRepository>D:/dev/maven-repository</localRepository>

  <!-- 2. 镜像源配置：国内优先使用阿里云镜像，解决国外中央仓库下载慢/失败问题 -->
  <mirrors>
    <mirror>
      <id>aliyunmaven</id>
      <name>阿里云公共仓库</name>
      <url>https://maven.aliyun.com/repository/public</url>
      <mirrorOf>central</mirrorOf>
    </mirror>
  </mirrors>

  <!-- 3. 全局JDK编译版本配置：避免每个项目都重复配置JDK版本 -->
  <profiles>
    <profile>
      <id>jdk-1.8</id>
      <activation>
        <activeByDefault>true</activeByDefault>
        <jdk>1.8</jdk>
      </activation>
      <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
      </properties>
    </profile>
  </profiles>

</settings>
```

## 五、Maven 核心概念与基础使用

### 1. 标准项目结构

Maven 遵循**约定大于配置**原则，所有项目必须遵循标准目录结构，无需额外配置即可自动识别：

```Plain Text
my-maven-project        # 项目根目录
├── pom.xml             # 项目核心配置文件，必须存在
├── src
│   ├── main
│   │   ├── java        # 项目Java源码目录
│   │   └── resources   # 项目配置文件目录
│   └── test
│       ├── java        # 测试代码目录
│       └── resources   # 测试配置文件目录
└── target              # 编译/打包输出目录，自动生成
```

### 2. 核心文件：pom.xml 详解

`pom.xml`（Project Object Model，项目对象模型）是 Maven 项目的灵魂，定义了项目的坐标、依赖、构建规则等，最小可运行模板如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- 项目GAV坐标：全球唯一标识，定位一个项目/依赖包 -->
    <groupId>com.example</groupId>    <!-- 组织ID，一般是公司域名倒序 -->
    <artifactId>my-demo</artifactId>  <!-- 项目/模块ID，项目唯一名称 -->
    <version>1.0.0</version>          <!-- 版本号，SNAPSHOT为快照版，RELEASE为正式版 -->
    <packaging>jar</packaging>        <!-- 打包类型，默认jar，可选war/pom等 -->

    <!-- 依赖管理：所有第三方jar包都在这里声明 -->
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>  <!-- 依赖作用范围 -->
        </dependency>
    </dependencies>

</project>
```

#### 依赖 scope 作用范围

|scope 值|作用|生效范围|示例|
|---|---|---|---|
|compile|默认值|编译、测试、运行、打包全阶段生效|spring-core|
|test|仅测试阶段生效|仅测试代码编译、执行时生效|junit|
|provided|编译测试生效，运行时不生效，不会打包|编译、测试|servlet-api（Tomcat 容器提供）|
|runtime|运行时生效，编译时不生效|测试、运行|mysql-connector-java|

### 3. 三大生命周期与核心命令

Maven 定义了三套生命周期，**执行后面的阶段，会自动执行前面的所有前置阶段**：

|生命周期|核心作用|
|---|---|
|clean|清理项目，删除 target 目录|
|default（build）|核心构建，完成编译、测试、打包、部署全流程|
|site|生成项目文档，日常开发极少使用|

#### 高频常用命令

|命令|核心作用|
|---|---|
|`mvn clean`|清理项目，删除 target 目录|
|`mvn compile`|编译主源码，检查编译错误|
|`mvn test`|执行所有测试用例|
|`mvn package`|编译测试后，打包成 jar/war 包|
|`mvn install`|打包后，安装到本地 Maven 仓库|
|`mvn dependency:tree`|打印依赖树，排查 Jar 包冲突|
|`mvn clean package -DskipTests`|打包时跳过测试，快速生成产物|

## 六、VSCode 中集成与使用 Maven

### 1. 前置准备

确保系统级 JDK、Maven 环境已配置完成，终端执行`mvn -v`可正常输出版本信息。

### 2. 安装必备插件

1. 打开 VSCode 扩展商店（快捷键`Ctrl+Shift+X`）

2. 搜索安装 **Extension Pack for Java**（微软官方 Java 全家桶），该插件包已内置`Maven for Java`核心插件，同时包含语言支持、调试器等所有 Java 开发必备工具

3. 安装完成后重启 VSCode

### 3. VSCode Maven 核心配置

默认情况下 VSCode 会自动读取系统环境变量和用户级`settings.xml`，识别失败可手动配置：

1. 打开 VSCode 设置（快捷键`Ctrl+,`），点击右上角打开`settings.json`

2. 添加以下配置，根据自己的路径修改：

```json
{
  // 指定Maven可执行文件路径
  "maven.executable.path": "D:\dev\apache-maven-3.9.15\bin\mvn.cmd",
  // 自定义Maven JVM参数
  "maven.terminal.customEnv": [
    {
      "environmentVariable": "MAVEN_OPTS",
      "value": "-Xms512m -Xmx1g -Dfile.encoding=UTF-8"
    }
  ],
  // 自动下载依赖源码和JavaDoc
  "maven.autoDownloadSources": true,
  "maven.autoDownloadJavadoc": true,
  // JDK路径配置
  "java.home": "D:\dev\jdk1.8.0_402",
  // 开启注解处理，解决Lombok不生效
  "java.annotations.processor.enabled": true,
  // 文件编码
  "files.encoding": "utf8"
}
```

### 4. 创建 / 导入 Maven 项目

#### 创建新项目

1. 打开命令面板（`Ctrl+Shift+P`），选择`Maven: Create Maven Project`

2. 选择项目模板，最常用`maven-archetype-quickstart`（标准 Java 项目）

3. 填写 GAV 坐标，选择保存路径，等待自动生成项目结构

#### 导入已有项目

1. 点击「文件」→「打开文件夹」，选择包含`pom.xml`的项目根目录

2. VSCode 会自动识别 Maven 项目，加载依赖，识别失败可右键`pom.xml`选择「Update Project Configuration」

### 5. 日常常用操作

1. **执行 Maven 命令**：左侧打开 Maven 侧边栏，展开项目→Lifecycle，双击对应命令即可执行

2. **自定义命令**：命令面板选择`Maven: Execute Commands`，输入自定义命令（如`dependency:tree`）

3. **运行调试**：右键 Java 主类，选择`Run Java`/`Debug Java`即可运行调试

4. **多模块项目**：打开父项目根目录，VSCode 会自动识别所有子模块，可单独对模块执行命令

## 七、常见问题与解决方案

### 1. `mvn`命令提示 “不是内部或外部命令”

- 必须打开新终端 / CMD 窗口，旧窗口无法读取新环境变量

- 检查`MAVEN_HOME`路径是否为 Maven 根目录，Path 中是否添加了`%MAVEN_HOME%\bin`

- 检查路径是否包含中文、空格，修改为纯英文路径

### 2. 报错`JAVA_HOME not set`

- 先配置 JDK 的`JAVA_HOME`环境变量，Maven 依赖 Java 环境

- 可在 VSCode 的`maven.terminal.customEnv`中单独配置 JAVA_HOME

### 3. 依赖下载慢 / 下载失败

- 检查`settings.xml`是否正确配置了阿里云镜像

- 删除本地仓库中对应依赖的`.lastUpdated`文件，重新刷新依赖

- 关闭 VPN，检查网络是否正常

### 4. Maven 项目识别失败，pom.xml 图标不对

- 检查是否安装了 Java 全家桶插件，重启 VSCode

- 确保打开了包含 pom.xml 的项目根目录

- 执行`Java: Clean Java Language Server Workspace`清理缓存

### 5. 中文乱码 / 编译报错

- 在`settings.xml`中配置 UTF-8 编码

- 在`MAVEN_OPTS`中添加`-Dfile.encoding=UTF-8`

- VSCode 文件编码设置为 UTF-8

### 6. Jar 包冲突（ClassNotFoundException/NoSuchMethodError）

- 执行`mvn dependency:tree`查看依赖树，定位冲突版本

- 在冲突依赖中使用`<exclusions>`排除不需要的版本

### 7. Lombok 注解不生效

- 安装 Lombok 插件，开启注解处理：`"java.annotations.processor.enabled": true`

### 8. 大项目打包内存溢出

- 调大`MAVEN_OPTS`的内存配置，比如改为`-Xms1g -Xmx2g`

## 核心知识点速览

- Maven 是 Java 项目的依赖管理与构建工具，核心遵循**约定大于配置**原则

- 系统环境变量核心是配置`MAVEN_HOME`+`PATH`，实现全局`mvn`命令调用

- `settings.xml`推荐使用用户级配置，可自定义本地仓库、阿里云镜像加速下载

- Maven 生命周期执行后序阶段会自动执行所有前置阶段，常用组合命令`mvn clean package`

- pom.xml 通过**GAV 坐标**唯一标识项目与依赖，scope 控制依赖的生效范围

- VSCode 集成 Maven 只需安装官方 Java 全家桶插件，可自动读取系统 Maven 配置

- 排查 Jar 包冲突可通过`mvn dependency:tree`查看完整依赖传递关系

- 所有配置路径严禁包含中文、空格，避免出现识别错误

- 国内开发必须配置阿里云镜像，解决中央仓库下载慢的问题

- 多版本 Maven 切换只需修改`MAVEN_HOME`路径，无需修改其他配置
