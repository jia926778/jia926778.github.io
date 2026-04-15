---
title: VS Code Data Wrangler使用教程
date: 2026-04-15 10:30:00
tags: [VS Code, 工具使用, Data Wrangler, 数据分析]
categories: 工具使用
toc: true
---
# VS Code Data Wrangler 详细使用教程

Data Wrangler 是微软官方推出的 VS Code 数据清洗工具，它将 Pandas 的强大能力与可视化交互界面结合，让你**不用手写代码，点点鼠标就能完成数据清洗，同时自动生成可复用的 Pandas 代码**，完美解决 “80% 时间做数据清洗” 的痛点。

---

## 一、前置条件与安装

### 1. 环境要求

- **Python 3.8+**：Data Wrangler 仅支持 3.8 及以上版本的 Python

- **VS Code**：最新稳定版即可

- **核心依赖**：`pandas`（数据处理）、`jupyter`（内核支持）

### 2. 安装插件

1. 打开 VS Code 扩展商店（左侧图标，或 `Ctrl+Shift+X`）

2. 搜索 **Data Wrangler**，选择微软官方发布的插件（ID: `ms-toolsai.data-wrangler`）

3. 点击「Install」安装，安装完成后重启 VS Code

### 3. 依赖安装

首次启动 Data Wrangler 时，它会自动检查并尝试安装 `pandas` 等依赖。如果自动安装失败，手动在终端执行：

```bash

pip install pandas jupyter
```

---

## 二、启动 Data Wrangler 的两种方式

Data Wrangler 所有操作都在**沙箱环境**中运行，原始数据不会被修改，直到你手动导出更改，非常安全。

### 方式 1：从 Jupyter Notebook 启动（最常用）

如果你已经在 Notebook 中加载了数据，这是最方便的方式，支持后续导出代码到 Notebook。

#### 三种启动入口：

1. **单元格输出按钮**
运行输出 DataFrame 的代码（比如 `df.head()`、`display(df)`、甚至直接 `df`），运行完成后，单元格输出底部会出现蓝色按钮：`Open 'df' in Data Wrangler`，点击即可启动。

    ```python

    import pandas as pd
    # 加载你的数据
    df = pd.read_csv("sales_data.csv")
    # 运行这行，触发输出按钮
    df.head()
    ```

2. **变量面板**
打开 Jupyter 的「Variables」变量面板，找到你的 DataFrame 变量，点击变量右侧的 📊 图标，即可直接打开。

3. **工具栏按钮**
点击 Notebook 顶部工具栏的「View data」，会弹出当前 Notebook 中所有 DataFrame 的列表，选择你要处理的变量即可。

### 方式 2：直接从本地文件启动

无需写任何代码，直接打开本地数据文件，适合快速预览和处理文件。

支持的文件类型：`.csv`/`.tsv`/`.xls`/`.xlsx`/`.parquet`

操作步骤：

1. 在 VS Code 左侧文件资源管理器中，找到你的数据文件

2. 右键点击文件，选择 **Open in Data Wrangler**

3. 如果是 CSV 文件，还可以自定义分隔符；如果是 Excel 文件，可以选择要加载的 Sheet

> 注意：如果打开文件遇到 `UnicodeDecodeError`，说明文件编码不是 UTF-8，建议改用 Notebook 方式加载，手动指定编码：
>
> ```python
>
> df = pd.read_csv("your_file.csv", encoding="gbk") # 根据实际编码调整
> ```
>
>

---

## 三、界面详解：两种工作模式

Data Wrangler 有两种工作模式，默认是查看模式，切换到编辑模式才能进行数据转换和代码生成。

### 1. 查看模式（Viewing Mode）

适合快速探索数据，查看分布、筛选排序，不会修改数据。

|区域|功能说明|
|---|---|
|📊 **数据摘要（Data Summary）**|左侧面板，显示整个数据集的统计：行数、列数、每列的缺失值数量|
|📈 **快速洞察（Quick Insights）**|列标题下方，自动显示每列的分布直方图、缺失值比例、唯一值数量，一眼就能看出数据问题|
|📋 **数据网格（Data Grid）**|中间的表格，可滚动查看所有数据，支持排序、筛选|
|🔍 **筛选排序**|点击列标题的下拉箭头，可快速按条件筛选行、排序|
### 2. 编辑模式（Editing Mode）

点击右上角的「Editing」下拉，切换到编辑模式，解锁所有数据清洗功能，自动生成 Pandas 代码。

|区域|功能说明|
|---|---|
|⚙️ **操作面板（Operations）**|左侧面板，所有内置的清洗操作都在这里，可搜索，按分类整理|
|📝 **清理步骤（Cleaning Steps）**|记录你所有的操作步骤，支持撤销某一步、修改最近的操作，可回溯|
|👀 **数据 Diff 视图**|当你应用操作时，表格中会高亮显示修改过的单元格，一目了然|
|💻 **代码预览（Code Preview）**|右下角，实时显示当前操作对应的 Pandas 代码，你甚至可以直接编辑代码，表格会同步更新效果|
|📤 **导出菜单**|顶部工具栏，处理完成后，导出代码或数据|
---

## 四、常用数据清洗操作（附自动生成代码）

以下是最常用的 10 个数据清洗操作，每个操作都有详细步骤和生成的代码示例，你可以直接跟着操作。

### 1. 快速查看缺失值

打开数据摘要面板，就能看到每列的缺失值数量，点击列标题的直方图，还能看到缺失值的分布。

> 快捷键：按住 `Alt` 点击列标题，快速打开该列的详细分布统计。
>
>

### 2. 筛选行（Filter Rows）

快速过滤出你需要的数据，比如只保留 2024 年的销售数据。

1. 点击列标题的下拉箭头，选择「Filter rows」

2. 设置条件，比如 `date >= 2024-01-01`

3. 点击「Apply」应用

自动生成的代码：

```python

# 筛选日期大于2024-01-01的行
df = df[df['date'] >= pd.Timestamp('2024-01-01')]
```

### 3. 处理缺失值

缺失值是数据清洗最常见的问题，Data Wrangler 提供两种处理方式：

#### 方式 1：删除包含缺失值的行

1. 在左侧操作面板，找到「Drop missing values」

2. 选择要检查的列，或者所有列

3. 应用即可

生成代码：

```python

# 删除price列有缺失值的行
df = df.dropna(subset=['price'])
```

#### 方式 2：填充缺失值

用均值、中位数或者自定义值填充空值，比如用价格的中位数填充空值。

1. 右键点击有缺失值的列，选择「Handle Missing Values」

2. 选择填充方式：Mean (均值)、Median (中位数)、Custom (自定义值)

3. 应用

生成代码：

```python

# 用price列的中位数填充缺失值
df['price'] = df['price'].fillna(df['price'].median())
```

### 4. 删除重复行

快速去重，比如删除用户 ID 重复的行。

1. 操作面板搜索「Drop duplicate rows」

2. 选择去重依据的列（比如用户 ID）

3. 应用

生成代码：

```python

# 按user_id列去重
df = df.drop_duplicates(subset=['user_id'], keep='first')
```

### 5. 转换列类型

经常遇到日期列被识别为字符串、数值列带单位的问题，一键转换类型。

1. 点击列标题旁的类型标识（比如 `abc` 代表字符串）

2. 选择目标类型：Number (数字)、Date (日期)、Boolean (布尔)

3. 系统会自动处理格式，有异常值会提示

生成代码：

```python

# 将date列从字符串转成日期类型
df['date'] = pd.to_datetime(df['date'])
# 将price列转成数值类型
df['price'] = pd.to_numeric(df['price'])
```

### 6. 文本格式处理

针对文本列的批量标准化，比如统一大小写、去除空格。

#### 去除首尾空格

1. 操作面板 → Format → Strip whitespace

2. 选择要处理的列

3. 应用

生成代码：

```python

# 去除city列的首尾空格
df['city'] = df['city'].str.strip()
```

#### 统一大小写

比如把城市名统一为首字母大写：

1. 操作面板 → Format → Capitalize first character

2. 选择列，应用

生成代码：

```python

df['city'] = df['city'].str.capitalize()
```

#### 拆分文本列

比如把 `“北京,朝阳区”` 拆成两个列：

1. 操作面板 → Format → Split text

2. 选择分隔符（比如逗号）

3. 自动生成新列

### 7. 分组聚合

按某列分组，统计聚合值，比如按城市统计平均销售额。

1. 操作面板 → Group by column and aggregate

2. 选择分组列（比如 city）

3. 选择要聚合的列和方式：比如 sales 列，mean (平均值)

4. 应用

生成代码：

```python

# 按city分组，计算sales的平均值
df_grouped = df.groupby('city')['sales'].mean().reset_index()
```

### 8. 添加新列（公式计算）

比如用单价和数量计算总价：

1. 操作面板 → Formulas → Create column from formula

2. 输入公式：`price * quantity`

3. 输入新列名：`total_price`

4. 应用

生成代码：

```python

# 新增总价列
df['total_price'] = df['price'] * df['quantity']
```

### 9. 列管理：重命名、删除、选择

- **重命名列**：右键列标题 → Rename，输入新名称

- **删除列**：右键列标题 → Drop column

- **选择保留列**：操作面板 → Schema → Select columns，勾选要保留的列，删除其他

### 10. 一键标准化：By Example

如果你有复杂的格式转换，比如日期格式从 `2024/01/01` 转成 `2024-01-01`，不用写正则，用「By Example」功能：

1. 操作面板 → DateTime formatting by example

2. 输入一两个转换后的示例，系统会自动识别模式，批量应用到所有行

---

## 五、导出结果：保存你的工作

处理完成后，有三种方式导出你的工作：

### 1. 导出代码到 Notebook

如果你是从 Notebook 启动的，点击顶部的「Export to notebook」，会自动在你的 Notebook 中插入一个新单元格，包含所有清洗步骤的完整代码，下次直接运行就能复现整个流程。

### 2. 导出处理后的数据文件

点击「Export as file」，可以把清洗后的数据保存为新的 CSV、Parquet 等文件，覆盖原文件或者另存为新文件。

### 3. 复制所有代码

点击「Copy all code」，把所有生成的 Pandas 代码复制到剪贴板，你可以粘贴到你的 Python 脚本中复用。

---

## 六、进阶技巧

### 1. 多表对比

同时打开两个 DataFrame，用 VS Code 的分栏功能并排显示，快速对比两个表的差异。

### 2. 自定义配置

打开 VS Code 设置（`Ctrl+,`），搜索 `dataWrangler`，可以自定义：

- 默认缺失值填充策略

- 预览最大行数（默认 1000，大文件可以调大）

- 代码生成风格

### 3. 处理超大数据集

对于超过内存的大数据集，可以配合 Dask 使用：

```python

import dask.dataframe as dd
ddf = dd.read_csv("large_data.csv")
# 采样后打开Data Wrangler，操作会自动适配Dask
ddf.sample(10000).compute()
```

### 4. 快速找列

如果你的表有几十上百列，点击顶部的「Go to column」，搜索列名就能快速定位。

---

## 七、常见问题排查

### 1. 为什么没有「Open in Data Wrangler」按钮？

- 检查 Python 版本是否 ≥3.8

- 检查变量是不是 Pandas DataFrame，Numpy 数组、列表不支持

- 检查当前 Python 环境是否安装了 pandas

- 重启 VS Code 和 Jupyter 内核

### 2. 打开文件报错 UnicodeDecodeError？

文件编码不是 UTF-8，改用 Notebook 方式加载，手动指定编码：

```python

df = pd.read_csv("your_file.csv", encoding="gbk") # 常见的中文编码是gbk
```

### 3. 内核连接失败？

打开命令面板（`Ctrl+Shift+P`），运行 `Data Wrangler: Clear cached runtime`，清除缓存后重新启动。

### 4. 操作错了怎么撤销？

在左侧「Cleaning Steps」面板，点击你要撤销的步骤，或者直接点击步骤旁的删除按钮，支持回滚到任意一步。

---

## 八、完整实战案例：销售数据清洗

我们用一个销售数据的例子，走一遍完整流程：

1. 右键 `sales.csv` → Open in Data Wrangler

2. 切换到编辑模式

3. 处理缺失值：用 median 填充 price 列的空值

4. 去重：按 order_id 删除重复订单

5. 转换类型：把 date 列转成日期类型

6. 筛选：只保留 2024 年的订单

7. 添加新列：计算总价 `price * quantity`

8. 导出：把代码复制到你的 Python 脚本中

整个过程只需要 1 分钟，不用写一行代码，就能得到可复用的清洗代码，效率提升 300%！

---

Data Wrangler 不是要替代 Pandas，而是帮你降低数据清洗的门槛，同时让你通过可视化操作学习 Pandas 代码，非常适合新手和需要快速处理数据的分析师。
