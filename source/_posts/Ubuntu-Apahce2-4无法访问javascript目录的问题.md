---
title: Ubuntu Apahce2.4无法访问javascript目录的问题
date: 2016-08-16 14:29:46
tags:
- Ubuntu
- Apache2.4
---


#### 问题描述
今天调试一个项目，项目放在`webroot`下面的`javascript`目录，无论怎么设置都是`403 You don't have permission to access /javascript/ on this server.`

<!--more-->

#### 思考过程
首先第一反应，Google了错误`You don't have permission to access /javascript/ on this server.`，然后发现大部分答案都说到了配置文件中的`Directory`设置。比如
```apache
<Directory />
   Options FollowSymLinks
   AllowOverride All
   Order deny,allow
   Allow from all
</Directory>
```
而我并不是整个`webroot`都不能访问，其他所有的文件夹都可以访问，就是目录`javascript`文件夹没有权限访问，我的设置是
```apache
<Directory /home/hfx/workspace/>
       Options Indexes FollowSymLinks
#       AllowOverride None
       AllowOverride All
       Require all granted
</Directory>
```
所以按理来说也是没有问题的，可以单单就是`javascript`目录无法访问，并且我尝试重新给`javascript`目录设置权限
```bash
chmod 777 -R javascript
```
然而还是并没有什么卵用，于是我尝试更改`javascript`变成`javaScript`或者新建其他的名字的文件夹，而它们均可以正常访问，所以猜想症结应该是那里的配置禁止了web根路径下名字为`javascript`的路径。

#### 发现问题
于是我看apache的所有配置文件，打开`/etc/apache2/conf-enable/`目录，发现了一个名为`javascript-common.conf`的配置文件，打开一看
```
Alias /javascript /usr/share/javascript/

<Directory "/usr/share/javascript/">
       Options FollowSymLinks MultiViews
</Directory>
```
其实我并不知道具体是什么意思，可是从字面意思大概看出，配置里面指定了一个别名`/javascript`到目录`/usr/share/javascript/`，应该访问`/javascript`就变成了访问目录`/usr/share/javascript/`而不是我们在web根目录下创建的`javascript`目录。

#### 解决方案
于是google一下`javascript-common.conf`，这才发现了很多真正与此相关的问题，所以真正的解决方案有很多种，有的直接修改别名的名称为`/javascript-common`，而我觉得最好的解决方案应该是
> You don't need to edit the conf file or purge the package just disable it.
>
>     a2disconf javascript-common
>     service apache2 reload
> If for some reason you want to use that conf:
>
>     a2enconf javascript-common
>     service apache2 reload

另外解释一下`a2disconf`和`a2enconf`的作用，官方的介绍是
> a2enconf, a2disconf - enable or disable an apache2 configuration file

从介绍就直接明了它的作用了。快捷的启用或者禁用一个apache模块。

另外还有类似的几个命令：

* apache2ctl - Apache HTTP server control interface
* a2enmod, a2dismod - enable or disable an apache2 module

#### 参考
* [Apache doesn't serve files in “javascript” directory. Why?](http://stackoverflow.com/questions/21706067/apache-doesnt-serve-files-in-javascript-directory-why)
* [a2enconf](http://manpages.ubuntu.com/manpages/trusty/man8/a2enconf.8.html)
* [apache2ctl](http://manpages.ubuntu.com/manpages/trusty/man8/apache2ctl.8.html)
* [a2dismod](http://manpages.ubuntu.com/manpages/trusty/man8/a2dismod.8.html)
