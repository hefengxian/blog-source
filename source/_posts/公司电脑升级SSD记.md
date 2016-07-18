---
title: 电脑升级SSD记
date: 2016-07-07 23:19:34
tags:
---
由于电脑使用的机械硬盘，太卡，所以想办法提升开发体验。电脑的CPU不算差，内存单条8G也还OK。

先去京东买个很便宜的240G的ADATA SSD。头天晚上下单，第二天早上上班就送到了。京东大城市的速度是毋庸置疑的。

硬件安装方面换硬盘是相当简单的，我是直接将原来的硬盘取下来不用了；安装过程很简单，不赘诉。
<!--more-->


### 系统安装

* 准备一个4G U盘
* 去官网下载Ubuntu 16.04
* 一个可用的windows + 安装盘制作软件

这里有个坑，开始用百度搜索出来的16.04的安装教程，推荐使用的制作软件是 Universal USB Installer，然而我制作了不下于5次，都无法正常启动。最终还是使用老牌的Ultraiso，制作过程网上一大把，大致说一下制作流程。

> 菜单->文件->打开下载好的系统->菜单->启动->写入硬盘映像->选好你的U盘 然后等待制作完成即可

将制作好的U盘插入到电脑，然后启动的时候注意选择启动项为制作的U盘，具体怎么选择启动项不同的电脑不一样，对了，别忘了在BIOS里面开启ACHI。DELL的电脑一般是F12选择启动项。

启动之后选择install ubuntu 或者 try ubuntu然后安装，安装过程基本上都是下一步，然后等待。

关于磁盘怎么选择分区这些问题，这里一概默认设置，因为240G并不大，而且我是准备只安装Ubuntu的，所以不考虑其他因素。用户名密码什么的安装步骤设置即可。

需要注意的是，先选择语言的时候可以不着急选中文（选中文也可以）因为选中文会导致系统菜单的：桌面/下载等目录都是汉字的，到时在命令行的时候会很不好使。等到系统内部在修改语言即可，后面会说到。

安装完成之后，输入密码即可进入到系统了，SSD速度果然不俗！！！秒进！！！

### 基础的设置

#### 更新软件源

第一步就是把更新的源给更换了！因为涉及到后面安装软件的速度和依赖问题。

> 系统设置->软件和更新->下拉里选择其他->自动选择最好的



#### 安装WizNote记录这一切

搜索为知笔记，找到官网，下载Linux版本。[为知笔记](http://www.wiz.cn/wiznote-linux.html)，安装官方的安装方法安装即可

```bash
$ sudo add-apt-repository ppa:wiznote-team
$ sudo apt-get update
$ sudo apt-get install wiznote
```

我开始安装还是遇到了问题，说是有个依赖找不到（因为那个时候我还没有切换源，切换之后再安装就正常了）



#### 安装搜狗输入法

安装完了为知笔记，发现没有输入法。那怎么写笔记呢？（英语太烂写不了）

> 如果开始是选择英文安装的，安装输入法之前应该要先把系统语言切换成中文。因为系统语言为中文的时候，会自动安装iBus和Fcitx（搜狗需要Fcitx），另外还有各种中文字体文件。
> 系统设置->语言支持（如果此时告诉你语言包不全，要记得让它自动安装）->然后把中文从最底下拉上来到第一位
> 然后点应用到整个系统，可能需要注销或者重启才能看到效果，那么我们先注销或者重启一下。这里会提示你要不要把文件夹切换成中文，选择不要切换并且不再提示。如果开始选择中文安装的也想把文件夹名字换成英文，此时应该也会提示你切换文件夹，那么你应该切换，然后注销之后再把语言改回来，这次就不要切换即可。

去搜狗官网下载[搜狗输入法](http://pinyin.sogou.com/linux/help.php)，按照安装文档安装（就是双击让软件中心自己安装），然后把输入法切换为Fcitx注销之后，就可以选择搜狗了。对了搜狗还有好看的输入法皮肤！！！



#### Google Chrome浏览器

作为Web开发者，Chrome是必不可少的。还有一点就是Chrome内建Flash，可以看某些视频网站。当然，一提到Google，请自备梯子！！！我是有买VPN账号的，当然也可以使用xx-net使用方法可能会比较复杂，可以去GitHub仓库看。

一般安装Chrome有好几种方法，可以使用第三方源，也可以使用Google提供的DEB包，这个自行选择，我选择使用去官方下载DEB安装包。[Chrome](https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb) 直接双击使用软件中心安装即可。

安装完成之后一般要进行一些设置，使用Chrome的好处就是账号同步功能了，我只要把账号登陆（自备梯子），所有的设置和插件都回来了。



### 开发环境的再次搭建

#### 安装JDK

由于开发使用的IDE都是Java系列的（JetBrains/Eclipse），有些部分也会使用Java做开发。
去Oracle下载[JDK1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html)最新版，进行安装。我把JDK放在`/opt/oracle/`目录下。

```bash
# 解压
tar -zxvf jdk-8u91-linux-x64.tar.gz
sudo mv jdk1.8.0_91/ /opt/oracle/jdk

# 设置环境变量（全局）
sudo vim /etc/profile

# 在文件的最后面加入如下语句，然后保存
export JAVA_HOME=/opt/oracle/jdk
export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib
export PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin

# 然后source一下profile文件，使设置生效
sudo source /etc/profile

# 测试Java是否安装成功
java -version
# 出现版本号则表明安装JDK已经成功了
```


#### 安装各种IDE

示例安装PhpStorm，其他的类似

```bash
# PhpStorm
sudo tar -zxvf PhpStorm-2016.1.2.tar.gz -C /opt/jetbrains/

# 到/opt/jetbrains/目录下改名字
sudo mv PhpStorm-145.1616.3/ phpstorm

# 进入bin目录，执行IDE
cd /opt/jetbrains/phpstorm/bin
./phpstorm.sh
```
另外一个小技巧，如果想让这些常用的IDE固定载导航栏（启动器）上，在第一次执行`./phpstorm.sh`之后，导航栏会出现一个图标，右键选择锁定到启动器即可。


#### 安装Node.js

由于有些项目，和自己的BLOG都是需要Node.js和NPM的，推荐使用`nvm`来安装，直接上GitHub上搜索nvm按照文档安装即可。
```bash
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.31.2/install.sh | bash

# source 一下.bashrc
source ~/.bashrc

# 查看Node.js的版本，选择安装一个
nvm ls-remote

# 比如我选择安装了5.12.0
nvm install v5.12.0
```


#### 安装Apache2.4

依然使用apt安装，安装之后直接范围`http://localhost`应该就可以看到欢迎界面。
```bash
sudo apt install a
```

#### 安装PHP7

16.04源里面自带的就是PHP7

```bash
sudo apt-get install php php7.0-mysql php7.0-tidy php7.0-curl php7.0-mbstring
```


#### 安装MySQL 和 MySQL Work Branch
直接使用apt安装，安装MySQL途中要设置root密码，设置一个既可。
```bash
sudo apt-get install mysql-server mysql-workbench
```

#### 安装GIT
直接时使用APT安装，安装完成之后直接配置全局信息。
```bash
git config --global user.name hfx
git config --global user.email jssmith883@gamil.com
```