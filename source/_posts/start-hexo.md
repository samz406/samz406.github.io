---
title: hexo博客搭建
date: 2021-06-16 18:32:46
tags: 技术
categories: 技术
---

hexo博客搭建

<!-- more -->

### 环境准备

- node.js 
  		node.js可以安装最新版本，目前我安装的是v.16.4.1

- Git 

- github仓库创建

   创建一个samz406.github.io的仓库

  ​		master分支放静态文件。也就是访问samz406.github.io可以看到的东西。

  ​		hexo就放写的文章

- 文章发版流程

  - 现在本地写好文章，生成静态文件，部署到github上。在云主机上clone一份仓库代码，每次有部署到github上就在云主机上git pull。静态代码是放在nginx上的。

- 多端写作

  ​		把hexo博客最基本配置都放到仓库上，拉下最新代码，本地执行下

  ​	hexo g，然后在安装next主题。基本可以跑起来了。感觉这个流程还可以优化下。

  > next的主题配置_config.yml是可以放到，仓库根目录下，命名为_config.next.yml

 

Tips:

​	我本地之前node版本是v8.16.0，安装Hexo是可以，但是执行Hexo命令失败。

会有如下提示

```js
samz406.github.io git:(hexo) ✗ hexo
console.js:35
    throw new TypeError('Console expects a writable stream instance');
    ^

TypeError: Console expects a writable stream instance
    at new Console (console.js:35:11)
    at Object.<anonymous> (/Users/lilin/.nvm/versions/node/v8.16.0/lib/node_modules/hexo-cli/node_modules/hexo-log/lib/log.js:31:17)
    at Module._compile (module.js:653:30)
    at Object.Module._extensions..js (module.js:664:10)
    at Module.load (module.js:566:32)
    at tryModuleLoad (module.js:506:12)
    at Function.Module._load (module.js:498:3)
    at Module.require (module.js:597:17)
    at require (internal/module.js:11:18)
    at Object.<anonymous> (/Users/lilin/.nvm/versions/node/v8.16.0/lib/node_modules/hexo-cli/lib/context.js:3:16)
    at Module._compile (module.js:653:30)
    at Object.Module._extensions..js (module.js:664:10)
    at Module.load (module.js:566:32)
    at tryModuleLoad (module.js:506:12)
    at Function.Module._load (module.js:498:3)
    at Module.require (module.js:597:17)
```



​	解决办法就是NVM(**Node Version Manager**)工具,我选择的是版本v.16.4.1

​	执行顺序

```
nvm install 16.4.1
nvm use 16.4.1  


```



### 安装Hexo 

	npm install -g hexo-cli

不出意外就安装成功了，

### 执行Hexo 命令

- 创建目录，folder为自定义

```
hexo init [folder]
```

-	创建文章 

```
hexo new [layout] <title>
```

​	如果没有设置 `layout` 的话，默认使用 [_config.yml](https://hexo.io/zh-cn/docs/configuration) 中的 `default_layout` 参数代替。如果标题包含空格的话，请使用引号括起来。

```
 hexo new "start hexo"
```

| 参数              | 描述                                          |
| :---------------- | :-------------------------------------------- |
| `-p`, `--path`    | 自定义新文章的路径                            |
| `-r`, `--replace` | 如果存在同名文章，将其替换                    |
| `-s`, `--slug`    | 文章的 Slug，作为新文章的文件名和发布后的 URL |

默认情况下，Hexo 会使用文章的标题来决定文章文件的路径。对于独立页面来说，Hexo 会创建一个以标题为名字的目录，并在目录中放置一个 `index.md` 文件。你可以使用 `--path` 参数来覆盖上述行为、自行决定文件的目录：



- 生成静态文件

```
$ hexo generate 或者hexo g
```

| 选项             | 描述                                                         |
| :--------------- | :----------------------------------------------------------- |
| `-d`, `--deploy` | 文件生成后立即部署网站                                       |
| `-w`, `--watch`  | 监视文件变动                                                 |
| `-b`, `--bail`   | 生成过程中如果发生任何未处理的异常则抛出异常                 |
| `-f`, `--force`  | 强制重新生成文件 Hexo 引入了差分机制，如果 `public` 目录存在，那么 `hexo g` 只会重新生成改动的文件。 使用该参数的效果接近 `hexo clean && hexo generate` |
|                  |                                                              |

部署

```
hexo deploy
```

这个可以将静态文件部署到github上，直接通过samz406.github.io。就可以访问部署后的网站。

这几个命令基本就可以额了。

### 设置主题 

 	这里选择next主题

​		进入博客目录执行

```shell
git clone https://github.com/theme-next/hexo-theme-next themes/next
```

​	然后就是一些基本配置，文件在 themes/next/_config.ym

### **选择Schemes**

打开 _config.next.yml，首先可以看到 Scheme Settings，里面提供了四种模式。

```text
# Schemes
scheme: Muse
#scheme: Mist
#scheme: Pisces
#scheme: Gemini
```

### **设置站点icon**

​	需要icon放在 source/img/目录下

```yaml
favicon:
 small: /img/avatar.jfif
 medium: /img/avatar.jfif
 apple_touch_icon: /img/avatar.jfif
 safari_pinned_tab: /images/logo.svg
```

### **设置菜单栏**

在menu下，可以设置菜单显示内容，此版本next支持font awesome图标，可以去**[官网](https://link.zhihu.com/?target=https%3A//fontawesome.com/)**寻找你喜欢的图标进行替换。注：相应的菜单栏需要有对应的页面才能打开，不然会404哦！

```yaml
menu:
 home: / || fa fa-home
 #about: /about/ || fa fa-user
 tags: /tags/ || fa fa-tags
 categories: /categories/ || fa fa-th
 archives: /archives/ || fa fa-archive
 guestbook: /guestbook/ || fa fa-book
 #schedule: /schedule/ || fa fa-calendar
 #sitemap: /sitemap.xml || fa fa-sitemap
 #commonweal: /404/ || fa fa-heartbeat
```

Tips:

>  创建分类 hexo new page categories	
>
> 创建 tags hexo new page tags

​		

### **设置已读进度条**



reading_progress的enable设置为true即可，color为颜色。

```yaml
# Reading progress bar
reading_progress:
 enable: true
 # Available values: top | bottom
 position: bottom
 color: "#37c6c0"
 height: 3px
```

### 常用配置

 怎么配置文章分类

​	1 在next主题_config_next.yml中配置中加入分类菜单

```yaml
menu:
  home: / || fa fa-home
  about: /about/ || fa fa-user
  #tags: /tags/ || fa fa-tags
  categories: /categories/ || fa fa-th
  archives: /archives/ || fa fa-archive
  #schedule: /schedule/ || fa fa-calendar
  #sitemap: /sitemap.xml || fa fa-sitemap
  #commonweal: /404/ || fa fa-heartbeat

# Enable / Disable menu icons / item badges.
```

​		

​	2 创建分类 hexo new page categories	

​    3  进入source/categories目录，修改index.md基本信息

```markdown
---
title: 分类
date: 2021-07-03 21:55:20
type: categories
comments: true
sitemap: false
---

```

​		



添加图片可以用这个插件

https://www.jianshu.com/p/f72aaad7b852