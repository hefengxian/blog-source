---
title: 自行修改Hexo主题
date: 2016-06-21 23:46:45
categories:
- 折腾
tags:
- 折腾
- Hexo
- Theme
---

折腾一下导航栏，相信很多用Hexo搭建博客的小伙伴们都使用了Apollo主题，这个主题配色简单大方，甚是喜爱。可是默认导航栏的样子是这样的
![Apollo默认导航栏](http://7d9jd9.com1.z0.glb.clouddn.com/2016-06-23-org-nav.png "默认导航栏的样子")
生命在于折腾，如何自定义导航栏呢？于是抽时间折腾了一下，变成了下面这样：
![修改之后的导航栏](http://7d9jd9.com1.z0.glb.clouddn.com/2016-06-23-nav-result.png "修改之后的样子")
记录一下如何折腾的，如果有遇到同样困难的小朋友或许可以参考一下
<!--more-->

#### 1. 删除主题子模块

先说一下博客项目的结构

* 博客托管在[Coding](https://coding.net)的，使用`hexo generate`之后的静态文件是放在`coding-pages`分支
* 而博客的项目源文件我是直接保存在`master`分支的

每次使用markdown写完文章，将源文件推送到`master`，然后将生成的静态页面发布到`coding-pages`。

如果按照[Apollo](https://github.com/pinggod/hexo-theme-apollo)主题官方的安装方式，使用`git clone`到`theme`文件夹，那么主题是一个匿名子模块（submodule）的方式存在于项目中的。如果是这样，我们修改主题文件会遇到问题，推送到`master`分支的文件是不包含子模块改动的，如果换一台电脑，`git clone/pull`下来的文件是不会包含主题改动的。保持子模块的好处是，如果Apollo主题的作者有更新主题，我们可以直接`cd`到子模块目录使用`git pull`更新即可。可是现在我们修改主题，那么我们则不再关注原主题是否更新了，所以要先删除主题作为子模块的存在，然后将修改的主题文件直接纳入博客项目源码的管理。删除子模块，在博客项目的根目录，执行如下代码
```bash
git rm --cached themes/apollo
```
再将主题目录下的文件add到git提交即可。


#### 2. 修改主题配置文件

找到apollo文件夹下的`_config.yml`文件，打开之后默认的目录数据结构是不满足我们修改要求的
```yaml
menu:
    Blog: /
    Archive: /archives/
    Weibo: http://weibo.com/sunchongsheng
    GitHub: https://github.com/pinggod
    Rss: /atom.xml
```
因为我们要为每个栏目配上图标。

但是发现这个`*.yml`有是个什么鬼文件？查一查发现，又有人发明了一个轮子叫[YAML](https://zh.wikipedia.org/wiki/YAML)。经查阅得知，YAML其实是包含JSON的，JSON是属于YAML的一个子集，意思就是我们可以直接写JSON格式；但是为了尊重这个轮子，我们还是用它新创建的语法更新一下我们的菜单结构
```yaml
menu:
  - name: Blog
    path: /
    icon: fa fa-home

  - name: Archive
    path: /archives/
    icon: fa fa-folder-open-o

  - name: Weibo
    path: http://weibo.com/fengxianhe
    icon: fa fa-weibo

  - name: GitHub
    path: https://github.com/hefengxian
    icon: fa fa-github-alt

  - name: Coding
    path: https://coding.net/u/hfx
    icon: iconfont coding

  - name: About
    path: /about/
    icon: fa fa-info-circle
```
代码的意思大致就是一个包含多个对象的数组，其中`name`表示要菜单要显示的文字，`path`表示url路径，`icon`表示我们要使用的图标（Font Awesome 格式）。这样更新之后去看下结果，肯定是菜单都不见了，因为我们改了数据结构。


#### 3. 修改其他相关模板文件


Apollo使用的是JADE，但是我并不懂JADE，但是看看`themes/apollo/layout/partial/nav.jade`的代码，虽然没接触过，但是凭借对其他编程语言的理解大致可以猜出语法，那么我们做一下改动
```jade
ul.nav.nav-list
    each value, key in theme.menu
        li.nav-list-item
            - var re = /^(http|https):\/\/*/gi;
            - var tar = re.test(value.path) ? "_blank" : "_self"
            a.nav-list-link(class={active: is_home() && value.path === "/"} href=value.path target=tar)
                i(class=value.icon) &nbsp;
                != value.name
```
使用hexo生成一下文件，看一下效果，发现菜单名字有了，但是还没有图标。

是因为我们还没有引入Font-Awesome相关的css文件，打开`themes/apollo/layout/partial/head.jade`，虽然还是不懂JADE是什么鬼，依葫芦画瓢，在原来的apollo.css那句后面或者前面加一句
```jade
link(rel="stylesheet", href=url_for("//cdn.bootcss.com/font-awesome/4.6.3/css/font-awesome.min.css"))
link(rel="stylesheet", href=url_for("css/apollo.css"))
```
在这里我用的是[bootcdn](http://www.bootcdn.cn/)的CDN。

再看一下，啊，大部分可以了，但是Font-Awesome这个项目并没有Coding的图标，但是我又很喜欢Coding，想把Coding放上去，那咋整呢？

以前记得淘宝好像有个字体图标的网站[ICONFONT](http://iconfont.cn/)，反正就是无意中发现的这么一个网站，往里面一搜coding，居然还真有人做了这个图标。

在网站上下载图标，按照文档，将字体文件复制到`themes/apollo/source/fonts/`目录下，在网站上下载图标，按照文档，将字体文件复制到`themes/apollo/source/fonts/`目录下,我们在目录`themes/apollo/source/css/`下加入`iconfont.css`文件
```css
@font-face {
    font-family: "iconfont";
    src: url('../fonts/iconfont.eot?t=1466522057'); /* IE9*/
    src: url('../fonts/iconfont.eot?t=1466522057#iefix') format('embedded-opentype'), /* IE6-IE8 */
    url('../fonts/iconfont.woff?t=1466522057') format('woff'), /* chrome, firefox */
    url('../fonts/iconfont.ttf?t=1466522057') format('truetype'), /* chrome, firefox, opera, Safari, Android, iOS 4.2+*/
    url('../fonts/iconfont.svg?t=1466522057#iconfont') format('svg'); /* iOS 4.1- */
}

.iconfont {
    font-family:"iconfont" !important;
    font-size:16px;
    font-style:normal;
    -webkit-font-smoothing: antialiased;
    -webkit-text-stroke-width: 0.2px;
    -moz-osx-font-smoothing: grayscale;
}

.coding:before { content: "\e604"; }

```
我们将新加css文件引入到页面，打开`themes/apollo/layout/partial/head.jade`，和前面引入Font-Awesome一样
```jade
link(rel="stylesheet", href=url_for("//cdn.bootcss.com/font-awesome/4.6.3/css/font-awesome.min.css"))
link(rel="stylesheet", href=url_for("css/iconfont.css"))
link(rel="stylesheet", href=url_for("css/apollo.css"))
```
这个时候再看一下效果，发现图标已经有了，可是大小好像有问题，很小...，于是我们打开Chrome调试工具，调整`.iconfont`样式的字体大小，发现`20px`差不多，那么再次修改一下文件`iconfont.css`
```css
.iconfont {
    font-family:"iconfont" !important;
    font-size:20px;
    font-style:normal;
    -webkit-font-smoothing: antialiased;
    -webkit-text-stroke-width: 0.2px;
    -moz-osx-font-smoothing: grayscale;
}
```
再刷新一下，终于可以了，得到了我们想要的结果。

多说一句，博客的Logo自己去找一个图片，保持格式名称一样（其实可以有其他的格式，但是自己可以去查一查favicon支持哪些格式），或者自己画一个替换`themes/apollo/source/favicon.png`

参考资料：

* YAML [YAML维基百科](https://zh.wikipedia.org/wiki/YAML)
* JADE [JADE官网](http://jade-lang.com/)
* Font-Awesome [font-awesome](http://fontawesome.io/)
* 淘宝 ICONFONT [ICONFONT](http://iconfont.cn/)
