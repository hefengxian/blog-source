---
title: Ubuntu 常用命令记录
date: 2016-11-22 10:09:38
categories:
- 日常笔记
tags:
- Ubuntu
- Linux
---

Linux有太多的命令，很多经常用却没有记住；这里统一收集一下平常用到的一些命令

<!--more-->
### 查看文件详情
一般用`ls`可以查看文件`la`（`ls -A`的别名）查看所有文件；但是一老没记住怎么查看文件的大小啊、权限啊什么的鬼；通过查看帮助`ls --help`
```bash
# 查看详情（帮助解释为：使用较长格式列出信息）
ls -l
```
但是这样有个问题，文件的大小是使用Byte为单位的，根本无法直接看出一个文件有多大；如果我们想要更好的可读性呢？
```bash
# -h 帮助解释为：--human-readable with -l and/or -s, print human readable sizes (e.g., 1K 234M 2G)
ls -lh
```

### 设置别名
比如我知道了`ls -lhA`很好使，我但是每次都输入这么多参数很不方便，那么我们怎么是其简单化呢？就像`la`一样；我们先看一下用户主目录下的`.bashrc`文件`sudo  vim ~/.bashrc`
```bash
# some more ls aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
```
看到这里应该明白怎么回事了，所以依葫芦画瓢，在上诉位置追加如下的命令，保存
```
alias lh='ls -lhA'
```
立即使用`lh`发现并没有什么用，因为`.bashrc`是启动时加载的，所以我们手动加载一下
```bash
source ~/.bashrc
```
现在用就看到效果了。
另外，设置别名的时候要注意有没有冲突，一个简单粗暴的验证方法是直接在命令行中输入你要准备使用的别名，如果还没使用，那么会提示你。

### 查看路由表
```bash
netstat -r

内核 IP 路由表
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         192.168.1.1     0.0.0.0         UG        0 0          0 enp2s0
0.0.0.0         192.168.2.1     0.0.0.0         UG        0 0          0 enp2s0
169.254.0.0     0.0.0.0         255.255.0.0     U         0 0          0 docker0
172.17.0.0      0.0.0.0         255.255.0.0     U         0 0          0 docker0
192.168.1.0     0.0.0.0         255.255.255.0   U         0 0          0 enp2s0
192.168.2.0     0.0.0.0         255.255.255.0   U         0 0          0 enp2s0
```
其他选项可以使用`netstat --help`

### 删除和设置默认网关
```bash
# 删除默认路由
sudo route del default

# 设置默认路由
sudo route add default gw 192.168.x.1
```

### Ubuntu桌面版的IP设置
现在网上的Ubuntu IP设置教程，一般都是在Server版下的；然而桌面版的除了使用图形界面设置，如果要想直接编辑对应的配置文件呢？

一般Server版的IP设置都在`/etc/network`目录下（当然桌面版也有），而桌面版除了这个还有另外的设置在目录`/etc/network`；在界面上设置的内容都是保存在这个目录下的。

比如我们打开主要的配置文件目录`/etc/NetworkManager/system-connections/`，里面可能有你设置的VPN，有线连接等等，一般我们都是默认叫有线连接，当然我们也可以叫其他的名字，你只要改一下这个配置文件名，那么你会看到右上角的网络里的名字页对于的改了

一般里面的内容都是类似如下的配置（文件是只读的，需要管理员权限），我们可以更改我们想改的配置。
```bash
[connection]
id=wired
uuid=5b6d5907-3636-401c-a4a0-cf0cc473f306
type=ethernet
autoconnect-priority=-999
permissions=
secondaries=
timestamp=1480391347

[ethernet]
duplex=full
mac-address=B0:83:FE:BA:A0:1D
mac-address-blacklist=

[ipv4]
address1=192.168.1.45/24,192.168.1.1
address2=192.168.2.45/24,192.168.2.1
dns=202.96.134.33;
dns-search=
method=manual

[ipv6]
addr-gen-mode=stable-privacy
dns-search=
ip6-privacy=0
method=auto
```