---
title: "Clash 教程"
date: 2020-04-02T21:10:52+08:00
slug: "clashtutorial"
type: posts
tags:
  - proxy
categories: 
  - Tech
---

## 前言



想写一个 Clash 的教程很久了，但由于学业繁重一直都没有抽出时间。最后决定拿碎片时间来攒出这篇文章了，因此上下逻辑可能不够紧密。

因为有很多小白不会使用 Clash 这款强大的软件，时常在群里询问相关问题，而笔者当初学习使用时也遇到了一些困难，因此想写一篇从零开始的教程。

由于笔者也算不上对其多么精通，因此必有疏漏，欢迎各位斧正。

笔者使用 Windows10 系统,Clash For Windows 0.9.1 替换 clashrdev v0.18.0.2 内核，subconverter 在文章部分使用 0.4.2 版本，部分使用 0.4.4 版本，应当不影响阅读。

文章篇幅较长，请确保有足够的时间来阅读，也可先收藏，日后慢慢阅读。~~不是来骗收藏的~~



## 文档



Updated at 2020-4-15 19:09:16.

补充一些文档。



[非常详细的英文文档](https://lancellc.gitbook.io/clash/)

[Clash 内核文档](https://clash.gitbook.io/doc/)

[Nameless13 的 Clashr 教程](https://docs.nameless13.com/shr)

[F 大的 CFW 文档](https://github.com/Fndroid/clash_for_windows_pkg/wiki)



## 介绍



本文的主角是 `Clash: A rule-based tunnel in Go.`，作者为 *Dreamacro*，[Github 地址](https://github.com/Dreamacro/clash)。

那么它与 ShadowsocksR 等类似客户端有何区别呢？笔者引用 Readme 中的 Features 进行说明。



### Features



- Local HTTP/HTTPS/SOCKS server with/without authentication

- VMess, Shadowsocks, Trojan (experimental), Snell protocol support for remote connections. UDP is supported.

- Built-in DNS server that aims to minimize DNS pollution attacks, supports DoH/DoT upstream. Fake IP is also supported.

- Rules based off domains, GEOIP, IP CIDR or ports to forward packets to different nodes

- Remote groups allow users to implement powerful rules. Supports automatic fallback, load balancing or auto select node based off latency

- Remote providers, allowing users to get node lists remotely instead of hardcoding in config

- Netfilter TCP redirecting. You can deploy Clash on your Internet gateway with iptables.

- Comprehensive HTTP API controller



简单地概括一下：



- 支持本地 HTTP/HTTPS/SOCKS 代理。

- 支持 VMess, Shadowsocks, Trojan(实验中)， Snell 协议，支持 UDP。

- 内置 DNS 服务器用于抗 DNS 污染，支持 DoH([DNS over HTTPS](https://zh.wikipedia.org/wiki/DNS_over_HTTPS)) 和 Dot([DNS over TLS](https://zh.wikipedia.org/wiki/DNS_over_TLS)) 以及 Fake IP。具体可以查看[官方文档](https://github.com/Fndroid/clash_for_windows_pkg/wiki/DNS%E6%B1%A1%E6%9F%93%E5%AF%B9Clash%EF%BC%88for-Windows%EF%BC%89%E7%9A%84%E5%BD%B1%E5%93%8D)。

- 根据域名、IP 所属地理位置、[IP CIDR](https://zh.wikipedia.org/wiki/%E6%97%A0%E7%B1%BB%E5%88%AB%E5%9F%9F%E9%97%B4%E8%B7%AF%E7%94%B1)、端口等规则将数据包分流到不同节点。

- 节点分组，组内支持 故障转移、负载均衡、延迟最低、手动选择 模式。

- 远程 providers 可以方便地更新节点。(当前较少机场提供，仅笔者个人认知。)

- 网络过滤 TCP 重定向。你可以在网关部署 Clash。(笔者对该点并不了解，概括很有可能错误。)

- 易于理解的 HTTP API controller。



而 Clash For Windows(下文简称 CFW ) 还有两个贴心的功能，[UWP Loopback](https://github.com/Fndroid/clash_for_windows_pkg/wiki/UWP%E5%BA%94%E7%94%A8%E8%81%94%E7%BD%91) 和 [TAP 模式](https://github.com/Fndroid/clash_for_windows_pkg/wiki/%E5%90%AF%E5%8A%A8TAP)，前者可以解决 win10 UWP 应用代理的问题，后者可以让所有流量都走代理。

笔者未使用过 `TAP 模式`，因此想了解可以查看相关文档。



Clash 功能核心在于**分流**，举个例子你可以让 Telegram 走游戏线路，让 Netflix 走解锁节点，让广告直接被拦截……

![RuleExample](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/RuleExample.png)

这是笔者使用的分组，将节点依照地区分组，使用延迟最低模式，再选择地区节点。



## 客户端



虽然有了 Clash Core，但一般人不能直接使用，要使用各平台对应开发的客户端。

Windows 下使用 Clash For Windows(CFW)，Android 下使用 Clash For Android(CFA)，macOS 可以使用 ClashX(笔者没有使用过 mac 系统)。

除此之外，还有替换内核的 ClashR 可以使用，支持了 ShadowsocksR 协议。

由于内核相同，各个平台使用应当大同小异，笔者将针对 CFW 进行讲解。



## 导入订阅



### 托管



如果你使用的机场支持 Clash 托管，那么使用会方便不少。只需要在 Profile 的面板中，将托管地址复制到输入框中，点击 Download 按钮，即可开始使用。

![ProfileDownloadExample](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/ProfileDownloadExample.png)

然后点击方片，待方片亮起后即为启用了。(笔者找不到什么名词来描述这个方块状的东西了)

在 Proxies 面板中应当可以看到节点和托管自带的规则。



### 订阅转换



如果你使用的机场没有对 Clash 进行支持，可以使用 [Subconverter](https://github.com/tindy2013/subconverter) 将 Shadowsocks 订阅转换为 Clash 订阅。

如果替换内核使用 ClashR，那么 Subconverter 也支持将 ShadowsocksR 订阅转换为 Clash 订阅。

此处将只介绍最简单的转换，更多的操作将在后文详细介绍。



#### 使用在线 API 转换



网上已有不少基于 Subconverter 搭建的订阅转换 API，可以直接使用。



- [https://bianyuan.xyz](https://bianyuan.xyz)

- [https://api.nameless13.com](https://api.nameless13.com)

- [https://gfwsb.114514.best](https://gfwsb.114514.best)

- [https://subconverter-web.now.sh](https://subconverter-web.now.sh)

- [https://subconverter.herokuapp.com](https://subconverter.herokuapp.com)



界面都大同小异，笔者以 https://bianyuan.xyz 进行举例。

选择基础使用版，将订阅链接扔到编辑框中。



![APIExample](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/APIexample.png)



可以看到生成了托管地址，随便选一个，按照托管的导入方法导入即可使用。

关于隐私问题，笔者认为并不需要过多考虑，如果实在在意，可以搭建本地 API 转换。



#### 搭建本地 API 转换



下载[最新版 Subconverter](https://github.com/tindy2013/subconverter/releases)，笔者使用的是 Subconverterv0.4.3。

将文件解压到一个单独的文件夹。



![SubconverterDir](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/SubconverterDir.png)



打开 subconverter.exe 后，会弹出一个黑色窗口。



![SubRunnning](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/SubRunning.png)



看到 `Startup completed.` 说明已经启动成功了。

要转换到 Clash，则托管地址为 `http://127.0.0.1:25500/sub?target=Clash&url=订阅地址`，后按导入托管的操作即可。

要转换到 ClashR，则托管地址为 `http://127.0.0.1:25500/sub?target=ClashR&url=订阅地址`

导入后，在黑窗中应能看到 `Generate completed.`，完成后关闭 Subconverter 即可，要更新订阅时重新打开。



## 应用代理



该内容面向 CFW，介绍几种使用代理的方式。



### System Proxy



打开 CFW 后，默认会开启 System Proxy(HTTP 代理)。



![CFWSYSPROXY](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/defaultsystemproxy.png)



地址为 `127.0.0.1:7812`，可以在面板中看到。

打开系统设置，Windows Settings -> Network & Internet -> Proxy，可以看到已经应用到系统代理了。



![ProxySettings](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/SettingsProxy.png)



此时 CFW 的托盘图标是一只棕色(或许我是个色盲)的小猫。



此时支持系统代理的应用都会走代理了。



### Socks5



然而有时我们希望更精细地控制代理，而非直接简单粗暴地设置成系统代理，避免很多时候不想代理的被代理了。

尽管 Clash 的分流已经是一种解决方案了，但有时不免有疏漏，而关闭系统代理，设置 Socks5 代理是一种行之有效的方法。



![ClashSocks5](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/ClashSocks5.png)



可以看到此处的 Socks Port 为 7813，那么本地 Socks5 代理地址就是 `127.0.0.1:7813`。

应用中需要有 Socks5 代理的设置项，才能为该应用启用代理。



#### Telegram



![TelegramSocks5](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/TelegramSocks5.png)

为 Telegram 启用 Socks5 代理，Settings -> Advanced -> Proxy settings -> Use custom proxy



#### Firefox



![FirefoxSocks5](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/FirefoxSocks5.png)

为 Firefox 启用 Socks5 代理，Settings -> General -> Network Settings -> Use custom proxy

建议勾选使用 SOCKS v5 时代理 DNS 查询。



#### Chrome



![ChromeSocks5](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/ChromeSocks5.png)

为 Chrome 启用 Socks5 代理，需要安装扩展，`Proxy SwitchyOmega`。



#### Other



其他应用可以在设置中寻找有关代理的设置，如果没有那就无法使用 Socks5 代理。



该方法的好处在于可以让想走代理的应用走代理，不想走的应用不走代理，但局限性在于有的应用不支持代理。

关闭 System Proxy 后，CFW 的托盘图标会变成一只小黑猫。



### UWP Loopback



Windows 10 下的 UWP 应用使用代理，需要额外设置。[详情可以点击了解](https://github.com/Fndroid/clash_for_windows_pkg/wiki/UWP%E5%BA%94%E7%94%A8%E8%81%94%E7%BD%91)。



![ClashUWP](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/ClashUWP.png)



打开时可能有报错，酌情选择，一般没什么关系。

打开后，界面如图：



![UWPHELPER](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/UWPHELPER.png)



勾选你想要代理的应用，或者直接点击 Exempt All 全部勾选，然后点击 Save Changes 即可。



### TAP



由于笔者没有使用过 TAP 模式，该部分内容暂不可用。



## 配置规则



分流规则，可定制性是很高的。这也是将 Clash 打造成你想要的样子的办法。



![ProfileBoard](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/ProfileBoard.png)



在 Profile 面板可以看到类似的东西，从左到右的图标依次为：文本模式编辑配置、节点分组、编辑规则、复制、信息、更新。

文本模式编辑配置是比较常用的方法，但考虑到上手较为困难，将放在最后讲解。



### 节点分组



顾名思义，将节点分为若干组。

分组的作用不只有美观好看，策略组是有实际功能的。



![ProxyGroup](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/ProxyGroup.png)



策略组有：延迟最低、故障转移、手动选择、负载均衡 四种模式。

延迟最低，顾名思义，每隔一段时间进行延迟测试，选择延迟最低的节点。

故障转移，每次都选组内第一个节点，无法使用再换到第二个，依次类推。

手动选择，顾名思义，没有特殊功能。

负载均衡，每个节点都用用，由于很多机场都有连接数的限制，因此实际使用较少。



![GroupBoard](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/GroupBoard.png)



在此页面可以编辑策略组。

可以向已有的策略组中拖动图标，向其中添加策略组或节点。

可用的选项分为三大类：  Special Proxies,Proxy Groups,Proxies.

Special Proxies，包括 DIRECT(直连) 和 REJECT(拦截)。

Proxy Groups，即用户自己添加编辑的策略组。

Proxies，即真实可用的节点。



笔者将真实可用的节点按地区划分，分入了地区组中，在地区组中使用**延迟最低**来选择节点，而在分流时直接选择地区节点。

仅供参考，分组可以按照个人喜好。



需要注意的是，有的机场并不喜欢用户进行 url-test 等操作，会给服务器周期性地带来大量连接，如 ssrcloud 就曾对连接数过多的用户进行警告。

这时可以选择 select 模式。

编辑完之后记得点击 Save。



### 编辑规则



![EditRules](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/EditRules.png)



一般来说，subconverter 转换的订阅和机场的托管都会自带一些规则。

Clash 的规则有三大要素： Content、Type 和 Proxy or Policy。



- Content 即关键字，往往配合 Type 起效。

- Type 即类型，有 DOMAIN-SUFFIX、DOMAIN、DOMAIN-KEYWORD、IP-CIDR、SRC-IP-CIDR、GEOIP、DST-PORT、SRC-PORT 和 MATCH。

- Proxy or Policy 即指定规则将分流到哪个节点/策略组。



当有多个规则可以匹配时，将会匹配到最先出现的规则。即在文本中位于最前面的规则。

笔者将对每个类型进行讲解。



#### DOMAIN-SUFFIX



域名后缀，顾名思义，举个简单的例子。

在 Content 中填写 `google.com`，那么：



- 访问 `www.google.com` 时，匹配到域名后缀为 `google.com`。

- 访问 `test.google.com` 时，匹配到域名后缀为 `google.com`。

- 访问 `google.com` 时，匹配到域名后缀为 `google.com`。



上述情况都会匹配。

该类型为最常用的类型之一，如果你遇到了无法访问的网站，就可以将其域名后缀添加为规则。



#### DOMAIN



匹配域名，相对于域名后缀更为严格。

比如在 Content 中填写 `google.com`，那么：



- 访问 `www.google.com` 时，域名为 `www.google.com`，不会匹配。

- 访问 `google.com` 时，域名为 `google.com`，可以匹配。



也是一种常用的类型，可以进行更细微的配置。



#### DOMAIN-KEYWORD



匹配域名关键字，更为模糊的规则。

比如在 Content 中填写 `google`，那么：



- 访问 `google.com` 时，匹配到关键字 `google`。

- 访问 `www.google.com` 时，匹配到关键字 `google`。

- 访问 `fakegoogle.com` 时，匹配到关键字 `google`。



需要注意的是，只能匹配域名而非整个网址，例如：

访问 `fake.com/google` 时，域名为 `fake.com`，不能匹配。



#### IP-CIDR



根据 ip 段进行分流。

这方面笔者不甚了解，在网上找到了一篇博文，[可以点击查看](https://blog.csdn.net/han156/article/details/77817031)。

如有大佬原意补充，可以回复评论。

根据我的理解举一个例子吧，比如在 Content 中填写 `172.111.0.0/16`，那么：



- 访问 `某一个网站` 时，解析 ip 属于 `172.111.0.0/16`，则匹配。



#### SRC-IP-CIDR



通过源 ip 所属的 ip 段进行分流。

这方面笔者亦不甚了解，猜测是根据客户端 ip 进行分流。

比如：

在局域网 `192.168.1.1/24` 中，有一台主机 `192.168.1.10` 开启了 Clash，而另一台主机 `192.168.1.9` 连接代理 `192.168.1.10:7813`。

此时其访问网站，可以匹配 `192.168.1.1/24` 的 SRC-IP-CIDR 的规则进行分流。



上述内容很有可能错误，如有大佬愿意补充……



#### GEOIP



通过解析的 ip 所属地进行分流。

比如在 Content 中填写 `CN`，那么：



- 访问 `www.baidu.com` 时，解析得到的 ip 在数据库中属于中国，匹配。

- 访问 `www.google.com` 时，解析得到的 ip 在数据库中不属于中国，不匹配。



可以以此达到绕过中国 IP 的效果，也可以用对应地区节点连接对应地区 IP。



#### DST-PORT



通过目标端口进行分流。



比如在 Content 中填写 `8080`，那么：



- 建立一个 `xxx.xxx.xxx.xxx:8080` 的 TCP 连接，匹配。

- 访问 `xxx.xxx.xxx:8080`，匹配。



再比如在 Content 中填写 `443`，那么：



- 访问 `https://xxx.xxx.xxx`，也会匹配。



#### SRC-PORT



通过源端口进行分流。



比如在 Content 中填写 `10086`，那么：



- 建立一个 `localhost:10086->xxx.xxx.xxx.xxx:xxx` 的连接，匹配。



#### MATCH



全匹配，一般放在最后用来捕捉漏网之鱼。



### 文本模式编辑配置



相信此时你已经知道如何对节点分组，如何定制自己的规则。

但使用 GUI 进行配置仍然有些低效，此时可以选择编辑文本配置文件。

在 Profile 面板点击小铅笔，Edit in Text Mode，使用你喜欢的工具打开。

应该长这个样子：



```
port: 
socks-port:
allow-lan:
mode: 
log-level:
external-controller:
experimental:
Proxy:
Proxy Group:
Rule:
```



具体可以查看[官方 README](https://github.com/Dreamacro/clash/blob/master/README.md) 中的 Config 部分的 example 进行了解。

其中最长的部分应当是 `Proxy` `Proxy Group` 和 `Rule`。



#### Proxy



即代理，用来添加代理节点。

一般来说，普通用户不会手动更改此处内容，而是使用 subconverter 转换。



#### Proxy Group



即对节点进行分组。



```
 - name: 组名
   type: 分组类型(select/urltest/fallback/load-balance)
   proxies:
     - 节点1
     - 节点2
     - 分组1
```



其中 proxies 不仅可以添加节点，还可以添加策略组。

分组在前文有提及。用文本编辑可能可以更方便地复制粘贴。



#### Rule



即规则，每条规则都应该是如下格式：



```
  - type,content,proxy
```



用文本编辑规则快捷方便，而且可以利用工具大量处理。

这就不得不提及 [ACL4SSR](https://github.com/ACL4SSR/ACL4SSR/tree/master/Clash) 了，比如拿到一个 list 文件，内容如下：



```
# OneDrive
PROCESS-NAME,OneDrive
PROCESS-NAME,OneDriveUpdater
USER-AGENT,OneDrive*
USER-AGENT,OneDriveiOSApp*
DOMAIN-KEYWORD,1drv
DOMAIN-KEYWORD,onedrive
DOMAIN-KEYWORD,skydrive
DOMAIN-SUFFIX,livefilestore.com
DOMAIN-SUFFIX,oneclient.sfx.ms
DOMAIN-SUFFIX,onedrive.com
DOMAIN-SUFFIX,onedrive.live.com
DOMAIN-SUFFIX,photos.live.com
DOMAIN-SUFFIX,skydrive.wns.windows.com
DOMAIN-SUFFIX,spoprod-a.akamaihd.net
DOMAIN-SUFFIX,storage.live.com
DOMAIN-SUFFIX,storage.msn.com
#DOMAIN-SUFFIX,aria.microsoft.com
```



而要让其走名为 `OneDrive` 的策略组，只需要在每行行尾增加 `,OneDrive` 即可。

当然，行首还需要添加 　　-　 。

其中有部分规则是不支持的，需要删去。



## 配置 subconverter



定制了自己的规则后，如果需要更新订阅，就会覆盖之前的配置。

除了重新配置之外，还可以通过配置 subconverter 来实现规则的可持久化。~~可持久化规则是什么奇怪的数据结构~~

前文已经提及了 subconverter 的简单用法，此时需要对其进行更深入的了解。

建议阅读一遍[官方文档](https://github.com/tindy2013/subconverter/blob/master/README-cn.md)。



### 创建 Profile



笔者推荐为每一个机场创建一个 Profile，不仅可以缩短订阅链接增加美观性，还能统一地进行管理(比如在手机端、PC 端上更新)。

在 subconverter 根目录下，找到名为 profiles 的子目录，进入，创建一个 ini 文件即可。



```
[Profile]
target=clashr
url=你的订阅链接
config=config/你的config配置文件.ini
emoji=true
filename=名字
```



在订阅链接中填写的配置，都可以放在 Profile 中填写。

此处的 config 文件可以在根目录下名为 config 的子目录找一个来填写，后文会介绍自己写 config 文件。

随后在根目录下打开 pref.ini 文件，找到：



```
;Access token used for performing critical action through Web interface
api_access_token=password
```



填一个密码，因为在内网部署一般不需要太高强度。

最后的订阅链接为：

`http://127.0.0.1:25500/getprofile?name=profiles/你的profile文件名.ini&token=你的密码`



### 创建 Config



建议对着 `ACL4SSR_Online_Full.ini` 先学习一番。



```

[custom]

surge_ruleset=🎯 全球直连,https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/LocalAreaNetwork.list

surge_ruleset=🛑 广告拦截,https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/BanAD.list

surge_ruleset=🍃 应用净化,https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/BanProgramAD.list

surge_ruleset=Ⓜ️ 微软云盘,https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/OneDrive.list

surge_ruleset=🍎 苹果服务,https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/Apple.list

surge_ruleset=📲 电报消息,https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/Telegram.list

surge_ruleset=📹 油管视频,https://raw.githubusercontent.com/lhie1/Rules/master/Surge/Surge%203/Provider/Media/YouTube.list

surge_ruleset=🎥 奈飞视频,https://raw.githubusercontent.com/lhie1/Rules/master/Surge/Surge%203/Provider/Media/Netflix.list

surge_ruleset=📺 巴哈姆特,https://raw.githubusercontent.com/lhie1/Rules/master/Surge/Surge%203/Provider/Media/Bahamut.list

surge_ruleset=🌏 亚洲媒体,https://raw.githubusercontent.com/lhie1/Rules/master/Surge/Surge%203/Provider/AsianTV.list

surge_ruleset=🌍 国外媒体,https://raw.githubusercontent.com/lhie1/Rules/master/Surge/Surge%203/Provider/GlobalTV.list

surge_ruleset=🎯 全球直连,https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/GoogleCN.list

surge_ruleset=🔰 节点选择,https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/ProxyGFWlist.list

;surge_ruleset=🎯 全球直连,https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/ChinaIp.list

surge_ruleset=🎯 全球直连,https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/ChinaDomain.list

surge_ruleset=🎯 全球直连,https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/ChinaCompanyIp.list

surge_ruleset=🎯 全球直连,https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/Download.list

surge_ruleset=🎯 全球直连,[]GEOIP,CN

surge_ruleset=🐟 漏网之鱼,[]FINAL



custom_proxy_group=🔰 节点选择`select`[]♻️ 自动选择`[]🇭🇰 香港节点`[]🇨🇳 台湾节点`[]🇸🇬 狮城节点`[]🇯🇵 日本节点`[]🇺🇲 美国节点`[]🚀 手动切换`[]DIRECT

custom_proxy_group=🚀 手动切换`select`.*

custom_proxy_group=♻️ 自动选择`url-test`.*`http://www.gstatic.com/generate_204`300

custom_proxy_group=📲 电报消息`select`[]🔰 节点选择`[]♻️ 自动选择`[]🇸🇬 狮城节点`[]🇭🇰 香港节点`[]🇨🇳 台湾节点`[]🇯🇵 日本节点`[]🇺🇲 美国节点`[]🚀 手动切换`[]DIRECT

custom_proxy_group=📹 油管视频`select`[]🔰 节点选择`[]♻️ 自动选择`[]🇸🇬 狮城节点`[]🇭🇰 香港节点`[]🇨🇳 台湾节点`[]🇯🇵 日本节点`[]🇺🇲 美国节点`[]🚀 手动切换`[]DIRECT

custom_proxy_group=🎥 奈飞视频`select`[]🎥 奈飞节点`[]🚀 手动切换`[]DIRECT

custom_proxy_group=📺 巴哈姆特`select`[]🇨🇳 台湾节点`[]🔰 节点选择`[]🚀 手动切换`[]DIRECT

custom_proxy_group=🌍 国外媒体`select`[]🔰 节点选择`[]♻️ 自动选择`[]🇭🇰 香港节点`[]🇨🇳 台湾节点`[]🇸🇬 狮城节点`[]🇯🇵 日本节点`[]🇺🇲 美国节点`[]🚀 手动切换`[]DIRECT

custom_proxy_group=🌏 亚洲媒体`select`[]DIRECT`[]🇭🇰 香港节点`[]🇨🇳 台湾节点`[]🇸🇬 狮城节点`[]🇯🇵 日本节点`[]🚀 手动切换

custom_proxy_group=Ⓜ️ 微软云盘`select`[]DIRECT`[]🔰 节点选择`[]🇺🇲 美国节点`[]🇭🇰 香港节点`[]🇨🇳 台湾节点`[]🇸🇬 狮城节点`[]🇯🇵 日本节点`[]🚀 手动切换

custom_proxy_group=🍎 苹果服务`select`[]DIRECT`[]🔰 节点选择`[]🇺🇲 美国节点`[]🇭🇰 香港节点`[]🇨🇳 台湾节点`[]🇸🇬 狮城节点`[]🇯🇵 日本节点`[]🚀 手动切换

custom_proxy_group=🎯 全球直连`select`[]DIRECT`[]🔰 节点选择`[]♻️ 自动选择

custom_proxy_group=🛑 广告拦截`select`[]REJECT`[]DIRECT

custom_proxy_group=🍃 应用净化`select`[]REJECT`[]DIRECT

custom_proxy_group=🐟 漏网之鱼`select`[]🔰 节点选择`[]♻️ 自动选择`[]DIRECT`[]🇭🇰 香港节点`[]🇨🇳 台湾节点`[]🇸🇬 狮城节点`[]🇯🇵 日本节点`[]🇺🇲 美国节点`[]🚀 手动切换

custom_proxy_group=🇭🇰 香港节点`url-test`(港|HK)`http://www.gstatic.com/generate_204`300

custom_proxy_group=🇨🇳 台湾节点`url-test`(台|新北|彰化|TW)`http://www.gstatic.com/generate_204`300

custom_proxy_group=🇸🇬 狮城节点`url-test`(新加坡|坡|狮城|SG)`http://www.gstatic.com/generate_204`300

custom_proxy_group=🇯🇵 日本节点`url-test`(日本|川日|东京|大阪|泉日|埼玉|沪日|深日|JP)`http://www.gstatic.com/generate_204`300

custom_proxy_group=🇺🇲 美国节点`url-test`(美|波特兰|达拉斯|俄勒冈|凤凰城|费利蒙|硅谷|拉斯维加斯|洛杉矶|圣何塞|圣克拉拉|西雅图|芝加哥|US)`http://www.gstatic.com/generate_204`300

custom_proxy_group=🎥 奈飞节点`select`(NF|解锁|Netflix|NETFLIX)



enable_rule_generator=true

overwrite_original_rules=true



clash_rule_base=https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/GeneralClashConfig.yml

```



一般来说，只需要增减改 surge_ruleset 与 custom_proxy_group 即可满足需求。



#### surge_ruleset



形如：



```
DOMAIN-KEYWORD,1drv
DOMAIN-KEYWORD,onedrive
DOMAIN-KEYWORD,skydrive
DOMAIN-SUFFIX,livefilestore.com
DOMAIN-SUFFIX,oneclient.sfx.ms
DOMAIN-SUFFIX,onedrive.com
DOMAIN-SUFFIX,onedrive.live.com
DOMAIN-SUFFIX,photos.live.com
DOMAIN-SUFFIX,skydrive.wns.windows.com
DOMAIN-SUFFIX,spoprod-a.akamaihd.net
DOMAIN-SUFFIX,storage.live.com
DOMAIN-SUFFIX,storage.msn.com
```



同时支持在线与本地，可以使用在线的 ACL4SSR 的 list，当然也可以本地。

一般来说，放在 rules 子目录中，在 config 中添加：



```
surge_ruleset=你想走的策略组或节点,rules/你的list.list
```



需要注意的是，根据出现先后顺序生成规则，因此如果你想让某个 list 有更高的优先级，在文件开头写它。



#### custom_proxy_group



这个可以玩出很多花样，推荐查看官方文档自行学习。

仅介绍最基础的形式：



```
custom_proxy_group=组名`类型`策略组1`策略组2`策略组3`
custom_proxy_group=组名`类型`(正则)`测速url(如果需要的话)`测速周期(如果需要的话)
```



更多的使用方法请查看官方文档，可以做到倍率分组等等等等等等好玩的功能。

It's up to you.



## More Possibilities



到此，笔者的文章差不多完结了。

上述的内容应当可以满足需求了，然而 Clash 的功能、特性还有很多，笔者也仅仅是个不熟练的使用者。

在本部分，笔者会补充一些可能并不常规但能让人眼前一亮的用法。

**以下内容纯属笔者臆想，未曾实践，仅供参考。如有尝试并成功者，欢迎评论告诉笔者 qwq**



### 局域网代理



Clash 的工作模式大概是这样：



Client -> Socks5/HTTP Proxy -> Clash -> Server



一般会使用设置系统默认 http 代理和用插件使用 Socks5 代理的方法来连接。

那么，这个代理地址是否可以给局域网内其他用户使用呢？

应该是可以的。



![AllowLan.png](https://cdn.jsdelivr.net/gh/Clouder0/myPics/img/AllowLan.png)



然后，在 General YAML 选项下打开配置文件，将 `allow-lan: ` 设为 `true`。

在别的主机使用 `内网ip:端口` 即可连接对应的代理。

笔者在本机开虚拟机用此办法成功连接了(使用 Socks5)，未做其他尝试。

比如在电脑开着 CFW，而在手机上直接使用 HTTP 代理。

再比如在某台性能强悍的电脑上开 CFW，在路由器中使用 `内网ip:端口` 的 Socks5 节点，将开 CFW 的机器加入黑名单，理论上能实现这样的效果：



Client -> Router -> PC with CFW -> Server



个人感觉可以缓解路由器性能不足的问题，但笔者不使用路由器科学上网，未曾尝试。



### 搭配其他代理软件



在 Trojan 尚未被支持的时候，笔者曾见到有人如此操作。

同样的，笔者也未曾亲自尝试，仅从理论上思考了可能性。

即开启其他的代理软件，如 Trojan SSR 或其他小众代理软件，一般都会开放一个 HTTP 代理/SOCKS5 代理。

而 Clash 是支持将其当做代理节点使用的。

那么将其当做代理节点使用，可以使其享受 Clash 的分流。



Client -> Socks5/HTTP Proxy(produced by Clash) -> Clash -> Socks5/HTTP Proxy(produced by Trojan/SSR etc.) -> Trojan/SSR etc. -> Server



问题是，这样只能将流量分去那个代理软件，而不能指定那个代理软件使用哪个节点。

因此可能缺乏实用性。



### 使用 Provider 提供节点



Clash-Provider，笔者认为是类似于 Nodelist 的东西，可以直接提供节点。

那么编辑本地配置文件后，只需要根据 Clash-Provider 更新节点信息就可以更新订阅，而不必覆盖配置了。

但笔者还没有使用过提供 Provider 的机场，因此无法详细介绍。

另外，subconverter 中也提供了以 Provider 格式生成的选项。

期待其普及的一天。



### 使用 Clash 中转



在某一次更新中，出现了一个叫做 `relay` 的策略组类型。

这是 example 中的配置：



```
  # relay chains the proxies. proxies shall not contain a proxy-group. No UDP support.
  # Traffic: clash <-> http <-> vmess <-> ss1 <-> ss2 <-> Internet
  - name: "relay"
    type: relay
    proxies:
      - http
      - vmess
      - ss1
      - ss2
```



看起来，用户以后可能可以自己折腾中转了。

但笔者考虑到，现在很多机场已经采用如下架构了：



国内入口 -> 公网隧道/专线 -> 国际出口(落地)



那么如果用户自己再加一层中转，就会变成：



国内入口 -> 公网隧道/专线 -> 落地 -> 国内入口 -> 公网隧道/专线 -> 另一个落地



当然了，如果用来拯救直连节点还是不错的。用户也可以自己购买国内的机器来中转了。

以上也是笔者主观臆断的结果，事实上笔者还没有更新到这个版本……



## 尾声



这篇有点长的、好几天攒出来的教程终于完结了。

在撰写过程中，笔者有很多概念并不清楚，在网上查阅了相关资料，但仍有部分笔者力所不及，已在文中写明，以尽力避免误导读者。

但即便如此，笔者也不能保证内容的正确性，难免有疏漏错误，还请各位指正。

也希望各位读者能有所收获。

初完稿于 2020 年 4 月 3 日 11 时 19 分 47 秒。