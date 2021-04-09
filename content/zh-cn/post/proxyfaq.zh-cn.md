---
title: "关于科学上网的流行说法"
date: 2020-05-04T16:17:00+08:00
slug: "proxyfaq"
tags: 
  - "Proxy"
type: post
categories:
  - Other
---

## 前言

本文意在分析一些经常被提及、引起争论的问题。

## 为什么不用 V2RAY/Trojan

由于直连打击力度较大，个人自建需要使用更不易被封锁的协议。很多人宣传 V2RAY Trojan 等，事实上对直连也许是有一定帮助的。

但是大部分机场都是中转线路，使用 V2RAY 显得太重，消耗服务器资源大，而实质上并没有什么帮助，因此更青睐老牌的、生态更好的 Shadowsocks 及 ShadowsocksR。

Trojan 也是类似的。

因此发表一些“不用 V2RAY 不怕被封吗”之类的言论，并不明智。

以下是部分大佬对此的看法。(如有冒犯，请联系删除)

![STC-nov2ray](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/nov2ray.png)

![ssrcloud-nov2ray2](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/nov2ray2.png)

![rixcloud-nov2ray3](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/nov2ray3.png)

![gu-notrojan](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/notrojan.png)
(本句在当时语境下类似于玩笑话，其本人无任何贬低 Trojan 的意思。)

不可否认，新的协议的出现是必然且有意义的，但其出现一般都是为了解决现有问题。

例如 V2RAY 试图解决的是严重封锁，而 Trojan 试图解决的是 V2RAY 过重的问题。

而机场线路，公网隧道、专线，大概没有此类需求。

此外，Shadowsocks 与 ShadowsocksR 的生态相对成熟。

关于 Shadowsocks 与 ShadowsocksR 有如下说法：

1. Shadowsocks 每个用户需要占用一个端口，部署可能不便。而 ShadowsocksR 支持端口复用模式[^1]。
2. ShadowsocksR 停止维护，且加密程度相对 Shadowsocks 更高，意味着效率、性能相对较劣，但有传 ShadowsocksR 能缓解 QOS 问题[^2]。

目前看来，Shadowsocks 协议支持最为广泛。

以上内容为笔者个人见解，如有谬误，敬请斧正。

## IPLC 天下第一吗？

首先，看看[毒药博客](https://www.duyaoss.com/archives/57/)中的介绍。

- IPLC: "International Private Leased Circuit"的缩写，即“国际专线”。不过大部分机场通常看到的 iplc，都只是阿里的经典网络，跨数据中心内网互通，阿里内网，并不是严格意义的 iplc 专线；当然也有其他渠道的，或真 iplc，不过比较少。阿里云的内网互通底层原理是通过采购多个点对点的 iplc 专线，来连接各个数据中心，从而把各个数据中心纳入到自己的一套内网里面来。这样做有两个好处，其一是 iplc 链路上的带宽独享，完全不受公网波动影响，其二是过境的时候不需要经过 GFW，确保了数据安全且不受外界各种因素干扰。但是需要注意一下阿里云的 iplc 也是有带宽上限的，如果过多的人同时挤到同一条专线上，峰值带宽超过专线的上限的话也同样会造成网络不稳定。其他渠道购买到的 iplc 价格很高，阿里云内网这种性价比超高这种好东西且用且珍惜。
- IEPL 国际以太网专线（International Ethernet Private Line,简称 IEPL)，构建于 MSTP 设备平台上，基于 SDH 传输技术，采用 GFP 封装，传输协议透明，物理层隔离，带宽保证，提供一层点对点数据专线服务。IEPL 是以太网接口的增强型的 IPLC 产品，是一个端到端的专享管理频宽服务，拥有高度灵活性及最高物理层网络安全功能，让客户轻松管理广域网和高带宽需求的应用。通俗的总结一下，IEPL 使用某种技术，实现了两点之间的高稳定性的数据传输。这与 IPLC,MPLS 是差不多的效果，只是实现的技术手段有区别而已。其实对机场来说，是 IPLC 或者 IEPL 或是 MPLS-VPN，都没有太大区别。认定是保障带宽的不过墙专线就行了，至于是用哪种技术方案实现的，根本不需要纠结。
- BGP：BGP 的实际意思通常是一个 IP 在多个运营商的网络中均为直连，不经过第三运营商，利用 iptables 或相关软件通过将去海外 VPS 的流量加一层国内转发。不过不同机场的定义也不一样。正常来说比如 rixcloud，指的是他们入口是阿里云、部分落地节点是自己起了 BGP（去 Peer 了 GitHub、微软、Cloudflare 等等）。不过多数机场把 BGP 定义为阿里云公网中转等。
- 中转：将数据从一个服务器重定向到另外一个服务器。机场中比较常见的是阿里云公网中转，其实也有很多其他中转，也有自己去买其他的 BGP 来中转的。
- 「原生 IP」：所谓原生 IP 通常意思为当地运营商原本就拥有的 IP；广播国家一般和注册国家相同（一般的 IP 库不会误判地区），一般不用于公有云计算服务或 IP 声誉好，一般能够用来解锁 Netflix、HBO、Hulu 以及其它有限制的流媒体服务
- 流媒体解锁：很多流媒体服务平台如 Netflix 会出于版权原因而限制一些特定 IP 的访问。一般来说网络运营商（如 HKT）自己持有的 IP，比如商宽、家宽，极少被屏蔽，因为这些 IP 大多是流媒体服务商的目标客户在使用。家宽 IP 被屏蔽的几率是最低的，很多 ISP 的家宽都是动态 IP 的，很难精准封杀。固定 IP 的商宽其次。这些流媒体服务商也怕误杀导致投诉，比如 GCP 的 IP 段被投诉之后又可以看 Netflix 了。IDC 商家所持有的 IP 一般会被屏蔽，越大越有名的 IDC 持有的 IP 被屏蔽的几率越高。很多 IDC 会租用运营商的 IP 从而绕过此类封杀，但是这种方式并不是万无一失的，翻车案例比较多我就不再一一例举了。所以除非是商宽、家宽，其他所谓的“原生 IP 解锁流媒体”都是有几率翻车的。
- CN2:是电信的精品骨干网。首先 CN2 是一张运营商骨干网，就像电信 163、中国移动这一类是同样性质的东西，但相对电信 163 来说会有更好的稳定性。不是什么专线（和 iplc 那类东西区分清楚）。既然是公网，那它本质上就是个很多人共享使用的东西。共享的东西都存在一定程度的不稳定性。区别在于它的超售比较低，相对电信 163 来说会有更好的质量。CN2 的跨国数据通讯也需要经过 GFW 审查，不存在不过 GFW 的公网。

在谈及 IPLC 时，一般泛指专线。专线相对于公网中转有如下优点：

1. 不经过 GFW，也就是所谓的不过墙。
2. 链路宽带独享，不受公网波动影响。

一个中转节点应该是这样的：

![Transfer](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/TransferPic.png)

### IPLC 速度快

一般来说，用户更关心的速度是“网速”，也就是宽带大小。

而使用专线并不能改变宽带的容量，因此盲目地认为 IPLC 网速快是毫无根据的。

### IPLC 延迟低

专线传输，理论上延迟的确更低。
排除掉机场过度超售、宽带跑满丢包、软件出锅等问题，专线在此方面的确有优势。
虽然一般用户无需过多在意延迟。

### IPLC 稳定

专线传输的稳定，主要体现在链路宽带独享，不容易受公网波动影响。
但上游依然会出问题，例如机房事故等等。
而高质量的公网可靠性并不差。

### IPLC 安全

不经过 GFW 的处理，在某种意义上安全了。
但专线传输数据仍受到监管，如 ssrcloud 的中日专线有人访问邪教网站遭到警告。

注重隐私请使用 no-log VPN、多层跳板等方式，而不要轻信提供科学上网服务的机场还能保护你的安全。
一般人无需对此过于敏感。

### IPLC 贵

专线宽带的价格的确高昂。

下图为花卷科技的 IPLC 专线宽带价格。

![IPLCPRICE](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/hjkjiplc.png)

即使有钻石分销[^3]的六折，每 Mbps 也高达 210RMB，每 100 Mbps 价格要达到两三万元。

然而，其实还有更高贵的。如下图，北京动态 BGP 全穿透多线宽带。

![ucacheprice](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/ucacheprice.png)

上图为微网聚力[^4]官网的价格表，可以看到最高贵的宽带每 100 Mbps 高达五万元每月。

事实上，单比较价格意义不大，因为可以这样部署：

![rixCloud](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/rixcloudmode.png)

### 总结

以上内容亦非绝对，笔者目的不在于否定专线等等，而在于提醒各位读者更理性、全面地看待线路，而不要迷信专线等。

100Mbps 的专线给 100 人用，体验可能并不如 1Gbps 的公网隧道给 100 人用。

其实，用户对节点线路无需过多关注，而只需要关注用户体验。用起来好就行，不必在意机场主用了什么神秘的操作。

最后，来点佩奇笑话。

![NOIEPL](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/ieplpeggy.png)
![HuaJi](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/huaji.png)

## 订阅转换偷订阅

不可否认，在技术上的确是可行的。
然而，让我们看看搭建订阅转换服务的都是什么大佬吧。

### bianyuan

[边缘订阅转换 API](https://bianyuan.xyz/)，笔者使用的第一个订阅转换 API。

Sabrina，博客地址为 [https://merlinblog.xyz/](https://merlinblog.xyz/)，N3RO 机场的大佬。

撰写了大量教程，对社区的贡献毋容置疑。对于转换偷订阅的说法，其本人亦有过回应：

![SabrinaReply](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/sabrinareply2.png)

### nameless13

[https://api.nameless13.com/](https://api.nameless13.com/)，也是相当知名的订阅转换 API 了。

Nameless13，魅影的管理员，撰写了[魅影各类教程](https://docs.nameless13.com/)。

### gfwsb.114514.best

[https://gfwsb.114514.best/](https://gfwsb.114514.best/)，这域名震撼我一万年……

TindyX，subconverter[^5] 项目的作者。

### 总结

以上只是订阅转换的一部分。笔者想借此说明的是，搭建这种公益服务者通常为大佬，并不会眼馋你的订阅。
当然并不排除有作恶者存在。
实在担心，建议使用本地版的 subconverter[^5]，避免此问题。

## Surfboard 快

笔者无端猜测，有小白看了某知名 youtuber 的各大软件速度对比后，认为 Surfboard 更快。

笔者未进行相关测试，但必须指出：
软件并不是网速的主要影响因素，且无明显缺陷的软件，在速度上并不会有太大差异。

网速的主导因素，是线路质量。包括但不限于设备到路由器、家庭宽带到国内入口、国内入口到国内出口、国内出口到国际落地、国际落地到目标服务器的线路质量。

Surfboard 是一个相当优秀的客户端，但请不要以“网速最快”等说法夸赞推荐它，否则可能引发不必要的争端。

### 关于网速的一些思考

使用 youtube 进行网速测试，会受到硬件性能等因素影响，测速应使用更专业的工具。

据传，旧版本的 Clash For Android 使用的内核有一定的性能问题，已经更换[^6]。(来源不详，不必当真)

理想的中高端机场，应当能达到 100Mbps 及以上的速率，选用软件对网速的影响不必多虑。

### 关于软件速度的一些思考

一般来说，功能越复杂，速度就越慢。在路由器上跑规则复杂的 Clash，可能对网速有较大影响。

使用 PC 客户端，基本不用考虑。移动设备可能有一定影响，但并不大，无需太在意。

此处速度为泛指，结论纯粹笔者臆想，不足为据。

## 总结

撰写本文的目的，并不是想否定、批评广为流行的某些说法，而是想否定盲目地迷信某些说法的行为。
例如，对机场不提供 V2RAY 提出质疑的小白，吹捧 IPLC 强无敌的小白，认为 Surfboard 天下第一的小白等等……

本文也无意提出什么说法，而仅仅想说明：没有什么是绝对的。请理性评判，不要迷信权威。

That's All.

[^1]: [ShadowsocksR 端口复用](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/portreuse.png) [原文](https://rixcloudkb.io/faq/)


[^2]: [ShadowsocksR 缓解 QOS 问题](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/ssrdeqos.png) [原文](https://rixcloudkb.io/faq/)


[^3]: (花卷科技钻石分销)[https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/zsfx.png] [原文](https://www.betaidc.com/pricing.html)


[^4]: 微网聚力价格表 [原文](https://www.ucache.cn/idc/services/hosting/)


[^5]: 几乎所有订阅转换都基于 [subconverter](https://github.com/tindy2013/subconverter)。


[^6]: [CFA Release](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/cfaupdate.png)
