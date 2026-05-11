---
title: VSCode启动Java项目教程
date: 2026-05-02 19:30:00
tags: [VSCode, 工具使用, java]
categories: 工具使用
toc: true
---

# VSCode启动Java项目详细教程

VSCode 凭借轻量、高效的特性，已经成为越来越多 Java 开发者的选择，相比传统重型 IDE，它启动更快、占用资源更少，同时通过插件生态可以覆盖 Java 开发的全流程需求。本教程将从环境准备到项目启动，一步步带你完成 VSCode Java 开发环境的搭建，覆盖单文件、Maven/Gradle、Spring Boot 等常见项目场景。

## 一、前置环境准备

在开始配置之前，我们需要先准备好 Java 开发的基础依赖。

### 1.1 JDK 安装与验证

JDK 是 Java 开发的核心依赖，没有它无法运行任何 Java 程序。

- **版本选择**：推荐选择 LTS 长期支持版本，比如 JDK 17 或 JDK 21，这两个版本拥有至少 6 年的官方维护周期，兼容性和稳定性最佳，也是目前主流的选择。

- **下载渠道**：可以从 OpenJDK 的开源发行版下载，比如 Eclipse Temurin、Amazon Corretto、Azul Zulu，也可以从清华大学开源镜像站下载，速度更快。

- **安装与验证**：

    1. 下载安装包后，按照指引完成安装，记住你的安装路径（比如 Windows 下的`C:\\Program Files\Java\jdk-17`，macOS 下的`/Library/Java/JavaVirtualMachines/jdk-17.jdk/Contents/Home`）。

    2. 安装完成后，配置系统环境变量：将 JDK 的`bin`目录添加到系统的`Path`变量中，同时配置`JAVA_HOME`变量指向 JDK 的根目录。

    3. 打开终端（CMD 或 Terminal），输入以下命令验证是否安装成功：

        ```bash
        java -version
        ```

        如果输出类似以下内容，说明 JDK 安装配置成功：

        ```Plain Text
        java version "17.0.8" 2023-07-18 LTS
        Java(TM) SE Runtime Environment (build 17.0.8+9-LTS-211)
        ```

### 1.2 VSCode 安装

VSCode 的安装非常简单，直接从[VSCode 官网](https://code.visualstudio.com/)下载对应系统的安装包，一键安装即可。

> 安装时建议勾选「添加到 PATH 环境变量」选项，方便后续在终端直接调用 VSCode。
>
>

如果你是 Windows 或 macOS 用户，也可以直接下载微软提供的**Java 编码包（Coding Pack for Java）**，它是 VSCode、JDK 和 Java 插件的整合包，一键就能完成基础环境的安装，非常适合新手。

## 二、核心插件安装

VSCode 本身不支持 Java 的原生开发，需要通过插件来实现代码补全、调试、项目管理等功能。

### 2.1 必装核心插件包

我们推荐直接安装微软官方的`Extension Pack for Java`插件包，它已经整合了 Java 开发所需的所有核心插件，无需单独安装：

1. 打开 VSCode，按下`Ctrl+Shift+X`（macOS 下是`Cmd+Shift+X`）打开扩展商店。

2. 在搜索框中输入`Extension Pack for Java`，找到 Red Hat 官方发布的插件包。

3. 点击「Install」按钮安装，安装完成后重启 VSCode 即可生效。

这个插件包包含了 6 个核心插件，覆盖了 Java 开发的全流程：

- Language Support for Java™ by Red Hat：提供 Java 语言支持、代码补全、语法检查

- Debugger for Java：Java 代码调试功能

- Test Runner for Java：单元测试运行支持

- Maven for Java：Maven 项目管理支持

- Project Manager for Java：Java 项目管理

- Visual Studio IntelliCode：AI 辅助的代码补全

### 2.2 可选扩展插件

根据你的开发场景，还可以安装以下可选插件，进一步提升开发效率：

- **Spring Boot Extension Pack**：如果你开发 Spring Boot 项目，这个插件包可以提供 Spring 项目的自动补全、依赖管理、Dashboard 等功能，大幅提升开发效率。

- **Gradle for Java**：如果你使用 Gradle 构建项目，安装这个插件可以获得 Gradle 项目的自动构建和管理支持。

- **Chinese \(Simplified\) Language Pack**：中文语言包，将 VSCode 界面转换为中文，降低英文界面的操作门槛。

## 三、基础环境配置

安装完插件后，我们需要对 VSCode 进行一些基础配置，让它能正确识别你的 Java 环境。

### 3.1 配置 JDK 路径

如果 VSCode 没有自动识别到你的 JDK，我们可以手动指定 JDK 的路径：

1. 按下`Ctrl+,`（macOS 下是`Cmd+,`）打开 VSCode 的设置界面。

2. 在搜索框中输入`java.home`，找到「Java: Home」配置项。

3. 点击「在 settings.json 中编辑」，在打开的配置文件中添加你的 JDK 路径：

    ```json
    // Windows示例
    "java.home": "C:\\Program Files\Java\jdk-17",
    // macOS/Linux示例
    // "java.home": "/Library/Java/JavaVirtualMachines/jdk-17.jdk/Contents/Home",
    ```

    > 注意：路径需要替换成你自己的 JDK 安装路径，Windows 下路径的反斜杠需要用双反斜杠`\\`转义。
    >
    >

### 3.2 常用优化配置

为了提升开发体验，我们还可以添加一些常用的优化配置，同样在`settings.json`中添加：

```json
// 解决中文乱码问题
"files.encoding": "utf-8",
"java.jdt.ls.vmargs": "-Dfile.encoding=utf-8",
// 配置Maven阿里云镜像，提升依赖下载速度
"maven.settings": "~/maven/settings.xml",
// 开启自动保存
"files.autoSave": "afterDelay",
```

## 四、不同类型 Java 项目的启动方式

根据你项目的类型，启动方式会略有不同，下面我们分别介绍最常见的几种场景。

### 4.1 单个 Java 文件快速运行

如果你只是想快速验证一段 Java 代码，不需要创建完整项目，可以直接运行单个 Java 文件：

1. 在 VSCode 中打开你的 Java 文件（比如`HelloWorld.java`），确保文件中包含`main`方法：

    ```java
    public class HelloWorld {
        public static void main(String[] args) {
            System.out.println("Hello, VS Code Java!");
        }
    }
    ```

2. 有三种方式可以运行这个文件：

    - 方式 1：点击代码行号左侧的绿色箭头，选择「Run Java」。

    - 方式 2：点击编辑器右上角的「运行」按钮。

    - 方式 3：在代码文件中右键，选择「Run Java」。

3. 运行完成后，VSCode 会自动打开终端，输出代码的运行结果。

> 注意：单个 Java 文件的文件名必须和公共类名完全一致，且区分大小写，否则会出现「找不到主类」的错误。
>

### 4.2 Maven 项目（含多模块）导入与启动

Maven 是目前企业级 Java 项目最常用的构建工具，VSCode 可以完美支持 Maven 项目的导入和运行：

1. **导入项目**：将你的 Maven 项目文件夹拖入 VSCode 窗口，VSCode 会自动识别项目中的`pom.xml`文件，自动开始下载项目依赖。等待依赖下载完成即可。

2. **启动项目**：

    - 方式 1：找到项目的主类（包含`main`方法的类），按照单个文件的方式，点击运行按钮即可启动。

    - 方式 2：使用 Maven 面板运行：

        1. 点击 VSCode 左侧的「Maven」图标，打开 Maven 项目面板。

        2. 展开你的项目，找到`Lifecycle`下的`tomcat7:run`（Web 项目）或者`spring-boot:run`（Spring Boot 项目），或者`exec:java`。

        3. 右键点击对应的任务，选择「Run」即可启动项目。

    - 方式 3：终端手动运行：打开 VSCode 的终端，进入项目根目录，执行 Maven 命令：

        ```bash
        # 普通Java项目
        mvn exec:java -Dexec.mainClass="com.example.Main"
        # Spring Boot项目
        mvn spring-boot:run
        # Web项目打包运行
        mvn tomcat7:run
        ```

> 对于多模块 Maven 项目，VSCode 会自动识别子模块，你只需要在 Maven 面板中选择对应的子模块，执行对应的任务即可。
>

### 4.3 Gradle 项目启动

如果你使用 Gradle 构建项目，步骤和 Maven 类似：

1. 将项目文件夹拖入 VSCode，VSCode 会自动识别`build.gradle`文件，自动下载依赖。

2. 启动项目：

    - 方式 1：直接运行主类，和单个文件的方式一致。

    - 方式 2：打开左侧的 Gradle 面板，找到对应的任务，右键点击运行。

    - 方式 3：终端执行 Gradle 命令：

        ```bash
        gradle bootRun
        ```

### 4.4 Spring Boot 项目启动

Spring Boot 项目是目前最流行的 Java 微服务项目，VSCode 对它有很好的支持：

1. 如果你是新建 Spring Boot 项目，可以安装`Spring Boot Extension Pack`插件后，按下`Ctrl+Shift+P`打开命令面板，输入`Spring Initializr`，按照指引快速创建 Spring Boot 项目。

2. 导入已有的 Spring Boot 项目后，等待依赖下载完成。

3. 启动项目：

    - 最简单的方式：找到项目的启动类（带有`@SpringBootApplication`注解的类），点击行号左侧的绿色箭头，选择「Run Java」即可启动。

    - 也可以使用 Maven/Gradle 命令运行，和之前的步骤一致。

    - 启动完成后，就可以访问你的 Spring Boot 服务了，比如默认的`http://localhost:8080`。

## 五、代码调试技巧

VSCode 的 Java 调试功能非常强大，可以帮助你快速排查代码问题：

1. **设置断点**：在你需要调试的代码行号左侧点击，会出现一个红色的圆点，这就是断点，代码运行到这里会暂停。

2. **启动调试**：点击行号左侧的绿色箭头，选择「Debug Java」，或者直接按下`F5`启动调试。

3. **调试操作**：

    - 步进（F10）：逐行执行代码，进入下一行。

    - 步入（F11）：进入当前调用的方法内部。

    - 步出（Shift+F11）：跳出当前方法。

    - 继续（F5）：继续执行到下一个断点。

4. 在调试过程中，你可以在左侧的调试面板中查看当前变量的取值，也可以在调试控制台输入表达式，实时查看变量的值。

## 六、常见问题与排查

在配置和运行的过程中，你可能会遇到一些常见问题，这里整理了对应的解决方法：

### ❌ 问题 1：提示「找不到主类」

- 检查文件名和公共类名是否完全一致，Java 严格区分大小写，比如类名是`HelloWorld`，文件名必须是`HelloWorld.java`。

- 检查项目的源代码目录是否配置正确，确保 Java 文件在`src/main/java`目录下。

- 重启 VSCode，重新加载项目。

### ❌ 问题 2：中文乱码

- 在`settings.json`中添加编码配置，将文件编码设置为 UTF-8，参考之前的优化配置部分。

- 检查你的 Java 文件的编码格式，确保文件本身的编码和配置的编码一致。

### ❌ 问题 3：Maven 依赖下载失败 / 慢

- 配置阿里云 Maven 镜像，替换默认的中央仓库，大幅提升下载速度。

- 删除本地 Maven 仓库的缓存（`\~/.m2/repository`），重新下载依赖。

- 检查你的网络连接，确保可以访问 Maven 仓库。

### ❌ 问题 4：VSCode 提示「找不到 JDK」

- 检查你在`settings.json`中配置的`java.home`路径是否正确，必须指向 JDK 的根目录，而不是 JRE。

- 检查系统环境变量中的`JAVA_HOME`是否配置正确。

### ❌ 问题 5：运行按钮不显示

- 检查你是否安装了`Extension Pack for Java`插件，并且重启了 VSCode。

- 检查你的 Java 文件中是否包含正确的`main`方法，VSCode 只有识别到主方法才会显示运行按钮。

## 总结

通过以上步骤，你已经完成了 VSCode Java 开发环境的搭建，并且可以运行不同类型的 Java 项目了。VSCode 的轻量特性可以让你快速启动项目，相比传统 IDE，它的启动速度更快，占用资源更少，非常适合日常的 Java 开发。

如果遇到其他问题，可以先查看 VSCode 的终端输出，大部分错误都会有明确的提示，根据提示排查即可解决。
