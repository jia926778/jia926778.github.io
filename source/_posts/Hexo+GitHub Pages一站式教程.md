---
title: Hexo+GitHub Pages一站式教程
date: 2026-04-13 14:30:00
tags: [Hexo, 博客搭建]
categories: 技术教程
toc: true
---

# Hexo+GitHub Pages一站式教程

> 本手册整合了完整的零基础搭建教程，以及部署、文章、主题阶段的所有常见问题排查方案，覆盖你遇到的所有问题，一站式解决，无需再翻找零散教程。
>
>

---

## 第一部分：完整零基础搭建教程

本部分为完整的 Hexo+GitHub Pages 搭建全流程，面向新手，步骤可复现。

### 一、前置准备（缺一不可）

搭建前必须完成 3 项基础环境 / 账号准备，提前做好可避免 90% 的新手踩坑。

#### 1. 安装 Node.js（Hexo 运行核心）

Hexo 基于 Node.js 开发，必须先安装 Node.js 环境，推荐安装**长期支持版 (LTS)**，避免版本兼容问题。

1. 官网下载：[Node.js 官网](https://nodejs.org/zh-cn/)，选择对应系统的 LTS 版本，默认下一步安装即可（Windows 系统建议勾选「Add to PATH」自动配置环境变量）。

2. 安装验证：安装完成后，打开终端（Windows 用 CMD/Git Bash，Mac/Linux 用系统终端），输入以下命令，输出版本号即安装成功。

    ```bash

    node -v
    npm -v
    ```

3. 可选优化（解决 npm 下载慢）：切换国内淘宝镜像源

    ```bash

    npm config set registry https://registry.npmmirror.com
    ```

#### 2. 安装 Git（部署与版本控制核心）

Hexo 需通过 Git 将本地博客文件推送到 GitHub 仓库，必须安装 Git 工具。

1. 官网下载：[Git 官网](https://git-scm.com/downloads)，选择对应系统版本，默认下一步安装即可。

2. 安装验证：终端输入以下命令，输出版本号即安装成功。

    ```bash

    git --version
    ```

3. 全局配置 Git（必须配置，否则无法提交代码）：

    ```bash

    # 替换为你的GitHub注册用户名与邮箱
    git config --global user.name "你的GitHub用户名"
    git config --global user.email "你的GitHub注册邮箱"
    ```

#### 3. 注册 GitHub 账号

GitHub Pages 是 GitHub 提供的免费静态页面托管服务，需先拥有 GitHub 账号。

1. 官网注册：[GitHub 官网](https://github.com/)，使用常用邮箱注册，记住用户名和邮箱（后续全程会用到）。

2. 账号完成邮箱验证，否则无法创建仓库和开启 Pages 服务。

### 二、本地 Hexo 环境搭建与初始化

完成前置准备后，先在本地搭建完整的 Hexo 博客，本地预览无误后再部署到线上。

#### 1. 全局安装 Hexo 脚手架

终端输入以下命令，全局安装 Hexo 官方脚手架，安装完成后可全局使用 `hexo` 命令。

```bash

npm install -g hexo-cli
```

安装验证：输入以下命令，输出版本号即安装成功。

```bash

hexo -v
```

#### 2. 初始化 Hexo 博客项目

1. 在电脑上新建一个文件夹（比如命名为 `my-hexo-blog`），作为博客的本地根目录，**路径不要包含中文和空格**。

2. 终端进入该文件夹（Windows：右键文件夹选择「Git Bash Here」；Mac/Linux：终端 cd 到文件夹路径）。

3. 执行初始化命令，Hexo 会自动生成博客所需的所有基础文件：

    ```bash

    hexo init
    ```

4. 安装项目依赖，初始化完成后执行以下命令，安装博客运行所需的所有依赖包：

    ```bash

    npm install
    ```

5. 初始化完成后，文件夹内会生成以下核心目录 / 文件，新手先记住核心作用即可：

    ```Plain Text

    my-hexo-blog/
    ├── _config.yml    # 博客核心配置文件，网站标题、作者、部署配置等全在这里修改
    ├── source/        # 资源文件夹，你的文章、图片都存在这里
    │   └── _posts/    # 所有博客文章都放在这个文件夹里，md格式
    ├── themes/        # 主题文件夹，存放博客主题，默认自带landscape主题
    └── package.json   # 项目依赖配置文件
    ```

#### 3. 本地启动预览博客

初始化完成后，即可在本地启动博客，预览效果。

1. 终端执行启动命令：

    ```bash

    hexo server
    # 简写 hexo s
    ```

2. 启动成功后，终端会提示 `Hexo is running at http://localhost:4000/`

3. 打开浏览器，访问 `http://localhost:4000`，即可看到 Hexo 默认的博客首页，说明本地环境搭建 100% 成功。

4. 停止本地服务：终端按 `Ctrl + C` 即可关闭。

> 小提示：如果提示 4000 端口被占用，可指定其他端口启动：`hexo s -p 5000`，访问对应端口即可。
>
>

### 三、GitHub Pages 仓库创建与 SSH 免密配置

本地博客搭建完成后，需要创建 GitHub 专属仓库，并配置 SSH 密钥，实现免密推送部署，避免每次部署都输入账号密码。

#### 1. 创建 GitHub Pages 专属仓库

**仓库名有严格规范，写错会导致 Pages 无法访问，必须严格按照要求填写**

1. 登录 GitHub，点击右上角 `+` 号，选择 `New repository` 新建仓库。

2. 仓库配置（核心必填项）：

    - Repository name（仓库名）：**必须是 ** **`你的GitHub用户名.github.io`**，比如你的用户名是 zhangsan，就填 `zhangsan.github.io`，大小写必须和用户名完全一致。

    - 权限：选择 `Public`（免费账号仅 Public 仓库可开启 GitHub Pages）。

    - 可选：勾选 `Add a README file`，自动生成 README 文件。

3. 点击 `Create repository`，完成仓库创建。

#### 2. 配置 SSH 密钥（免密部署核心）

SSH 密钥用于本地设备和 GitHub 之间的安全加密连接，配置后无需每次部署都输入账号密码，是部署成功的关键步骤。

1. 终端输入以下命令，生成 SSH 密钥对（替换为你的 GitHub 注册邮箱）：

    ```bash

    ssh-keygen -t ed25519 -C "你的GitHub注册邮箱"
    ```

2. 执行后全程按回车即可（无需设置密码，默认保存到用户目录的.ssh 文件夹下），直到出现密钥指纹和随机画，说明生成成功。

3. 找到生成的公钥文件：

    - Windows 系统：路径一般为 `C:\Users\你的用户名.ssh\id_ed25519.pub`

    - Mac/Linux 系统：路径一般为 `~/.ssh/id_ed25519.pub`

4. 用记事本 / 文本编辑器打开 `.pub` 后缀的公钥文件，**复制文件内的全部内容**（不要漏字符、不要改内容）。

5. 打开 GitHub，点击右上角头像 → `Settings` → 左侧找到 `SSH and GPG keys` → 点击 `New SSH key`。

6. 填写配置：

    - Title：随便填，比如「个人笔记本」，用于区分设备。

    - Key type：默认 `Authentication Key` 即可。

    - Key：粘贴刚才复制的公钥全部内容。

7. 点击 `Add SSH key`，完成密钥添加。

#### 3. 验证 SSH 连接

终端输入以下命令，验证本地和 GitHub 的连接是否成功：

```bash

ssh -T git@github.com
```

执行后会提示是否继续连接，输入 `yes` 回车。
如果终端出现 `Hi 你的用户名! You've successfully authenticated`，说明 SSH 配置成功，可正常免密部署。

### 四、Hexo 部署配置与线上发布

完成仓库和 SSH 配置后，只需修改 Hexo 核心配置，执行部署命令，即可将本地博客推送到 GitHub Pages，实现线上访问。

#### 1. 安装 Git 部署插件

Hexo 无法直接部署到 Git 仓库，必须先安装官方部署插件，否则执行部署命令会报错。
终端进入博客根目录，执行以下命令安装：

```bash

npm install hexo-deployer-git --save
```

#### 2. 修改 Hexo 核心配置文件_config.yml

打开博客根目录的 `_config.yml` 文件（用 VS Code / 记事本 / 其他文本编辑器都可），修改以下核心配置，**注意 YAML 语法规范：所有冒号后面必须加一个空格，缩进用 2 个空格，禁止用 Tab 键**。

##### ① 基础网站信息配置（可选，建议修改）

找到以下字段，替换为你自己的博客信息：

```yaml

# 网站标题
title: 我的个人博客
# 网站副标题
subtitle: 记录生活与技术
# 网站描述
description: 这是我用Hexo搭建的个人博客，分享技术、生活与思考
# 你的名字
author: 你的昵称
# 网站语言，简体中文填zh-CN
language: zh-CN
# 时区，国内填Asia/Shanghai
timezone: Asia/Shanghai
```

##### ② 网站 URL 配置（必须修改，否则样式会丢失）

找到 `url` 和 `root` 字段，修改为：

```yaml

# 替换为你的GitHub Pages地址，即https://你的用户名.github.io
url: https://zhangsan.github.io
# 根仓库部署，固定填/即可
root: /
```

##### ③ 部署配置（必须修改，核心中的核心）

拉到文件最底部，找到 `deploy` 字段，修改为以下内容，**严格注意缩进和空格**：

```yaml

deploy:
  type: git
  # 替换为你的GitHub仓库SSH地址（在仓库页面→Code→SSH里复制）
  repo: git@github.com:zhangsan/zhangsan.github.io.git
  # 分支名，GitHub默认主分支是main，老仓库可能是master，按实际填写
  branch: main
```

> 小提示：仓库 SSH 地址获取方式：打开你创建的仓库 → 点击绿色的 `Code` 按钮 → 切换到 `SSH` 选项卡 → 复制地址即可。
>
>

修改完成后，保存 `_config.yml` 文件。

#### 3. 执行部署命令上线

终端进入博客根目录，按顺序执行以下 3 条命令，**每次修改博客内容后重新部署，都要按这个顺序执行**：

1. 清除缓存和旧的静态文件（避免缓存导致的异常）

    ```bash

    hexo clean
    ```

2. 生成静态网页文件

    ```bash

    hexo generate
    # 简写 hexo g
    ```

3. 部署到 GitHub 仓库

    ```bash

    hexo deploy
    # 简写 hexo d
    ```

执行完成后，终端提示 `Deploy done: git`，说明部署成功。

#### 4. 访问你的线上博客

1. 部署成功后，等待 1-5 分钟（GitHub Pages 有缓存更新延迟）。

2. 打开浏览器，访问你的仓库名地址，比如 `https://zhangsan.github.io`，即可看到和本地预览一致的博客页面，恭喜你，个人博客正式上线！

### 五、核心操作：写文章、更换主题

#### 1. 发布你的第一篇博客文章

Hexo 文章采用 Markdown 格式，所有文章都存放在 `source/_posts` 目录下，有两种创建方式：

##### 方式一：命令创建（推荐，自动生成规范格式）

终端进入博客根目录，执行以下命令：

```bash

hexo new "我的第一篇博客文章"
```

执行后，会自动在 `source/_posts` 目录下生成 `我的第一篇博客文章.md` 文件。

##### 方式二：手动创建

直接在 `source/_posts` 目录下新建 `.md` 后缀的 Markdown 文件即可。

##### 文章编写规范

打开生成的 md 文件，顶部是文章的基础配置（Front-matter），下方是文章正文，用标准 Markdown 语法编写即可。

```markdown

---
title: 我的第一篇博客文章  # 文章标题，必填
date: 2024-05-20 14:30:00  # 文章发布时间，自动生成
tags: [Hexo, 博客搭建]      # 文章标签，可选
categories: 技术教程        # 文章分类，可选
---

这里是文章正文，用Markdown语法编写，支持标题、列表、图片、代码块等所有Markdown格式。

## 二级标题
这是正文内容，我的第一篇Hexo博客！
```

文章编写完成后，执行 `hexo clean && hexo g && hexo s` 本地预览，无误后执行 `hexo d` 部署到线上即可。

#### 2. 更换博客主题（以热门 Next 主题为例）

Hexo 拥有丰富的开源主题，默认的 landscape 主题比较基础，新手推荐先上手热门的 Next 主题，简洁美观、文档完善、扩展性强。

1. 安装主题：终端进入博客根目录，执行以下命令，将 Next 主题克隆到 themes 文件夹下：

    ```bash

    git clone https://github.com/next-theme/hexo-theme-next themes/next
    ```

2. 启用主题：打开根目录的 `_config.yml`，找到 `theme` 字段，将默认的 `landscape` 改为 `next`：

    ```yaml

    # 改成你要启用的主题名，和themes文件夹下的主题文件夹名完全一致
    theme: next
    ```

3. 预览与部署：执行 `hexo clean && hexo g && hexo s` 本地预览主题效果，无误后执行 `hexo d` 部署到线上即可。

4. 主题自定义：Next 主题的所有配置都在 `themes/next/_config.yml` 文件中，可根据官方文档修改布局、配色、插件等。

---

## 第二部分：部署阶段常见问题排查

本部分覆盖部署过程中最常见的报错与问题，按顺序排查即可快速解决。

### 1. 部署报错：Author identity unknown（Git 身份配置错误）

#### 报错现象

执行 `hexo deploy` 时，终端报错：

```Plain Text

Author identity unknown
*** Please tell me who you are.
Run
  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"
to set your account's default identity.
```

#### 根因说明

Git 在每次提交代码时，必须明确提交者的用户名和邮箱，Hexo 部署本质是通过 Git 向 GitHub 仓库提交代码，缺少身份信息就会触发该报错。

#### 修复步骤

1. 打开终端，执行以下 2 条命令，**必须替换成你自己的 GitHub 账号真实信息**：

    ```bash

    # 替换为你的GitHub用户名（和GitHub主页显示的用户名完全一致）
    git config --global user.name "你的GitHub用户名"
    # 替换为你的GitHub注册邮箱（和GitHub账号绑定的邮箱完全一致）
    git config --global user.email "你的GitHub注册邮箱"
    ```

2. 验证配置是否生效：

    ```bash

    # 查看已配置的用户名
    git config user.name
    # 查看已配置的邮箱
    git config user.email
    ```

3. 重新执行部署命令：

    ```bash

    hexo clean && hexo generate && hexo deploy
    ```

#### 兜底方案

如果全局配置后仍报错，给当前博客项目单独配置身份信息：

```bash

# 进入博客根目录
cd D:\jll-hexo-blog
# 单独配置当前项目的身份
git config user.name "你的GitHub用户名"
git config user.email "你的GitHub注册邮箱"
# 重新部署
hexo clean && hexo g && hexo d
```

---

### 2. URL 配置错误：样式丢失、404 问题修复

#### 常见故障现象

- 博客上线后样式完全丢失，只有纯文字

- 点击文章 / 分页链接出现 404

- Hexo 启动提示`Config validation error`

#### 根因说明

90% 的情况是**YAML 语法不规范**、**地址与 GitHub 真实信息不匹配**、**配套 root 配置缺失 / 错误**这三类问题。

#### 标准正确配置（根仓库部署）

针对你当前的`用户名.github.io`根仓库，标准配置如下，严格遵守即可解决问题：

```yaml

# 强制规范：冒号后必须加1个英文空格，地址完整无多余斜杠，大小写和GitHub完全一致
url: https://jia026778.github.io
# 根仓库部署，root必须固定为/，和url配套
root: /
```

#### 分步修复流程

1. **核对 GitHub 真实信息**
确保仓库名是 `jia026778.github.io`，和你的 GitHub 用户名**大小写完全一致**，GitHub Pages 的官方访问地址和你配置的 url 完全一样。

2. **修正_config.yml 配置**
打开博客根目录的`_config.yml`，找到 URL 配置块，完整替换为以下内容：

    ```yaml

    # URL
    ## 根仓库部署标准配置
    url: https://jia026778.github.io
    root: /
    permalink: :year/:month/:day/:title/
    permalink_defaults:
    pretty_urls:
      trailing_index: true
      trailing_html: true
    ```

3. **清除缓存并验证**

    ```bash

    # 清除旧缓存
    hexo clean
    # 用新配置重新生成
    hexo generate
    # 本地预览
    hexo server
    ```

    本地确认正常后，执行部署：

    ```bash

    hexo deploy
    ```

4. **线上验证**
部署后等待 3-5 分钟，浏览器按`Ctrl+F5`强制刷新，访问你的博客地址确认正常。

#### 必须避开的错误写法

- 冒号后无空格：`url:https://jia026778.github.io`（YAML 解析直接失败）

- 地址不完整：`url: jia026778.github.io`（缺少 https://）

- 结尾多余斜杠：`url: https://jia026778.github.io/`（路径拼接出错）

- 根仓库错误配置 root：`root: /jia026778.github.io/`

---

## 第三部分：文章与主题阶段常见问题排查

本部分覆盖文章渲染、主题侧边栏、导航的常见问题，解决`(no title)`显示异常。

### 1. 侧边栏「最新文章」显示 `(no title)` 修复

#### 故障现象

侧边栏的「最新文章」组件中，部分文章显示为 `(no title)`，无法正常显示文章标题。

#### 根因说明

Hexo 渲染侧边栏时，无法从文章中读取到有效的标题信息，因此用占位符 `(no title)` 显示。

#### 修复步骤

##### 第一步：修复文章 Front-matter（90% 问题直接解决）

Hexo 读取文章标题的优先级是：**Front-matter 中的 ** **`title`** ** 字段 > 文件名**。如果 `title` 字段缺失、格式错误，就会触发该问题。

1. 打开 `source/_posts/` 目录下的所有 `.md` 文章，检查文件开头的 Front-matter：

    ```markdown

    ---
    # 必须有 title 字段，冒号后必须加 1 个英文空格，不能用中文符号
    title: 大模型微调与RAG技术问答
    date: 2026-04-13 10:00:00
    tags: [大模型, RAG, 微调]
    categories: 技术教程
    ---
    ```

2. 清理无效文件：删除空文件、临时草稿、未写完的 `.md` 文件，给不需要发布的草稿添加`published: false`：

    ```markdown

    ---
    title: 临时草稿
    published: false
    ---
    ```

##### 第二步：主题渲染兜底修复（文章正常仍报错时用）

如果所有文章 Front-matter 都正常，修改主题的渲染逻辑，强制兜底显示：

1. 打开 Next 主题的最新文章模板：`themes/next/layout/_widgets/latest-posts.njk`

2. 修改标题渲染代码，添加兜底逻辑：

    ```twig

    {# 修改后：优先读 title，无 title 则用文件名兜底 #}
    <a class="post-title-link" href="{{ post.permalink }}">
      {{ post.title | default(post.slug) }}
    </a>
    ```

##### 第三步：验证与部署

```bash

hexo clean && hexo g && hexo s
# 本地确认正常后部署
hexo d
```

---

### 2. 文章页「前 / 后一篇」导航显示 `(no title)` 修复

#### 故障现象

文章详情页底部的「前一篇 / 后一篇」导航中，部分文章显示为 `(no title)`，无法正常显示标题。

#### 根因说明

和侧边栏问题根因一致：Hexo 在渲染文章导航时，无法读取到目标文章的有效标题，因此用占位符显示。

#### 修复步骤

##### 第一步：修复文章 Front-matter（95% 问题直接解决）

和侧边栏修复的第一步完全一致，检查并补全所有文章的 Front-matter，清理无效文件。

##### 第二步：主题导航模板兜底修复（文章正常仍报错时用）

以 Next 主题为例，修改导航模板的渲染逻辑：

1. 打开 Next 主题的导航模板：`themes/next/layout/_partials/post/post-nav.njk`

2. 修改前后篇导航的标题渲染代码，添加兜底逻辑：

    ```twig

    {% if post.prev %}
      <a class="extend prev" rel="prev" href="{{ url_for(post.prev.path) }}">
        <i class="fa fa-angle-left"></i>{{ post.prev.title | default(post.prev.slug) }}
      </a>
    {% endif %}

    {% if post.next %}
      <a class="extend next" rel="next" href="{{ url_for(post.next.path) }}">
        {{ post.next.title | default(post.next.slug) }}<i class="fa fa-angle-right"></i>
      </a>
    {% endif %}
    ```

> 说明：`post.slug` 会自动读取文章的文件名作为兜底标题，即使没有 `title` 字段也能正常显示。
>
>

##### 第三步：验证与部署

```bash

hexo clean && hexo g && hexo s
# 本地确认正常后部署
hexo d
```

---

## 第四部分：Hexo 常用命令速查

|完整命令|简写|核心作用|
|---|---|---|
|hexo init|-|初始化 Hexo 博客项目|
|hexo new "文章标题"|hexo n|新建一篇博客文章|
|hexo new page "页面名称"|-|新建一个独立页面（关于我 / 分类页等）|
|hexo clean|-|清除缓存和已生成的静态文件|
|hexo generate|hexo g|生成静态网页文件|
|hexo server|hexo s|本地启动预览服务（默认 4000 端口）|
|hexo deploy|hexo d|部署到远程仓库|
|hexo g -d|-|生成静态文件后直接部署（一步到位）|
---
