---
layout: post
title: "Eclipse远程debug最简单教程"
date: 2017-11-06 
description: "Eclipse,debug"
tag: java,Eclipse
--- 

  

## 原理就是在启动tomcat的时候加上jvm参数
>在apache-tomcat-7.xx/bin/catalina.bat(linux 下  .sh)
增加一句话

```xml
SET CATALINA_OPTS=-server -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8000
```

##### 但是很多人不知道在哪个位置加上 现在有一个批处理文件 放在 /bin下就可以了，端口是8787 可以更改，然后debug启动就可以调试了

> bat文件
```xml
cd %CATALINE_HOME%/bin 
set JPDA_ADDRESS=8787 
set JPDA_TRANSPORT=dt_socket 
set CATALINA_OPTS=-server -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8787 
startup
```

![运行](http://4315e09a.wiz03.com/share/resources/d954f0fe-e8a6-465b-a72a-ca38d165a1c8/index_files/8b9ac31e674adf711d9d01cabbccbccb.png "运行")
![图例](http://4315e09a.wiz03.com/share/resources/d954f0fe-e8a6-465b-a72a-ca38d165a1c8/index_files/0a47ca8b4e88b201cf4625271e921dae.png "图例")
