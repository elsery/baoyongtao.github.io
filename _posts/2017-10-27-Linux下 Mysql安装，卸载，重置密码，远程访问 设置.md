---
layout: post
title: "Linux下 Mysql安装，卸载，重置密码，远程访问 设置"
date: 2017-10-27 
description: "Linux下 Mysql安装，卸载，重置密码，远程访问 设置"
tag: java,Mysql,Linux
--- 

  

## 1、安装
查看有没有安装过：

```shell
  yum list installed mysql*
   rpm -qa | grep mysql*
```

查看有没有安装包：

```shell
yum list mysql*
```

安装mysql客户端：

```shell
 yum install mysql
```

安装mysql 服务器端：

```shell
yum install mysql-server
yum install mysql-devel
```
## 2、启动&&停止
 
##### 1,数据库字符集设置

>mysql配置文件/etc/my.cnf中加入
>```shell
default-character-set=utf8
```
 
启动mysql服务

```shell
   service mysqld start
```
或者

```shell
 /etc/init.d/mysqld start
```
开机启动：

```shell
chkconfig -add mysqld
```

查看开机启动设置是否成功

```shell
chkconfig --list | grep mysql*
[root@hadoop 桌面]# chkconfig --list | grep mysql*
mysqld         	0:关闭	1:关闭	2:启用	3:启用	4:启用	5:启用	6:关闭
[root@hadoop 桌面]# 
```

停止：

```shell
service mysqld stop
```

##### 2、登录
 
创建root管理员（知道密码）：

```shell
  mysqladmin -u root password 123456
```

登录：

```shell
          mysql -u root -p输入密码即可。
```

忘记密码：

```shell
service mysqld stop
mysqld_safe --user=root --skip-grant-tables
mysql -u root;
use mysql;
update user set password=password("new_pass") where user="root";
flush privileges;  
[root@localhost 桌面]# mysqld_safe --user=root --skip-grant-tables
170315 15:14:31 mysqld_safe Logging to '/var/log/mysqld.log'.
170315 15:14:31 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
[root@localhost 桌面]# mysql -u root
Welcome to the MySQL monitor. Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.1.73-log Source distribution
Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A
Database changed
mysql> update user set password=password("root") where user="root";
Query OK, 0 rows affected (0.07 sec)
Rows matched: 3 Changed: 0 Warnings: 0
mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)
mysql> exit
[root@localhost 桌面]#   service mysqld restart
停止 mysqld：                                              [确定]
正在启动 mysqld：                                          [确定]
[root@localhost 桌面]# 
[root@localhost 桌面]#  mysql -uroot -proot
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 7
Server version: 5.1.73-log Source distribution
Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql> 
```
##### 3、远程访问
 
开放防火墙的端口号
> mysql增加权限：mysql库中的user表新增一条记录host为“%”，user为“root”。

##### 4、Linux MySQL的几个重要目录

 
>数据库目录
           /var/lib/mysql/
>配置文件
          /usr/share /mysql（mysql.server命令及配置文件）
>相关命令
           /usr/bin（mysqladmin mysqldump等命令）
>启动脚本
           /etc/rc.d/init.d/（启动脚本文件MySQL的目录）

>卸载mysql

```shell
yum -y remove mysql*
```

如果是rpm安装的话：

```shell
rpm -e mysql
```