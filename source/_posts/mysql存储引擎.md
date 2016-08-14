title: mysql存储引擎
date: 2016-05-10 21:22:44
tags:
 mysql
 存储引擎
categories:
 mysql
---

>mysql> select version();
+-----------------+
| version()       |
+-----------------+
| 5.7.12-0ubuntu1 |
+-----------------+

<!-- more-->

**mysql支持的存储引擎**

>mysql> SHOW ENGINES;

| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
| :--:               | :--:    | :--:                                                           | :--:         | :--: | :--:       |
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |


# MyISAM
MyISAM是mysql5.5之前默认的存储引擎，不支持失误，页不支持外键，访问速度快，适用于对事物完整性没有要求，或者以Select,Insert为主的应用。

MyISAM支持3种不同的存储格式：
1. 静态表（字段长度固定）
2. 动态表（字段长度不固定
3. 压缩表

# InnoDB
InnoDB提供了具有，提交，回滚，崩溃能力恢复的事物安全。但是对比MyISAM,InnoDB的处理效率差一些，会占用更多的磁盘空间以保留数据和索引。
## 自动增长列
可以手动插入，如果插入的值为空，或者为0，则实际插入的值为自动增长后的值。自动增长列必须是索引，如果是组合索引，页必须是组合索引的第一列，对于MyISAM，自动增长列可以是组合所应的其他列。

## 外键约束
只有InnoDB
