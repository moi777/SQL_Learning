### Notification

主要内容来源：极客时间--->SQL 必会必知

### 01丨了解SQL：一门半衰期很长的语言

SQL(Structured Query Language)：结构化查询语言，可以用于和数据库进行交互。

SQL 的特点：简单易学，通用性强，半衰期长（长期变化不大）。



### 02丨DBMS的前世今生

DBMS(Database Management System)：数据库管理系统，是一种操纵和管理数据库的大型软件，用于建立、使用和维护数据库。

数据库类型有：关系型数据库、键值型数据库、文档型数据库、搜索引擎、列式数据库、图形数据库，其中关系型数据库（RDBMS）最为主流，常见的有：Oracle，MySQL，SQL Server。



### 03丨学会用数据库的方式思考SQL是如何执行的

**Oracle 中的 SQL 是如何执行的**

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6qrwiynhej30wo0bogms.jpg)

1. 语法检查：检查拼写是否正确。
2. 语义检查：检查访问的对象是否存在。
3. 权限检查：检查用户是否访问该数据的权限。
4. 共享池检查：共享池（Shared Pool)最主要的作用是缓存 SQL 语句和该语句的执行计划。如果存在执行计划，则进行软解析，如果没有，则进入优化器步骤。
5. 优化器：进行硬解析，比如创建解析树，生成执行计划。
6. 执行器：有了解析树和执行计划之后，就知道 SQL 该怎么被执行，这样就可以在执行器中执行 SQL 语句了。

为了提升 SQL 的执行效率，应该尽量避免硬解析，可以通过绑定变量的形式来提升软解析的可能性，举例：

```
SELECT * FROM player WHERE player_id = 10001; -- 查询 player_id = 10002 时需要进行硬解析

SELECT * FROM player WHERE player_id = :player_id; -- 查询 player_id = 10002 时进行软解析
```



**MySQL 中的 SQL 是如何执行的**

MySQL 是典型的 C/S 架构，即 Client/Server 架构，服务端程序使用的是 mysqld。

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6qrxg74iej30p00dcq4c.jpg)

MySQL 由三层组成：

1. 连接层：客户端和服务器端建立连接，客户端发送 SQL 到服务器端。
2. SQL 层：对 SQL 语句进行查询处理。
3. 存储引擎层：与数据库文件打交道，负责数据的存储和读取。

SQL 层的结构如下：

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6qrx4npl6j30cy0gm74x.jpg)

1. 查询缓存：查看缓存中是否有这条 SQL 语句，如果有就直接将结果返回客户端，如果没有，进入解析器阶段。因为查询效率不高，MySQL 8.0 之后抛弃了这个功能。
2. 解析器：对 SQL 语句进行语法分析、语义分析。
3. 优化器：确定 SQL 语句的执行路径。
4. 执行器：执行前判断用户是否有权限。

与 Oracle 不同的是，MySQL 的存储引擎采用了插件的形式，每个存储引擎都面向一种特定的数据库应用环境。MySQL 还允许开发人员设置自己的存储引擎，以下是常见的存储引擎：

- InnoDB：MySQL 5.5 之后的默认存储引擎，支持事务、行级锁定、外键约束。
- MyISAM: MySQL 5.5 之前默认的存储引擎，不支持事务，也不支持外键，但是速度快、占用资源少。
- Memory: 使用系统内存作为存储介质，响应速度快。但 mysqld 进程崩溃会导致所有数据丢失，所以一般只在数据是临时的情况下才使用。
- NDB: 也叫做 NDB Cluster，用于 MySQL 分布式集群环境。
- Archive: 有很好的压缩机制，用于文件归档。在请求写入时会进行压缩，所以也经常用来做仓库。

数据库的设计在于表的设计，MySQL 中的每个表都可以采用不同的存储引擎，可以根据实际的数据处理需求来选择存储引擎。

**查看 SQL 执行的资源（时间）：**

```
SELECT @@PROFILING;

SET PROFILING = 1; -- profiling=0 表示关闭，=1 表示开启

SELECT * FROM player; -- 执行一个 SQL 查询

SHOW PROFILE; -- 获取执行时间

SHOW VERSION(); -- 查看 MySQL 版本情况
```



###  04丨使用DDL创建数据库&数据表时需要注意什么?

**DDL 的基础语法及设计工具**

DDL(Data Definition Language)：数据定义语言，它定义了数据库和数据表的结构。

DDL 的常用指令为：CERATE、DROP、ALTER，DDL 无需 COMMIT 就可以完成。

1. 对数据库进行定义

```
CREATE DATABASE nba; -- 创建一个名为 nba 的数据库

DROP DATABASE nba; -- 删除一个名为 nba 的数据库
```

2. 对数据表进行定义

```
CREATE TABLE niubi; -- 创建一个名为 niubi 的数据表
	
DROP TABLE niubi; -- 删除一个名为 niubi 的数据表
```



**创建表结构**

1. 通过 SQL 语句创建表结构

```
CREATE TABLE player  (
  player_id int(11) NOT NULL AUTO_INCREMENT,
  player_name varchar(255) NOT NULL
); -- 语句最后以分号作为结束符，最后一个字段定义的最后没有逗号
```



2. 通过 Navicat 等软件创建表结构，创建完成后可以导出为 SQL 语句

```
DROP TABLE IF EXISTS `player`;
CREATE TABLE `player`  (
  `player_id` int(11) NOT NULL AUTO_INCREMENT,
  `team_id` int(11) NOT NULL,
  `player_name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `height` float(3, 2) NULL DEFAULT 0.00,
  PRIMARY KEY (`player_id`) USING BTREE,
  UNIQUE INDEX `player_name`(`player_name`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
```



**修改表结构**

```
ALTER TABLE player ADD (age int(11)); -- 增加字段 age

ALTER TABLE player RENAME COLUMN age to player_age; -- 修改字段名

ALTER TABLE player MODIFY (player_age float(3,1)); -- 修改字段数据类型

ALTER TABLE player DROP COLUMN player_age; -- 删除字段
```



**数据表的常见约束**

1. 主键约束：不能重复，不能为空，即 UNIQUE + NOT NULL。
2. 外键约束：用于确保表与表之间引用的完整性。可以重复，可以为空。
3. 唯一性约束：使得字段在表中的数值是唯一的。
4. NOT NULL 约束：字段不为空。
5. DEFAULT：字段的默认值，如果在插入数据时，这个字段没有取值，则设置为默认值。
6. CHECK 约束：检查特定字段取值范围的有效性，CHECK 的约束结果不能为 FALSE。



**设计数据表的原则**

核心原则是「简单复用」：

1. 数据表的个数越少越好
2. 数据表中字段的个数越少越好
3. 数据表中联合主键的字段个数越少越好
4. 使用主键和外键越多越好



### 05丨检索数据：你还在SELECT * 么？

**查询列**

```
SELECT name FROM heros; -- 查询 heros 表中的 name 列

SELECT name, id FROM heros; -- 查询 heros 表中的 name 和 id 列

SELECT * FROM heros; -- 查询 heros 表中的所有列
```



**起别名**

```
SELECT name AS n, hp_max AS hm, mp_max AS mm, attack_max AS am, defense_max AS dm FROM heros
```



**查询常数**

```
SELECT '王者荣耀' as platform, name FROM heros; -- 在查询结果前面增加一列字段 platform，如果常数是字符串，需要加上单引号，否则会当做字段名进行查询，如果是数值，则不需要引号
```

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6qsmvq1ljj30cv05a3yt.jpg)

**去除重复行**

```
SELECT DISTINCT attack_range FROM heros;

SELECT DISTINCT attack_range, name FROM heros; -- DISTINCT 是对后面所有列名的组合去重
```



**检索排序**

```
SELECT name, hp_max FROM heros ORDER BY hp_max DESC;

SELECT name, hp_max FROM heros ORDER BY mp_max, hp_max DESC; -- 递增使用 ASC，递减使用 DESC
```



**约束返回结果的数量**

```
SELECT name, hp_max FROM heros ORDER BY hp_max DESC LIMIT 5; -- MySQL/PostgreSQL/MariaDB/SQLite 中使用 LIMIT 关键字

SELECT name, hp_max FROM heros ORDER BY hp_max DESC FETCH FIRST 5 ROWS ONLY; -- DB2 语法

SELECT name, hp_max FROM heros WHERE ROWNUM <=5 ORDER BY hp_max DESC; -- Oracle 语法
```



**SELECT 的执行顺序**

1. 关键字顺序

```
SELECT ... FROM ... WHERE ... GROUP BY ... HAVING ... ORDER BY ...
```

2. SELECT 语句的执行顺序

```
FROM > WHERE > GROUP BY > HAVING > SELECT 的字段 > DISTINCT > ORDER BY > LIMIT
```



**如何提升 SELECT 查询效率**

1. 写清列名，可以减少数据表查询的网络传输量，不推荐直接使用 SELECT * 进行查询。
2. 约束返回记录的数量。



### 06丨数据过滤：SQL数据过滤都有哪些方法？

在 SQL 中，可以在 WHERE 子句对条件进行筛选，经常会和比较运算符、逻辑运算符、通配符一起使用。

**比较运算符**

```
SELECT name, hp_max FROM heros WHERE hp_max > 6000; -- 从 heros 表中取出 hp_max > 
6000 的记录

SELECT name, hp_max FROM heros WHERE hp_max BETWEEN 5399 AND 6811; -- 从 heros 表中取出 hp_max 在 5399 和 6811 之间的记录

SELECT name, hp_max FROM heros WHERE hp_max IS NULL; -- 从 heros 表中取出 hp_max 值为空的记录
```

**逻辑运算符**

```
SELECT name, hp_max, mp_max FROM heros WHERE hp_max > 6000 AND mp_max > 1700 ORDER BY (hp_max+mp_max) DESC; -- AND 运算符的使用，满足所有条件的记录才会被取出

SELECT name, hp_max, mp_max FROM heros WHERE ((hp_max+mp_max) > 8000 OR hp_max > 6000) AND mp_max > 1700 ORDER BY (hp_max+mp_max) DESC; -- OR 运算符的使用，满足其中一个条件的记录就会被取出
```

**通配符**

```
SELECT name FROM heros WHERE name LIKE '% 太 %'; -- % 可以匹配任意字符出现的任意次数
SELECT name FROM heros WHERE name LIKE '_% 太 %'; -- _ 可以匹配一个字符
```



### 07丨什么是SQL函数？为什么使用SQL函数可能会带来问题？

函数可以把我们经常使用的代码封装起来，需要的时候直接调用。这样既提高了代码效率，又提高了可维护性。

SQL 提供了一些常用的内置函数，可以分为四类：算数函数、字符串函数、日期函数、转换函数。



**算数函数**

```
SELECT ABS(-2); -- 取 -2 的绝对值

SELECT MOD(101,3); -- 取 101 除以 3 的余数

SELECT ROUND(37.25, 1); -- 37.23 四舍五入，保留 1 为销售
```



**字符串函数**

```
SELECT CONCAT('abc',123); -- 字符串拼接，运行结果为 abc123

SELECT LENGTH('你好'); -- 计算字段长度，一个汉字算三个字符，一个数字或字母算一个字符，运行结果为 6

SELECT CHAR_LENGTH('你好'); -- 计算字段长度，一个汉字、一个数字或字母算一个字符，运行结果为 2

SELECT LOWER('ABC'); -- 将字符串中的字符转化为小写，运行结果为 abc

SELECT UPPER('abc'); -- 将字符串中的字符转化为大写，运行结果为 ABC

SELECT REPLACE('fabcd','abc',123); -- 替换函数，运行结果为 f123d

SELECT SUBSTRING('fabcd',1,3); -- 截取字符串，运行结果为 fab
```



**日期函数**

```
SELECT CURRENT_DATE(); -- 系统当前日期

SELECT CURRENT_TIME(); -- 系统当前时间，没有日期

SELECT CURRENT_TIMESTAMP(); -- 系统当前时间，含日期和时间

SELECT EXTRACT(YEAR FROM '2019-09-06'); -- 抽取具体的年，运行结果为 2019

SELECT DATE('2019-09-06 12:00:07'); -- 返回时间的日期部分

SELECT YEAR('2019-09-06 12:00:07'); -- 返回时间的年的部分

SELECT MONTH('2019-09-06 12:00:07'); -- 返回时间的月的部分

SELECT DAY('2019-09-06 12:00:07'); -- 返回时间的日的部分

SELECT HOUR('2019-09-06 12:00:07'); -- 返回时间的时的部分

SELECT MINUTE('2019-09-06 12:00:07'); -- 返回时间的分的部分

SELECT SECOND('2019-09-06 12:00:07'); -- 返回时间的秒的部分
```



**转换函数**

```
SELECT CAST(123.123 AS INT); -- 浮点型和整数型不能互相转换，结果会报错

SELECT CAST(123.125 AS DECIMAL(8,2)); -- 将 123.123 转换为两位小数，整数位加上小数位不能超过 8 位。注意使用 CAST 做转换时不会进行四舍五入，运行结果为 123.12

SELECT COALESCE(NULL,1,2); -- 返回第一个非空数值，运行结果为 1
```



**使用 SQL 函数对王者荣耀数据库做处理**

```
SELECT name, ROUND(attack_growth, 1) FROM heros;

SELECT MAX(hp_max) FROM heros;

SELECT name, hp_max FROM heros WHERE hp_max = (SELECT MAX(hp_max) FROM heros);

SELECT CHAR_LENGTH(name), name FROM heros;

SELECT name, EXTRACT(YEAR FROM birthdate) FROM heros WHERE birthdate IS NOT NULL;

SELECT name, YEAR(birthdate) FROM heros WHERE birthdate IS NOT NULL;

SELECT * FROM heros WHERE DATE(birthdate) > '2016-10-01';

SELECT AVG(hp_max), AVG(mp_max), MAX(attack_max) FROM heros WHERE DATE(birthdate) > '2016-10-01'
```



**为什么使用SQL函数可能会带来问题？**

两点考虑：Python 版本的差异性、DBMS 之间的差异性。



### 08丨什么是SQL的聚集函数，如何利用它们汇总表的数据？

聚集函数是对一组数据进行汇总的函数，输入时一组数据的集合，输出是单个值。



**聚集函数有哪些**

SQL 中的聚集函数一共包括 5 个，COUNT()、MAX()、MIN()、SUM()、AVG()。

```
SELECT COUNT(*) FROM heros WHERE hp_max > 6000;

SELECT COUNT(role_assist) FROM heros WHERE hp_max > 6000; -- COUNT(role_assist) 会忽略值为 NULL 的数据行，而 COUNT(*) 只是统计数据行数，不论字段的值是否为 NULL

SELECT MAX(hp_max) FROM heros WHERE role_main = '射手' OR role_assist = '射手';

SELECT COUNT(*), AVG(hp_max), MAX(mp_max), MIN(attack_max), SUM(defense_max) FROM heros WHERE role_main = '射手' OR role_assist = '射手'; -- AVG, MAX, MIN 会自动忽略值为 NULL 的数据行

SELECT MIN(CONVERT(name USING gbk)), MAX(CONVERT(name USING gbk)) FROM heros; -- MAX 和 MIN 可用于字符串类型数据的统计，如果是英文字母，按照 A-Z 的顺序排列，如果是汉字则按照全拼拼音进行排列；gbk 表示 按汉字首字母排序

SELECT COUNT(DISTINCT hp_max) FROM heros; -- 可以先使用 DISTINCT 去重，再使用聚集函数

SELECT ROUND(AVG(DISTINCT hp_max), 2) FROM heros;
```



**如何对数据进行分组，并进行聚集统计**

对数据进行分组，使用 GROUP BY 子句。

```
SELECT COUNT(*), role_main FROM heros GROUP BY role_main; --按照 role_main 对 heros 表的数据进行分组

SELECT COUNT(*), role_assist FROM heros GROUP BY role_assist;

SELECT COUNT(*) AS num, role_main, role_assist FROM heros GROUP BY role_main, role_assist ORDER BY num DESC;
```



**如何使用 HAVING 过滤分组，它与 WHERE 的区别是什么？**

对分组进行过滤使用 HAVING，对数据行进行过滤使用 WHERE。

```
SELECT COUNT(*) AS num, role_main, role_assist FROM heros GROUP BY role_main, role_assist HAVING num > 5 ORDER BY num DESC; -- HAVING 用于对分组进行过滤

SELECT COUNT(*) AS num, role_main, role_assist FROM heros WHERE hp_max >6000 GROUP BY role_main, role_assist HAVING num > 5 ORDER BY num DESC;
```



SELECT 查询中，关键字的顺序`SELECT ... FROM ... WHERE ... GROUP BY ... HAVING ... ORDER BY ...`

