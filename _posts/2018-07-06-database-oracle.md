---
layout: post
title: "ORACLE入门"
date: 2018-07-06 05:25:06 -0700
comments: false
---

主流数据库介绍

	公司					产品名称
	微软				   	access/sqlserver
	瑞典AB公司				mysql
	IBM					db2/informix
	Sysbase					sysbase
	oracle					oracle
 

1、oracle的安装

sys超级用户，具有sysdba角色，最高权限，默认密码是change_on_install，具有create database权限

system管理操作员，具有sysoper角色，默认密码是manager,无create database权限

2、常用工具及命令

sqlplus、企业管理器、sqlplusw、pl/sql developer

连接conn 用户名/密码@网络服务名[as sysdba/sysoper]

断开disc

修改密码passw

显示当前用户show user

退出exit

文件操作命令

start/@ sql脚本

edit

spool sql脚本 spool off 输出执行结果到指定脚本

显示和设置环境变量 set linesize/set pagesize

oracle用户管理

创建用户create user 用户名 identified by 密码

修改密码password

删除用户drop user 用户名[cascade]

oracle权限与角色讲解：

1、系统权限（注：用户对数据库的相关权限）

2、对象权限（注：用户对其他用户的数据对象操作的权限）

3、预定义角色与自定义角色

指定权限grant 权限 to 用户

(示：grant select on scott.emp to 用户 with grant option)对象权限

(示：grant connect on scott.emp to 用户 with admin option)系统权限

级联收回权限revoke(示：revoke select on scott.emp from 用户)

使用profile管理用户口令

create profile lock_account limit failed_login_attempts 3 password_lock_time 2;

alter user 用户名 profile lock_account

oracle-表查询

查看表结构 desc dept

取消重复行 distinct

处理null值：nvl函数

通配符like：%代表任何多个字符，_代表任何单个字符

in与is null

数据分组-max,min,avg,sum,count

group by用于对查询的结果分组统计

having子句用于限制分组显示结果(示：select avg(sal),max(sal),depno,job from emp group by deptno,job查询各部门各岗位的平均与最大工资

多表查询是指基于两个和两个以上的表或是视图的查询

oracle-表管理

表字段数据类型

字符型char定长 varchar2变长 clob(character large object)字符型大对象

数字型number 示number(5,2)表示2位小数范围-999.99-999.99

日期型date 年月日时分秒timestamp 对date数据类型的扩展,精度高

图片blob 二进制数据

创建表

示：create table student(id number(4),name varchar2(20),sex char(2),birthday date,sal number(7,2));

修改默认日期格式alter session set nls_date_format = 'yyyy-mm-dd'

插入数据 insert into student values(1,lee,'男','12-10月-2013',199.99);

更新数据 update student set sal = sal/2 where sex='男';

删除数据 delete from student;

删除表结构 drop table student;

清空数据，不写日志，效率高 truncate table student;

日志保存点savepoint 回滚rollback to