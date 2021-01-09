---
title: Ubuntu20.04设置开机自启
tags:
  - Linux
categories: 学习笔记
description: 设置Ubuntu20.4开机自行执行脚本
top_img: 'https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201120231500.jpg'
cover: 'https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201120231500.jpg'
abbrlink: '2579'
date: 2020-12-23 23:21:08
---

# Ubuntu20.04 设置开机自启

## 第一步

执行 **ls /lib/systemd/system** 你可以看到有很多启动脚本，其中就有我们需要的 **rc-local.service**

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201223232207.png)

## 第二步

打开 **rc-local.service**脚本内容，内容如下：

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201223232257.png)

一般正常的启动文件主要分成三部分

- **[Unit]** 段: 启动顺序与依赖关系
- **[Service]** 段: 启动行为,如何启动，启动类型
- **[Install]** 段: 定义如何安装这个配置文件，即怎样做到开机启动

可以看出，**/etc/rc.local** 的启动顺序是在网络后面，但是显然它少了 **Install** 段，也就没有定义如何做到开机启动，所以显然这样配置是无效的。 因此我们就需要在后面帮他加上 **[Install]** 段:

```
[Install]
WantedBy=multi-user.target  
Alias=rc-local.service
```

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201223232359.png)

PS：添加了[Install]内容后，下面两行的**WantedBy**和**Alias**两个英文跟上面的都是绿色的，要是绿色才有用。

一般需要先修改**rc-local.service**的权限才可以进行编辑。

```
sudo chmod 777 /lib/systemd/system/rc-local.service
```

## 第三步

查看系统中有无 **/etc/rc.local** 这个文件，没有则自己创建一个。

然后把你需要启动脚本写入 **/etc/rc.local** ，我们不妨写一些测试的脚本放在里面，以便验证脚本是否生效.

```shell
#!/bin/sh
echo "看到这行字，说明添加自启动脚本成功。" > /usr/local/test.log
exit 0
```

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201223232505.png)

PS：**#!/bin/sh** 这一行一定要加上！一定要加上！

## 第四步

给**rc.local**加上权限

```bash
sudo chmod +x /etc/rc.local
```

做完这一步，还需要最后一步 前面我们说 **systemd** 默认读取 **/etc/systemd/system** 下的配置文件, 所以还需要在 **/etc/systemd/system** 目录下创建软链接。

```bash
ln -s /lib/systemd/system/rc.local.service /etc/systemd/system/ 
```

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201223232632.png)

## 第五步

重启**Ubuntu**后，去**/usr/local**下看看有没有生成**test.log**这个文件以及这个文件的内容。

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201223232706.png)

## 第六步

如果在 **/etc/rc.local** 中添加的是 **./test.sh** 这种类型的，要在末尾加上&，如下图， **不然重启ubuntu的时候会卡在启动界面进不去系统。** 

![](https://cdn.jsdelivr.net/gh/Willis-zzx/PicGO/img/20201223232744.png)