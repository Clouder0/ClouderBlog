---
title: "抓取 Pixiv 元数据导入 Eagle"
date: 2022-12-03T02:35:44+08:00
slug: "pixiv2eagle"
type: post
tags:
  - python
categories:
  - Programming
---

# 抓取 Pixiv 元数据导入 Eagle

> Life is literally about doing something meaningless. Anything meaningful just provides you with the opportunity to devote yourself to what seems to make no sense.
>
> How absurd. But that's how art works.
>

## 前情提要

近期考虑给我的 AcgPic 做一点图片管理的工作。

考察了免费使用的 Billfish 后发现了若干的问题：

* Billfish 跑路记

  * 最终还是打算放弃 Billfish 了。这里写一写遇到的问题。
  * 首先，导入素材库如果选择索引模式，在原目录是不会监控的，新增的文件不会被自动索引。只能选择剪切、复制这两个才能做到动态更新。
  * 然后，大量标签(我这里是 6k 左右)打开标签管理假死，最后能勉强打开。在标签管理中搜索标签，动态刷新，在数量大时很卡，基本是不可用状态。
  * 标签相关的批量操作基本都没有，比如批量删除标签，批量将标签添加至标签分组。这部分功能我基本都要手写脚本来处理，直接对着数据库 hack.
  * 我这里有 svg 无法正常显示。  
    ​![image](https://assets.b3logfile.com/siyuan/1609132319768/assets/image-20221202194914-48qs565.png)​
  * 搜索备注时不全，似乎是不支持模糊搜索。这个很是神秘，也是我最终放弃使用的原因。
  * ​![image](https://assets.b3logfile.com/siyuan/1609132319768/assets/image-20221202195314-qmali5c.png)​
  * ​![image](https://assets.b3logfile.com/siyuan/1609132319768/assets/image-20221202195333-y2yfxol.png)  
    这张就没有搜出来。

> 拿着 [Pixiv2Billfish](https://github.com/Clouder0/Pixiv2Billfish) 这小脚本做了一些修改，做了标签过滤啥的，还是输在了 Billfish 本身不行上……唉，还是用 Eagle 吧。
>
> 不过如果我把 Artist 加成 Tag 似乎也可以做到我想要的功能……Hmmmm
>

于是我转战 Eagle，借鉴了 [Pixiv2Eagle](https://github.com/WriteCode-ChangeWorld/Tools/blob/master/0x09-Pixiv%E6%8F%92%E7%94%BBtag%E6%95%B0%E6%8D%AE%E5%AF%BC%E5%85%A5Eagle/pixiv2eagle.py) 与 [Pixiv2Billfish](https://github.com/Ai-desu-2333/Pixiv2Billfish) 后，由于它们的原代码风格不太符合我的习惯，于是另起炉灶从零开始写了一个小脚本。

​![image](https://assets.b3logfile.com/siyuan/1609132319768/assets/image-20221203021030-yufh1rz.png)​

​![image](https://assets.b3logfile.com/siyuan/1609132319768/assets/image-20221203021038-4elkwrv.png)​

## 开发记录

首先是调用 Eagle 的 API，没错！Eagle 有 API！当初对着 Billfish 直接调数据库果然还是太不优雅了。获取图片列表，进行下一步的处理。

接着解析图片名称获得 Pixiv Id，抓取 Pixiv 上的数据，这里直接参考了 pixiv2eagle 的做法，访问 `https://www.pixiv.net/ajax/illust/{pid}`​ 即可。

解析本身就有的 tags，再通过元数据生成 extra tags. 再通过元数据生成 Annotation.

最后把这些玩意扔回来，调用 API 写到 Eagle 里就好了。

Eagle 在标签很多情况下依然维持着良好的性能，并且标签可以批量加入分组，值得肯定。

不过遗憾的是 Eagle 没有提供标签相关的 API，如果后续做标签清理过滤的时候，可能还需要直接访问 json 文件。

## 使用

代码在 [Github](https://github.com/Clouder0/pixiv-metadata)上开源。使用 Python 编写，不建议完全无相关基础的小白使用。(我太懒了)

需要做一些简单的代码修改操作，主要是更改从文件名中提取 Pixiv Id 的正则串(在代码注释中提供了一种常见的格式)，还有调整获取目标 Eagle 图片的范围。(我是直接限定文件夹，更多的参数可以查看[文档](https://api.eagle.cool/item/list))

更具体的操作请看 README.

Made with Love & Leisure. Why so serious? Just some anime pics, kid.

> 30 天过期之后我就用不起 Eagle 了，怎么办呢……
>
