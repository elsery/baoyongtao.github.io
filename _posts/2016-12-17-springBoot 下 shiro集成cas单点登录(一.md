---
layout: post
title: "springBoot 下 shiro集成cas单点登录(一"
date: 2016-12-17
description: "java,cas,sso,shiro"
tag: java,shiro,cas
--- 

>  单点登录（Single Sign On , 简称 SSO ）是目前比较流行的服务于企业业务整合的解决方案之一， SSO 使得在多个应用系统中，用户只需要登录一次就可以访问所有相互信任的应用系统。CAS(Central Authentication Service)是一款不错的针对 Web 应用的单点登录框架，本文介绍了 CAS 的原理、协议、在 Tomcat 中的配置和使用，对于采用 CAS 实现轻量级单点登录解决方案的入门读者具有一定指导作用。

## CAS 介绍

CAS 是 Yale 大学发起的一个开源项目，旨在为 Web 应用系统提供一种可靠的单点登录方法，CAS 在 2004 年 12 月正式成为 JA-SIG 的一个项目。CAS 具有以下特点：

- 开源的企业级单点登录解决方案。
- CAS Server 为需要独立部署的 Web 应用。
- CAS Client 支持非常多的客户端(这里指单点登录系统中的各个 Web 应用)，包括 Java, .Net, PHP, Perl, Apache, uPortal, Ruby 等。

### CAS 原理和协议

从结构上看，CAS 包含两个部分： CAS Server 和 CAS Client。CAS Server 需要独立部署，主要负责对用户的认证工作；CAS Client 负责处理对客户端受保护资源的访问请求，需要登录时，重定向到 CAS Server。图1 是 CAS 最基本的协议过程：

##### 图 1. CAS 基础协议
![CAS 基础协议](http://4315e09a.wiz03.com/share/resources/0d05ee77-f6f9-4d9a-91b0-0fda65647310/index_files/0.0828712994698435.png)
CAS Client 与受保护的客户端应用部署在一起，以 Filter 方式保护受保护的资源。对于访问受保护资源的每个 Web 请求，CAS Client 会分析该请求的 Http 请求中是否包含 Service Ticket，如果没有，则说明当前用户尚未登录，于是将请求重定向到指定好的 CAS Server 登录地址，并传递 Service （也就是要访问的目的资源地址），以便登录成功过后转回该地址。用户在第 3 步中输入认证信息，如果登录成功，CAS Server 随机产生一个相当长度、唯一、不可伪造的 Service Ticket，并缓存以待将来验证，之后系统自动重定向到 Service 所在地址，并为客户端浏览器设置一个 Ticket Granted Cookie（TGC），CAS Client 在拿到 Service 和新产生的 Ticket 过后，在第 5，6 步中与 CAS Server 进行身份合适，以确保 Service Ticket 的合法性。

在该协议中，所有与 CAS 的交互均采用 SSL 协议，确保，ST 和 TGC 的安全性。协议工作过程中会有 2 次重定向的过程，但是 CAS Client 与 CAS Server 之间进行 Ticket 验证的过程对于用户是透明的。

另外，CAS 协议中还提供了 Proxy （代理）模式，以适应更加高级、复杂的应用场景，具体介绍可以参考 CAS 官方网站上的相关文档。

### 一)[准备工作]

[本文中的例子以 tomcat7, cas4.0.0 为例进行讲解]

- [CAS Server版本：cas-server-4.0.0]
- [CAS Client版本：cas-client-3.1.12]

>  CAS 官网 : [https://www.apereo.org/](https://www.apereo.org/)
> 
> 
> 
> 
> 
> 项目主页 : [https://www.apereo.org/projects/cas](https://www.apereo.org/projects/cas)
> 
> 
> 
> 
> 
> 下载主页:[https://www.apereo.org/projects/cas/download-cas](https://www.apereo.org/projects/cas/download-cas)

## 二)[创建证书](http://www.ibm.com/developerworks/cn/opensource/os-cn-cas/index.html)

>  证书是单点登录认证系统中很重要的一把钥匙，客户端于服务器的交互安全靠的就是证书；本教程由于是演示所以就自己用JDK自带的keytool工具生成证书；如果以后真正在产品环境中使用肯定要去证书提供商去购买，证书认证一般都是由VeriSign认证，中文官方网站：[http://www.verisign.com/cn/](http://www.verisign.com/cn/)

用JDK自带的keytool工具生成证书：

    

    
    keytool -genkey -alias wsria -keyalg RSA -keystore d:/keys/wsriakey
    
    

wsria :证书别名

d:/keys/wsriakey :生成位置

借用网上证书解释![用keytool生成证书](http://4315e09a.wiz03.com/share/resources/0d05ee77-f6f9-4d9a-91b0-0fda65647310/index_files/0.42969394382089376.png)
[https://www.apereo.org/](https://www.apereo.org/)
注意!注意!注意!!第一项 您的名字与姓氏 一定写上一会要用得host 域名 ,记住了下面我们就靠这个了[http://www.ja-sig.org/downloads/cas/cas-server-3.1.1-release.zip](http://www.ja-sig.org/downloads/cas/cas-server-3.1.1-release.zip)

 

修改  C:\Windows\System32\drivers\etc\hosts  添加本地域映射

127.0.0.1    sso.myhost.org  ###这里得sso.myhost.org是我配置得就是名字和姓氏

这样在访问sso.myhost.org的时候其实是访问的127.0.0.1也就是本机

**严重提醒**：提示输入域名的时候不能输入IP地址 (localhost ,127.0.0.1,192.168.xx.xx都不可以)

## 三)、导出证书

D:\keys>keytool -export -file d:/keys/wsria.crt -alias wsria -keystore d:/keys/wsriakey

导出得证书wsriakey 导出为  : wsria.crt

**特别提示：**如果提示：(就是密码不正确得时候)

    keytool error: java.io.IOException: Keystore was tampered with, or password was incorrect

那么请输入密码：**changeit**

![用keytool导出证书](http://4315e09a.wiz03.com/share/resources/0d05ee77-f6f9-4d9a-91b0-0fda65647310/index_files/0.28842349094338715.png)**
**

至此导出证书完成，可以分发给应用的JDK使用了，接下来讲解客户端的JVM怎么导入证书。

## 四)、为客户端的JVM导入证书

导入得目录一定是jre得 不要选中jdk得

    keytool -import -keystore D:\tools\jdk\1.6\jdk1.6.0_20\jre\lib\security\cacerts -file D:/keys/wsria.crt -alias wsria

网上得图看看

![用keytool导出证书](http://4315e09a.wiz03.com/share/resources/0d05ee77-f6f9-4d9a-91b0-0fda65647310/index_files/0.36022721068002284.png)

可以打开证书看看

![](http://4315e09a.wiz03.com/share/resources/0d05ee77-f6f9-4d9a-91b0-0fda65647310/index_files/10fef397-ffff-4d70-af73-2edcd1baa5fa.png)

 

## 五)、应用证书到Web服务器-Tomcat 支持HTTPS协议

打开tomcat目录的conf/server.xml文件，开启83和87行的注释代码，并设置keystoreFile、keystorePass修改结果如下：

![](http://4315e09a.wiz03.com/share/resources/0d05ee77-f6f9-4d9a-91b0-0fda65647310/index_files/7249c743-cb92-4d73-840e-4de653787198.png)
 

``` xml   
 <Connector port="8443" protocol="org.apache.coyote.http11.Http11Protocol"
        maxThreads="150"SSLEnabled="true" scheme="https" secure="true"
	    keystoreFile="D:\keys\wsriakey"
		 keystorePass="admin123"
        clientAuth="false" sslProtocol="TLS"/>
```    


keystoreFile 证书位置  
keystorePass 你设置得密码

好了，到此Tomcat的SSL启用完成，现在你可以启动tomcat试一下了，例如tocmat地址：https://sso.myhost.org:8443/ 

![](http://4315e09a.wiz03.com/share/resources/0d05ee77-f6f9-4d9a-91b0-0fda65647310/index_files/e65441b1-1003-4134-b801-890755faf255.png)\

## 六)部署cas-server.war

[![](http://4315e09a.wiz03.com/share/resources/0d05ee77-f6f9-4d9a-91b0-0fda65647310/index_files/30536618.png)](![](/share/resources/0d05ee77-f6f9-4d9a-91b0-0fda65647310/index_files/30536618.png))

改名为 cas.war 放在tomcat里

开启tomcat  访问主页  [https://sso.myhost.org:8443/cas/login](https://sso.myhost.org:8443/cas/login)

![](http://4315e09a.wiz03.com/share/resources/0d05ee77-f6f9-4d9a-91b0-0fda65647310/index_files/ee95ce5a-2e24-40ba-b5d5-16a6866bc039.png)
 选择个语言

下面登陆  在cas 4.0以前默认用户 /密码是:admin/admin

4.0以上为用户 /密码:casuser/Mellon
![](http://4315e09a.wiz03.com/share/resources/0d05ee77-f6f9-4d9a-91b0-0fda65647310/index_files/8c9cc266-7740-4720-9b37-d0ce11097cdd.png)

登陆成功