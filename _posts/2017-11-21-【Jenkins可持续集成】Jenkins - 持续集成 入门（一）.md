---
layout: post
title: "【Jenkins可持续集成】Jenkins - 持续集成 入门（一）"
date: 2017-11-21 
description: "Jenkins是基于Java开发的一种持续集成工具，用于监控持续重复的工作"
categories: Jenkins
--- 

  

**Jenkins是基于Java开发的一种持续集成工具，用于监控持续重复的工作，功能包括：**
* 持续的软件版本发布/测试项目。

* 监控外部调用执行的工作。
------------------------
### 安装：
>官网 ： https://jenkins.io/index.html

[![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/51371bd1-320c-4edd-9433-75a9d56625a9.png)](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/51371bd1-320c-4edd-9433-75a9d56625a9.png)

-------------------------------------------------------------------------------------------------------

**当然还有对应不同的容器的 我们就下载war并且部署在tomcat中**

![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/adc67354-ffd5-4da7-87a2-d82e87ec3d2a.png)

**有时候运行时间长的生活会抛异常 ： PreGerm space 内存移除  我们在启动tomcat的时候加上jvmm参数
为``%T0OMCAT_HOME%/bin`` 文件夹下的 catalina.bat (linux  是.sh)**

![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/792c36f6-1156-44f9-8529-0b1244474d0e.png)
>本文直接使用war包安装 下载地址：https://jenkins-ci.org/content/thank-you-downloading-windows-installer/
-------------------------------------------------------------------------------------------------------
**打开 加入**

```
set "Java_OPTS=-Xms512m -Xmx768m -XX:MaxNewSize=256m -XX:MaxPermSize=128m"
```

**在conf/server.xml加入 容器编码  URIEncoding="UTF-8"**

![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/0.882891870219132.png)

#### 启动 tomcat  
-------------------------------------------------------------------------------------------------------
**启动完毕 访问 项目 就可以继续了 下面我用linux演示**
![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/1bcdc6ce-2c80-4c27-b53d-90cb33607167.png)
#### 启动日志
![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/c8a1cf8c-6fe3-433b-ac0c-d96617479253.png)
** 注意我画圈的 **

**访问的时候需要一个password  位置他会提示你**
![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/3563e9df-c20d-440d-9d6d-935d800cde6d.png)
**我们把那个文件的值复制出来  我的是：**

``
f5f935ee1a7a49e78511fbb51a048071
``
** 继续;
![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/d3d7aa70-65eb-43bb-a902-77e7cb474ad0.jpg)**
**我选建议安装的插件安装**
-------------------------------------------------------------------------------------------------------
**这些都是插件  同时后台开始下载我们等着就行**
![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/dd65eb05-8bcc-4365-9a16-6ba3cf70ce96.png)
#### 开始创建用户
![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/79ec3fbd-1cac-4d4b-a303-45716411db9b.png)
####  安装完成

 [![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/abe506a0-ba4c-4114-b535-0f107a5e8428.png)](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/abe506a0-ba4c-4114-b535-0f107a5e8428.png)
-------------------------------------------------------------------------------------------------------
**注意：**
**  如果你忘记了用户的密码 那么请到位置  /root/.jenkins/config.xml  删除我圈出来的内容  重启tomcat 直接访问**
 ![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/2448e422-6a28-4517-97cb-63f3c388e31d.png)
 ##### 开始全局设置 包括jdk mavan ant之类的配置环境
[![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/5d829494-d1f2-4eb1-918d-66aee782e93c.png)](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/5d829494-d1f2-4eb1-918d-66aee782e93c.png)
-------------------------------------------------------------------------------------------------------
##### 配置jdk
[![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/680f01ce-1f6a-434f-8535-f8b626706d9f.png)](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/680f01ce-1f6a-434f-8535-f8b626706d9f.png)

-------------------------------------------------------------------------------------------------------
##### 安装maven
[![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/e3bd7693-18b4-4654-a6ae-335240bb2b6a.png)](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/e3bd7693-18b4-4654-a6ae-335240bb2b6a.png)
-------------------------------------------------------------------------------------------------------
**接下来我们要自动部署任务把 ，在部署maven项目前  需要安装maven的插件  ：Maven Integration plugin **
[![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/a174f417-9f14-47c7-a0ea-4dc31162506c.png)](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/a174f417-9f14-47c7-a0ea-4dc31162506c.png)

--------------------------------------------------

#####  新建一个JOB
  不安装那个插件是没有maven项目构建的
![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/8b5266b4-ccae-493a-bdd2-65ba43b450d7.png)
[![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/d6ce777e-f702-469d-83ab-cd402e3c1542.png)](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/d6ce777e-f702-469d-83ab-cd402e3c1542.png)
-------------------------------------------------------------------------------------------------------
** 在这里选Git或者svn都可以 我选GIT 我去git。osc找一个私有项目  出现如下现象**
[![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/8db9ca02-7bb5-4b4c-8323-e8fbdcc9cb01.png)](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/8db9ca02-7bb5-4b4c-8323-e8fbdcc9cb01.png)
-------------------------------------------------------------------------------------------------------
**说明不能链接仓库 引文我们没有经过认证 我们填写认证  点击add**
[![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/ef6b1e01-3885-4b1e-93df-924626be9a48.png)](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/ef6b1e01-3885-4b1e-93df-924626be9a48.png)
-------------------------------------------------------------------------------------------------------
**填写账号密码**
 ![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/f153b4c9-c1ce-4514-96df-4a0cac17ea14.png) 

**这里我们采用shh链接 , 首先生成ssh密钥 **

```shell
[root@localhost .ssh]# ssh-keygen -t rsa -C "15550407334@163.com"
```

在

```shell
/root/.ssh
```

生成

 我们 复制 id_rsa.pub（公钥）的内容
![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/52f7944d-b3d7-4469-8294-05dc184e5fe9.png)
打开 git。osc 添加个公钥 
便签为
---------------------------------------------------------------------------------------------------------
**15550407334@163.com**
--------------------------------------------------------------------------------------------------------

** 公钥为 xxxxxxxxxxxxxxxxxxxxxxxxxx 复制的内容  保存**

** 在回到Jenkins中  添加认证**

** 用户名就是刚刚那个生成公钥的邮箱**
![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/ca7c5327-0c9d-41f0-89b8-0a57f6c71bc2.png)

----------------------------------------------------------------------------------------------------------------------
> kind选shh 用户私钥链接

**这时候不报错就是连接上了**
 [![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/6d512611-8dc5-4b4d-b5fd-f7948789d3e9.png)](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/6d512611-8dc5-4b4d-b5fd-f7948789d3e9.png)
------------------------------------------------------------------------------------------------------
**下面开始构建 构建后的部署**
[![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/d8ffd124-d7df-44db-85fb-20740403cf62.png)](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/d8ffd124-d7df-44db-85fb-20740403cf62.png)
---------------------------------------------------------------------------------------------------------
**选中部署war 到一个容器**
![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/clip_image0019d2dce2d-430e-4c26-996f-13399724a6cf.png)
[![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/clip_image0019d2dce2d-430e-4c26-996f-13399724a6cf.png)](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/clip_image0019d2dce2d-430e-4c26-996f-13399724a6cf.png)
-------------------------------------------------------------------------------------------------------------------
** 远程部署 在tomcat添加角色用户用来部署**
** 打开tomcat7下的``~/conf/tomcat-users.xml``文件,关于用户角色、管理员的信息都在这个配置文件中。   在配置文件``omcat-users>``下添加如下xml**

```xml
<role rolename="manager-gui"/> 
<role rolename="manager-script"/>
<role rolename="manager-jmx"/> 
<role rolename="manager-status"/>
<user username="admin" password="admin" roles="manager-gui,manager-script,manager-jmx,manager-status"/>
```

**完成后启动tomcat**

** 回到Jenkins 点击开始**
** 这时候看到我们正在从git下载项目**

![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/67448c95-ae15-498e-a4a6-36efceeadf1b.png)

![](http://4315e09a.wiz03.com/share/resources/a7f5677b-7a58-4e2c-a8eb-61d69ea507f0/index_files/0.7941991102388417.png)
**下载的位置是 ： /root/.jenkins/workspace/test**
** 下载完后会自动构建项目并发布到指定的容器**
![](http://4315e09a.wiz03.com/share/resources/a24925ba-9e8f-4e24-bec1-643b4016d1e4/index_files/6f405743-a1fd-4467-8d88-08c4a0f0e936.png)

--------------------------------------------------------------------------------------------------------------------
------------------------------------------------------------------------------------------------------------------------
** 到此结束，只是作为研究使用，有什么疑问请联系我！**
