+++
date = "2020-05-18T00:00:00Z"
tags = ["database", "ORM"]
categories = ["database"]
title = "Atomic DDL"

+++

[上回书](https://wpchou.github.io/post/2020-05-17-typeorm-database-migration)
说到原子性 DDL 是将数据库自动迁移应用到生产环境的关键保障。然后，不幸的是
[mysql 文档](https://dev.mysql.com/doc/refman/8.0/en/atomic-ddl.html)
明确说了：

> Atomic DDL is not transactional DDL. DDL statements, atomic or otherwise, implicitly end any transaction that is active in the current session, as if you had done a COMMIT before executing the statement. This means that DDL statements cannot be performed within another transaction, within transaction control statements such as START TRANSACTION ... COMMIT, or combined with other statements within the same transaction.

任何的 DDL(数据定义语句)执行前都会自动作一个 `commit` 的动作. 世界上最流行的开源数据库 GAME OVER!
流行的果然是辣鸡！我们来看看世界上最先进的开源数据库。

![postgresql](/assets/google-postgresql.png)

果然是最先进的开源数据库，请进入

https://wiki.postgresql.org/wiki/Transactional_DDL_in_PostgreSQL:_A_Competitive_Analysis

看 pg 吊打一众主流数据库，尤其是 mysql.

```
$ psql mydb
mydb=# DROP TABLE IF EXISTS foo;
NOTICE: table "foo" does not exist
DROP TABLE
mydb=# BEGIN;
BEGIN
mydb=# CREATE TABLE foo (bar int);
CREATE TABLE
mydb=# INSERT INTO foo VALUES (1);
INSERT 0 1
mydb=# ROLLBACK;
ROLLBACK
mydb=# SELECT * FROM foo;
ERROR: relation "foo" does not exist
mydb=# SELECT version();
version
----------------------------------------------------------------------
PostgreSQL 8.3.7 on i386-redhat-linux-gnu, compiled by GCC gcc (GCC) 4.3.2 20081105 (Red Hat 4.3.2-7)
(1 row)
```

```
mysql> drop table if exists foo;
Query OK, 0 rows affected (0.00 sec)

mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> create table foo (bar int) type=InnoDB;
Query OK, 0 rows affected, 1 warning (0.03 sec)

mysql> insert into foo values (1);
Query OK, 1 row affected (0.00 sec)

mysql> rollback;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from foo;
+------+
| bar |
+------+
| 1 |
+------+
1 row in set (0.00 sec)

mysql> select version();
+--------------------------+
| version() |
+--------------------------+
| 5.0.32-Debian_7etch1-log |
+--------------------------+
1 row in set (0.00 sec)
```

2016 年有一篇很有影响的 uber 的为什么放弃 pg 的文章。
他们最终的选择是在 mysql 上加一层。具体可见
https://eng.uber.com/postgres-to-mysql-migration/

国内广泛地使用 mysql, 主要是历史惯性，积攒下了大量的优秀 mysql
运维人才。同时，近几个版本 mysql 的效率和特性都有很大改善。

不管怎么说，在应用原型阶段和中小应用中，pg 仍不失为一种优秀的选择。
至多用 ORM 保持数据库独立，需要的时候做次迁移就行了。
