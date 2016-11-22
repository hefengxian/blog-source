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
**查看文件详情**
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

**设置别名**
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

