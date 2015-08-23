---
layout: post
title: "有关博客的加载速度"
date: 2015-01-03 21:34:00 +0800
comments: true
categories: [blog,配置]
---
## 起因
有童鞋反应说加载速度很慢，这是一个关于举着保护旗号却在做愚蠢的事的悲伤的故事。
<!--more-->
## GFW
GFW 是一切的原因，因为octopress需要加载google的字体库，本来是件多好到事情，怎耐GFW插足，你说有GFW在能不慢吗？网上找到了解决方法，发现360竟然把这字体库搬到墙里来了，这是360唯一一次让我还觉得做了件靠谱的事了。

那么怎么解决问题呢？除了修改_config.yml，将墙外到花花世界注释掉之外，主要从两head.html进行配置调整。
## 开始调整
### 1. /source/_include/custom/head.html
网上大多的方法是去掉这个文件里的google字体设置，但这个配置就两行google的字体库，删除之后还有存在到意义吗？这时候看到说，360把这个字体库搬进来了，真实干了件靠谱到事了。如是修改：

	//fonts.googleapis.com/
修改为

	//fonts.useso.com/
即可。
好激动，立马rake preview，咋回事，没效果，底部怎么在加载 'ajax.googleapis.com'，又是哪个api在加载？经过一番排查，终于找到。
### 2. /source/_include/head.html
这个配置里面搜索一下，找到'googleapis'关键字，然后忍痛把这行脚本注释了。重新preview, 终于是加载快了，但是，总觉得这样不好。
## 我的想法
google 的许多api是多么好到福利，减轻了多少开发者的工作量，然而这么好到工具只因社会意识形态和政治原因而粗暴地屏蔽，历史上也有闭关锁国的愚蠢行为，希望以史为鉴乃。
