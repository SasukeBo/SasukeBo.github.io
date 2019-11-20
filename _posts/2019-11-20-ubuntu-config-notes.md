---
title:  "Ubuntu常用配置笔记"
date:   2019-11-20 08:27:00 +0800
categories: ubuntu
header:
  overlay_image: /assets/images/banners/ubuntu.png
  teaser: /assets/images/teaser/ubuntu-logo14.png
---

> 此篇笔记主要记录我在学习使用**Ubuntu**时遇到的配置问题。

#### Ubuntu18 配置网络

Ubuntu18使用`netplan`管理网络配置，netplan的配置文件一般在`/etc/netplan`中，例如文件`50-cloud-init.yaml`，
该配置文件为只读文件，修改需要root权限。下面给出服务器多网络接口的配置文件样例：
```yaml
# This file is generated from information provided by
# the datasource.  Changes to it will not persist across an instance.
# To disable cloud-init's network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        enp3s0:
            addresses: [192.168.8.111/24]
            gateway4: 192.168.8.1
            dhcp4: no
            optional: true
        enp1s0f0:
            addresses: [192.168.8.118/24]
            gateway4: 192.168.8.1
            dhcp4: no
            optional: true
        enp1s0f1:
            addresses: [192.168.9.102/24]
            gateway4: 192.168.9.1
            dhcp4: no
            optional: true
    version: 2
```

此设备有三个网络端口，如果使用DHCP动态获取IP，则只需要配置`dhcp4: yes`，此时DHCP客户端会自动获取IP，自动配置网络，
如果要设置静态IP则需要将`dhcp4`设置为`no`，需要给出静态IP地址以及子网编码长度，还有网关IP。
配置完成后执行`netplan apply`使配置生效。 也可以加上`--dubug`选项打印日志，最后使用`ip a`查看所有网络端口信息。
```shell
$ sudo netplan --debug apply # 应用配置
$ ip a # 查看所有网络端口信息
```

#### Ubuntu18 配置自启动

Ubuntu18使用`systemd`配置开机启动项，`systemd`默认读取`/etc/systemd/system/`路径下的配置文件，该目录会链接
`/lib/systemd/system/`下的文件，一般安装完系统会在该路径下生成一些启动配置文件，其中包括`rc-local.service`
文件，但是默认是没有链接到`/etc/systemd/system/`文件夹下的，查看该文件的初始内容：

```conf
[Unit]
 Description=/etc/rc.local Compatibility
 Documentation=man:systemd-rc-local-generator(8)
 ConditionFileIsExecutable=/etc/rc.local
 After=network.target

[Service]
 Type=forking
 ExecStart=/etc/rc.local start
 TimeoutSec=0
 RemainAfterExit=yes
 GuessMainPID=no

[Install]
 WantedBy=multi-user.target
 Alias=rc-local.service
```
1. [Unit]主要用于配置启动顺序和依赖关系
1. [Service]主要用于定义启动行为，启动类型。
1. [Install]主要定义如何按安装这个配置文件。

> 系统的学习如何使用`systemd`应该查看官方文档：[SystemdForUpstartUsers](https://wiki.ubuntu.com/SystemdForUpstartUsers)。

上面的配置[Service]块指出执行`/etc/rc.local`文件，当然也可以是其他可执行文件。
创建`/etc/rc.local`可执行文件：
```shell
$ touch /etc/rc.local
$ chmod 755 /etc/rc.local
```

在`rc.local`文件中添加启动时需要执行的任务：
```shell
#!/bin/bash

echo "test rc.local" > /var/test.log
```

重启验证效果。

另外需要注意的是，如果执行一些任务的前提条件不存在，需要调整启动顺序配置，上面配置的是`After=network.target`，
具体配置需要结合实际情况。
