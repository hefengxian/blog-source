---
title: nvm-npm加速
date: 2017-03-26 21:17:57
categories:
- JavaScript
tags:
- npm
- nvm
- node.js
---

## NVM 加速

安装 Node.js 一般都是用 [nvm](https://github.com/creationix/nvm)

但是安装之后使用体验就不咋地了，由于某些不可变原因，国内网络就不好吐槽了；就连执行一下 `nvm ls-remote` 都要很久。

当然“道”高一尺“魔”高一丈，聪明的国人肯定也想出了对应的解决方案，就是建立了各种各样的镜像；比如淘宝源 [淘宝 NPM 镜像](https://npm.taobao.org/)

所以加速 nvm 的方法就是使用 NPM 淘宝的 node.js 镜像。

### 临时方案：

在运行 `nvm` 命令之前执行设置一个变量的命令

```bash
NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/mirrors/node
```

再执行 `nvm` 命令，那么速度就非常快了；这种方案就是每次使用你都要手工执行设置变量的命令，所以也就有了永久的方案。

### 永久方案：

其实很简单，就是在 `.bashrc` 中加入上面的设置变量的命令，这样就可以保证每次一开机，系统就自动执行了这个命令了。


## NPM 加速

如果在安装完 node 之后，使用 `npm install` 速度也是非常感人的；所以同样的我们也需要使用镜像来加速依赖的安装。

设置也比较简单，只需要设置 npm 的全局配置即可

```bash
# 查看默认的 registry 地址
npm config -g get registry
https://registry.npmjs.org/

# 设置 npm 淘宝镜像
npm config -g set registry https://registry.npm.taobao.org
```

这样设置之后安装就非常快了

<!--more-->