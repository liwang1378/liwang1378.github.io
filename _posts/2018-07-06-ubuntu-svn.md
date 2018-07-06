---
layout: post
title: "ubuntu搭建svn服务并同步网站发布目录"
date: 2018-07-06 03:25:06 -0700
comments: false
---

安装软件包：

sudo apt-get install subversion

之后选择SVN服务文件及配置文件的放置位置。本机安装放在了/srv下的svn目录。

	cd /srv
	
	sudo mkdir svn

本次的svn版本仓库叫liw_src

	cd /srv/svn
	
	sudo mkdir liw_src

目录建好后　创建版本仓库

	sudo svnadmin create /srv/svn/liw_src

执行之后 liw_src下文件结构如下：　　

```
tone@ubuntu:/srv/svn/liw_src$ ls -l
总用量 24
drwxr-xr-x 2 root root 4096 1月 15 10:52 conf
drwxr-sr-x 6 root root 4096 1月 15 14:52 db
-r--r--r-- 1 root root 2 1月 15 10:50 format
drwxr-xr-x 2 root root 4096 1月 15 10:50 hooks
drwxr-xr-x 2 root root 4096 1月 15 10:50 locks
-rw-r--r-- 1 root root 246 1月 15 10:50 README.txt
```

下面进行配置：

我们需要修改conf目录下的三个文件，authz;passwd;svnserve.conf

编辑svnserve.conf

	[general]
	#匿名用户不可读
	anon-access = none
	#权限用户可写
	auth-access = write
	#密码文件为passwd
	password-db = passwd
	#权限文件为authz
	authz-db = authz

编辑authz 制定管理员组 即admin组的用户为liw admin组有rw（读写权限） 所有人有r（读权限）

	[groups]
	admin= liw
	
	[/]
	@admin =rw
	*=r

这里组的名字 不一定叫admin 你的管理员组名 可以叫做任意的名字，另外比如admin组还有其他用户，可以这样制定 admin=tone，tone1,tone2 类似这样的写法

编制passwd 文件 设定用户密码

	[users]
	# harry = harryssecret
	# sally = sallyssecret
	liw=123456

liw的密码为123456  明文的。

以上都做完之后，就可以开启你的svn服务器了。

	sudo svnserve -d -r /srv/svn/

-d 已守护模式启动

-r 制定svn版本库根目录 这样是便于客户端不用输入全路径 就可以访问版本库了

例如：svn：//127.0.0.1/liw_src

加入svn服务开机启动

在init.d目录建立一个脚本文件svnd.sh


	# cd /etc/init.d
	# vim svnd.sh


输入svnd.sh内容如下（/srv/svn 为svn仓库目录）：
	
	#!/bin/bash
	#svnserve startup
	svnserve -d -r /srv/svn

保存退出。
更新，修改权限：

	#update-rc.d svnd.sh defaults
	#chmod 777 svnd.sh

 

SVN同步版本库与网站目录

定义:
SVN版本库 = /srv/svn/liw_src
网站目录 = /var/www/html
1.检出一个项目到网站目录

	#svn checkout file:///srv/svn/liw_src /var/www/html
网站目录已成为SVN的工作副本，接下来要做的就是让这个工作副本自动更新。

3.增加hooks(钩子)文件

	vi /srv/svn/liw_src/hooks/post-commit ，内容如下
	#!/bin/sh
	export LANG=en_US.UTF-8#防止乱码
	svn update /var/www/html --username liw --password 123456 --no-auth-cache
保存后修改文件权限为777，否则SVN无法调用执行

	#chmod 777 /srv/svn/liw_src/hooks/post-commit