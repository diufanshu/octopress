---
layout: post
title: "TexLive 跨版本更新"
date: 2016-06-20 22:25:52 +0800
comments: true
categories: [latex]
---

前段时间，TexLive 2016 发布了，也就意味着2015 版本不再更新。官方是不让跨版本升级的，各大论坛也如是说。然而我发现并非如此，而且需要做的很简单。
<!--more-->

举例来说，如果从[中科大镜像站](http://mirrors.ustc.edu.cn/CTAN/)作为仓库源，那么下载[update-tlmgr-latest.sh](http://mirrors.ustc.edu.cn/CTAN/systems/texlive/tlnet/update-tlmgr-latest.sh)，然后执行，就可以了。如果你觉得要再做什么，那么就把本地的```2015```路径名改为``2016``，并修改相应环境变量为```2016```即可，然后愉快地更新吧。