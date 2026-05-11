---
title: Spark大数据处理 学习手册
date: 2026-05-07 18:30:00
tags: [Spark, 大数据处理, Agent]
categories: 数据处理
toc: true
---

# Spark 与大数据处理学习手册


## 文档核心说明

本文档面向**大数据开发入门学习者**，整合了 Spark 核心技术与大数据生态的完整知识体系，涵盖：

- Spark 基础概念与核心数据结构

- PySpark 环境搭建与完整实操代码

- Spark SQL 核心用法与高级特性

- Spark 与 Pandas 的差异对比与适用场景

- 2026 年主流大数据处理框架全景与选型指南

## 一、Spark 基础入门

### 1\.1 Spark 核心简介

Apache Spark 是一个快速、通用的大数据处理引擎，专为大规模数据处理而设计。它提供了高级 API，支持 Scala、Java、Python 和 R 等多种编程语言，并内置了丰富的库，包括 Spark SQL、Spark Streaming、MLlib（机器学习）和 GraphX（图计算）。

#### 核心优势

- **速度快**：比 Hadoop MapReduce 快 100 倍（内存计算）或 10 倍（磁盘计算）

- **易用性**：提供多种语言的高级 API，代码简洁易读

- **通用性**：一站式解决批处理、流处理、机器学习和图计算

- **兼容性**：可运行在 Hadoop、Mesos、Kubernetes 或独立集群上

- **丰富的数据源**：支持 HDFS、HBase、Cassandra、S3 等多种数据源

#### 整体架构

Spark 采用主从架构，主要由以下组件组成：

- **Driver Program**：驱动程序，运行应用程序的 main \(\) 函数，创建 SparkSession

- **Cluster Manager**：集群管理器，负责资源分配（Standalone、YARN、Mesos）

- **Worker Node**：工作节点，负责执行任务

- **Executor**：执行器，在工作节点上运行的进程，负责执行任务并存储数据

- **Task**：任务，被发送到 Executor 上执行的工作单元

### 1\.2 Spark 核心数据结构

Spark 提供了三种核心数据抽象：**RDD**、**DataFrame** 和 **DataSet**。它们一脉相承，各有侧重，适用于不同的场景。

#### 1\.2\.1 RDD（Resilient Distributed Dataset）

**定义**：弹性分布式数据集，是 Spark 最基础的分布式数据抽象，是不可变的、分区的对象集合。

核心特性：

- **不可变性**：RDD 一旦创建就不能修改，只能通过转换操作生成新的 RDD

- **分区性**：数据被划分为多个分区，分布在集群的不同节点上

- **弹性容错**：通过血缘关系（Lineage）记录依赖，某分区数据丢失可通过父 RDD 重算恢复

- **惰性计算**：转换操作不会立即执行，只有遇到行动操作时才会真正计算

- **持久化**：可以将 RDD 缓存在内存或磁盘中，供后续操作重复使用

适用场景：

- 非结构化数据处理（如文本文件、日志文件）

- 需要精细控制数据处理流程的场景

- 复杂的迭代计算（如机器学习算法）

#### 1\.2\.2 DataFrame

**定义**：分布式的二维表结构数据集合，包含行和列，且有明确的 schema（字段名和数据类型）。

核心特性：

- **有 Schema**：类似数据库表结构，预先定义列名和类型

- **优化执行**：依赖 Catalyst 优化器，自动优化执行计划（如谓词下推、列裁剪）

- **高性能**：使用 Tungsten 二进制格式存储数据，减少内存占用和 GC 开销

- **支持 SQL**：可以直接使用 SQL 查询数据

- **支持嵌套数据类型**：struct、array、map 等复杂类型

适用场景：

- 结构化和半结构化数据处理

- ETL（抽取、转换、加载）操作

- 交互式数据分析

- 需要 SQL 查询的场景

#### 1\.2\.3 DataSet

**定义**：强类型的分布式数据集合，是 DataFrame 的扩展，结合了 RDD 的类型安全和 DataFrame 的性能优势。

核心特性：

- **强类型**：编译时类型检查，提前发现类型错误

- **高性能**：使用 Encoder 在 JVM 对象和 Tungsten 二进制格式之间高效转换

- **面向对象**：可以直接操作自定义类的对象

- **与 DataFrame 兼容**：DataFrame 是 DataSet \[Row\] 的别名

注意事项：

- **Python 不支持强类型 DataSet**：因为 Python 是动态类型语言，所以在 PySpark 中，DataSet 和 DataFrame 是同一个概念

- **Scala 和 Java 支持**：只有在 Scala 和 Java 中才能使用强类型 DataSet

#### 1\.2\.4 三种数据结构对比

|维度|RDD|DataFrame|DataSet \(强类型\)|
|---|---|---|---|
|内部数据结构|Java/Kryo 序列化对象|Tungsten 二进制格式的 Row 集合|Encoder 转化的 Tungsten 二进制格式|
|Schema|无，需人工维护|有（列名和类型）|有|
|类型安全|编译时类型安全（泛型）|运行时检查（弱类型）|编译时类型安全（强类型）|
|优化引擎|无（函数黑盒）|Catalyst 优化器 \+ Tungsten|Catalyst 优化器 \+ Tungsten|
|性能|低（对象开销大，GC）|高（二进制处理，堆外内存）|最高（Encoder 优化）|
|适用语言|Scala, Java, Python, R|Scala, Java, Python, R|Scala, Java|
|适用场景|非结构化数据，精细控制|结构化数据，ETL，SQL|类型安全，高性能需求|

## 二、环境搭建与快速上手

### 2\.1 环境安装

#### 前置条件

- **Java 11**（推荐，兼容绝大多数 Spark 版本）

- **Python 3\.8\+**（用于 PySpark）

#### 安装步骤

1. 安装 PySpark：

```bash
pip install pyspark
```

2. 配置环境变量（可选）：

```bash
# Linux/macOS
export SPARK_HOME=/opt/spark
export PATH=$PATH:$SPARK_HOME/bin
export PYSPARK_PYTHON=python3
```

### 2\.2 第一个 PySpark 程序

下面是一个完整的学生成绩分析示例，可直接运行：

```python
# 导入必要的模块
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, avg, sum, count

def main():
    # 1. 创建 SparkSession（Spark 2.0+ 的统一入口点）
    spark = SparkSession.builder \
        .appName("StudentGradeAnalysis") \
        .master("local[*]")  # 使用本地所有 CPU 核心运行
        .config("spark.driver.memory", "2g") \
        .config("spark.sql.shuffle.partitions", "10") \
        .getOrCreate()

    # 设置日志级别为 WARN，减少不必要的输出
    spark.sparkContext.setLogLevel("WARN")

    print("=" * 50)
    print("Spark 程序启动成功")
    print("=" * 50)

    # 2. 创建示例数据
    student_data = [
        (1, "张三", "一班", "数学", 85),
        (2, "李四", "一班", "数学", 92),
        (3, "王五", "一班", "数学", 78),
        (4, "赵六", "二班", "数学", 90),
        (5, "钱七", "二班", "数学", 88),
        (1, "张三", "一班", "语文", 88),
        (2, "李四", "一班", "语文", 76),
        (3, "王五", "一班", "语文", 95),
        (4, "赵六", "二班", "语文", 82),
        (5, "钱七", "二班", "语文", 91)
    ]

    # 定义列名
    columns = ["student_id", "name", "class", "subject", "score"]

    # 3. 创建 DataFrame
    df = spark.createDataFrame(student_data, schema=columns)

    print("\n原始数据：")
    df.show()

    # 4. 基本数据操作
    print("\n分数统计信息：")
    df.describe("score").show()

    # 按班级分组统计各科平均分
    print("\n各班各科平均分：")
    class_subject_avg = df.groupBy("class", "subject") \
                          .agg(avg("score").alias("average_score")) \
                          .orderBy("class", "subject")
    class_subject_avg.show()

    # 计算每个学生的总分和平均分
    print("\n每个学生的总分和平均分：")
    student_total = df.groupBy("student_id", "name", "class") \
                      .agg(
                          sum("score").alias("total_score"),
                          avg("score").alias("average_score"),
                          count("subject").alias("subject_count")
                      ) \
                      .orderBy(col("total_score").desc())
    student_total.show()

    # 5. 停止 SparkSession
    spark.stop()
    print("\n" + "=" * 50)
    print("Spark 程序执行完成")
    print("=" * 50)

if __name__ == "__main__":
    main()
```

## 三、PySpark 核心操作

### 3\.1 SparkSession 详解

**SparkSession** 是 Spark 2\.0 引入的统一入口，它封装了 SparkContext、SQLContext 和 HiveContext 的所有功能。

#### 创建方式

```python
from pyspark.sql import SparkSession

# 基本创建方式
spark = SparkSession.builder \
    .appName("MyApp") \
    .master("local[*]") \
    .getOrCreate()

# 完整配置示例
spark = SparkSession.builder \
    .appName("MyApp") \
    .master("local[*]") \
    .config("spark.driver.memory", "4g") \
    .config("spark.executor.memory", "8g") \
    .config("spark.sql.shuffle.partitions", "20") \
    .config("spark.sql.adaptive.enabled", "true") \
    .enableHiveSupport()  # 启用 Hive 支持
    .getOrCreate()
```

### 3\.2 数据读写

Spark SQL 支持多种数据源的读写操作，这是它最强大的功能之一。

#### 通用读写 API

```python
# 通用读取方式
df = spark.read.format("csv") \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .load("data.csv")

# 通用写入方式
df.write.format("parquet") \
    .mode("overwrite") \
    .save("output.parquet")
```

**写入模式（mode）**：

- `append`：追加到现有数据

- `overwrite`：覆盖现有数据

- `ignore`：如果数据已存在则不执行任何操作

- `error`（默认）：如果数据已存在则抛出异常

#### 常用数据源详解

##### CSV 文件

```python
# 读取 CSV
df = spark.read.csv(
    "data.csv",
    header=True,        # 第一行作为列名
    inferSchema=True,   # 自动推断数据类型
    sep=",",            # 分隔符
    encoding="UTF-8",   # 编码
    nullValue="NA"      # 空值表示
)

# 写入 CSV
df.write.csv(
    "output.csv",
    header=True,
    mode="overwrite",
    encoding="UTF-8"
)
```

##### JSON 文件

```python
# 读取 JSON
df = spark.read.json("data.json")

# 写入 JSON
df.write.json("output.json", mode="overwrite")
```

##### Parquet 文件（推荐）

Parquet 是 Spark 推荐的列式存储格式，具有以下优势：

- 高压缩比（比 CSV 小 5\-10 倍）

- 高效的查询性能（只读取需要的列）

- 支持复杂数据类型

- 与 Hadoop 生态系统完美兼容

```python
# 读取 Parquet
df = spark.read.parquet("data.parquet")

# 写入 Parquet
df.write.parquet("output.parquet", mode="overwrite")
```

##### JDBC 数据库

```python
# 读取 MySQL
df = spark.read \
    .format("jdbc") \
    .option("url", "jdbc:mysql://localhost:3306/mydb") \
    .option("dbtable", "employees") \
    .option("user", "root") \
    .option("password", "password") \
    .option("driver", "com.mysql.cj.jdbc.Driver") \
    .load()

# 写入 MySQL
df.write \
    .format("jdbc") \
    .option("url", "jdbc:mysql://localhost:3306/mydb") \
    .option("dbtable", "employees_result") \
    .option("user", "root") \
    .option("password", "password") \
    .option("driver", "com.mysql.cj.jdbc.Driver") \
    .mode("append") \
    .save()
```

### 3\.3 DataFrame 基本操作

#### 查看数据

```python
# 显示前 20 行数据
df.show()

# 显示前 10 行数据
df.show(10)

# 显示完整内容（不截断）
df.show(truncate=False)

# 打印 Schema
df.printSchema()

# 查看列名
print(df.columns)

# 查看数据行数
print(df.count())

# 查看基本统计信息
df.describe("age", "salary").show()
```

#### 选择与过滤

```python
from pyspark.sql.functions import col

# 选择列
df.select("name", "age").show()
df.select(df.name, df.age).show()
df.select(col("name"), col("age")).show()

# 选择并重命名列
df.select(col("name").alias("employee_name"), col("age")).show()

# 过滤数据
df.filter(col("age") > 25).show()
df.filter((col("age") > 25) & (col("gender") == "F")).show()
df.filter(col("name").startswith("A")).show()
```

#### 添加与修改列

```python
# 添加新列
df.withColumn("age_plus_10", col("age") + 10).show()

# 修改列
df.withColumn("salary", col("salary") * 1.1).show()

# 重命名列
df.withColumnRenamed("salary", "monthly_salary").show()

# 删除列
df.drop("gender").show()
```

#### 分组与聚合

```python
from pyspark.sql.functions import avg, max, min, sum, count

# 简单分组聚合
df.groupBy("gender").count().show()

# 多列分组
df.groupBy("gender", "department").count().show()

# 多种聚合函数
df.groupBy("gender").agg(
    avg("salary").alias("avg_salary"),
    max("salary").alias("max_salary"),
    min("salary").alias("min_salary"),
    sum("salary").alias("total_salary")
).show()
```

#### 排序

```python
# 升序排序
df.orderBy("age").show()

# 降序排序
df.orderBy(col("age").desc()).show()

# 多列排序
df.orderBy(col("department").asc(), col("salary").desc()).show()
```

## 四、Spark SQL 详解

### 4\.1 Spark SQL 核心概念

Spark SQL 是 Apache Spark 用于处理**结构化和半结构化数据**的核心模块，它将关系型数据库的 SQL 查询能力与 Spark 的分布式计算引擎完美结合。

核心优势：

- **性能卓越**：比传统 MapReduce 快 100 倍以上

- **多语言支持**：Scala、Java、Python、R

- **无缝集成**：与 Spark Streaming、MLlib、GraphX 深度集成

- **企业级特性**：支持事务、ACID、数据湖（Delta Lake/Iceberg/Hudi）

### 4\.2 临时视图与全局视图

```python
# 创建临时视图（仅当前会话可见）
df.createOrReplaceTempView("employees")

# 创建全局临时视图（跨会话可见）
df.createOrReplaceGlobalTempView("global_employees")

# 查询全局临时视图
spark.sql("SELECT * FROM global_temp.global_employees").show()

# 删除视图
spark.catalog.dropTempView("employees")
```

### 4\.3 常用 SQL 查询示例

```python
# 基本查询
spark.sql("SELECT name, age, salary FROM employees WHERE age > 25").show()

# 分组聚合
spark.sql("""
    SELECT gender, AVG(salary) as avg_salary
    FROM employees
    GROUP BY gender
    HAVING avg_salary > 10000
""").show()

# 排序
spark.sql("SELECT * FROM employees ORDER BY salary DESC LIMIT 5").show()

# 连接查询
spark.sql("""
    SELECT e.name, e.salary, d.department_name
    FROM employees e
    JOIN departments d ON e.department_id = d.id
""").show()

# 子查询
spark.sql("""
    SELECT name, salary
    FROM employees
    WHERE salary > (SELECT AVG(salary) FROM employees)
""").show()
```

### 4\.4 高级特性

#### 复杂数据类型处理

Spark SQL 支持多种复杂数据类型，包括数组、映射和结构体。

```python
from pyspark.sql.functions import explode, col, struct

# 数组类型
df = spark.createDataFrame([
    ("Alice", ["Math", "English", "Science"]),
    ("Bob", ["History", "Geography"])
], ["name", "subjects"])

# 展开数组
df.select("name", explode("subjects").alias("subject")).show()

# 结构体类型
df = spark.createDataFrame([
    ("Alice", struct("New York", "NY").alias("address")),
    ("Bob", struct("Los Angeles", "CA").alias("address"))
], ["name", "address"])

# 访问结构体字段
df.select("name", "address.city", "address.state").show()
```

#### 窗口函数

窗口函数是 Spark SQL 最强大的功能之一，它允许你在不分组的情况下对数据进行聚合和排名。

```python
from pyspark.sql.window import Window
from pyspark.sql.functions import rank, dense_rank, row_number, sum

# 定义窗口规范
windowSpec = Window.partitionBy("department").orderBy(col("salary").desc())

# 排名函数
df.withColumn("rank", rank().over(windowSpec)) \
  .withColumn("dense_rank", dense_rank().over(windowSpec)) \
  .withColumn("row_number", row_number().over(windowSpec)) \
  .show()

# 累计求和
windowSpec2 = Window.partitionBy("department").orderBy("salary").rowsBetween(Window.unboundedPreceding, 0)
df.withColumn("cumulative_salary", sum("salary").over(windowSpec2)).show()
```

#### 用户自定义函数（UDF）

当内置函数无法满足需求时，你可以创建自定义函数。

```python
from pyspark.sql.functions import udf
from pyspark.sql.types import StringType

# 定义 Python 函数
def get_salary_level(salary):
    if salary < 8000:
        return "低"
    elif salary < 15000:
        return "中"
    else:
        return "高"

# 注册 UDF
salary_level_udf = udf(get_salary_level, StringType())

# 使用 UDF
df.withColumn("salary_level", salary_level_udf(col("salary"))).show()

# 在 SQL 中使用 UDF
spark.udf.register("salary_level", get_salary_level, StringType())
spark.sql("SELECT name, salary_level(salary) as level FROM employees").show()
```

### 4\.5 性能优化技巧

#### 数据存储优化

- **使用列式存储格式**：优先使用 Parquet 或 ORC 格式

- **合理分区**：按常用过滤条件分区（如日期、地区）

- **分桶**：对经常连接的列进行分桶

- **压缩**：使用 Snappy 或 Gzip 压缩

#### 查询优化

- **谓词下推**：尽量将过滤条件放在查询的最前面

- **列裁剪**：只选择需要的列，避免使用 `SELECT \*`

- **避免 Shuffle**：尽量减少不必要的分组和连接操作

- **广播小表**：在连接操作时，将小表广播到所有节点

```python
# 广播连接示例
from pyspark.sql.functions import broadcast

large_df.join(broadcast(small_df), "id").show()
```

#### 资源配置优化

- **合理设置分区数**：`spark\.sql\.shuffle\.partitions`（默认 200）

- **调整内存配置**：`spark\.driver\.memory` 和 `spark\.executor\.memory`

- **启用自适应执行**：`spark\.sql\.adaptive\.enabled=true`（Spark 3\.0\+）

## 五、Spark vs Pandas 对比

### 5\.1 核心定位与架构对比

|维度|Pandas|Apache Spark \(PySpark\)|
|---|---|---|
|**设计目标**|单机高性能数据分析库|分布式大数据处理引擎|
|**运行架构**|单节点单进程（可利用多线程）|主从分布式架构（Driver \+ Executor）|
|**数据规模**|适合 **GB 级以下** 数据（受单机内存限制）|适合 **TB\-PB 级** 数据（可横向扩展）|
|**核心优势**|简单易用、API 丰富、开发效率高|可扩展性强、容错性好、支持大规模并行计算|
|**核心劣势**|无法处理超过单机内存的数据|有集群调度开销、小数据处理效率低|

### 5\.2 执行模型对比

**Pandas：立即执行模型**

- 每一行代码都会立即执行并返回结果

- 执行顺序严格按照代码编写顺序

- 没有优化器，完全依赖开发者手动优化

- 优点：直观易懂，调试方便

- 缺点：无法进行全局优化，性能依赖开发者经验

**Spark：惰性执行模型**

- 转换操作（Transformation）不会立即执行，只会记录计算逻辑

- 只有遇到行动操作（Action）时，才会生成执行计划并真正计算

- 内置 Catalyst 优化器，会自动优化执行计划（谓词下推、列裁剪、join 重排等）

- 优点：自动优化，性能更好，支持复杂计算

- 缺点：调试困难，错误信息不直观

### 5\.3 性能对比

|数据规模|Pandas|Spark|说明|
|---|---|---|---|
|\&lt; 100MB|🚀 极快|🐢 慢|Spark 有集群启动和调度开销|
|100MB\-1GB|🚀 快|🐇 较快|Pandas 仍有优势，但差距缩小|
|1GB\-10GB|🐌 慢或 OOM|🚀 快|Pandas 开始出现内存压力|
|\&gt; 10GB|❌ 无法处理|🚀 快|Spark 可以横向扩展|

### 5\.4 Pandas API on Spark

Spark 3\.2\+ 引入了 **Pandas API on Spark**（以前称为 Koalas），这是一个重大突破。它允许用户使用几乎与 Pandas 完全相同的 API 来操作 Spark DataFrame，同时获得 Spark 的分布式计算能力。

```python
# 导入 Pandas API on Spark
import pyspark.pandas as ps

# 创建 DataFrame（与 Pandas 完全相同）
df = ps.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie'],
    'age': [25, 30, 35],
    'score': [85, 92, 78]
})

# 操作与 Pandas 完全相同
print(df[df['age'] > 28])
print(df.groupby('age')['score'].mean())
print(df.sort_values('score', ascending=False))
```

### 5\.5 适用场景与选择建议

#### 什么时候用 Pandas？

- 数据量小于单机内存（通常 \&lt; 10GB）

- 快速数据探索和原型开发

- 复杂的统计分析和时间序列处理

- 需要与 Scikit\-learn 等机器学习库紧密集成

- 交互式数据分析（Jupyter Notebook）

#### 什么时候用 Spark？

- 数据量超过单机内存（\&gt; 10GB）

- 需要处理 TB\-PB 级别的大数据

- 批处理和流处理结合的场景

- 需要在集群上并行计算

- 企业级数据处理和 ETL 管道

- 大规模机器学习训练

## 六、大数据生态全景（2026 版）

### 6\.1 技术栈整体架构

现代大数据技术栈已经形成了一个**分层、模块化、云原生**的完整生态系统：

```Plain Text
┌─────────────────────────────────────────────────────────────┐
│ 应用层：数据可视化、BI报表、AI应用、业务系统                │
├─────────────────────────────────────────────────────────────┤
│ 计算引擎层：批处理、流处理、交互式查询、机器学习、图计算    │
├─────────────────────────────────────────────────────────────┤
│ 数据湖/仓库层：表格式、数据仓库、数据集市                  │
├─────────────────────────────────────────────────────────────┤
│ 数据集成层：数据采集、同步、转换、治理                      │
├─────────────────────────────────────────────────────────────┤
│ 存储层：分布式文件系统、对象存储、消息队列、数据库          │
├─────────────────────────────────────────────────────────────┤
│ 基础设施层：云平台、Kubernetes、物理服务器                  │
└─────────────────────────────────────────────────────────────┘
```

### 6\.2 核心计算引擎对比

|框架|处理模型|延迟|核心优势|适用场景|
|---|---|---|---|---|
|Spark|批流一体（微批）|秒级|生态完善、API 丰富、通用性强|通用大数据处理、ETL、机器学习|
|Flink|批流一体（原生流）|毫秒级|流处理能力强、事件时间准确|实时流处理、实时分析、风控|
|Trino|批处理|秒级|交互式性能好、多数据源联邦|交互式数据分析、数据湖查询|
|ClickHouse|批流一体|亚秒级|单表查询性能极致|实时数据分析、日志分析|
|Hive|批处理|分钟级|生态完善、成熟稳定|离线数据仓库、大规模 ETL|

### 6\.3 数据湖表格式对比

|特性|Apache Iceberg \(1\.6\.0\)|Delta Lake \(3\.2\.0\)|Apache Hudi \(0\.15\.0\)|
|---|---|---|---|
|**发起公司**|Netflix|Databricks|Uber|
|**多引擎支持**|极佳（Spark/Flink/Trino/Hive/ClickHouse）|良好（Spark 为主，Flink/Trino 支持）|良好（Flink 深度集成，Spark 支持）|
|**Schema 演化**|完整支持（添加 / 重命名 / 删除列）|完整支持|有限支持（主要支持添加列）|
|**实时写入**|中等（分钟级）|中等（秒到分钟级）|极佳（毫秒到秒级）|
|**增量查询**|良好|良好|极佳|
|**ACID 事务**|完整支持|完整支持|完整支持|
|**时间旅行**|支持|支持|支持|
|**适用场景**|跨引擎数据湖架构，长期演进|Spark 生态为主，批处理优先|实时数据入湖，CDC 场景|

### 6\.4 2026 年技术趋势

1. **流批一体全面普及**：实时计算与批量计算的技术鸿沟彻底消除

2. **湖仓一体成为事实标准**：数据湖与数据仓库的界限越来越模糊

3. **AI 与大数据深度融合**：大数据平台内置 AI 能力，大模型用于数据治理

4. **云原生与 Serverless 化**：所有主流框架都支持 Kubernetes 部署，按需付费

5. **实时化与低延迟**：亚秒级甚至毫秒级的数据分析成为常态

6. **国产化替代加速**：国产大数据框架快速发展，政府和金融行业优先采用

### 6\.5 企业级技术栈选型指南

|业务场景|推荐技术栈|
|---|---|
|**离线 ETL 与批处理**|Spark \+ Iceberg \+ Parquet|
|**实时流处理**|Flink \+ Kafka \+ Hudi|
|**交互式数据分析**|Trino \+ Iceberg 或 ClickHouse|
|**实时数据仓库**|Apache Doris / StarRocks|
|**日志分析**|ClickHouse \+ Elasticsearch|
|**机器学习**|Spark \+ Ray \+ MLflow|
|**数据湖架构**|Iceberg \+ Spark \+ Flink \+ Trino|

## 核心知识点速览

- **Spark 核心数据结构**：RDD（基础）、DataFrame（主流）、DataSet（强类型），优先使用 DataFrame

- **SparkSession**：所有 Spark 程序的统一入口，替代了旧的 Context

- **数据存储**：优先使用 Parquet 列式存储格式，性能和压缩比最优

- **执行模型**：Spark 采用惰性执行，只有 Action 才会触发计算

- **Spark SQL**：支持标准 SQL，支持临时视图，可用于多数据源查询

- **Pandas vs Spark**：小数据用 Pandas，大数据用 Spark，Pandas API on Spark 可无缝衔接

- **流处理**：Flink 是实时流处理的事实标准，毫秒级延迟

- **数据湖**：Iceberg/Delta/Hudi 三大表格式，实现湖仓一体

- **OLAP 分析**：ClickHouse/Doris/StarRocks 提供极致的查询性能

- **选型原则**：根据数据规模和业务场景选择合适的技术栈，通常组合使用多个框架
