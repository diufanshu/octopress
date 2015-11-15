---
layout: post
title: "科学上网"
date: 2015-02-15 10:51:58 +0800
comments: true
categories: [代理,网络]
---
{% blockquote Sky Chan %}
科学上网是必须技能。
{% endblockquote %}

##Goagent 的局域网/公网配置
goagent 的强大不必多说了吧，项目在github托管:[goagent](https://github.com/goagent/goagent)，大家可以通过git clone 的方式来获得最新版本，常见问题在其[FAQ](https://github.com/goagent/goagent/blob/wiki/FAQ.md)里都有解答，“为什么要叫 goagent，而不叫 GoProxy？”“一开始叫 GoProxy 的，后来 Hewig 说软件名字带有 proxy 字样不祥，于是就改成了 goagent。”<!--more-->

goagent 时常会更新版本，更新后要及时看文档，如果说要重新上传appid，那就再上传一次。一般来说，启动的时候，会自动导入证书，但是也不排除例外，如果出现类似于 The requested URL /_gh/ was not found on this server. 的问题，Don’t panic! 在浏览器里重新导入一次local 目录里的 CA.crt证书即可。

基本用法大家想必都会，就讲一讲局域网内共用一个goagent 的实例吧。

### 实验环境

模拟大多数高校实验室以及公司内部上网的环境，有一台服务器，可以直接连Internet，处在同一网段内的其他终端通过http/https 代理的方式来上网，比如说其主机IP为：10.11.12.13，port为8080，这时候要怎么设置才能科学上网捏？

### 配置方法

+ __方法1__：在自己的终端配置goagent

这个设置比较简单，只要直接修改 proxy.ini 即可，找到[proxy]项，这样填写：

	[proxy]
	enable = 1
	autodetect = 1
	host = 10.11.12.13
	prot = 8080
	username = [如果需要就填]
	password = [和上面一样]

然后就可以了，和平常一样。

+ __方法2__ :在服务器配置goagent

在服务器上运行goagent，然后其他终端只要填写其监听地址即可，可以通过SwitchyProxy/SwitchyOmega/Proxyfoxy 这些代理切换插件来切换监听地址。这个方法的设置也不复杂，只需修改 proxy.ini 即可(不列举每项的所有栏目了)：

	[listen]
	ip = 0.0.0.0
	port = 8087
	...

	[dns] (如果启用的话)
	enable = 1
	listen = 0.0.0.0:53

然后在代理切换插件(以SwitchyOmega为例)里新建情景模式作为科学上网用：

__代理服务器__: 10.11.12.13

__代理端口__: 8087

至此，就可以快乐的上网了。这里提供的这两种方法，其实都是参考了项目主页里的[Configurations](https://github.com/goagent/goagent/blob/wiki/ConfigIntroduce.md.ini)。

那么，还能再给力点吗，我喜欢用pac 方式自动判断需要代理和不需要代理的网站，关键是这种方式能有效过滤广告乃！经过一番实验，终得实现，下面来分享一下过程。

+ __隐藏方法__ : 基于方法2

首先想到的是，在代理切换插件里面新建pac情景模式，然后导入pac文件就行。说干就干，以SwitchyProxy为例，在新建的pac情景模式中，填写PAC 网址：

	http://10.11.12.13:8086/proxy.pac
注意端口是proxy.ini 里[pac]项设定的端口，此处默认设定是8086。这个proxy.pac文件其实位于10.11.12.13服务器goagent目录里的local目录下。随后，规则就导入进来了，测试了一下，科学上网毫无压力，然而，那些本不需要代理的网站都不能上了。排查原因，原来是这个proxy.pac 的规则中，所有不需要代理的网址都归类成'Direct'，就是不使用任何代理，所以终端的机器当然不能访问这些网站了。那么，就直接把它替换掉，也就是说，编辑proxy.pac，然后把所有 'Direct'替换成：

	'PROXY 10.11.12.13:8080'
也就是用来普通代理上网的代理地址。在SwitchyProxy 中成功看见了规则的在线更新，然后就愉快的上网去咯！

然而，故事还没结束，在第二天，国内的网站又不能上了。怎么回事，PAC 规则怎么又变成原来的样子了！原来这个proxy.pac 文件是自动生成的，由于规则是直接从其他地方获取，然后每天重新生成一次，可以从proxy.ini 中的[pac]项看出：

	[pac]
	enable = 1
	ip = 0.0.0.0
	port = 8086
	file = proxy.pac
	admode = 1
	adblock = https://easlist-downloads.adblockplus.org/easylistchina.txt
	gfwlist = https://autoproxy-gfwlist.google.com/svn/trunk/gfwlist.txt
	expired = 86400
里面基本都说明了，所以说，要不就不让它更新，也就是注释掉expired 项目，要不就要修改proxy.pac 的生成代码了。自动更新这么好的东西怎么能把它关掉，当然是修改代码咯。

于是乎，在proxy.py 的881 行，找到了需要修改的位置，源码如下：

	default = 'PROXY %s:%s' % (common.PROXY_HOST,common.PROXY_PORT) if common.PROXY_ENABLE else 'DIRECT'

修改为：

	default = 'PROXY %s:%s' % (common.PROXY_HOST,common.PROXY_PORT) if common.PROXY_ENABLE or common.PAC_IP != '0.0.0.0' else 'DIRECT'
	
这么修改后，需要将proxy.ini 中的[pac]项的ip 和[proxy]项进行修改

	[pac]
	enable = 1
	ip = 10.11.12.13
	...

	[proxy]
	enable = 0
	autodetect = 1
	host = 10.11.12.13
	prot = 8080
	...

注意，是关闭proxy，但指定了监听地址，也就是说，通过判断pac的监听ip来确定是否是在用局域网代理上网。当然了，生效的话需要等到下一次更新proxy.pac，等不及了，就把[pac]项的expired 修改为1，然后重启goagent，虽然不是立竿见影，但一会儿后，实验用的终端机器可以真正使用pac 方式科学上网了，查看pac文件，确实按照我想要的方式变化了。当然，最后还是要把失效时间改回来，毕竟每秒都更新也太喜新厌旧了吧！

### 相关福利
在google code issue 里面看到有位热心的同学贡献了一堆appid，这是福利，你懂的！！

	zimofanqiang|zimofanqiang1|zimofanqiang10|zimofanqiang11|zimofanqiang12|zimofanqiang13|zimofanqiang14|zimofanqiang15|zimofanqiang16|zimofanqiang17|zimofanqiang18|zimofanqiang19|zimofanqiang2|zimofanqiang20|zimofanqiang21|zimofanqiang22|zimofanqiang23|zimofanqiang24|zimofanqiang3| zimofanqiang4|zimofanqiang5|zimofanqiang6|zimofanqiang7|zimofanqiang8|zimofanqiang9|zhangnraka|zhangnraka1|zhangnraka10| ​

#### 福利更新

	kqshu00|kqshu01|kqshu02|kqshu03|kqshu04|kqshu05|kqshu06|kqshu07|kqshu08|kqshu09|kqshu10|kqshu11a|kqshu12|kqshu13|kqshu14|kqshu15|kqshu16|kqshu17|kqshu18|kqshu19|kqshu20|kqshu21|kqshu22|kqshu23|kqshu24
	
## IP 更新

经常使用得同学会发现，一段时间后goagent 里面满是黄色红色，这是因为IP 被墙了，仅仅更新goagent 有时不是很有用，这个时候，需要一款叫做[GoGoTester](https://github.com/azzvx/gogotester) 的软件来测试仍然有效的IP ，把proxy.ini 里面[iplist] 项中的google_cn 和google_hk 替换即可。

## ShadowSocks

需要有vps主机，可以找到很多免费的，用法比较简单。

## fqrouter

Android 端利器，可以分享给同网段任意手机甚至是PC！

## OpenVPN
又一移动端利器，似乎要配合一些例如Ovpn之类对配置文件获取应用才能使用，Google Play 毫无压力。
 
## 保持更新

前段段时间期末，一大堆作业，那次写的匆匆忙忙，这次稍作修改，持续保持更新！

