---
layout: post
title: "MYSQL数据库"
date: 2018-07-06 04:25:06 -0700
comments: false
---


SQL代码

	/etc/mysql/my.cnf  
	[mysql]  
	default-character-set = utf8  
	[mysqld]  
	character-set-server = utf8  
	   
select version();  
select now();  
select user();  
select database();  
  
show warnings; #显示错误  
show create database test;  
create database if not exists t1 character set gbk;  
alter database t1 character set utf8; #数据库修改  
drop database if exists t1; #删除数据库  
set names utf8  
  
数据类型  
整型  tinyint/smallint/mediumint/int/bigint  
浮点数     float/double  
日期  year/time/date/datetime/timestamp  
字符型  char/varchar/tinytext/text/mediumtext/longtext/enum/set  
  
创建数据表  

	show tables [from db_name]  
	show columns from tb1;  
	desc tb1;  
	create table if not exists table_name(column_name data_type,...);  
	insert into table_name [column_name,...] values(val,...);  
  
约束  
数据的完整性和一致性，分为表级约束和列级约束(非空、主键、唯一、默认、外键约束)  
  
主键约束  
primary key  
每张数据表只能存在一个主键  
主键保证记录的唯一性  
主键自动为not null  
  
唯一约束  
可以存在多个唯一约束，可以为空(null)  
  
外键约束foreign key  
show create table user;   #创建数据表的sql  
show indexes from user\G; #显示索引  
举例：  

	create table province( id smallint unsigned primary key auto_increment, pname varchar(20) not null); #父表  
	create table user( id smallint unsigned primary key auto_increment, username varchar(10) not null, pid smallint unsigned,foreign key (pid) references province (id));#子表  
  
修改数据表  

	alter table table_name add|drop col_name column_definition  
修改数据表约束  

	alter table user2 add constraint pk_user2_id primary key(id);  
	alter table user2 add foreign key (pid) references province(id);  
  
插入记录  

	insert into users values (default,'jack','123',25,1);  
	insert users set username='ben',password='456';  
	insert tb1 (username) select username from users where age>30;  
更新记录  

	update users set age = age + 9,sex=0 where id=3;  
删除记录  

	delete from users where id=3;  
  
子查询  
any some all  
in|not in |[not] exists  
  
insert tdb_goods_cates(cate_name) select goods_cate from tdb_goods group by goods_cate;  
连接  
inner join 内连接  
select goods_id,goods_name from tdb_goods as g inner join tdb_goods_cates as c on g.cate_id = c.cate_id;  
  
left join  左外连接 #显示左表的全部记录及右表符合连接条件的记录  
  
right join 右外连接 #显示右表的全部记录及左表符合连接条件的记录  

# [on/where/having查询条件的区别](https://www.cnblogs.com/sky6699/p/5238584.html)
  
多表更新 
 
	update tdb_goods inner join tdb_goods_cates on goods_cate=cate_name set goods_cate = cate_id;  
	  
	create table tdb_goods_brands( brand_id smallint unsigned primary key auto_increment, brand_name varchar(40) not null ) select brand_name from tdb_goods group by brand_name;  
  
多表查询 
 
	select goods_id,goods_name,cate_name,brand_name,goods_price from tdb_goods as g ,tdb_goods_cates as c ,tdb_goods_brands as b where g.cate_id=c.cate_id and g.brand_id = b.brand_id;  
  
多表删除  

	delete t1 from tdb_goods as t1 left join (select goods_id,goods_name from tdb_goods group by goods_name having count(goods_name)>1) as t2 on t1.goods_name=t2.goods_name where t1.goods_id > t2.goods_id;  
  
无限分类表设计 
 
	Create TABLE `tdb_goods_types` (  
	  `type_id` smallint(5) unsigned NOT NULL AUTO_INCREMENT,  
	  `type_name` varchar(20) NOT NULL,  
	  `parent_id` smallint(5) unsigned NOT NULL DEFAULT '0',  
	  PRIMARY KEY (`type_id`)  
	) ENGINE=InnoDB AUTO_INCREMENT=16 DEFAULT CHARSET=utf8  
	  
	select s.type_id 子分类id,s.type_name 子分类,p.type_name 父分类 from tdb_goods_types as s left join tdb_goods_types p on s.parent_id = p.type_id;  
	  
	select p.type_id 父分类id,p.type_name 父分类,count(s.type_name) 子分类数 from tdb_goods_types p left join tdb_goods_types s on p.type_id = s.parent_id group by p.type_name order by p.type_id;  
mysql密码找回

如果忘记了mysql root密码，可按如下方法重置
1、以mysqld --skip-grant-tables方式启动服务
2、mysql直接登录命令台

	>use mysql
	>update user set password=password('123456') where user='root';
	>flush privileges;
3、重新正常启动mysql服务即恢复为新密码

mysql定时备份

数据库的手动备份

cmd控制台： mysqldump -u root -p databaseName[tableName]  > path

 备份恢复 mysql控制台：source  path

定时器定时备份

windows系统的任务定时器，执行bat批处理文件

php.exe c:\myTask.php

PHP代码

<?php
//myTask.php 
date_default_timezone_set('PRC');//设置时区
$bakfilename = date("YmdHis",time());
//备份temp数据库的dept表
$command = "mysqldump -u root -p test dept > d:\\{$bakfilename}";
exec($command);
Linux系统以crontab调用，

增量备份：mysql数据库会以二进制形式，自动把用户对mysql数据库的操作记录到文件，可使用增量备份文件进行数据恢复。

	1、配置my.ini中的[mysqld]添加 log-bin=D:/binlog/mylog
	
	2、mysqlbinlog d:\binlog\mylog.000001可以查看记录日志
	
	mysqlbinlog --stop-datetime[--start-datetime]="2014-3-8 17:46:00" d:\binlog\mylog.000001|mysql -u root -p //以时间点恢复
	
	mysqlbinlog --stop-position[--start-position]="267" d:\binlog\mylog.000001|mysql -u root -p //以位置点恢复

建议：每周以mysqldump做全备份，启用增量备份，配置过期日志过期时间(-expire_logs_days)>=7 day