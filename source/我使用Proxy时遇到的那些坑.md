---
title: "我使用proxy时遇到的那些坑"
brief: ""
date: 2020-01-18
tags: ["socks5", "proxy"]
---

作为一个经常利用国外资源进行学习的程序员，由于众所周知的原因，无时无刻都需要使用proxy来访问互联网。因为我在使用proxy的过程中遇到过非常多的坑，所以想要写一篇文章把这些坑记录下来，一是当作自己的一份笔记，二是如果有人遇到相同的问题，并通过搜索引擎查到这篇文章，也许能获得解决方案。
P.S.我对网络协议栈的了解非常浅显，所以这篇文章或许会有大量错误。

## 第一阶段：在路由器上使用V2Ray客户端

这是我在大一到大三时的方案。实现这个方案具体的步骤有：

- 有一台支持安装V2Ray客户端的路由器，我使用的路由器是斐讯 K2p A2，这也算是一台非常热门的路由器了，在恩山上能找到很多它的固件，大部分都已安装了SS/V2Ray的客户端。
- 有一台稳定可使用的V2Ray服务器，我自己是在搬瓦工上用19.99$/year的vps搭的，在校园网下表现异常优秀，我使用两年从未被封，带宽也足以在Youtube上观看4K视频。另外，我对低延迟没有需求。
- 在路由器上配置好转发规则，主要目的是让国内流量不走代理，否则访问国内网站时会出现一些问题。

这个方案我用得非常爽，完成配置后所有连上路由器的设备的流量都会在路由器上自动转发，完全不需要考虑其他问题。

最终不再使用这个方案的契机是，由于自建proxy server存在局限性，一是如果VPS不支持CN2等，在校园网之外的其他出口带宽不足的网络环境下（比如上海电信），带宽会严重不足；二是自建proxy server在近年来愈发严格的封ip策略下风险太高。所以我不再自建proxy server而是转用机场。

我使用过的机场的一大特点是，单台机器通常无法稳定使用，在风紧扯呼的时期，几乎需要每天更新订阅并频繁切换服务器。
由于K2p羸弱的性能，它的固件web端十分难用，在K2p的固件上使用SS/V2Ray的订阅完全是种折磨。

我最终没有选择换路由器（因为好的路由器太贵且体积太大，寝室没地方摆），而是开始为每台终端安装特定的客户端。此次切换正是我接下来诸多问题的开始。

## 第二阶段：在Windows上使用Clash的系统代理

[Clash](https://github.com/Dreamacro/clash)是大多数机场推荐使用的Windows客户端，它非常好用，体验远超我在Windows上使用过的其他所有VPN/其他proxy客户端。这里不再赘述它的优点，相信使用过的人都有所体会。

使用Clash的步骤简单无比，在Profiles页面添加订阅，再在首页开启System Proxy，即可。

Clash的默认架构大概是，在`localhost:7890`启了一个proxy server来转发http和https流量，然后在`localhost:7891`启了一个socks5的proxy server。流量经由这两个本地的proxy server发往远端的proxy server。

而在Clash上点开System Proxy，实际上是在windows的proxy settings里设置了如下内容

![test](https://raw.githubusercontent.com/Sphish/gitpress-docs/master/pictures/Image_20200118.png)

问题就出在这个windows的系统代理上。

我在使用系统代理的过程中遇到两个主要的问题：

- WSL的流量不走系统代理
- Windows Store里的应用，也就是UWP应用，由于某种安全策略，在system proxy被设置为`localhost:xxxx`时会不work。

第一个问题是我在使用git的时候发现的，某一天晚上我在使用git时奇慢无比，我一开始以为用https方式git clone应该是走系统代理，就以为速度慢是github的锅（毕竟也有先例）。但后来过了一晚上还是有这个问题，我就尝试了一些其他绕过GFW的办法，发现速度一下子变快了！这才知道WSL不走系统代理。

第二个问题就是巨坑了。我频繁使用的UWP应用有三个，Mail、Microsot To-Do和Zenkit。我发现在开启代理的时候，这三个应用怎么也连不上网，windows也没有给出任何提示，我在duckduckgo上搜索"Windows 10 mail app doesn't work with system proxy"也根本无法找到解决方案！我一开始还以为是这几个App自身的问题。几经辗转，才搜索得知[所有Windows Store应用都有这个问题！](http://disq.us/p/187u6gk)（解决方案也在这个链接下有提到，另外，Clash客户端也有一键解决这个问题的按钮，就是首页的"UWP Loopback"）。

由这两件事，我认识到“不确定哪些App使用了proxy”实在是危害巨大，所以，我不再使用system proxy。

## 第三阶段：为每个应用分别设置proxy

在windows上，我常用的需要走代理的软件无非两个，Firefox和Telegram。这两个软件都支持socks proxy，所以直接在设置里面把socks proxy设置为`127.0.0.1:7891`即可。

### 在WSL里使用proxy

我主要使用的WSL是Ubuntu 18.04，我在网上查到的设置系统代理的方式是设置`HTTP_PROXY`, `HTTPS_PROXY`和`ALL_PROXY`这几个环境变量。

在我还没有决定不再使用系统代理的时候，我在WSL里设置了这几个环境变量。

但我马上就遇到了问题，比如pip install不work。我没有保存错误信息的截图，自此之后我就也不再在WSL里使用环境变量来设置代理了，原因与不在Windows里使用系统代理相似。

总之，在WSL里最需要用到代理的软件也就是git, pip, apt, wget等。在网上都能搜到诸多解决方案。比如pip可以使用[pysocks](https://pypi.org/project/PySocks/)。值得一提的是，类似以ssh的方式git clone时，由于不是http/https流量，所以设置代理的方式会有些特殊，比如[How to force Git to use socks proxy over its ssh connection?](https://stackoverflow.com/a/58253407)。

## 在Android上使用Sufboard

Android上使用proxy就没有那么多坑，现在大多数App都支持按照App来转发流量，比如我使用的Sufboard。

---
总而言之，在使用代理的时候，只要清楚地知道哪些App走了代理，哪些App没有走代理，就能避免掉很多潜在地坑，即使遇到问题，也能快速排查。

## 其他可能的方案

- [在 Windows 10 上使用 Hyper-V 安装 LEDE 软路由](https://blog.skk.moe/post/hyper-v-win10-lede/)
