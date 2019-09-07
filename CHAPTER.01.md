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
