---
layout: post
title: "SSH 通道的代理设置"
date: 2015-05-11 20:45:38 +0800
comments: true
categories: [ssh,代理,git,配置]
---
## 背景和问题
在学校实验室，一般都通过服务器配置的代理来上网，因为这样方便，但更重要的是网速！然而代理上网常常会遇到一些小麻烦，在之前的网志里面也讲到一些，今天要来分享的是和ssh 通道相关的代理配置的解决方案。

<!--more-->
需要用到ssh 通道的地方还挺多的，比如这个问题的源头：通过代理方式向我的github 托管库中提交代码。由于系统已经设配置好了代理，可以通过http 通道直接提交代码，然而这样的话每次都要输入密码，而使用ssh 通道会提示连接失败的错误。网上有类似的问题提出，并且也有许多解决办法。有些是通过修改git 的配置来实现的，感觉这不是从本质上解决问题，本质上应该是配置ssh 来解决问题。当然，我提供的解决方案也是站在巨人的肩上。

## connect
[connect](https://bitbucket.org/gotoh/connect) 提供了可以让ssh 通道走代理的小工具源码,拿到后编译成可执行文件，记得修改权限：

```
$ gcc -o connect connect.c
$ sudo chmod +x connect
$ mv connect ~/bin/connect
```
然后在``` ~/.ssh/``` 中创建config 文件如下：
	
	ProxyCommand /home/username/bin/connect -S proxyhost:port %h %p  
	Host github.com
	User git
	Port 22
	Hostname github.com
	IdentityFile "/home/username/.ssh/id_rsa"
	TCPKeepAlive yes
	IdentitiesOnly yes

	Host ssh.github.com
	User git
	Port 443
	Hostname ssh.github.com
	IdentityFile "/home/username/.ssh/id_rsa"
	TCPKeepAlive yes
	IdentitiesOnly yes
把其中的_username_ 替换为你的主机名字，_proxyhost:port_ 替换为你使用的代理地址和端口，然后测试了以下，可以使用代理走通ssh 通道了：

```
$ ssh -T git@github.com
Hi diufanshu! You've successfully authenticated, but GitHub does not provide  access.
```

## 新的问题
还没来得及高兴，旧发现了一个小问题，我在实验室的另一台机器上部署了git 服务，供实验室内部使用，通过上面的修改，竟然不能在本地的git 服务器上提交代码了。也就是说这个小工具在走代理的时候不能走局域网了，当然最好的办法是把这个小工具的代码改一改，然后重新编译。然而经过一些折腾，发现有更好的办法，就是修改上面的config 文件：
	
	Host github.com
		User git
		Port 22
		Hostname github.com
		IdentityFile "/home/username/.ssh/id_rsa"
		TCPKeepAlive yes
		IdentitiesOnly yes
		ProxyCommand /home/username/bin/connect -S proxyhost:port %h %p  

	Host ssh.github.com
		User git
		Port 443
		Hostname ssh.github.com
		IdentityFile "/home/username/.ssh/id_rsa"
		TCPKeepAlive yes
		IdentitiesOnly yes
		ProxyCommand /home/username/bin/connect -S proxyhost:port %h %p  

这么改的话，只有当请求github.com 和 ssh.github.com 时会使用这个小工具进行代理。测试了一下，果然成功了。

## For Windows
在Windows 下当然也可以用类似的方法，把connect.c 编译成可执行文件就行了，不过，所谓踏破铁鞋无觅处，git 在 win 下的版本自带了这个工具，在git 的安装目录下bin 文件夹里，这个工具已经编译成了connect.exe 可执行文件，直接用吧。当然需要在你的 ssh 默认配置文件夹里添加这个config文件，下面供参考：

	Host github.com
		User git
		Port 22
		Hostname github.com
		IdentityFile "/d/Users/username/.ssh/id_rsa"
		TCPKeepAlive yes
		IdentitiesOnly yes
		ProxyCommand /bin/connect.exe -S proxyhost:port %h %p  

	Host ssh.github.com
		User git
		Port 443
		Hostname ssh.github.com
		IdentityFile "/d/Users/username/.ssh/id_rsa"
		TCPKeepAlive yes
		IdentitiesOnly yes
		ProxyCommand /bin/connect.exe -S proxyhost:port %h %p  

接下来，愉快地玩耍吧。
