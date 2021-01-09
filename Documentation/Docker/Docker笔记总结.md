---
title: Docker笔记总结
tags:
  - Docker
categories: 学习笔记
description: 整理总结一下Docker的有关知识点
top_img: 'https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201225221357.png'
cover: 'https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201225220416.png'
abbrlink: 8bff
date: 2020-12-20 23:31:12
---

**Docker**，什么是**Docker**？

引用官方文档的概述：

- **Docker**是一个用于开发，发布和运行应用程序的开放平台。**Docker**使您能够将应用程序与基础架构分开，从而可以快速交付软件。借助**Docker**，您可以以与管理应用程序相同的方式来管理基础架构。通过利用**Docker**的方法来快速交付，测试和部署代码，您可以大大减少编写代码和在生产环境中运行代码之间的延迟。

我打算分三个部分完成这篇文章：

1. **Docker** 简介
2. **Docker** 架构
3. **Docker** 简单使用

# Docker 简介

## 环境配置的难题

环境配置是最最麻烦的事情，一台电脑必须保证：操作系统的正确设置、各种库和组件的正确安装，只有做到这两点，软件才可以运行起来。从事软件开发行业的人员，拿到一个新电脑，最开始做的就是配置环境，例如：**Java**开发人员配置**Java**环境（如**JDK**、**Tocmat**服务器等等）。

一个软件的无到有、再到正式上线，不单单只有开发，还有后期的软件测试、软件运行和维护。

开发人员、测试人员、运维人员的电脑环境配置有可能是不同的，这也是导致一个软件在开发人员的电脑可以跑起来，而在测试人员的电脑中运行不起来。如果某些老旧的模块与当前的环境不兼容，那就更麻烦了。

环境配置如此麻烦，换一台机器，就要重来一次，旷日费时。很多人想到，能不能从根本上解决问题，软件可以带环境安装？也就是说，安装的时候，把原始环境一模一样地复制过来。

## 虚拟机

虚拟机是带环境安装的一种解决方案。

虚拟机可以在一种操作系统里面运行另一种操作系统，比如在**Window 10**系统里面运行**Linux**系统（微软为了拥抱**Linux**而推出的**WSL**，目前是**WSL2**版本），或者是利用虚拟机软件**VMware Workstation**，在这个软件中安装别的系统。

使用**VMware Workstation**或者是**WSL**（**VMware Workstation**安装操作系统后可以有图形化界面，**WSL**我则利用**VS code**来使用，**WSL**也可以使用图形化界面，但利用**VS code**会更加方便），就跟使用一台全新的操作系统电脑一般，与真实系统一模一样，无论在虚拟机中怎么使用，都不会影响到外面的这个主操作系统，因为对于这个主操作系统而言，无论是**VMware Workstation**也好，或是**WSL**也好，都只是一个普通文件夹，不需要了可以删掉，对于其他部分是没有任何影响的。

虽然用户可以通过虚拟机还原软件的原始环境，但是，这个方案有几个缺点：

1. 资源占用过多

   虚拟机会独占一部分内存和硬盘空间。它运行的时候，其他程序就不能使用这些资源了。哪怕虚拟机里面的应用程序，真正使用的内存只有 1MB，虚拟机依然需要几百 MB 的内存才能运行。

2. 冗余步骤多

   虚拟机是完整的操作系统，一些系统级别的操作步骤，往往无法跳过，比如用户登录。

3. 启动慢

   启动操作系统需要多久，启动虚拟机就需要多久。可能要等几分钟，应用程序才能真正运行。

## Docker 的前生——Linux容器（容器虚拟化技术）

由于虚拟机存在以上三个缺点，**Linux**发展出了另一种虚拟化技术：**Linux**容器（**Linux Containers**，缩写为 **LXC**）。

与虚拟机不同，**Linux容器不是模拟一个完整的操作系统，而是对进程进行隔离**，或者换一种说法，在正常进程的外面套上一个保护套，对于容器里面的进程来说，它接触到的各种资源都是虚拟的，从而实现与底层操作系统的隔离。

由于容器是进程级别的，相比于虚拟机有许多优势：

1. 启动快

   容器里面的应用，直接就是底层系统的一个进程，而不是虚拟机内部的进程。所以，启动容器相当于启动本机的一个进程，而不是启动一个操作系统，速度就快很多。

2. 资源占用少

   容器只占用需要的资源，不占用那些没有用到的资源；虚拟机由于是完整的操作系统，不可避免要占用所有资源。另外，多个容器可以共享资源，虚拟机都是独享资源。

3. 体积小

   容器只要包含用到的组件即可，而虚拟机是整个操作系统的打包，所以容器文件比虚拟机文件要小很多。

总之，容器有点像轻量级的虚拟机，能够提供虚拟化的环境，但是成本开销小得多。

| 特性       | 容器               | 虚拟机        |
| ---------- | ------------------ | ------------- |
| 启动       | 秒级               | 分钟级        |
| 硬盘使用   | 一般为 **MB**      | 一般为 **GB** |
| 性能       | 接近原生           | 弱于          |
| 系统支持量 | 单机支持上千个容器 | 一般几十个    |

## 什么是 Docker

**Docker**是一个用于开发，发布和运行应用程序的开放平台。**Docker**使您能够将应用程序与基础架构分开，从而可以快速交付软件。借助**Docker**，您可以以与管理应用程序相同的方式来管理基础架构。通过利用**Docker**的方法来快速交付，测试和部署代码，您可以大大减少编写代码和在生产环境中运行代码之间的延迟。（引自于官方文档）

用简单的话来说，**Docker**可以让开发者打包他们的应用以及各种依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的操作系统中，也可以实现虚拟化。

**Docker属于Linux容器的一种封装，提供简单简易的容器使用接口。**它是目前最流行的 **Linux**容器解决方案。

**Docker** 将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器里运行，就好像在真实的物理机上运行一样。有了 **Docker**，就不用担心环境问题。

总体来说，**Docker** 的接口相当简单，用户可以方便地创建和使用容器，把自己的应用放入容器。容器还可以进行版本管理、复制、分享、修改，就像管理普通的代码一样。

## Docker 与 Linux 容器的关系

虚拟化技术已经走过了三个时代，没有容器化技术的演进就不会有**Docker**技术的诞生！

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20210109133628.png)

- 物理机时代：多个应用程序可能跑在一台机器上。

  <img src="https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20210109133828.png" style="zoom:50%;" />

- 虚拟机时代：一台物理机安装多个虚拟机（VM），一个虚拟机跑多个程序。

  <img src="https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20210109133853.png" style="zoom:50%;" />

- 容器化时代：一台物理机安装多个容器实例，一个容器跑多个程序。

  <img src="https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20210109133921.png" style="zoom: 50%;" />

**Docker**并不是**Linux**容器的替代品，**Docker**底层使用**Linux**容器来实现，**Linux**容器将**Linux**进程沙盒化，使得进程之间相互隔离。在**Linux**容器的基础上，**Docker**提供了一系列更加强大的功能。可以理解为**Docker**是**Linux**容器的升级版。

## 为什么 Docker 越来越受欢迎

**一处构建、随处运行**

这是**Docker**最大的优点。

1. 更快速的应用交付和部署

   传统的应用开发完成后，需要提供一堆安装程序和配置说明文档，安装部署后需根据配置文档进行繁杂的配置才可以正常运行。

   使用**Docker**，只需要交付少量容器镜像文件，在正式生产环境加载镜像并运行即可，应用安装、环境配置等等应用运行的前提条件，都已经在镜像中内置好，大大节省部署配置和测试验证时间。

2. 更便捷的升级和扩缩容

   随着微服务架构和**Docker**的发展，大量的应用会通过微服务方式架构，应用的开发构建将变成搭乐高积木一样，每一个**Docker**容器就变成了一块积木，应用的升级会变得更加容易。

   每次需要对某一个应用模块进行升级，可以通过镜像运行新的容器进行快速升级。

3. 更简单的系统运维

   应用容器化运行后，生产环境运行的应用可与开发、测试环境的应用高度一致，容器会将应用程序相关的环境和状态完全封装起来，不会因为底层基础架构和操作系统的不一致性给应用带来影响，产生新的**BUG**。

   当出现程序异常时，也可以通过测试环境的相同容器进行快速定位和修复。

4. 更高效的计算资源利用

   **Docker**是内核级虚拟化，其不像传统的虚拟化技术一样需要额外的**Hypervisor**支持，所以在一台物理机上可以运行多个容器实例，可大大提升物理服务器的**CPU**和内存的利用率。

## Docker 的用途

**Docker**的主要用途，目前有三大类：

1. **提供一次性的环境**

   比如：本地测试他人的软件、持续集成的时候提供单元测试和构建的环境。

2. **提供弹性的云服务**

   因为Docker容器可以随开随关，很适合动态扩容和缩容。

3. **组建微服务架构**

   通过多个容器，一台机器可以跑多个服务，因此在本地机器就可以模拟出微服务架构。

## Docker 改变了什么

- 面向产品：产品交付
- 面向开发：简化环境配置
- 面向测试：多版本测试
- 面向运维：环境一致性
- 面向架构：自动化扩容（微服务）

# Docker 架构

## Docker 引擎

**Docker Engine**是具有以下主要组件的客户端-服务器应用程序：

- 服务器是一种长期运行的程序，称为守护程序进程（ **dockerd**命令）。

- REST API，它指定程序可以用来与守护程序进行通信并指示其操作的接口。

- 命令行界面（**CLI**）客户端（**docker**命令）。

![Docker引擎](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201220224351.png)

**CLI**使用**Docker REST API**通过脚本或直接**CLI**命令来控制**Docker**守护程序或与**Docker**守护程序进行交互。许多其他**Docker**应用程序都使用基础**API**和**CLI**。

守护程序创建和管理**Docker**对象，例如图像，容器，网络和卷。

## Docker 架构

**Docker**使用客户端-服务器（**C/S**）架构模式。使用远程**API**来管理和创建**Docker**容器

**Docker** 容器通过 **Docker** 镜像来创建。

容器与镜像的关系类似于面向对象编程中的对象与类。

| Docker | 面向对象 |
| ------ | -------- |
| 容器   | 对象     |
| 镜像   | 类       |

**Docker**客户端与**Docker**守护程序进行对话，该守护程序完成了构建，运行和分发**Docker**容器的繁重工作。

**Docker**客户端和守护程序可以在同一系统上运行，或者可以将**Docker**客户端连接到远程**Docker**守护程序。

**Docker**客户端和守护程序在**UNIX**套接字或网络接口上使用**REST API**进行通信。

![Docker架构](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201220224651.png)

| 概念                             | 说明                                                         |
| -------------------------------- | :----------------------------------------------------------- |
| **Docker** 镜像（**Images**）    | **Docker** 镜像是用于创建 **Docker** 容器的模板，比如 **Ubuntu** 系统。 |
| **Docker** 容器(**Container**)   | 容器是独立运行的一个或一组应用，是镜像运行时的实体           |
| **Docker** 客户端(**Client**)    | **Docker** 客户端通过命令行或者其他工具使用 **Docker API** (https://docs.docker.com/reference/api/docker_remote_api) 与 **Docker** 的守护进程通信。 |
| **Docker**守护程序               | **Docker**守护程序（**dockerd**）侦听**Docker API**请求并管理**Docker**对象，例如图像，容器，网络和卷。守护程序还可以与其他守护程序通信以管理**Docker**服务。 |
| **Docker** 主机(**Docker_Host**) | 一个物理或者虚拟的机器用于执行 **Docker** 守护进程和容器。   |
| **Docker** 仓库(**Registry**)    | **Docker** 仓库用来保存镜像，可以理解为代码控制中的代码仓库。<br />**Docker Hub**(https://hub.docker.com) 提供了庞大的镜像集合供使用。 |
| **Docker Machine**               | **Docker Machine**是一个简化**Docker**安装的命令行工具，通过一个简单的命令行即可在相应的平台上安装**Docker**，比如**VirtualBox**、 **Digital Ocean**、**Microsoft Azure**。 |

# Docker 简单使用

## Ubuntu下安装 Docker

### 卸载旧版本

旧版本的 **Docker** 称为 **docker** 或者 **docker-engine**，使用以下命令卸载旧版本：

```bash
$ sudo apt-get remove docker \
               docker-engine \
               docker.io
```

![卸载Docker旧版本](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201222211131.png)

### 使用 apt 安装

由于 **apt** 源使用 **HTTPS** 以确保软件下载过程中不被篡改。因此，我们首先需要添加使用 **HTTPS** 传输的软件包以及 **CA** 证书。

```bash
$ sudo apt-get update

$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201222211242.png)

鉴于国内网络问题，强烈建议使用国内源，官方源请在注释中查看。

为了确认所下载软件包的合法性，需要添加软件源的 **GPG** 密钥。

```bash
$ curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
#PS:这里用的Ubuntu软件源是中科大的软件源，如果想切换软件源的文末有其他的软件源

# 官方源
# $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201222211405.png)

然后，我们需要向 **sources.list** 中添加 **Docker** 软件源

```bash
$ sudo add-apt-repository \
    "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu \
    $(lsb_release -cs) \
    stable"


# 官方源
# $ sudo add-apt-repository \
#    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
#    $(lsb_release -cs) \
#    stable"
```

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201222211459.png)

更新 **apt** 软件包缓存，并安装 **docker-ce**：

```bash
$ sudo apt-get update

$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201222211555.png)

### 使用脚本自动安装

```bash
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

### 启动 Docker

```bash
$ sudo systemctl enable docker
$ sudo systemctl start docker
```

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201222211726.png)

### 安装完Docker后，守护进程启动，检查Docker是否正在运行

```bash
$ sudo systemctl status docker
#终端出现如下图的信息，表明该服务处于活动状态并正在运行
```

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201222211826.png)

### 建立 docker 用户组

默认情况下，**docker** 命令会使用 **Unix socket** 与 **Docker** 引擎通讯。而只有 **root** 用户和 **docker** 组的用户才可以访问 **Docker** 引擎的 **Unix socket**。出于安全考虑，一般 **Linux** 系统上不会直接使用 **root** 用户。因此，更好地做法是将需要使用 **docker** 的用户加入 **docker** 用户组。

建立 **docker** 组：

```bash
$ sudo groupadd docker
```

将当前用户加入 **docker** 组：

```bash
$ sudo usermod -aG docker $USER
```

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201222211949.png)

退出当前终端并重新登录，进行测试。

### 永远滴神——HelloWord

```bash
#先拉取hello-world镜像
$ sudo docker pull hello-world
#再运行hello-world镜像
$ sudo docker run hello-world
```

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201222212108.png)

```bash
#查看hello-world的镜像
$ sudo docker images
```

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201222212226.png)

### 镜像加速

国内从 DockerHub 拉取镜像有时会遇到困难，此时可以配置镜像加速器。Docker 官方和国内很多云服务商都提供了国内加速器服务，例如：

> - 网易：https://hub-mirror.c.163.com/
> - 阿里云：https://<你的ID>.mirror.aliyuncs.com
> - 七牛云加速器：https://reg-mirror.qiniu.com

我这里以网易的为例说一说如何进行配置

目前主流 **Linux** 发行版均已使用 **systemd** 进行服务管理，这里介绍如何在使用 **systemd** 的 **Linux** 发行版中配置镜像加速器。

请首先执行以下命令，查看是否在 **docker.service** 文件中配置过镜像地址。

```bash
$ systemctl cat docker | grep '\-\-registry\-mirror'
```

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201222212332.png)

如果该命令有输出，那么请执行 `$ systemctl cat docker` 查看 `ExecStart=` 出现的位置，修改对应的文件内容去掉 `--registry-mirror` 参数及其值，并按接下来的步骤进行配置。

如果以上命令没有任何输出，像我的一样，那么就可以在 `/etc/docker/daemon.json` 中写入如下内容（如果文件不存在请新建该文件）：

```bash
{
  "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ]
}
```

注意，一定要保证该文件符合 json 规范，否则 Docker 将不能启动。

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201222212427.png)

之后重新启动服务。

```bash
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

检查加速器是否生效：

执行 `$ docker info`，如果从结果中看到了如下内容，说明配置成功。

```bash
Registry Mirrors:
 https://hub-mirror.c.163.com/
```

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201222212543.png)

### 其余各类操作系统安装 Docker 官方文档

> - [Mac](https://docs.docker.com/docker-for-mac/install/)
> - [Windows](https://docs.docker.com/docker-for-windows/install/)
> - [Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
> - [Debian](https://docs.docker.com/engine/install/debian/)
> - [CentOS](https://docs.docker.com/engine/install/centos/)
> - [Fedora](https://docs.docker.com/engine/install/fedora/)
> - [其他 Linux 发行版](https://docs.docker.com/engine/install/binaries/)

## 镜像使用

### 获取镜像

从 **Docker** 镜像仓库获取镜像的命令是 **docker pull**。其命令格式为：

```bash
$ docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
```

具体的选项可以通过 **docker pull --help** 命令看到，这里我们说一下镜像名称的格式。

- **Docker** 镜像仓库地址：地址的格式一般是 **<域名/IP>[:端口号]**。默认地址是 **Docker Hub(docker.io)**。
- 仓库名：如之前所说，这里的仓库名是两段式名称，即 **<用户名>/<软件名>**。对于 **Docker Hub**，如果不给出用户名，则默认为 **library**，也就是官方镜像。

比如：

```bash
$ docker pull ubuntu:18.04
18.04: Pulling from library/ubuntu
bf5d46315322: Pull complete
9f13e0ac480c: Pull complete
e8988b5b3097: Pull complete
40af181810e7: Pull complete
e6f7c7e5c03e: Pull complete
Digest: sha256:147913621d9cdea08853f6ba9116c2e27a3ceffecf3b492983ae97c3d643fbbe
Status: Downloaded newer image for ubuntu:18.04
```

上面的命令中没有给出 **Docker** 镜像仓库地址，因此将会从 **Docker Hub** 获取镜像。而镜像名称是 **ubuntu:18.04**，因此将会获取官方镜像 **library/ubuntu** 仓库中标签为 **18.04** 的镜像。

从下载过程中可以看到我们之前提及的分层存储的概念，镜像是由多层存储所构成。下载也是一层层的去下载，并非单一文件。下载过程中给出了每一层的 ID 的前 12 位。并且下载结束后，给出该镜像完整的 sha256 的摘要，以确保下载一致性。

在使用上面命令的时候，你可能会发现，你所看到的层 ID 以及 sha256 的摘要和这里的不一样。这是因为官方镜像是一直在维护的，有任何新的 bug，或者版本更新，都会进行修复再以原来的标签发布，这样可以确保任何使用这个标签的用户可以获得更安全、更稳定的镜像。

如果从 Docker Hub 下载镜像非常缓慢，可以参照镜像加速一节，更换国内源。

### 列出镜像

要想列出已经下载下来的镜像，可以使用 `docker image`  命令。

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201225211110.png)

列表包含了 仓库名、标签、镜像 ID、创建时间 以及 所占用的空间。

其中仓库名、标签在之前的基础概念章节已经介绍过了。镜像 ID 则是镜像的唯一标识，一个镜像可以对应多个 标签。

### 删除本地镜像

如果要删除本地的镜像，可以使用 `docker image rm` 命令，其格式为：

```bash
$ docker image rm [选项] <镜像1> [<镜像2> ...]
```

```bash
#先用命令查看所有本地镜像
$ docker images
#再用删除命令
$ docker image rm <IMAGE ID>
#先用命令查看所有本地镜像，检查是否删除成功
$ docker images
```

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201225211547.png)

例外状况：

```bash
#可能会出现因操作失误，导致不同的镜像会有相同的IMAGE ID号
#直接使用docker image rm <IMAGE ID>会报错
#要换一个命令：
$ docker image rm REPOSITORY:TAG
```

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201225211740.png)

### 上传镜像到阿里云仓库

```bash
sudo docker login --username=[你的阿里云用户名] registry.cn-shenzhen.aliyuncs.com
sudo docker tag [ImageId] registry.cn-shenzhen.aliyuncs.com/zhouzixin/docker1:[镜像版本号]
sudo docker push registry.cn-shenzhen.aliyuncs.com/zhouzixin/docker1:[镜像版本号]

#registry.cn-shenzhen.aliyuncs.com/zhouzixin/docker1：更改为你的阿里云Docker仓库地址
```

### 从阿里云仓库拉取镜像

```bash
sudo docker login --username=[用户名] registry.cn-shenzhen.aliyuncs.com
sudo docker pull registry.cn-shenzhen.aliyuncs.com/zhouzixin/docker1:[镜像版本号]
#registry.cn-shenzhen.aliyuncs.com/zhouzixin/docker1：更改为你的阿里云Docker仓库地址
```



### 使用Dockerfile构建镜像

[我另写了一篇笔记。]( https://williscode.gitee.io/posts/12a3.html)

## 操作容器

### 启动

启动容器有两种方式，一种是基于镜像新建一个容器并启动，另外一个是将在终止状态（`stopped`）的容器重新启动。

因为 **Docker** 的容器实在太轻量级了，很多时候用户都是随时删除和新创建容器。

#### 新建并启动

命令：

```bash
$ docker run
```

例如，下面的命令输出一个 “Hello World”，之后终止容器。

```bash
$ docker run ubuntu:20.04 /bin/echo 'Hello world'
Hello world
```


这跟在本地直接执行 **/bin/echo 'hello world'** 几乎感觉不出任何区别。

下面的命令则启动一个 bash 终端，允许用户进行交互。

```bash
$ docker run -t -i ubuntu:20.04 /bin/bash
root@af8bae53bdd3:/#
```

其中，**-t** 选项让**Docker**分配一个伪终端（**pseudo-tty**）并绑定到容器的标准输入上， **-i** 则让容器的标准输入保持打开。

在交互模式下，用户可以通过所创建的终端来输入命令，例如

```bash
root@af8bae53bdd3:/# pwd
/
root@af8bae53bdd3:/# ls
bin boot dev etc home lib lib64 media mnt opt proc root run sbin srv sys tmp usr var
```


当利用 **docker run** 来创建容器时，**Docker** 在后台运行的标准操作包括：

- 检查本地是否存在指定的镜像，不存在就从公有仓库下载
- 利用镜像创建并启动一个容器
- 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
- 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
- 从地址池配置一个 ip 地址给容器
- 执行用户指定的应用程序
- 执行完毕后容器被终止

#### 启动已终止容器

可以利用 **docker container star**t 命令，直接将一个已经终止的容器启动运行。

### 后台运行

更多的时候，需要让 **Docker** 在后台运行而不是直接把执行命令的结果输出在当前宿主机下。此时，可以通过添加 `-d` 参数来实现。

```bash
$ docker run -d ubuntu:20.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
77b2dc01fe0f3f1265df143181e7b9af5e05279a884f4776ee75350ea9d8017a
```

此时容器会在后台运行并不会把输出的结果 (**STDOUT**) 打印到宿主机上面(输出结果可以用 **docker logs** 查看)。

注： 容器是否会长久运行，是和 **docker run** 指定的命令有关，和 **-d** 参数无关。

使用 **-d** 参数启动后会返回一个唯一的 **id**，也可以通过 **docker container ls** 命令来查看容器信息。

要获取容器的输出信息，可以通过 **docker container logs** 命令。

```bash
$ docker container logs [container ID or NAMES]
hello world
hello world
hello world
. . .
```

### 终止容器

可以使用 **docker container stop** 来终止一个运行中的容器。

```bash
$ docker container stop <CONTAINER ID>
```

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201225212153.png)

处于终止状态的容器，可以通过 `docker container start` 命令来重新启动。

此外，`docker container restart` 命令会将一个运行态的容器终止，然后再重新启动它。

### 进入容器

在使用 `-d` 参数时，容器启动后会进入后台。

某些时候需要进入容器进行操作，包括使用 `docker attach` 命令或 `docker exec` 命令

### 导出和导入容器

- 导出容器：

如果要导出本地某个容器，可以使用 **docker export** 命令。

```bash
#先查看容器
$ docker container ls -a
#再用命令导出
$ docker export <CONTAINER ID> > hello.tar
```

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201225212431.png)

- 导入容器快照：

可以使用 `docker import` 从容器快照文件中再导入为镜像

### 删除容器

可以使用 **docker container rm** 来删除一个处于终止状态的容器。

```bash
#先用命令查看所有容器
$ docker ps -a
#再用删除命令删除容器
$ docker container rm <CONTAINER ID>
#先用命令查看所有容器，检查是否已删除
$ docker ps -a
```

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201225212802.png)

如果要删除一个运行中的容器，可以添加 **-f** 参数。**Docker** 会发送 **SIGKILL** 信号给容器。

清理所有处于终止状态的容器

用 **docker container ls -a** 命令可以查看所有已经创建的包括终止状态的容器，如果数量太多要一个个删除可能会很麻烦，用下面的命令可以清理掉所有处于终止状态的容器。

```bash
$ docker container prune
```

# 参考链接

## 参考文档

- 官方文档：https://docs.docker.com/get-started/overview/

- 博客：
  - http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html
  - https://docker_practice.gitee.io/zh-cn/
- 知乎：https://zhuanlan.zhihu.com/p/89587030

- W3Cschool：https://www.w3cschool.cn/docker/docker-tutorial.html


## Ubuntu国内镜像源

### 阿里云源

```bash
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
```

### 清华源

```bash
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
```

### 中科大源

```bash
deb https://mirrors.ustc.edu.cn/ubuntu/ focal main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ focal-security main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal-security main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
```

### 网易163源

```bash
deb http://mirrors.163.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ focal-proposed main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ focal-backports main restricted universe multiverse
```

