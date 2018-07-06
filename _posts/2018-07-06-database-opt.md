---
layout: post
title: "数据库设计规范"
date: 2018-07-06 03:25:06 -0700
comments: false
---

数据库设计规范

关系数据库:mysql、SQLServer、Oracle、PgSQL

NoSQL系统：Mongo、Memcache、Redis

数据库设计：

 1. 减少数据冗余
 1. 避免数据维护异常
 1. 节约存储空间
 1. 高效的访问


需求分析->逻辑设计->物理设计->维护优化

1. 实体与实体之间的关系
2. 实体包含的属性
3. 选择属性或属性的组合来唯一标识一个实体

第一范式：数据库中的所有字段都是单一属性，不可再分，这个单一属性是由基本的数据类型所构成，如整数，浮点数，字符串。如果一个关系的每个字段都只含有原子值，而不是一个表或集合，那么称这个关系为第一范式（1NF）。


第二范式：
数据库的表中不存在非关键字段对任一侯选字段的部分函数依赖。
部分函数依赖是指存在着组合关键字中的某一关键字决定非关键字的情况。

要求表中的每一行，所有属性都完全依赖于这一行的主键，换句话说，一行下有唯一主键标识，就可以说满足了第二范式。

举例说明:

商名名称	供应商名称	价格	描述		重量	﻿	供应商电话	﻿有效期﻿	﻿分类

﻿
ipad	apple		2999	 	250g	400-025-123	2014.12	饮料


ipad	Microsoft	1999	 	﻿250g﻿	800-025-321	2014.9	饮料

供应商与商品之间是多对多关系，以上例（商口名称与供应商名称）组合关键字才能唯一标识一件商品。

上表存在以下的部分函数依赖关系

(商品名称) - > (价格，描述，重量，商品有效期)

(供应商名称) - > (供应商电话)

 

第三范式：

如果数据表中不存在非关键字段对任意候选关键字段的传递函数依赖则符合第三范式。

 

商名名称	价格	描述	重量	﻿有效期﻿	﻿分类﻿	分类描述

ipad	2999	 	250g	2014.12	电子产品	日常应用

ipad	1999	 	﻿250g﻿	2014.9	 移动设备	移动办公

﻿(商品名称) -> (分类) -> (分类描述)，即非关键字段"分类描述"对关键字段"商品名称"的传递函数依赖


BC范式(BCNF)

数据库表中如果不存在任何字段对任一候选关键字段的传递函数依赖，即如果是复合关键字，则复合关键字之间也不能存在函数依赖关系。
 

供应商	商品ID	供应商联系人	商品数量
联想	1	张三	10
IBM	2	李四	20
 假定'候选主键字段'的决定关系为：

(供应商，商品ID) -> (联系人，商品数量)

(联系人，商品ID) -> (供应商，商品数量)

则存在如下的依赖关系：

(供应商) -> ( 供应商联系人)

(供应商联系人) -> (供应商)

Mysql优化
========

Mysql数据库的优化技术

1、表的设计合理化（3NF范式）

1NF即表的列属性具有原子性，不可再分解,关系型数据库自动满足1NF要求

2NF即表中的记录是唯一的，以主键设计来确定

3NF,表中字段不能有冗余，

2、设计适当的索引(index)，普通索引、主键索引primary、唯一索引unique、全文索引、空间索引、复合索引

3、分表技术（水平、垂直分割）

水平分割：以多个结构相同的多表存储数据，关键是确定分表的标准

垂直分割：提取表中的某些字段（一般为数据量大的），单独存放在另一个表中

4、读写分离

原理：MasterDatabase(主数据库，负责数据的增删改操作，即写)与多台slaveDatabase(从数据库，负责查询，即读)，主数据库定时同步从数据库，利用负载均衡技术实现。

5、存储过程，优点：效率高，模块化。缺点：可移植性差

6、mysql配置优化（最大并发数、缓存大小）

利用MySQL的控制台定位慢查询
=========================

默认MYSQL不会记录查询，可以以mysqld --safe-mode --slow-query-log 安全模式[日志、回滚等功能]启动，

日志慢查询记录log存放目录，可在my.ini配置datadir//对应于mysql5.5版本

show variables like 'long_query_time'; //查看慢查询参数

set long_query_time = 1; //修改慢查询参数(秒)，默认为10秒

show status like 'slow_queries';

7、mysql服务器硬件升级(超过4G内存，推荐使用64操作系统与64位Mysql)

8、定期清理数据垃圾，进行碎片整理(针对MyISAM搜索引擎)

 

SQL优化之索引
===========

利用MySQL的控制台定位慢查询

默认MYSQL不会记录慢查询日志，可以以mysqld --safe-mode --slow-query-log 安全模式[日志、回滚等功能]启动，

日志慢查询记录log存放目录，可在my.ini配置datadir//对应于mysql5.5版本

show variables like 'long_query_time'; //查看慢查询参数

set long_query_time = 1; //修改慢查询参数(秒)，默认为10秒

show status like 'slow_queries';

索引

1、主键索引 alter tableName add primary key(id);

2、普通索引 create index indexName on tableName(id);

3、全文索引 FULLTEXT 只对MYISAM存储引擎

SQL代码

	//fulltext索引只针对MYISAM   
	//只对英文有效   
	create talbe articles(   
	    id int unsigned auto_increment not null primary key,   
	    title varchar(200),   
	    body text,   
	    FULLTEXT(title,body)   
	)engine = MyISAM charset utf8;   
   
select * from 表名 where match(全文索引字段名) against('查询关键字');   
4、唯一索引,unique可以有多个null值

create unique index indexName on tableName(fieldName);

查询索引

desc tableName;

show index from tableName;

show keys from tableName;

删除索引

alter table 表名 drop index 索引名；

alter table 表名 drop primary key;

创建索引的选择建议：

a、在where查询条件中经常使用

b、该字段的内容不容易重复

c、该字段的内容不经常频繁的修改变动

索引的使用及注意事项

explain命令可以显示的SQL预执行的详细信息，利于分析优化SQL

explain select * from dept where name = '515ai'\G

1、创建多列复合索引，只要查询条件使用最左边的索引，索引生效

2、模糊查询,like '%515ai'不会使用索引，like '515ai%''则可以使用索引

3、条件or,要求使用查询的所有字段都必须建立索引，索引才会生效

查询索引的使用率 show status like 'handler_read%';

存储引擎的选择

![存储引擎]({{ site.baseurl}}/images/mysql/engine.jpg)

MyISAM：以查询和添加为主，对事务处理要求不高

Innodb：对事务处理要求高，保存数据重要，

memory：数据变化频繁，入库较少，同时经常查询与修改，速度快

MyISAM的定期优化：optimize table tableName