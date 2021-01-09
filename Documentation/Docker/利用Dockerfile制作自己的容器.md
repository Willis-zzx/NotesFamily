---
title: 利用Dockerfile制作自己的容器
tags:
  - Docker
categories: 学习笔记
description: >-
  最近接触了Docker，玩一下利用Docker构建容器，这个容器包含了JDK1.8和Tomcat服务器，文章里面的Collector和Monitor的jar包、war包单纯只是为了验证JDK环境、Tomcat服务器是否能正常运行而已，别无他用。整理一下知识笔记，方便下次查阅。
top_img: 'https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201225220042.png'
cover: 'https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201225220042.png'
abbrlink: 12a3
date: 2020-12-25 21:36:29
---

# 利用 Dockerfile 构建自己的镜像

## 第一步——准备工作

在 `/usr/local` 目录下新建 `docker` 文件夹

- 新建 `Dockerfile` 文件
- 将 `JDK1.8` 和 `tomcat8` 的压缩包提取到 `/usr/local/docker` 目录下，并分别更名为 `jdk1.8` 和 `tomcat8`
- 上传`collector`和`monitor`的`war`包、`jar`包，各自所需的配置文件

```bash
cd /usr/local
mkdir docker
touch Dockerfile
```

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201225214010.png)

## 第二步——编写文件

编写 `Dockerfile` 文件

```bash
vim Dockerfile
```

```bash
#指定基础镜像
From ubuntu

#创建者
MAINTAINER zzx

#工作目录
WORKDIR /root

#将宿主机Dockerfile当前目录下的jdk目录下的文件添加至镜像容器的/opt目录下，并改名为jdk
ADD jdk1.8 /opt/jdk

#将宿主机Dockerfile当前目录下的tomcat8目录下的文件添加至镜像的/opt目录下，并改名为tomcat
ADD tomcat8 /opt/tomcat8

#设置JDK环境变量
ENV JAVA_HOME=/opt/jdk
ENV JRE_HOME=$JAVA_HOME/jre
ENV CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV PATH=/sbin:$JAVA_HOME/bin:$PATH

#新建配置文件的文件夹
RUN cd /home
RUN mkdir pi
RUN cd pi
RUN mkdir wsp
RUN cd wsp
RUN mkdir collector
RUN mkdir shell 
RUN mkdir jar
RUN cd ..

#上传配置文件
ADD collector /home/pi/wsp/collector/collector
ADD config /home/pi/config
ADD cd2000init_v3 /home/pi/wsp/cd2000init_v3

#上传Collector的jar包
ADD Collector-all.jar /home/pi/wsp/jar/Collector-all.jar
#上传执行Collectorjar包的脚本
COPY collector.sh /home/pi/wsp/jar/
RUN chmod 777 /home/pi/wsp/jar/collector.sh

#Monitor的war包放在tomcat下的webapps下，不用再上传

#执行collector.sh
RUN cd ~
RUN cd /home/pi/wsp/jar
CMD ["collector.sh","-g","daemon off"]

#设置公开端口
EXPOSE 8080

#设置启动命令
ENTRYPOINT ["/opt/tomcat8/bin/catalina.sh","run"]
```

在`/usr/local/docker`目录下新建一个脚本文件,命名为`collector.sh`

```shell
#!/bin/sh

#jar包文件路径及名称（目录按照各自配置）
APP_NAME=/home/pi/wsp/jar/Collector-all.jar
#查询进程，并杀掉当前jar/java程序

id=`ps -ef|grep $APP_NAME | grep -v grep | awk '{print $2}'`
sudo kill -9 $pid
echo "$pid进程终止成功"

sleep 2

#判断jar包文件是否存在，如果存在启动jar包，并时时查看启动日志

if test -e $APP_NAME
then
echo '文件存在,开始启动此程序...'

# 启动jar包，指向日志文件，2>&1 & 表示打开或指向同一个日志文件
sudo java -jar $APP_NAME 

echo '$APP_NAME 启动成功...'
else
	echo '$APP_NAME 文件不存在,请检查。'
fi
```

## 第三步——构建容器

构建命令格式：

```bash
docker build -t 容器名:版本号 .
```

**PS：版本号后要空一格，在加一个英文的小数点**

在`Dockerfile`的目录下执行如下命令：

```bash
docker build -t docker:0.5 . 
```

终端出现如下信息表示构建成功：

```bash
Sending build context to Docker daemon  589.8MB
Step 1/29 : From ubuntu
 ---> f643c72bc252
Step 2/29 : MAINTAINER zzx
 ---> Using cache
 ---> dc49c0692f34
Step 3/29 : WORKDIR /root
 ---> Using cache
 ---> 380860a682b4
Step 4/29 : ADD jdk1.8 /opt/jdk
 ---> Using cache
 ---> a8f36e2a7f05
Step 5/29 : ADD tomcat8 /opt/tomcat8
 ---> Using cache
 ---> ede1318972d6
Step 6/29 : ENV JAVA_HOME=/opt/jdk
 ---> Using cache
 ---> 95074f69fa5b
Step 7/29 : ENV JRE_HOME=$JAVA_HOME/jre
 ---> Using cache
 ---> c3cfe2b55be3
Step 8/29 : ENV CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
 ---> Using cache
 ---> 2ae0c5a711c8
Step 9/29 : ENV PATH=/sbin:$JAVA_HOME/bin:$PATH
 ---> Using cache
 ---> 0ce51f63d830
Step 10/29 : RUN cd /home
 ---> Using cache
 ---> 81577b7f1b43
Step 11/29 : RUN mkdir pi
 ---> Using cache
 ---> c62b55c4b3e8
Step 12/29 : RUN cd pi
 ---> Using cache
 ---> ba3d8b3bfe2b
Step 13/29 : RUN mkdir wsp
 ---> Using cache
 ---> 3d6b309749c3
Step 14/29 : RUN cd wsp
 ---> Using cache
 ---> c5a24cf47385
Step 15/29 : RUN mkdir collector
 ---> Using cache
 ---> 8cda5d67e04a
Step 16/29 : RUN mkdir shell
 ---> Using cache
 ---> 5e9355a33c4b
Step 17/29 : RUN mkdir jar
 ---> Running in 5448dbd14af2
Removing intermediate container 5448dbd14af2
 ---> a7360732be9b
Step 18/29 : RUN cd ..
 ---> Running in b157b20e2545
Removing intermediate container b157b20e2545
 ---> 00619bdeeb50
Step 19/29 : ADD collector /home/pi/wsp/collector/collector
 ---> 7fe9e028953b
Step 20/29 : ADD config /home/pi/config
 ---> 027dec1998e7
Step 21/29 : ADD cd2000init_v3 /home/pi/wsp/cd2000init_v3
 ---> dab7c54f498f
Step 22/29 : ADD Collector-all.jar /home/pi/wsp/jar/Collector-all.jar
 ---> 928207063cbd
Step 23/29 : COPY collector.sh /home/pi/wsp/jar/
 ---> ecfb89b4a1b8
Step 24/29 : RUN chmod 777 /home/pi/wsp/jar/collector.sh
 ---> Running in 787c7797f7ea
Removing intermediate container 787c7797f7ea
 ---> f00b5f1c162a
Step 25/29 : RUN cd ~
 ---> Running in d29a81008606
Removing intermediate container d29a81008606
 ---> 9e5a5a3552eb
Step 26/29 : RUN cd /home/pi/wsp/jar
 ---> Running in 3d4c33dbb8cf
Removing intermediate container 3d4c33dbb8cf
 ---> c513c756d807
Step 27/29 : CMD ["collector.sh","-g","daemon off"]
 ---> Running in 62b9d172e1c1
Removing intermediate container 62b9d172e1c1
 ---> afe4bbd1ea73
Step 28/29 : EXPOSE 8080
 ---> Running in c99d7142478d
Removing intermediate container c99d7142478d
 ---> 4e38633312ca
Step 29/29 : ENTRYPOINT ["/opt/tomcat/bin/catalina.sh","run"]
 ---> Running in 86263414e6a9
Removing intermediate container 86263414e6a9
 ---> 276db8259e6e
Successfully built 276db8259e6e
Successfully tagged docker:0.5
```

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201225214219.png)

## 第四步——运行容器

```bash
docker run -d -p 10085:8080 docker:0.5
```

第一个`10082`端口为本地主机的端口，第二个`8080`端口为容器的端口

参数说明：

`-d`：后台运行容器，并返回容器ID

`-p`: 指定端口映射，格式为：主机(宿主)端口:容器端口

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201225214527.png)

这里使用后台运行容器，可以使用命令打印运行日志

```
docker container logs [container ID or NAMES]
```

## 第五步——导出容器

导出本地容器命令：

```bash
#先查看容器
docker container ls -a
#再用命令导出
docker export <CONTAINER ID> > hello.tar
```

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201225214818.png)

导出的容器在当前目录下：

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201225214852.png)

## 第六步——导出镜像

```bash
docker  save  -o  tar包的名字  镜像名
```

## 第七步——上传阿里云Docker仓库

```bash
sudo docker login --username=[你的阿里云用户名] registry.cn-shenzhen.aliyuncs.com
sudo docker tag [ImageId] registry.cn-shenzhen.aliyuncs.com/zhouzixin/docker1:[镜像版本号]
sudo docker push registry.cn-shenzhen.aliyuncs.com/zhouzixin/docker1:[镜像版本号]

#registry.cn-shenzhen.aliyuncs.com/zhouzixin/docker1:更改为你的阿里云Docker仓库地址
```

## 第八步——过程中碰见的错误

### Docker镜像源问题，导致拉取失败

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201225214956.png)

- 本身网速太慢，无法下载；
- 在Docker容器中配置的镜像有误，或镜像太水

把`Docker`的镜像源更改为国内源的即可。

步骤：

修改或新建`/etc/docker/daemon.json`文件

```bash
cd /etc/docker         #进入/etc/docker目录下
sudo vim daemon.json   #修改daemon.json
```

更改为：

```
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
```

注意，一定要保证该文件符合 `json` 规范，否则 `Docker` 将不能启动。

之后重新启动服务。

```bash
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

检查加速器是否生效：

执行 `$ docker info`，如果从结果中看到了如下内容，说明配置成功。

```bash
Registry Mirrors:
 https://docker.mirrors.ustc.edu.cn
```

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201225215356.png)

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201225215419.png)

参考链接：

- https://blog.csdn.net/yyj108317/article/details/105875582
- https://www.jianshu.com/p/34d3b4568059


### 端口占用问题

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201225215526.png)

宿主机的`8080`已被占用，`Dockerfile`中换一个。

### Dockerfile 文件配置书写错误

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201225215648.png)

`Dokcerfile`的第一行：`FROM`的`debian`的d需要小写，`debian`后不要加版本号

换成`ubuntu`的也是一样的。

### 小问题

启动容器后，通过`docker ps` 命令看到刚刚启动的容器没在运行，再通过`docker ps -a`命令，看到我们刚刚启动的容器，启动之后就退出了。

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201225215748.png)

接着通过`docker logs -f -t --tail 20 [容器ID]`看一下容器退出的原因

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201225215828.png)

这个错误，主要是由于不兼容引起的。主要包括几种不兼容：

- 硬件架构不兼容。在 `amd` 和 `arm` 架构下构建的镜像很有可能不能互通。

  解决办法：针对不同的硬件架构构建不同的镜像，或者构建跨架构（`multi-arch`）的镜像。

- `shell` 执行不兼容。脚本可能是基于 `bash` 写的，不同的 `shell` 解释器存在不兼容的情况，而有些 `Linux` 发行版可能没有 `bash`，或者默认的 `shell` 解释器不是 `bash`。

  解决办法：在 `shell` 脚本的开头指定需要使用的解释器，比如`#!/bin/bash`，并且注意使用的语法。

- 存在非 `*NIX` 环境的换行符。比如在 `Windows` 环境下编写的一些代码，移植到 `Linux` 环境下可能会出问题。

我看了下执行`collector jar`包的`collector.sh`文件，该文件开头的`#!/bin/bash`有错！换成`#!/bin/sh`

### 删除容器与删除镜像

删除容器步骤：（要先停止再删除）

```bash
#查看正在运行的容器
docker ps
#停止正在运行的容器
docker container stop <CONTAINER ID>
#再查看容器是否已经停止了
docker ps
#查看所有容器，包括停止的容器
docker ps -a
#删除已停止的容器
docker container rm <CONTAINER ID>
```

删除镜像步骤：（要先删除使用镜像的容器，再删除镜像）

```bash
#先删除容器
#查看镜像
docker images
#再用删除命令
docker image rm <IMAGE ID>
#先用命令查看所有本地镜像，检查是否删除成功
docker images
#如果出现不同的镜像有相同的IMAGE ID，则用下面的命令删除镜像
docker image rm REPOSITORY:TAG
```

### 构建镜像时命令不对

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201225215929.png)