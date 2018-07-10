---
layout: post
title: "ubuntu搭建lamp"
date: 2018-07-06 02:25:06 -0700
comments: false
---

ubuntu搭建lamp
=============

```
1、sudo vi /etc/network/interfaces
//添加以下内容
auto eth0 #设置自动启动eth0接口
iface eth0 inet static #配置静态IP
address 192.168.1.10 $IP地址
netmask 255.255.255.0 #子网掩码
gateway 192.168.1.1 #默认网关
2、重启网络
sudo /etc/init.d/networking restart
ifconfig

sudo lsb_release -a #查看ubuntu版本
uname -r #查看ubuntu内核

修改在线apt-get镜像服务器 
sudo vi /etc/apt/sources.list 
将原有 http://us.archive.ubuntu.com/ 改为国内 http://mirrors.163.com

:%s/us.archive.ubuntu.com/mirrors.163.com/g

ps -e|grep ssh #查看ssh服务
sudo apt-get update
sudo apt-get install apache2 php5 mysql-server php5-mysql
sudo apt-get install openssh-server #安装ssh服务

sudo apt-get install php5-gd curl libcurl3 libcurl3-dev php5-curl#安装php扩展包gd、curl

cat /etc/php5/conf.d/mysql.ini #确认成功安装mysql扩展

安装 Mcrypt 包, 安装 php 开发环境
sudo apt-get install php5-mcrypt php5-dev
将配置文件链接给 PHP
sudo ln -s /etc/php5/conf.d/mcrypt.ini /etc/php5/mods-available
开启 Mcrypt 模块
sudo php5enmod mcrypt
sudo service apache2 restart

配置dns(此配置重启后会重置)
1、sudo /etc/resolv.conf #添加nameserver 192.168.1.1
2、sudo /etc/init.d/networking restart
3、ping www.baidu.com #测试是否成功

配置永久dsn

sudo vi /etc/resolvconf/resolv.conf.d/base，添加内容

nameserver 8.8.8.8 #谷歌dns

nameserver 8.8.4.4

保存后执行sudo resolvconf -u

查看是否生效 cat /etc/resolv.conf

配置mysql远程访问
sudo vi /etc/mysql/my.cnf #将bind-address注释

配置apache2
/etc/apache2/apache2.conf #核心配置文件
mods* #apache模块,available可用的；enabled已启用
sites* #虚拟主机

mysql核心配置文件 /etc/mysql/my.cnf
php核心配置文件 /etc/php5/apache2/php.ini
```

ubuntu虚拟主机配置
=================

```
1、mkdir /wwwroot/{video,bbs,oa} rm删除文件 rmdir删除目录
2、/etc/apache2/sites-available下default是默认配置文件

<VirtualHost *:80>
ServerAdmin webmaster@localhost
ServerName video.imooc.com
DocumentRoot /wwwroot/video
<Directory />
Options FollowSymLinks
AllowOverride None
</Directory>
<Directory /wwwroot/video/>
Options Indexes FollowSymLinks MultiViews
AllowOverride None
order allow,deny
allow from all
</Directory>
ScriptAlias /cgi-bin/ /usr/lib/cgi-bin/
<Directory "/usr/lib/cgi-bin">
AllowOverride None
Options +ExecCGI -MultiViews +SymLinksIfOwnerMatch
order allow,deny
Allow from all
</Directory>
ErrorLog ${APACHE_LOG_DIR}/error.log
# Possible values include: debug, info, notice, warn, error, crit,
# alert, emerg.
LogLevel warn
CustomLog ${APACHE_LOG_DIR}/access.log combined
Alias /doc/ "/usr/share/doc/"
<Directory "/usr/share/doc/">
Options Indexes MultiViews FollowSymLinks
AllowOverride None
order deny,allow
Deny from all
Allow from 127.0.0.0/255.0.0.0 ::1/128
</Directory>
</VirtualHost>


3、/etc/apache2/sites-enabled下建立软链接 
sudo ln -s ../sites-available/video video
sudo ln -s ../sites-available/bbs bbs
sudo ln -s ../sites-available/oa oa
4、重启sudo service apache2 restart
```

mysql数据存储目录的迁移
====================

```
1、sudo service mysql stop
2、mysql数据存储目录在/var/lib/mysql,需迁移到的目录必须与此mysql目录具有相同的所属用户与用户组及权限
sudo mkdir mysqldata #建立迁移目录
chown -vR mysql:mysql mysqldata #changed ownership of `mysqldata' from root:root to mysql:mysql
chmod -vR 700 mysqldata #mode of `mysqldata' changed from 0755 (rwxr-xr-x) to 0700 (rwx------)
3、以root身份备份数据到mysqldata 
cp -av /var/lib/mysql/* /var/lib/mysqldata
4、sudo vi /etc/mysql/my.cnf #修改datadir配置项为新的迁移目录
5、ubuntu apparmor安全应用修改
sudo vi /etc/apparmor.d/usr.sbin.mysqld
将此两个配置修改为迁移目录
/var/lib/mysql/ r,
/var/lib/mysql/** rwk,
6、sudo service mysql start 启动mysql
```

ubuntu14.04.2配置基于端口的虚拟主机
================================

```
/etc/apache2目录的ports.conf添加

Listen 8080

sudo a2ensite video
```

ubuntu14.04.2配置FTP服务
=======================

```python
sudo apt-get install vsftpd

useradd -d /var/ftp -s /usr/sbin/nologin username /#设置仅做ftp登录的账号，访问目录/var/ftp
passwd username #修改ftp登录用户密码

chown -R username:username /var/ftp #设置ftp目录所属
#配置/etc/vsftpd.conf
#service vsftpd restart
#非常重点：/etc/pam.d目录下，将vsftpd文件内容全部注释
#或将vsftpd.conf的配置pam_service_name=vsftpd注释，否则出现530拒绝错误
```

安装tomcat
==========

```
vim /etc/profile	#配置环境变量
export JAVA_HOME=/usr/lib/jdk/jdk1.8
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=.:${JAVA_HOME}/bin:$PATH
```