---
layout: post
title: "使用 Docker 搭建 Java Web 运行环境"
date: 2017-11-09 
description: "Docker,Java"
tag: Docker,Java
--- 

  

> Docker 是 2014 年最为火爆的技术之一，几乎所有的程序员都听说过它。Docker 是一种“轻量级”容器技术，它几乎动摇了传统虚拟化技术的地位，现在国内外已经有越来越多的公司开始逐步使用 Docker 来替换现有的虚拟化平台了。作为一名 Java 程序员，我们是时候一起把 Docker 学起来了！

###### 本文会对虚拟化技术与 Docker 容器技术做一个对比，然后引出一些 Docker 的名词术语，比如：容器、镜像等，随后将使用 Docker 搭建一个 Java Web 运行环境，最后将对本文做一个总结。


**我们先来回顾一下传统虚拟化技术的体系架构：**

![](http://4315e09a.wiz03.com/share/resources/059972e8-c7b4-4407-af11-bce448321d08/index_files/1dd93314-b6a2-43a0-9097-5bc571444b4d.png)

**可见，我们在宿主机的操作系统上，可安装了多个虚拟机，而在每个虚拟机中，通过虚拟化技术，实现了一个虚拟操作系统，随后，就可以在该虚拟操作系统上，安装自己所需的应用程序了。这一切看似非常简单，但其中的技术细节是相当高深莫测的，大神级人物都不一定说得清楚。
凡是使用过虚拟机的同学，应该都知道，启动虚拟机就像启动一台计算机，初始化过程是相当慢的，我们需要等很久，才能看到登录界面。一旦虚拟机启动以后，就可以与宿主机建立网络连接，确保虚拟机与宿主机之间是互联互通的。不同的虚拟机之间却是相互隔离的，也就是说，彼此并不知道对方的存在，但每个虚拟机占用的都是宿主机的硬件与网络资源。**

> 我们再来对比一下 Docker 技术的体系架构吧：

![](http://4315e09a.wiz03.com/share/resources/059972e8-c7b4-4407-af11-bce448321d08/index_files/5591d92d-f6fd-46db-beeb-9c4453b22b26.png)

**可见，在宿主机的操作系统上，有一个 Docker 服务在运行（或者称为“Docker 引擎”），在此服务上，我们可开启多个 Docker 容器，而每个 Docker 容器中可运行自己所需的应用程序，Docker 容器之间也是相互隔离的，同样地，都是占用的宿主机的硬件与网络资源。**

**Docker 容器相对于虚拟机而言，除了在技术实现上完全不一样以外，启动速度较虚拟机而言有本质的飞跃，启动一个容器只在眨眼瞬间。不管是虚拟机还是 Docker 容器，它们都是为了隔离应用程序的运行环境，节省我们的硬件资源，为我们开发人员提供福利。**

> 我们再来看看 Docker 的 Logo 吧：

![](http://4315e09a.wiz03.com/share/resources/059972e8-c7b4-4407-af11-bce448321d08/index_files/91c5109e-bf61-4d04-82d5-2822f23beb8a.png)

**很明显，这是一只鲸鱼，它托着许多集装箱。我们可以把宿主机可当做这只鲸鱼，把相互隔离的容器可看成集装箱，每个集装箱中都包含自己的应用程序。这 Logo 简直的太形象了！**

- 需要强调的是，笔者并非否定虚拟化技术，而是想通过本文让更多的读者了解如何使用 Docker 技术，让大家知道除了虚拟化技术以外，还有另一种替代技术，也能让应用程序隔离起来。

> 下面，我们将结合一个 Java Web 应用的部署过程，来描述如何“烹饪”Docker 这份美味佳肴。您准备好了吗？我们现在就开始！

- 原料

 -  前提条件

> 首先，您要准备一个 CentOS 的操作系统，虚拟机也行。总之，可以通过 Linux 客户端工具访问到 CentOS 操作系统就行。

**需要说明的是，Ubuntu 或其它 Linux 操作系统也能玩 Docker，只不过本文选择了以 CentOS 为例，仅此而已。**

CentOS 具体要求如下：
> 必须是 64 位操作系统
建议内核在 3.8 以上
通过以下命令查看您的 CentOS 内核：

- uname -r
- 如果执行以上命令后，输出的内核版本号低于 3.8，请参考下面的方法来来升级您的 Linux 内核。

**对于 CentOS 6.5 而言，内核版本默认是 2.6。首先，可通过以下命令安装最新内核：**
```shell
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -ivh http://www.elrepo.org/elrepo-release-6-5.el6.elrepo.noarch.rpm
yum -y --enablerepo=elrepo-kernel install kernel-lt
```
随后，编辑以下配置文件：

```shell
vi /etc/grub.conf
```
**将default=1修改为default=0。**

- 最后，通过reboot命令重启操作系统。
- 重启后如果不出意外的话，再次查看内核，您的 CentOS 内核将会显示为 3.10。
- 如果到这里，您和我们所期望的结果是一致的。恭喜您！下面我们就一起来安装 Docker 了。

##   安装 Docker

只需通过以下命令即可安装 Docker 软件：
```shell
rpm -Uvh http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
yum -y install docker-io
```
##### 可使用以下命令，查看 Docker 是否安装成功：
```shell
docker version
```
**若输出了 Docker 的版本号，则说明安装成功，我们下面就可以开始使用 Docker 了。**

可通过以下命令启动 Docker 服务：

```shell
ervice docker start
```
##### -做法

>就像曾经安装软件一样，我们首先需要有一张刻录了该软件的光盘，如果您使用的是虚拟光驱，那么就需要运行一种名为“镜像”的文件，通过它来安装软件。在 Docker 的世界里，也有一个名为“镜像”的东西，已经安装我们所需的操作系统，我们一般成为“Docker 镜像”，本文简称“镜像”。

#### 那么问题来了，我们从哪里下载镜像呢？

> Docker 官网 确实已经提供了所有的镜像下载地址，可惜在国内却是无法访问的。幸好国内好心人提供了一个Docker 中文网，在该网站上可以下载我们所需的 Docker 镜像。

##### 下载镜像

>我们不妨还是以 CentOS 为例，通过以下步骤，下载一个 CentOS 的镜像。

**首先，访问 Docker 中文网，在首页中搜索名为“centos”的镜像，在搜索的结果中，有一个“官方镜像”，它就是我们所需的。**

**   然后，进入 CentOS 官方镜像页面，在“Pull this repository”输入框中，有一段命令，把它复制下来，在自己的命令行上运行该命令，随后将立即下载该镜像。**

**最后，使用以下命令查看本地所有的镜像：**

```shell
docker images
```
当下载完成后，您应该会看到：

```shell
REPOSITORY TAG            IMAGE ID          CREATED             VIRTUAL           SIZE
docker.cn/docker/centos   centos6           25c5298b1a36      7 weeks ago      215.8 MB
```
**
如果看到以上输出，说明您可以使用“docker.cn/docker/centos”这个镜像了，或将其称为仓库（Repository），该镜像有一个名为“centos6”的标签（Tag），此外还有一个名为“25c5298b1a36 ”的镜像 ID（可能您所看到的镜像 ID 与此处的不一致，那是正常现象，因为这个数字是随机生成的）。此外，我们可以看到该镜像只有 215.8 MB，非常小巧，而不像虚拟机的镜像文件那样庞大。
**
>现在镜像已经有了，我们下面就需要使用该镜像，来启动容器。

#### 启动容器

**容器是在镜像的基础上来运行的，一旦容器启动了，我们就可以登录到容器中，安装自己所需的软件或应用程序。既然镜像已经下载到本地，那么如何才能启动容器呢？**

>只需使用以下命令即可启动容器：

```shell
docker run -i -t -v /root/software/:/mnt/software/ 25c5298b1a36 /bin/bash
```
这条命令比较长，我们稍微分解一下，其实包含以下三个部分：
```shell
docker run <相关参数> <镜像 ID> <初始命令>
```
其中，相关参数包括：
>-i：表示以“交互模式”运行容器
-t：表示容器启动后会进入其命令行
-v：表示需要将本地哪个目录挂载到容器中，格式：-v <宿主机目录>:<容器目录>
假设我们的所有安装程序都放在了宿主机的/root/software/目录下，现在需要将其挂载到容器的/mnt/software/目录下。


**需要说明的是，不一定要使用“镜像 ID”，也可以使用“仓库名:标签名”，例如：docker.cn/docker/centos:centos6。**

- 初始命令表示一旦容器启动，需要运行的命令，此时使用“/bin/bash”，表示什么也不做，只需进入命令行即可。

#### 安装相关软件

**为了搭建 Java Web 运行环境，我们需要安装 JDK 与 Tomcat，下面的过程均在容器内部进行。我们不妨选择/opt/目录作为安装目录，首先需要通过cd /opt/命令进入该目录。**

- 安装 JDK

  - 首先，解压 JDK 程序包：

```shell
tar -zxf /mnt/software/jdk-7u67-linux-x64.tar.gz -C .
```
  - 然后，重命名 JDK 目录：

```shell
mv jdk1.7.0_67/ jdk/
```
  - 安装 Tomcat

  - 首先，解压 Tomcat 程序包：

```shell
tar -zxf /mnt/software/apache-tomcat-7.0.55.tar.gz -C .
```
  - 然后，重命名 Tomcat 目录：

```shell
mv apache-tomcat-7.0.55/ tomcat/
```
  - 设置环境变量

  - 首先，编辑.bashrc文件

```shell
vi ~/.bashrc
```
  - 然后，在该文件末尾添加如下配置：

```shell
export JAVA_HOME=/opt/jdk
export PATH=$PATH:$JAVA_HOME
```
  - 最后，需要使用source命令，让环境变量生效：

```
source ~/.bashrc
```
  - 编写运行脚本

我们需要编写一个运行脚本，当启动容器时，运行该脚本，启动 Tomcat，具体过程如下：

  - 首先，创建运行脚本：

```
vi /root/run.sh
```
  - 然后，编辑脚本内容如下：

```
#!/bin/bash
source ~/.bashrc
sh /opt/tomcat/bin/catalina.sh run
```
  - 注意：这里必须先加载环境变量，然后使用 Tomcat 的运行脚本来启动 Tomcat 服务。

  - 最后，为运行脚本添加执行权限：

```
chmod u+x /root/run.sh
```
  - 退出容器

**当以上步骤全部完成后，可使用exit命令，退出容器。**

**随后，可使用如下命令查看正在运行的容器：**

```
docker ps
```
此时，您应该看不到任何正在运行的程序，因为刚才已经使用exit命令退出的容器，此时容器处于停止状态，可使用如下命令查看所有容器：

```
docker ps -a
```
输出如下内容：
```
CONTAINER ID        IMAGE                             COMMAND             CREATED             STATUS                      PORTS               NAMES
57c312bbaad1        docker.cn/docker/centos:centos6   "/bin/bash"         27 minutes ago      Exited (0) 19 seconds ago                       naughty_goldstine
```
记住以上CONTAINER ID（容器 ID），随后我们将通过该容器，创建一个可运行 Java Web 的镜像。

创建 Java Web 镜像

使用以下命令，根据某个“容器 ID”来创建一个新的“镜像”：

```
docker commit 57c312bbaad1 else/javaweb:0.1
```
该容器的 ID 是“57c312bbaad1”，所创建的镜像名是“huangyong/javaweb:0.1”，随后可使用镜像来启动 Java Web 容器。

- 启动 Java Web 容器

    - 有必要首先使用docker images命令，查看当前所有的镜像：

```
REPOSITORY                TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
huangyong/javaweb         0.1                 fc826a4706af        38 seconds ago      562.8 MB
docker.cn/docker/centos   centos6             25c5298b1a36        7 weeks ago         215.8 MB
```
>可见，此时已经看到了最新创建的镜像**“else/javaweb:0.1”**，其镜像 ID 是**“fc826a4706af”**。正如上面所描述的那样，我们可以通过“镜像名”或“镜像 ID”来启动容器，与上次启动容器不同的是，我们现在不再进入容器的命令行，而是直接启动容器内部的 Tomcat 服务。此时，需要使用以下命令：

```
docker run -d -p 58080:8080 --name javaweb huangyong/javaweb:0.1 /root/run.sh
```
- 稍作解释：

>-d：表示以“守护模式”执行/root/run.sh脚本，此时 Tomcat 控制台不会出现在输出终端上。
-p：表示宿主机与容器的端口映射，此时将容器内部的 8080 端口映射为宿主机的 58080 端口，这样就向外界暴露了 58080 端口，可通过 Docker 网桥来访问容器内部的 8080 端口了。
--name：表示容器名称，用一个有意义的名称命名即可。
关于 Docker 网桥的内容，需要补充说明一下。实际上 Docker 在宿主机与容器之间，搭建了一座网络通信的桥梁，我们可通过宿主机 IP 地址与端口号来映射容器内部的 IP 地址与端口号，

**在一系列参数后面的是“镜像名”或“镜像 ID”，怎么方便就怎么来。最后是“初始命令”，它是上面编写的运行脚本，里面封装了加载环境变量并启动 Tomcat 服务的命令。**

当运行以上命令后，会立即输出一长串“容器 ID”，我们可通过docker ps命令来查看当前正在运行的容器。

```
CONTAINER ID        IMAGE                   COMMAND             CREATED             STATUS              PORTS                     NAMES
82f47923f926        else/javaweb:0.1   "/root/run.sh"      4 seconds ago       Up 3 seconds        0.0.0.0:58080->8080/tcp   javaweb
```
品尝

在浏览器中，输入以下地址，即可访问 Tomcat 首页：


>http://192.168.65.132:58080/

注意：这里使用的是宿主机的 IP 地址，与对外暴露的端口号 58080，它映射容器内部的端口号 8080。

总结

***通过本文，我们了解了 Docker 是什么？它与虚拟机的差别在哪里？以及如何安装 Docker？如何下载 Docker 镜像？如何运行 Docker 容器？如何在容器内安装应用程序？如何在容器上创建镜像？如何以服务的方式启动容器？这一切看似简单，但操作也是相当繁琐的，不过熟能生巧，需要我们不断地操练。***

***除了这种手工生成 Docker 镜像的方式以外，还有一种更像是写代码一样，可以自动地创建 Docker 镜像的方式。只需要我们编写一个 Dockerfile 文件，随后使用docker build命令即可完成以上所有的手工操作。***


