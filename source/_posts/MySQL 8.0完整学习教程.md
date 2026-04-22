---
title: MySQL 学习教程
date: 2026-03-21 18:30:00
tags: [MySQL, 技术分享, 关系型数据库, DB]
categories: 数据库
toc: true
---

# MySQL 8\.0完整学习教程

本教程基于**MySQL 8\.0 LTS**长期支持版本编写，覆盖从环境部署、基础语法、核心特性到性能优化、企业级运维的全链路知识，所有示例均可直接复制执行，兼顾新手入门与开发者进阶需求。

## 一、MySQL 基础认知

### 1\.1 什么是 MySQL

MySQL 是 Oracle 旗下开源的**关系型数据库管理系统（RDBMS）**，采用客户端 \- 服务器架构，使用结构化查询语言（SQL）进行数据管理，是 Web 开发、企业级系统中应用最广泛的数据库之一。

- 核心优势：开源免费、性能稳定、轻量易用、支持高并发、完善的事务机制、丰富的生态工具

- 核心存储引擎：默认使用**InnoDB**（支持事务、行级锁、外键、崩溃恢复），替代引擎 MyISAM（不支持事务，已逐步淘汰）

- 版本选择：生产环境优先选择 8\.0 稳定版（8\.0\.30\+），相比 5\.7 版本，优化了事务性能、索引算法、安全机制，新增窗口函数、CTE 等高级特性

### 1\.2 SQL 语言核心分类

|分类|全称|核心作用|核心语句|
|---|---|---|---|
|DDL|数据定义语言|库、表、索引等结构的创建 / 修改 / 删除|CREATE、ALTER、DROP|
|DML|数据操纵语言|表中数据的增删改|INSERT、UPDATE、DELETE|
|DQL|数据查询语言|数据查询检索（核心高频）|SELECT|
|DCL|数据控制语言|用户权限管理|GRANT、REVOKE、CREATE USER|
|TCL|事务控制语言|事务提交与回滚|COMMIT、ROLLBACK、SAVEPOINT|

## 二、环境部署与客户端连接

### 2\.1 全平台安装部署

#### 2\.1\.1 Windows 安装（图形化安装）

1. 进入 MySQL 官网，下载**MySQL Installer for Windows**社区版，选择完整安装包（with Community）

2. 运行安装器，选择「Full」完整安装，自动安装 Server、Workbench 可视化工具等组件

3. 依赖检测：自动安装缺失的 Visual Studio Runtime 组件

4. 服务配置：

    - 端口默认 3306，被占用可修改为 3307

    - 认证方式推荐强密码加密（SHA256，8\.0 默认）

    - 设置 root 用户强密码（必须包含大小写、数字、特殊符号），可额外创建普通用户

    - 启动类型选择系统服务，开机自动启动

5. 安装完成后，通过 MySQL Workbench 或命令行验证连接

#### 2\.1\.2 Linux（CentOS 7\+/Ubuntu）安装

以 CentOS 为例，核心步骤如下：

```bash
# 1. 配置MySQL 8.0 yum源
rpm -ivh https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm

# 2. 安装MySQL Server
yum install -y mysql-community-server

# 3. 启动服务并设置开机自启
systemctl start mysqld
systemctl enable mysqld

# 4. 获取初始临时密码
grep 'temporary password' /var/log/mysqld.log

# 5. 登录并修改root密码
mysql -u root -p
# 输入临时密码后执行
ALTER USER 'root'@'localhost' IDENTIFIED BY '你的强密码@2026';
FLUSH PRIVILEGES;

# 6. 安全初始化（可选，生产环境必做）
mysql_secure_installation
# 按提示完成：删除匿名用户、禁止root远程登录、删除test库、刷新权限
```

#### 2\.1\.3 Docker 快速部署（推荐本地测试）

```bash
# 1. 拉取MySQL 8.0镜像
docker pull mysql:8.0

# 2. 启动容器，端口映射、密码设置、数据持久化
docker run -d \
  --name mysql8 \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=你的强密码@2026 \
  -v /mysql/data:/var/lib/mysql \
  --restart=always \
  mysql:8.0

# 3. 进入容器登录MySQL
docker exec -it mysql8 mysql -u root -p
```

### 2\.2 服务启停与客户端连接

#### 2\.2\.1 服务启停命令

|系统|启动命令|停止命令|重启命令|查看状态|
|---|---|---|---|---|
|Windows|net start mysql|net stop mysql|服务面板手动操作|sc query mysql|
|Linux|systemctl start mysqld|systemctl stop mysqld|systemctl restart mysqld|systemctl status mysqld|
|Docker|docker start mysql8|docker stop mysql8|docker restart mysql8|docker ps|

#### 2\.2\.2 命令行连接

```bash
# 基础连接命令
mysql -u 用户名 -p -h 主机地址 -P 端口号

# 示例1：本地连接
mysql -u root -p
# 回车后输入密码即可登录

# 示例2：远程连接
mysql -u root -p -h 192.168.1.100 -P 3306
```

#### 2\.2\.3 可视化客户端工具

- 官方工具：MySQL Workbench（免费，功能全面）

- 主流第三方工具：Navicat、DBeaver、DataGrip、SQLyog

- 连接核心参数：主机 IP、端口（默认 3306）、用户名、密码

## 三、DDL 库表操作与数据类型

### 3\.1 数据库（库）核心操作

所有库操作命令执行后，可通过`SHOW DATABASES;`验证结果。

```sql
-- 1. 创建数据库（指定字符集和排序规则，8.0默认utf8mb4，支持emoji）
CREATE DATABASE IF NOT EXISTS test_db
DEFAULT CHARACTER SET utf8mb4
DEFAULT COLLATE utf8mb4_unicode_ci;

-- 2. 查看所有数据库
SHOW DATABASES;

-- 3. 查看数据库创建语句
SHOW CREATE DATABASE test_db;

-- 4. 使用/切换数据库（后续表操作默认基于该库）
USE test_db;

-- 5. 修改数据库字符集
ALTER DATABASE test_db DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

-- 6. 删除数据库（高危操作！生产环境禁用）
DROP DATABASE IF EXISTS test_db;
```

### 3\.2 MySQL 核心数据类型

数据类型选择直接影响数据库性能和存储空间，核心原则：**够用就小，精准匹配**。

#### 3\.2\.1 数值类型

|类型|存储空间|取值范围|适用场景|
|---|---|---|---|
|TINYINT|1 字节|\-128\\127 / 0\\255|状态值、性别、年龄等极小数字|
|INT|4 字节|\-21 亿～21 亿 / 0\~42 亿|主键 ID、数量、普通整数|
|BIGINT|8 字节|\-9e18\~9e18|超大主键、订单号、高并发自增 ID|
|FLOAT/DOUBLE|4/8 字节|浮点型|非精准小数，如温度、重量（不推荐金额）|
|DECIMAL\(M,D\)|自定义|精准小数|金额、价格等高精度场景，如 DECIMAL \(10,2\)|

#### 3\.2\.2 字符串类型

|类型|存储空间|核心特点|适用场景|
|---|---|---|---|
|CHAR\(N\)|固定长度 N，最大 255 字符|定长，性能高，浪费空间|固定长度内容，如手机号、身份证号、UUID|
|VARCHAR\(N\)|可变长度，最大 65535 字节|变长，节省空间，需额外存储长度|用户名、地址、标题等变长内容|
|TEXT|大文本，最大 64KB|无需指定长度，不能有默认值|文章详情、备注、长文本描述|
|LONGTEXT|超大文本，最大 4GB|存储超长文本|富文本内容、大段日志|

#### 3\.2\.3 日期时间类型

|类型|格式|核心特点|适用场景|
|---|---|---|---|
|DATE|YYYY\-MM\-DD|仅存储日期|生日、入职日期、业务日期|
|TIME|HH:MM:SS|仅存储时间|时段、打卡时间|
|DATETIME|YYYY\-MM\-DD HH:MM:SS|范围 1000\-9999 年，与时区无关|通用创建时间、更新时间（最常用）|
|TIMESTAMP|YYYY\-MM\-DD HH:MM:SS|范围 1970\-2038 年，与时区相关，自动更新|数据修改时间、行记录变更时间|

### 3\.3 数据表核心操作

#### 3\.3\.1 创建表（核心语法）

```sql
-- 语法模板
CREATE TABLE IF NOT EXISTS 表名(
  字段名1 数据类型 [约束] [注释],
  字段名2 数据类型 [约束] [注释],
  ...
  [表级约束]
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='表注释';

-- 完整示例：用户表
CREATE TABLE IF NOT EXISTS sys_user (
  id BIGINT AUTO_INCREMENT PRIMARY KEY COMMENT '主键ID',
  username VARCHAR(50) NOT NULL UNIQUE COMMENT '用户名',
  password VARCHAR(100) NOT NULL COMMENT '密码',
  phone CHAR(11) COMMENT '手机号',
  age TINYINT UNSIGNED COMMENT '年龄',
  gender ENUM('男','女','未知') DEFAULT '未知' COMMENT '性别',
  balance DECIMAL(10,2) DEFAULT 0.00 COMMENT '账户余额',
  create_time DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  update_time DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  is_deleted TINYINT DEFAULT 0 COMMENT '是否删除 0-否 1-是'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='系统用户表';
```

#### 3\.3\.2 表结构查看与修改

```sql
-- 1. 查看库中所有表
SHOW TABLES;

-- 2. 查看表结构
DESC sys_user;

-- 3. 查看表创建语句
SHOW CREATE TABLE sys_user;

-- 4. 修改表名
ALTER TABLE sys_user RENAME TO system_user;

-- 5. 添加字段
ALTER TABLE system_user ADD COLUMN email VARCHAR(100) COMMENT '邮箱' AFTER phone;

-- 6. 修改字段类型/约束/注释
ALTER TABLE system_user MODIFY COLUMN email VARCHAR(150) NOT NULL COMMENT '用户邮箱';

-- 7. 修改字段名
ALTER TABLE system_user CHANGE COLUMN email user_email VARCHAR(150) COMMENT '用户邮箱';

-- 8. 删除字段
ALTER TABLE system_user DROP COLUMN user_email;

-- 9. 删除表（高危操作！生产环境禁用）
DROP TABLE IF EXISTS system_user;

-- 10. 清空表数据（自增ID重置，比DELETE全表删除快）
TRUNCATE TABLE sys_user;
```

### 3\.4 表约束（数据完整性保障）

约束是对表中数据的强制校验规则，确保数据的准确性和一致性，InnoDB 支持以下 6 种约束：

1. **主键约束（PRIMARY KEY）**：唯一标识一行记录，非空且唯一，一个表只能有一个主键，推荐使用 BIGINT 自增主键

2. **非空约束（NOT NULL）**：字段值不允许为 NULL，必须填写

3. **唯一约束（UNIQUE）**：字段值在表中唯一，允许为 NULL（可多个）

4. **默认约束（DEFAULT）**：字段未赋值时，使用默认值

5. **检查约束（CHECK）**：8\.0 原生支持，校验字段值符合业务规则，如`CHECK\(age\&gt;0 AND age\&lt;150\)`

6. **外键约束（FOREIGN KEY）**：关联两张表的数据，保证关联数据的一致性，生产环境慎用（影响性能）

```sql
-- 约束示例：订单表（关联用户表）
CREATE TABLE IF NOT EXISTS sys_order (
  order_id BIGINT AUTO_INCREMENT PRIMARY KEY COMMENT '订单ID',
  order_no VARCHAR(32) NOT NULL UNIQUE COMMENT '订单编号',
  user_id BIGINT NOT NULL COMMENT '下单用户ID',
  order_amount DECIMAL(10,2) NOT NULL CHECK(order_amount >= 0) COMMENT '订单金额',
  order_status TINYINT DEFAULT 0 COMMENT '订单状态 0-待付款 1-已付款',
  create_time DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  -- 外键约束：关联用户表主键
  FOREIGN KEY (user_id) REFERENCES sys_user(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='订单表';
```

## 四、DML 数据操作与基础查询 DQL

### 4\.1 数据插入 INSERT

```sql
-- 1. 插入单行数据（指定字段，推荐写法）
INSERT INTO sys_user (username, password, phone, age, gender)
VALUES ('zhangsan', '123456', '13800138000', 25, '男');

-- 2. 插入单行全字段数据（不推荐，表结构变更会失效）
INSERT INTO sys_user
VALUES (NULL, 'lisi', '654321', '13900139000', 30, '女', 100.00, DEFAULT, DEFAULT, 0);

-- 3. 批量插入（性能远高于单行循环插入，推荐）
INSERT INTO sys_user (username, password, phone, age, gender)
VALUES
('wangwu', '111111', '13700137000', 28, '男'),
('zhaoliu', '222222', '13600136000', 22, '女'),
('sunqi', '333333', '13500135000', 35, '男');

-- 4. 插入数据，存在则更新，不存在则插入（基于唯一键）
INSERT INTO sys_user (username, password, phone)
VALUES ('zhangsan', '666666', '13800138000')
ON DUPLICATE KEY UPDATE password = '666666', phone = '13800138000';
```

### 4\.2 数据更新 UPDATE

⚠️ 高危操作：必须加 WHERE 条件，否则会更新全表数据！

```sql
-- 标准更新语法
UPDATE sys_user
SET age = 26, balance = 200.00
WHERE id = 1;

-- 批量更新
UPDATE sys_user
SET balance = balance + 100
WHERE age > 25;

-- 关联更新
UPDATE sys_user u
JOIN sys_order o ON u.id = o.user_id
SET u.balance = u.balance - o.order_amount
WHERE o.order_id = 1;
```

### 4\.3 数据删除 DELETE

⚠️ 高危操作：必须加 WHERE 条件，否则会删除全表数据！

```sql
-- 标准删除语法
DELETE FROM sys_user
WHERE id = 5;

-- 批量删除
DELETE FROM sys_user
WHERE is_deleted = 1 AND create_time < '2025-01-01';

-- 关联删除
DELETE u, o
FROM sys_user u
JOIN sys_order o ON u.id = o.user_id
WHERE u.id = 1;
```

> 补充：DELETE vs TRUNCATE
>
> - DELETE：DML 语句，可加 WHERE 条件删除指定行，可回滚，不会重置自增 ID，删除速度慢
>
> - TRUNCATE：DDL 语句，清空全表数据，不可回滚，重置自增 ID，删除速度极快
>
>

### 4\.4 基础查询 SELECT（核心高频）

查询是 MySQL 最核心的操作，语法执行顺序：`FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT`

```sql
-- 基础语法模板
SELECT [DISTINCT] 字段1, 字段2, 聚合函数...
FROM 表名
[WHERE 条件]
[GROUP BY 分组字段]
[HAVING 分组后过滤]
[ORDER BY 排序字段 排序规则]
[LIMIT 分页限制];
```

#### 4\.4\.1 基础查询示例

```sql
-- 1. 查询全表所有字段（生产环境禁用！性能极差）
SELECT * FROM sys_user;

-- 2. 查询指定字段（推荐写法，减少IO）
SELECT id, username, phone, age, balance FROM sys_user;

-- 3. 字段别名（AS可省略）
SELECT id AS 用户ID, username 用户名, balance 账户余额 FROM sys_user;

-- 4. 去重查询
SELECT DISTINCT gender FROM sys_user;

-- 5. 常量与运算查询
SELECT username, balance + 100 AS 余额加100, '测试' AS 备注 FROM sys_user;
```

#### 4\.4\.2 条件查询 WHERE

```sql
-- 1. 等值查询
SELECT * FROM sys_user WHERE id = 1;
SELECT * FROM sys_user WHERE username = 'zhangsan';

-- 2. 范围查询
SELECT * FROM sys_user WHERE age BETWEEN 20 AND 30;
SELECT * FROM sys_user WHERE id IN (1,3,5);

-- 3. 模糊查询（%匹配任意字符，_匹配单个字符）
SELECT * FROM sys_user WHERE username LIKE 'zhang%'; -- 以zhang开头
SELECT * FROM sys_user WHERE phone LIKE '%8000'; -- 以8000结尾
SELECT * FROM sys_user WHERE username LIKE '_hang%'; -- 第二个字符是hang

-- 4. 空值判断
SELECT * FROM sys_user WHERE phone IS NULL;
SELECT * FROM sys_user WHERE phone IS NOT NULL;

-- 5. 逻辑运算符（AND/OR/NOT）
SELECT * FROM sys_user WHERE age > 25 AND gender = '男' AND balance > 0;
SELECT * FROM sys_user WHERE age < 20 OR age > 35;
SELECT * FROM sys_user WHERE NOT gender = '未知';

-- 6. 比较运算符
SELECT * FROM sys_user WHERE age >= 25;
SELECT * FROM sys_user WHERE balance != 0;
```

#### 4\.4\.3 排序与分页

```sql
-- 1. 排序（ASC升序，默认；DESC降序）
-- 单字段排序
SELECT * FROM sys_user ORDER BY age DESC;
-- 多字段排序
SELECT * FROM sys_user ORDER BY balance DESC, age ASC;

-- 2. 分页查询（LIMIT 偏移量, 每页条数）
-- 第1页，每页10条
SELECT * FROM sys_user LIMIT 0, 10;
-- 第3页，每页10条（偏移量 = (页码-1)*每页条数）
SELECT * FROM sys_user LIMIT 20, 10;
-- 简写：只取前5条
SELECT * FROM sys_user LIMIT 5;
```

## 五、SQL 进阶查询

### 5\.1 聚合函数与分组查询

聚合函数对一组数据进行计算，返回单个结果，**NULL 值不参与计算**。

|函数|作用|
|---|---|
|COUNT\(\)|统计行数，COUNT \(\*\) 统计全表行数，COUNT \(字段\) 统计非空行数|
|SUM\(\)|求和，仅支持数值类型|
|AVG\(\)|求平均值，仅支持数值类型|
|MAX\(\)|求最大值|
|MIN\(\)|求最小值|

```sql
-- 1. 基础聚合函数使用
SELECT
  COUNT(*) AS 用户总数,
  SUM(balance) AS 总余额,
  AVG(age) AS 平均年龄,
  MAX(balance) AS 最高余额,
  MIN(age) AS 最小年龄
FROM sys_user;

-- 2. 分组查询 GROUP BY
-- 按性别分组，统计各性别的用户数、平均余额
SELECT
  gender,
  COUNT(*) AS 用户数,
  AVG(balance) AS 平均余额
FROM sys_user
GROUP BY gender;

-- 3. 分组后过滤 HAVING
-- 统计平均年龄大于25的性别分组，且用户数大于2
SELECT
  gender,
  COUNT(*) AS 用户数,
  AVG(age) AS 平均年龄
FROM sys_user
GROUP BY gender
HAVING 平均年龄 > 25 AND 用户数 > 2;
```

> 关键区别：WHERE vs HAVING
>
> - WHERE：分组前过滤，不能使用聚合函数，先过滤再分组
>
> - HAVING：分组后过滤，可使用聚合函数，先分组再过滤
>
>

### 5\.2 联表查询 JOIN

联表查询用于关联多张表的数据，核心分为内连接、外连接、交叉连接，关联条件推荐使用主键与外键关联。

#### 5\.2\.1 核心联表语法示例

```sql
-- 1. 内连接 INNER JOIN：只返回两张表中匹配关联条件的数据
-- 查询有订单的用户及其订单信息
SELECT
  u.id, u.username, o.order_no, o.order_amount, o.order_status
FROM sys_user u
INNER JOIN sys_order o ON u.id = o.user_id;

-- 2. 左连接 LEFT JOIN：返回左表所有数据，右表匹配不到的显示NULL
-- 查询所有用户，及其订单信息（无订单的用户也会显示）
SELECT
  u.id, u.username, o.order_no, o.order_amount
FROM sys_user u
LEFT JOIN sys_order o ON u.id = o.user_id;

-- 3. 右连接 RIGHT JOIN：返回右表所有数据，左表匹配不到的显示NULL
SELECT
  u.id, u.username, o.order_no, o.order_amount
FROM sys_user u
RIGHT JOIN sys_order o ON u.id = o.user_id;

-- 4. 多表联查
SELECT
  u.id, u.username, o.order_no, o.order_amount, p.product_name, p.price
FROM sys_user u
LEFT JOIN sys_order o ON u.id = o.user_id
LEFT JOIN order_item i ON o.order_id = i.order_id
LEFT JOIN product p ON i.product_id = p.id;
```

### 5\.3 子查询

子查询是嵌套在另一个 SQL 语句中的查询语句，结果作为外层查询的条件 / 数据源，优先执行子查询，再执行外层查询。

```sql
-- 1. 标量子查询（子查询返回单个值）
-- 查询余额大于平均余额的用户
SELECT * FROM sys_user
WHERE balance > (SELECT AVG(balance) FROM sys_user);

-- 2. 列子查询（子查询返回一列数据），搭配IN/ANY/ALL使用
-- 查询有下单记录的用户
SELECT * FROM sys_user
WHERE id IN (SELECT DISTINCT user_id FROM sys_order);

-- 3. 行子查询（子查询返回一行数据）
-- 查询和用户ID=1的年龄、性别都相同的用户
SELECT * FROM sys_user
WHERE (age, gender) = (SELECT age, gender FROM sys_user WHERE id = 1);

-- 4. 表子查询（子查询返回多行多列），作为临时表使用
-- 查询订单金额大于100的订单，及其用户信息
SELECT u.username, t.order_no, t.order_amount
FROM sys_user u
JOIN (SELECT * FROM sys_order WHERE order_amount > 100) t ON u.id = t.user_id;

-- 5. EXISTS子查询（判断子查询是否有结果，有则返回true）
-- 查询有订单的用户（比IN效率更高，大数据量推荐）
SELECT * FROM sys_user u
WHERE EXISTS (SELECT 1 FROM sys_order o WHERE o.user_id = u.id);
```

### 5\.4 常用内置函数

#### 5\.4\.1 字符串函数

```sql
-- 1. 字符串拼接
SELECT CONCAT(username, '-', phone) FROM sys_user;
-- 带分隔符拼接
SELECT CONCAT_WS('-', username, gender, age) FROM sys_user;

-- 2. 字符串长度
SELECT username, LENGTH(username) 字节长度, CHAR_LENGTH(username) 字符长度 FROM sys_user;

-- 3. 截取字符串
SELECT SUBSTRING(phone, 8, 4) 手机号后四位 FROM sys_user; -- 从第8位开始，截取4位

-- 4. 大小写转换
SELECT UPPER(username), LOWER(username) FROM sys_user;

-- 5. 替换字符串
SELECT REPLACE(phone, '138', '139') FROM sys_user;

-- 6. 去除首尾空格
SELECT TRIM(username) FROM sys_user;
```

#### 5\.4\.2 日期时间函数

```sql
-- 1. 获取当前时间
SELECT NOW(); -- 日期+时间
SELECT CURDATE(); -- 当前日期
SELECT CURTIME(); -- 当前时间

-- 2. 日期格式化
SELECT DATE_FORMAT(create_time, '%Y-%m-%d') 日期, DATE_FORMAT(create_time, '%H:%i:%s') 时间 FROM sys_user;

-- 3. 日期加减
SELECT DATE_ADD(NOW(), INTERVAL 7 DAY); -- 加7天
SELECT DATE_SUB(NOW(), INTERVAL 1 MONTH); -- 减1个月

-- 4. 计算日期差值
SELECT DATEDIFF(NOW(), create_time) 注册天数 FROM sys_user;

-- 5. 提取日期部分
SELECT YEAR(create_time) 年, MONTH(create_time) 月, DAY(create_time) 日 FROM sys_user;
```

#### 5\.4\.3 数值函数

```sql
-- 四舍五入
SELECT ROUND(balance, 2) FROM sys_user;
-- 向上取整
SELECT CEIL(age/10) FROM sys_user;
-- 向下取整
SELECT FLOOR(balance) FROM sys_user;
-- 取模（余数）
SELECT MOD(age, 2) FROM sys_user;
-- 随机数
SELECT RAND();
```

#### 5\.4\.4 流程控制函数

```sql
-- 1. IF函数：IF(条件, 满足值, 不满足值)
SELECT username, IF(balance > 100, '高余额', '低余额') 余额等级 FROM sys_user;

-- 2. IFNULL函数：IFNULL(值, 默认值)，NULL值替换
SELECT username, IFNULL(phone, '未填写手机号') 手机号 FROM sys_user;

-- 3. CASE WHEN 函数（多条件分支）
SELECT
  username,
  age,
  CASE
    WHEN age < 18 THEN '未成年'
    WHEN age BETWEEN 18 AND 30 THEN '青年'
    WHEN age BETWEEN 31 AND 60 THEN '中年'
    ELSE '老年'
  END AS 年龄分段
FROM sys_user;
```

### 5\.5 合并查询

```sql
-- UNION ALL：合并两个查询结果，不去重，性能高
SELECT id, username, phone FROM sys_user WHERE gender = '男'
UNION ALL
SELECT id, username, phone FROM sys_user WHERE age > 30;

-- UNION：合并两个查询结果，去重，性能略低
SELECT id, username, phone FROM sys_user WHERE gender = '男'
UNION
SELECT id, username, phone FROM sys_user WHERE age > 30;
```

> 注意：两个查询的字段数量、数据类型必须一致，字段名以第一个查询为准。
>
>

## 六、MySQL 核心高级特性

### 6\.1 事务管理

事务是一组 SQL 操作的最小逻辑单元，要么全部执行成功，要么全部执行失败回滚，是关系型数据库的核心特性，仅 InnoDB 引擎支持。

#### 6\.1\.1 事务 ACID 四大特性

|特性|全称|核心含义|底层实现|
|---|---|---|---|
|原子性（Atomicity）|事务是不可分割的最小单元，操作要么全成功，要么全回滚|Undo Log（回滚日志），记录数据修改前的状态，异常时反向撤销||
|一致性（Consistency）|事务执行前后，数据的业务规则和约束保持一致|原子性 \+ 隔离性 \+ 持久性 \+ 业务约束共同保障||
|隔离性（Isolation）|多个并发事务之间，操作互不干扰，由隔离级别控制|锁机制 \+ MVCC 多版本并发控制||
|持久性（Durability）|事务提交后，数据变更永久生效，即使数据库崩溃也不会丢失|Redo Log（重做日志），WAL 预写机制，先写日志再刷盘||

#### 6\.1\.2 事务隔离级别

SQL 标准定义了 4 种隔离级别，MySQL 默认隔离级别为**可重复读（REPEATABLE READ）**。

|隔离级别|脏读|不可重复读|幻读|特点|
|---|---|---|---|---|
|读未提交（READ UNCOMMITTED）|✅|✅|✅|最低级别，可读取其他事务未提交的数据，生产环境禁用|
|读已提交（READ COMMITTED）|❌|✅|✅|只能读取其他事务已提交的数据，解决脏读，Oracle 默认级别|
|可重复读（REPEATABLE READ）|❌|❌|✅|同一事务内多次读取同一数据结果一致，解决脏读、不可重复读，MySQL 默认|
|串行化（SERIALIZABLE）|❌|❌|❌|最高级别，事务串行执行，完全解决并发问题，性能极差，极少使用|

> 并发问题说明：
>
> - 脏读：事务 A 读取了事务 B 未提交的数据，B 回滚后，A 读取的数据无效
>
> - 不可重复读：同一事务内，两次读取同一数据，中间被其他事务修改，两次结果不一致
>
> - 幻读：同一事务内，两次查询同一范围的数据，中间被其他事务插入 / 删除，两次行数不一致
>
>

#### 6\.1\.3 事务实操语法

```sql
-- 1. 查看当前隔离级别
SELECT @@transaction_isolation;

-- 2. 修改隔离级别（会话级）
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- 3. 开启事务
START TRANSACTION;
-- 或简写
BEGIN;

-- 4. 事务内SQL操作
UPDATE sys_user SET balance = balance - 100 WHERE id = 1;
UPDATE sys_user SET balance = balance + 100 WHERE id = 2;

-- 5. 提交事务（成功执行）
COMMIT;

-- 6. 回滚事务（异常时执行，撤销所有操作）
ROLLBACK;

-- 7. 事务保存点（部分回滚）
BEGIN;
UPDATE sys_user SET balance = balance - 50 WHERE id = 1;
SAVEPOINT sp1; -- 创建保存点
UPDATE sys_user SET balance = balance - 50 WHERE id = 2;
ROLLBACK TO sp1; -- 回滚到保存点，只撤销第二个更新
COMMIT;
```

### 6\.2 视图

视图是基于 SQL 查询结果的虚拟表，本身不存储数据，数据来自底层的基础表，简化复杂查询，提升数据安全性。

```sql
-- 1. 创建视图
CREATE VIEW user_order_view AS
SELECT
  u.id, u.username, u.phone, o.order_no, o.order_amount, o.create_time order_time
FROM sys_user u
LEFT JOIN sys_order o ON u.id = o.user_id;

-- 2. 查询视图（和查询普通表语法一致）
SELECT * FROM user_order_view WHERE order_amount > 100;

-- 3. 修改视图
CREATE OR REPLACE VIEW user_order_view AS
SELECT
  u.id, u.username, o.order_no, o.order_amount, o.order_status
FROM sys_user u
LEFT JOIN sys_order o ON u.id = o.user_id;

-- 4. 删除视图
DROP VIEW IF EXISTS user_order_view;
```

### 6\.3 存储过程与自定义函数

#### 6\.3\.1 存储过程

存储过程是预先编译并存储在数据库中的一组 SQL 语句集合，可重复调用，支持参数传递，减少网络 IO，提升执行效率。

```sql
-- 1. 创建存储过程（无参数）
DELIMITER // -- 临时修改语句结束符为//，避免;提前结束
CREATE PROCEDURE get_user_count()
BEGIN
  SELECT COUNT(*) AS 用户总数 FROM sys_user;
END //
DELIMITER ; -- 恢复默认结束符;

-- 调用存储过程
CALL get_user_count();

-- 2. 创建带参数的存储过程（IN入参、OUT出参、INOUT入出参）
DELIMITER //
CREATE PROCEDURE get_user_by_id(
  IN in_user_id BIGINT, -- 入参：用户ID
  OUT out_username VARCHAR(50), -- 出参：用户名
  INOUT inout_balance DECIMAL(10,2) -- 入出参：账户余额
)
BEGIN
  -- 查询用户名赋值给出参
  SELECT username INTO out_username FROM sys_user WHERE id = in_user_id;
  -- 更新余额
  UPDATE sys_user SET balance = balance + inout_balance WHERE id = in_user_id;
  -- 查询更新后的余额赋值给入出参
  SELECT balance INTO inout_balance FROM sys_user WHERE id = in_user_id;
END //
DELIMITER ;

-- 调用带参数的存储过程
SET @add_balance = 100;
CALL get_user_by_id(1, @username, @add_balance);
-- 查看结果
SELECT @username, @add_balance;

-- 3. 删除存储过程
DROP PROCEDURE IF EXISTS get_user_by_id;
```

#### 6\.3\.2 自定义函数

自定义函数是用户自定义的功能函数，必须有返回值，可在 SQL 语句中直接调用。

```sql
-- 1. 创建自定义函数
DELIMITER //
CREATE FUNCTION calc_user_age_level(age INT)
RETURNS VARCHAR(20)
DETERMINISTIC -- 相同输入返回相同输出
BEGIN
  DECLARE age_level VARCHAR(20);
  IF age < 18 THEN SET age_level = '未成年';
  ELSEIF age BETWEEN 18 AND 30 THEN SET age_level = '青年';
  ELSEIF age BETWEEN 31 AND 60 THEN SET age_level = '中年';
  ELSE SET age_level = '老年';
  END IF;
  RETURN age_level;
END //
DELIMITER ;

-- 2. 调用函数
SELECT username, age, calc_user_age_level(age) 年龄等级 FROM sys_user;

-- 3. 删除函数
DROP FUNCTION IF EXISTS calc_user_age_level;
```

### 6\.4 触发器

触发器是与表绑定的特殊存储过程，当表发生 INSERT/UPDATE/DELETE 事件时，自动触发执行，可用于数据校验、日志记录、数据同步等场景。

```sql
-- 1. 创建触发器：用户数据更新时，记录操作日志
-- 先创建日志表
CREATE TABLE IF NOT EXISTS user_operate_log (
  id BIGINT AUTO_INCREMENT PRIMARY KEY,
  user_id BIGINT NOT NULL,
  operate_type VARCHAR(20) NOT NULL,
  old_value TEXT,
  new_value TEXT,
  operate_time DATETIME DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户操作日志表';

-- 创建更新触发器
DELIMITER //
CREATE TRIGGER user_update_trigger
BEFORE UPDATE ON sys_user -- 触发时机：更新前，触发事件：UPDATE
FOR EACH ROW -- 行级触发器，每更新一行触发一次
BEGIN
  -- 插入日志，OLD代表更新前的数据，NEW代表更新后的数据
  INSERT INTO user_operate_log(user_id, operate_type, old_value, new_value)
  VALUES (OLD.id, 'UPDATE', CONCAT('旧余额：', OLD.balance), CONCAT('新余额：', NEW.balance));
END //
DELIMITER ;

-- 2. 测试触发器
UPDATE sys_user SET balance = 500 WHERE id = 1;
-- 查看日志表
SELECT * FROM user_operate_log;

-- 3. 查看触发器
SHOW TRIGGERS;

-- 4. 删除触发器
DROP TRIGGER IF EXISTS user_update_trigger;
```

### 6\.5 锁机制

锁是 MySQL 实现事务隔离性的核心机制，用于控制并发事务对共享资源的访问，避免并发问题。InnoDB 锁分为以下几类：

1. **按粒度划分**

    - 全局锁：锁定整个数据库，用于全库备份，会阻塞所有读写操作

    - 表级锁：锁定整张表，分为表共享读锁、表排他写锁，开销小，加锁快，锁冲突概率高

    - 行级锁：锁定表中的指定行，分为行共享读锁、行排他写锁，开销大，加锁慢，锁冲突概率低，并发性能高（InnoDB 独有）

2. **按功能划分**

    - 共享锁（S 锁）：读锁，多个事务可同时加 S 锁，互不阻塞，加锁后只能读不能写

    - 排他锁（X 锁）：写锁，一个事务加 X 锁后，其他事务不能加任何锁，阻塞所有读写

    - 意向锁：表级锁，分为意向共享锁（IS）、意向排他锁（IX），用于快速判断表中是否有行锁，提升表锁加锁效率

3. **InnoDB 行锁算法**

    - 记录锁（Record Lock）：锁定单行索引记录

    - 间隙锁（Gap Lock）：锁定索引记录之间的间隙，防止幻读

    - 临键锁（Next\-Key Lock）：记录锁 \+ 间隙锁，InnoDB 默认行锁算法，可完全解决幻读问题

```sql
-- 手动加锁示例
-- 1. 加共享锁（S锁）
BEGIN;
SELECT * FROM sys_user WHERE id = 1 LOCK IN SHARE MODE;
COMMIT; -- 事务提交后释放锁

-- 2. 加排他锁（X锁）
BEGIN;
SELECT * FROM sys_user WHERE id = 1 FOR UPDATE;
UPDATE sys_user SET balance = 1000 WHERE id = 1;
COMMIT;
```

## 七、索引与 SQL 性能优化

索引是提升 MySQL 查询性能的核心手段，本质是**通过数据结构对表中数据进行排序，减少查询时的磁盘 IO 次数**，InnoDB 默认使用 B \+ 树作为索引数据结构。

### 7\.1 索引核心原理

#### 7\.1\.1 B \+ 树索引结构

B \+ 树是一种平衡多路查找树，特点：

- 非叶子节点只存储索引键，不存储数据，磁盘页可存储更多索引，树的高度更低（通常 3\-4 层即可支撑千万级数据）

- 叶子节点存储所有索引键和对应的数据，叶子节点之间通过双向链表连接，便于范围查询

- 所有查询最终都会落到叶子节点，查询性能稳定

#### 7\.1\.2 聚簇索引与二级索引

1. **聚簇索引**：主键索引，叶子节点存储整行数据，一个表只能有一个聚簇索引。如果没有定义主键，InnoDB 会自动选择唯一非空索引作为聚簇索引，没有则隐式创建一个 6 字节的 ROWID 作为聚簇索引。

2. **二级索引（辅助索引）**：非主键索引，叶子节点只存储索引键和主键值。通过二级索引查询时，先找到主键值，再通过聚簇索引查询整行数据，这个过程称为**回表**。

### 7\.2 索引的类型与创建

```sql
-- 1. 主键索引（创建表时通过PRIMARY KEY定义）
ALTER TABLE sys_user ADD PRIMARY KEY (id);

-- 2. 唯一索引（索引值必须唯一，允许NULL）
CREATE UNIQUE INDEX idx_username ON sys_user(username);

-- 3. 普通索引（最常用，无特殊约束）
CREATE INDEX idx_phone ON sys_user(phone);

-- 4. 联合索引（多个字段组合创建的索引，最左前缀原则）
CREATE INDEX idx_gender_age_balance ON sys_user(gender, age, balance);

-- 5. 全文索引（用于大文本字段的模糊查询，支持CHAR/VARCHAR/TEXT）
CREATE FULLTEXT INDEX idx_username_text ON sys_user(username);

-- 6. 查看表中所有索引
SHOW INDEX FROM sys_user;

-- 7. 删除索引
DROP INDEX idx_phone ON sys_user;
ALTER TABLE sys_user DROP INDEX idx_username;
```

### 7\.3 索引生效规则与失效场景

#### 7\.3\.1 最左前缀原则

联合索引必须遵循最左前缀原则，查询时必须匹配索引的最左 N 个字段，否则索引会失效。
例如联合索引`idx\_gender\_age\_balance\(gender, age, balance\)`：

- 生效场景：`gender`、`gender\+age`、`gender\+age\+balance`

- 失效场景：`age`、`age\+balance`、`gender\+balance`（仅 gender 字段生效，后面字段失效）

#### 7\.3\.2 索引失效的常见场景

```sql
-- 1. 索引列使用函数、表达式、算术运算
SELECT * FROM sys_user WHERE age + 1 = 26; -- 失效
SELECT * FROM sys_user WHERE SUBSTRING(phone, 1, 3) = '138'; -- 失效

-- 2. 隐式类型转换（字符串字段不加引号，数值字段加引号）
SELECT * FROM sys_user WHERE phone = 13800138000; -- phone是字符串，失效

-- 3. 模糊查询左通配符
SELECT * FROM sys_user WHERE username LIKE '%zhang'; -- 失效
SELECT * FROM sys_user WHERE username LIKE 'zhang%'; -- 生效（右通配）

-- 4. OR连接非索引列
SELECT * FROM sys_user WHERE username = 'zhangsan' OR age = 25; -- age无索引，全表扫描

-- 5. 联合索引违背最左前缀原则
SELECT * FROM sys_user WHERE age = 25 AND balance = 100; -- 失效

-- 6. 使用NOT、!=、<>、IS NOT NULL（部分场景）
SELECT * FROM sys_user WHERE age != 25; -- 失效

-- 7. 优化器判断全表扫描比走索引更快时（如小表、查询数据占全表大部分）
SELECT * FROM sys_user WHERE age > 0; -- 全表数据都符合，不走索引
```

### 7\.4 EXPLAIN 执行计划分析

EXPLAIN 是 MySQL 性能优化的核心工具，可查看 SQL 语句的执行计划，判断索引是否生效、表的关联顺序、扫描行数等核心信息。

#### 7\.4\.1 基础用法

```sql
EXPLAIN SELECT * FROM sys_user WHERE id = 1;
```

#### 7\.4\.2 核心字段详解

|字段|核心含义|优化重点|
|---|---|---|
|id|SQL 执行的序列号，id 越大越先执行，相同 id 从上到下执行|子查询、联表查询的执行顺序|
|select\_type|查询类型，如 SIMPLE（简单查询）、PRIMARY（主查询）、SUBQUERY（子查询）、DERIVED（派生表）|避免出现 DERIVED、DEPENDENT SUBQUERY，性能差|
|type|访问类型，性能从优到劣：system \&gt; const \&gt; eq\_ref \&gt; ref \&gt; range \&gt; index \&gt; ALL|优化目标：至少达到 range 级别，最好达到 ref，杜绝 ALL（全表扫描）|
|possible\_keys|可能用到的索引|与 key 对比，判断索引选择是否合理|
|key|实际用到的索引，NULL 表示未使用索引|核心优化字段，确保查询命中合适的索引|
|key\_len|索引使用的字节长度|联合索引中，可判断用到了索引的哪些字段|
|rows|扫描的行数，估算值|数值越小越好，代表扫描的数据量越少|
|Extra|额外信息，核心优化字段||

#### 7\.4\.3 Extra 关键字段说明

- **Using index**：覆盖索引，查询的字段都在索引中，无需回表，性能最优

- **Using where**：使用 WHERE 条件过滤数据，无索引时代表全表扫描后过滤

- **Using temporary**：使用临时表存储结果，常见于 GROUP BY、ORDER BY，性能差，需优化

- **Using filesort**：文件排序，无法使用索引完成排序，性能极差，必须优化

- **Using index condition**：索引条件下推，优化了联表查询，减少回表次数

### 7\.5 通用 SQL 优化技巧

1. **查询字段优化**：禁止使用`SELECT \*`，只查询需要的字段，减少 IO 和回表次数

2. **索引优化**：为 WHERE、JOIN、ORDER BY、GROUP BY 的字段建立合适的索引，避免冗余索引、失效索引，单表索引数量控制在 5 个以内

3. **分页优化**：深分页场景优化，如`SELECT \* FROM sys\_user LIMIT 100000, 10`，优化为`SELECT \* FROM sys\_user WHERE id \&gt; 100000 LIMIT 10`

4. **联表优化**：JOIN 的表数量控制在 3 个以内，关联字段必须建立索引，小表驱动大表

5. **避免索引失效**：杜绝上述索引失效场景，严格遵循最左前缀原则

6. **批量操作优化**：批量插入替代循环单行插入，批量更新替代循环单行更新

7. **避免大事务**：大事务会导致锁持有时间过长，阻塞其他事务，引发主从延迟

8. **排序优化**：ORDER BY 的字段必须建立索引，避免出现 Using filesort

## 八、MySQL 运维与安全管理

### 8\.1 用户与权限管理

MySQL 权限管理基于「用户 \+ 主机」的模式，核心原则：**最小权限原则**，只给用户分配必要的权限，禁止普通用户拥有超级权限。

#### 8\.1\.1 用户管理

```sql
-- 1. 创建用户（%代表允许所有远程主机登录，localhost仅本地登录）
CREATE USER 'test_user'@'%' IDENTIFIED BY 'Test@User2026';
-- 8.0创建用户后必须单独授权，5.7可创建时直接授权

-- 2. 修改用户密码
ALTER USER 'test_user'@'%' IDENTIFIED BY 'New@Test2026';

-- 3. 重命名用户
RENAME USER 'test_user'@'%' TO 'app_user'@'%';

-- 4. 删除用户
DROP USER IF EXISTS 'app_user'@'%';

-- 5. 查看所有用户
SELECT user, host FROM mysql.user;
```

#### 8\.1\.2 权限管理

```sql
-- 1. 授予权限
-- 授予test_db库所有表的查询、插入权限给test_user
GRANT SELECT, INSERT ON test_db.* TO 'test_user'@'%';
-- 授予所有库所有表的所有权限（超级权限，仅root可使用，禁止给普通用户）
GRANT ALL PRIVILEGES ON *.* TO 'admin_user'@'%' WITH GRANT OPTION;

-- 2. 刷新权限，授权后必须执行
FLUSH PRIVILEGES;

-- 3. 查看用户权限
SHOW GRANTS FOR 'test_user'@'%';

-- 4. 撤销权限
REVOKE INSERT ON test_db.* FROM 'test_user'@'%';
-- 撤销所有权限
REVOKE ALL PRIVILEGES, GRANT OPTION FROM 'test_user'@'%';
```

### 8\.2 数据备份与恢复

数据备份是数据库安全的最后一道防线，生产环境必须制定完善的备份策略，定期执行备份并验证恢复能力。

#### 8\.2\.1 逻辑备份（mysqldump）

mysqldump 是 MySQL 自带的逻辑备份工具，备份为 SQL 文件，可跨版本恢复，适合中小规模数据库。

```bash
# 1. 全库备份
mysqldump -u root -p --all-databases > all_database_backup.sql

# 2. 单库备份
mysqldump -u root -p test_db > test_db_backup.sql

# 3. 单表备份
mysqldump -u root -p test_db sys_user > sys_user_backup.sql

# 4. 只备份表结构，不备份数据
mysqldump -u root -p test_db --no-data > test_db_schema.sql

# 5. 数据恢复
# 方式1：登录MySQL后执行
mysql -u root -p
USE test_db;
source /root/test_db_backup.sql;

# 方式2：命令行直接恢复
mysql -u root -p test_db < test_db_backup.sql
```

#### 8\.2\.2 物理备份

物理备份直接备份数据库的数据文件，速度快，适合大规模数据库，主流工具为 Percona XtraBackup（开源免费，支持 InnoDB 热备，不锁表）。

- 核心优势：备份速度快，支持增量备份，备份时不锁表，恢复速度快

- 适用场景：生产环境 TB 级数据库，全量 \+ 增量备份策略

#### 8\.2\.3 备份策略建议

- 中小规模数据库：每日凌晨全量备份，binlog 开启实时备份

- 大规模数据库：每周日全量备份，每日凌晨增量备份，binlog 实时备份

- 备份文件：异地存储，定期验证恢复能力，保留至少 3 个备份周期

### 8\.3 日志管理

MySQL 日志是故障排查、性能优化、数据恢复的核心依据，核心日志分为以下 5 类：

|日志类型|核心作用|配置建议|
|---|---|---|
|错误日志（Error Log）|记录 MySQL 启动、运行、关闭过程中的错误信息，故障排查核心|强制开启，默认开启|
|二进制日志（Binlog）|记录所有数据变更操作，用于主从复制、数据恢复|生产环境强制开启，格式选择 ROW，过期时间 7\-30 天|
|慢查询日志（Slow Query Log）|记录执行时间超过阈值的 SQL 语句，性能优化核心|生产环境开启，阈值 long\_query\_time=1s|
|通用查询日志（General Log）|记录所有客户端连接和执行的 SQL 语句|生产环境关闭，仅调试时临时开启|
|中继日志（Relay Log）|主从复制中，从库用于存储主库的 Binlog 日志|从库开启，主库无需开启|

#### 8\.3\.1 Binlog 核心配置与操作

```sql
-- 1. 查看Binlog是否开启
SHOW VARIABLES LIKE 'log_bin';

-- 2. 查看所有Binlog文件
SHOW BINARY LOGS;

-- 3. 查看当前正在写入的Binlog文件
SHOW MASTER STATUS;

-- 4. 手动刷新Binlog，生成新的Binlog文件
FLUSH LOGS;

-- 5. 清空所有Binlog文件（高危操作！主从环境禁用）
RESET MASTER;
```

#### 8\.3\.2 慢查询日志配置

```sql
-- 1. 查看慢查询日志配置
SHOW VARIABLES LIKE 'slow_query%';
SHOW VARIABLES LIKE 'long_query_time';

-- 2. 临时开启慢查询日志（重启后失效，永久生效需修改my.cnf配置文件）
SET GLOBAL slow_query_log = 'ON';
-- 设置慢查询阈值1秒
SET GLOBAL long_query_time = 1;
-- 记录未使用索引的SQL
SET GLOBAL log_queries_not_using_indexes = 'ON';
```

### 8\.4 安全配置建议

1. 禁止 root 用户远程登录，仅本地登录，创建专用的业务用户分配最小权限

2. 强密码策略：所有用户密码必须包含大小写、数字、特殊符号，长度不低于 8 位，定期更换

3. 修改默认端口 3306，避免被端口扫描攻击

4. 防火墙限制：仅允许业务服务器 IP 访问数据库端口，禁止公网全开放

5. 关闭不必要的功能：如 LOAD DATA LOCAL INFILE、符号链接等

6. 定期更新 MySQL 版本，修复安全漏洞

7. 开启审计日志，记录用户的关键操作，便于安全审计

8. 防范 SQL 注入：业务代码禁止使用字符串拼接 SQL，使用预编译语句

## 九、企业级进阶实战

### 9\.1 主从复制

MySQL 主从复制是高可用架构的基础，实现数据冗余、读写分离、故障切换，核心原理基于 Binlog 日志实现。

#### 9\.1\.1 主从复制原理

1. 主库（Master）：数据变更操作写入 Binlog 日志

2. 从库（Slave）：IO 线程读取主库的 Binlog 日志，写入本地的中继日志（Relay Log）

3. 从库（Slave）：SQL 线程读取中继日志，重放 SQL 语句，实现数据同步

#### 9\.1\.2 一主一从搭建核心步骤

1. 主库配置（my\.cnf）

```ini
[mysqld]
server-id=1 # 唯一ID，主从不能重复
log_bin=mysql-bin # 开启Binlog
binlog_format=ROW # Binlog格式，推荐ROW
expire_logs_days=7 # Binlog过期时间
binlog_do_db=test_db # 需要同步的库，不配置则同步所有库
```

2. 重启主库，创建主从复制专用用户

```sql
CREATE USER 'repl'@'%' IDENTIFIED BY 'Repl@2026';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
FLUSH PRIVILEGES;
-- 锁表，禁止数据写入，备份数据
FLUSH TABLES WITH READ LOCK;
-- 查看主库状态，记录File和Position值
SHOW MASTER STATUS;
```

3. 全量备份主库数据，导入到从库，保证主从初始数据一致

4. 从库配置（my\.cnf）

```ini
[mysqld]
server-id=2 # 与主库不同
relay_log=relay-bin # 开启中继日志
read_only=1 # 设置只读，超级用户除外
log_slave_updates=1 # 级联复制开启
```

5. 重启从库，配置主从连接

```sql
CHANGE MASTER TO
MASTER_HOST='主库IP',
MASTER_PORT=3306,
MASTER_USER='repl',
MASTER_PASSWORD='Repl@2026',
MASTER_LOG_FILE='mysql-bin.000001', # 主库SHOW MASTER STATUS的File值
MASTER_LOG_POS=156; # 主库SHOW MASTER STATUS的Position值

-- 启动主从复制
START SLAVE;

-- 查看主从复制状态
SHOW SLAVE STATUS\G
-- 核心指标：Slave_IO_Running=Yes、Slave_SQL_Running=Yes，代表复制正常
```

6. 主库解锁表

```sql
UNLOCK TABLES;
```

### 9\.2 读写分离

基于主从复制实现读写分离，**写操作全部走主库，读操作全部走从库**，分摊主库的压力，提升数据库并发性能。

- 实现方案：

    1. 代码层实现：业务代码中根据 SQL 类型，动态切换数据源，简单灵活，无额外组件依赖

    2. 中间件实现：通过数据库中间件实现自动读写分离，如 MyCat、Sharding\-JDBC、ProxySQL，支持负载均衡、故障切换，适合大规模集群

### 9\.3 分库分表

当单表数据量超过千万级、单库容量超过 TB 级时，数据库性能会急剧下降，此时需要进行分库分表，将海量数据分散到多个库、多个表中存储。

#### 9\.3\.1 分库分表类型

1. **垂直分库**：按业务模块拆分，将不同业务的表拆分到不同的数据库中，如用户库、订单库、商品库，降低单库压力

2. **垂直分表**：将大表按字段拆分，把高频访问的字段和低频访问的字段拆分到不同的表中，如用户基础信息表、用户详情表，提升单表查询性能

3. **水平分表**：将同一张表的数据按分片规则拆分到多个结构相同的表中，如 user\_0、user\_1、user\_2\.\.\.，解决单表海量数据的性能问题

#### 9\.3\.2 常用分片规则

- 范围分片：按时间、ID 范围分片，如按月份拆分订单表

- 哈希分片：对分片键取模，均匀分散数据，最常用，如按用户 ID 取模

- 一致性哈希：解决动态扩容的问题，适合集群节点动态变化的场景

#### 9\.3\.3 主流中间件

- 客户端层：Sharding\-JDBC（轻量，无额外部署，Java 生态首选）

- 代理层：MyCat、ProxySQL（支持多语言，功能全面，需独立部署）

### 9\.4 高可用架构

生产环境核心业务必须使用高可用架构，避免单点故障，保障数据库服务持续可用，主流方案：

1. **主从 \+ Keepalived**：一主一从架构，通过 Keepalived 实现虚拟 IP 漂移，主库故障时自动切换到从库，实现故障自动转移

2. **MGR（MySQL Group Replication）**：MySQL 官方原生的组复制技术，基于 Paxos 协议，支持多主模式，数据强一致性，自动故障检测与切换，是 MySQL 高可用的主流方案

3. **集群架构**：基于 MyCat、ShardingSphere 的分布式数据库集群，支持读写分离、分库分表、高可用，适合超大规模业务场景

## 十、学习路径与进阶建议

1. **入门阶段**：熟练掌握环境部署、库表操作、增删改查、基础查询，多动手实操，完成简单的业务 SQL 编写

2. **进阶阶段**：深入学习联表查询、子查询、事务、索引、存储过程，掌握 EXPLAIN 执行计划分析，能完成基础的 SQL 优化

3. **高级阶段**：深入理解 InnoDB 存储引擎原理、事务底层实现、MVCC 机制、锁机制、索引底层结构，能解决复杂的性能问题

4. **运维阶段**：掌握用户权限管理、备份恢复、日志分析、主从复制搭建、故障排查，能完成数据库日常运维工作

5. **架构阶段**：掌握读写分离、分库分表、高可用架构设计，能根据业务场景设计合理的数据库架构

> 官方参考文档：[MySQL 8\.0 Reference Manual](https://dev.mysql.com/doc/refman/8.0/en/)，是最权威、最全面的 MySQL 学习资料。
>
