## 安装git

## 安装Node.js
见notes的另一篇文章

## 安装hexo

## 创建GithubPage

## 创建hexo站点

## 配置next主题
### 1、基本信息配置
> 基本信息包括：博客标题、作者、描述、语言等等。

打开 **站点配置文件** ，找到``Site``模块
```yaml
title: 标题
subtitle: 副标题
description: 描述
author: 作者
language: 语言（简体中文是zh-Hans, 英文是en）
timezone: 网站时区（Hexo 默认使用您电脑的时区，不用写）
```
关于 **站点配置文件** 中的其他配置可参考[站点配置](https://hexo.io/zh-cn/docs/configuration.html)

### 2、菜单设置
> 菜单包括：首页、归档、分类、标签、关于等等

我们刚开始默认的菜单只有首页和归档两个，不能够满足我们的要求，所以需要添加菜单，打开**主题配置文件**找到`Menu Settings`

```yaml
menu:
  home: / || home                          //首页
  archives: /archives/ || archive          //归档
  categories: /categories/ || th           //分类
  tags: /tags/ || tags                     //标签
  about: /about/ || user                   //关于
  #schedule: /schedule/ || calendar        //日程表
  #sitemap: /sitemap.xml || sitemap        //站点地图
  #commonweal: /404/ || heartbeat          //公益404
  ```
看看你需要哪个菜单就把哪个取消注释打开就行了；   
关于后面的格式，以`archives: /archives/ || archive`为例：   
`||` 之前的`/archives/`表示标题“归档”，关于标题的格式可以去`themes/next/languages/zh-Hans.yml`中参考或修改   
`||` 之后的`archive`表示图标，可以去[Font Awesome](https://fontawesome.com/icons?from=io)中查看或修改，`Next`主题所有的图标都来自[Font Awesome](https://fontawesome.com/icons?from=io)。   
### 3、Next主题样式设置
我们百里挑一选择了`Next`主题，不过`Next`主题还有4种风格供我们选择，打开**主题配置文件**找到`Scheme Settings`
```yaml
# Schemes
# scheme: Muse
# scheme: Mist
# scheme: Pisces
scheme: Gemini
```
4种风格大同小异，本人用的是`Gemini`风格，你们可以选择自己喜欢的风格。
### 4、侧栏设置
> 侧栏设置包括：侧栏位置、侧栏显示与否、文章间距、返回顶部按钮等等

打开**主题配置文件**找到`sidebar`字段
```yaml
sidebar:
# Sidebar Position - 侧栏位置（只对Pisces | Gemini两种风格有效）
  position: left        //靠左放置
  #position: right      //靠右放置

# Sidebar Display - 侧栏显示时机（只对Muse | Mist两种风格有效）
  #display: post        //默认行为，在文章页面（拥有目录列表）时显示
  display: always       //在所有页面中都显示
  #display: hide        //在所有页面中都隐藏（可以手动展开）
  #display: remove      //完全移除

  offset: 12            //文章间距（只对Pisces | Gemini两种风格有效）

  b2t: false            //返回顶部按钮（只对Pisces | Gemini两种风格有效）

  scrollpercent: true   //返回顶部按钮的百分比
```
### 5、头像设置
打开**主题配置文件**找到`Sidebar Avatar`字段
```yaml
# Sidebar Avatar
avatar: /images/header.jpg
```
这是头像的路径，只需把你的头像命名为`header.jpg`（随便命名）放入`themes/next/source/images`中，将`avatar`的路径名改成你的头像名就OK啦！
### 6、设置RSS(未验证)
1、先安装 hexo-generator-feed 插件
```shell
$ npm install hexo-generator-feed --save
```
2、打开**站点配置文件**找到`Extensions`在下面添加
```yaml
# RSS订阅
feed:
  type: atom
  path: atom.xml
  limit: 20
  hub:
  content:
  content_limit: 140
  content_limit_delim: ' '
```
3、打开 主题配置文件 找到rss，设置为
```yaml
rss: /atom.xml
```
### 7、添加分类模块
1、新建一个分类页面
```shell
$ hexo new page categories
```
2、你会发现你的`source`文件夹下有了`categorcies/index.md`，打开`index.md`文件将`title`设置为`title: 分类`
3、打开**主题配置文件**找到`menu`，将`categorcies`取消注释，分类模块即添加完成   

4、写文章时只需在文章的顶部标题下方添加categories字段，即可自动创建分类名并加入对应的分类中
举个栗子：
```markdown
title: 分类测试文章标题
categories: 分类名
```
### 8、添加标签模块
1、新建一个标签页面
```shell
$ hexo new page tags
```
2、你会发现你的`source`文件夹下有了`tags/index.md`，打开`index.md`文件将`title`设置为`title: 标签`
3、打开**主题配置文件**找到`menu`，将`tags`取消注释，标签模块即添加完成    

4、写文章时只需在文章的顶部标题下方添加tags字段，即可自动创建标签名并归入对应的标签中
举个栗子：
```markdown
title: 标签测试文章标题
tags: 
  - 标签1
  - 标签2
```
### 9、添加关于模块
1、新建一个关于页面
```shell
$ hexo new page about
```
2、你会发现你的`source`文件夹下有了`about/index.md`，打开`index.md`文件即可编辑关于你的信息，可以随便编辑。
3、打开 主题配置文件 找到`menu`，将`about`取消注释

### 10、添加搜索功能
1、安装 hexo-generator-searchdb 插件
```shell
$ npm install hexo-generator-searchdb --save
```
2、打开**站点配置文件**找到`Extensions`在下面添加
```yaml
# 搜索
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```
3、打开 主题配置文件 找到`Local search`，将`enable`设置为`true`

