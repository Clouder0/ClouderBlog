---
title: "一种基于 OICQ 框架的自动化群员活跃指数统计方法"
date: 2022-08-27T16:17:03+08:00
slug: "water-kings"
type: post
tags:
  - javascript
categories:
  - Programming
---

标题人话版：调库统计某群中群员发言次数并降序排序。

今天 ~~闲着没事~~忙里偷闲 打算加一下计算机科学大群里发言过的同学的 QQ 好友。

经过了手动翻找后，我备受折磨、饱感痛苦。~~都是你们太能水了！~~

于是我决定写一个程序来 automate 这个无聊的重复劳动。

算法设计：

* 调用 API 获取历史聊天记录
* 获取发言人名单列表，再去重导出即可。

由于需求过于简单，只需要学习 OICQ 很小的一个 subset.

OICQ 基于 node.js，刚好我几天前速成前端学了 javascript，因此应当能比较快上手。

## Hands On

按照 [README](https://github.com/takayama-lily/oicq) 中的说法，使用 Template 快速创建。

```bash
git clone https://github.com/takayama-lily/oicq-template.git
cd oicq-template/
npm i
```

根据建议阅读 node.js 入门，typescript 5 分钟上手。  
最后我懒得读了，直接上吧。

修改 `index.js` 中的 account，然后 `npm run dev` 直接开始。

同 ip 下手机扫码登陆即可。

### Plugin Development

基于需求写一个 plugin 加载就可以了。

![image](https://assets.b3logfile.com/siyuan/1609132319768/assets/image-20220827125823-crxmkvn.png)​

在 `index.js` 中加载自己的插件。

插件的操作主要都是 event based，那我希望直接获取，简单粗暴，因此参考上线问候插件：

![image](https://assets.b3logfile.com/siyuan/1609132319768/assets/image-20220827125934-9ty2r7y.png)​

绑定 `system.online` 事件。

我需要主动根据群号获取到 Group Instance，然后 fetch 消息记录，那么需要引入 client.

可以看到在 `index.js` 中，调用 createClient 方法创建了 bot 变量，那么引入这个 bot 就行了。

使用 `client.pickGroup()` 获取 Group Instance.

![image](https://assets.b3logfile.com/siyuan/1609132319768/assets/image-20220827130457-ctd8x86.png)​

再调用 `getChatHistory` 来获取历史消息记录。

![image](https://assets.b3logfile.com/siyuan/1609132319768/assets/image-20220827130649-j28j1ei.png)​

从最后一条发言往前反复获取。

不如再顺便记录一下每个 id 的发言次数。既然如此，那把群昵称也加上吧。

写 async/await. 感觉代码写得一塌糊涂，果然对 js 还是太不熟练了。。。

大体就是：

* 先通过群号获得 Group 对象。
* 调用 Group 对象的 getChatHistory 方法，将消息加入 msgs 数组中。记录上一次的最早消息，便于下次接着获取。
* 遍历 msgs 数组，获取 sender 的 QQ 号，统计每个 QQ 号的消息条数。

  * 同时维护一个 QQ号 -> User 对象的 map，便于后续操作（获取 nickname）

    * ~~调 QQ 号获取 User 的 API 失败的权宜之计罢了~~
* 将 map 对象按消息条数降序排序。
* 遍历 map，输出 用户群昵称-QQ号-发言次数。

![image](https://assets.b3logfile.com/siyuan/1609132319768/assets/image-20220827160032-jsvh264.png)​

然后 `npm run dev` 运行即可。

## 结语

OICQ 似乎没有加好友的 API....

最终还是要手动加好友。

把前一百(30 msgs+)的都手动申请了一遍，重复体力劳动还是没有成功避免。

一共统计了 10w 条消息。

![image](https://assets.b3logfile.com/siyuan/1609132319768/assets/image-20220827155927-dfba67e.png)​

~~我亦是水王榜前三！~~

拙劣的隐私保护。

顺便说一下，我校的各种公共事务完全没有个人隐私的意识啊，全系的姓名学号手机号乃至身份证号都一览无余。。。

盒战争风险微存。~~表表主义的坏处！~~

![Cemetery94886135_p0](https://assets.b3logfile.com/siyuan/1609132319768/assets/Cemetery94886135_p0-20220827161137-0h4v1bw.jpg)​

## 未来方向

虽然这个随手为之的小玩具肯定没有未来，但不妨写写缺憾。

* ~~断点续传~~ 断点续统计。统计到一半卡一卡就前功尽弃了，有点难顶。实现这个可以定期把内存中的数据保存到硬盘上。
* 真正的活跃指数统计，比如持续发言加权、发言时间较近的加权等等。
* 基于本地 QQ 聊天记录分析，而不抓取漫游信息。

完。
