---  
title: "AnkiLink 完全体构想"  
date: 2021-07-12T21:14:57+08:00  
slug: "ankilink-complete"  
type: post  
tags:  
  - python  
  - anki  
categories:  
  - Programming  
---
  
AnkiLink 完全实现，必须依赖有块级别 ID 绑定的编辑器。目前来看，思源笔记天然具有这个优势。  
  
AnkiLink，之所以叫做「Link」，改名的时候就有考虑过。那就是双向性。  
AnkiImporter 是起点而非终点，AnkiLink **不仅仅要实现从「文档」到「Anki」，还要实现从「Anki」到「文档」的双向链接。**  
  
目前对于导入，将 AnkiIn 单独抽象出来成为一个库之后，加入思源格式的 json 解析只是加一个 Parser 的问题，可以说兼容非常好做。  
而主要难点就集中在**如何获取需要导入的内容**，这个大概率要用到思源内部的数据库查询。  
HTTP API 未开放之前，应该都只能观望。更具体地，对性能有要求，那么要查询的就是「有某种特殊标记的、在某个特定时间后被更改过的块」。  
  
思源内部的「属性」为配置提供了绝佳平台。AnkiIn 库本身也已经实现了配置功能，再加一层转换，将思源的属性转化为 AnkiIn 的配置，就能无缝对接了。  
这样一来，原本写在内容中的强干扰的配置就被屏蔽了。  
  
为了性能，不可能解析导入所有潜在的块，而要进行配置。对于这个问题，AnkiIn 加入了 `skip` 配置项，设定是否要跳过当前块。只需要将默认值设为 `true`，然后手动设置 `false`，就能做到选择性导入。  
此外，还有一个树形依赖的问题。在 AnkiIn 的 Markdown Parser 中，对语义树进行了从上到下的搜索，而将父亲节点的配置顺延到儿子节点中，同时让儿子节点重载配置项。这意味着，如果沿用这个模式，我们必须拥有在思源中找到最顶层的有效节点的方法。  
在 Markdown Parser 中，根节点天然是文档本身，而以块为粒度的思源结构却有希望做到更细化的程度。但这并不是简单的事情，所以可能还是要采用从最顶层节点开始遍历的方案。但还有一条路：自下向上地遍历。  
  
在与思源的对接中，需要注意的是：修改内容的模式发生了变化。往常的 AnkiIn 的工作模式是这样的：写好一个文档，然后转换它。但是在 AnkiLink 中，我们希望能随时随地修改内容，并且随时随地更新。也就是说，**碎片化的读写操作取代了原本的文档式的读写操作**，如果依然使用整个文档粒度的导入，那么效率是非常低下的。  
但我们前面提到了，我们配置了 `skip` 功能，**那么只要标记规范，实际上可以极大程度避免这个问题，无效的向下遍历会被快速剪枝，事实上需要遍历的只是有标记的子树。**但这样的效率有一个前提，**就是我们必须能以树形结构获取数据。**也就是说，我希望获取某个节点的儿子、父亲是谁，却不想要它的儿子、父亲的所有详细数据。  
**基于文件的数据获取是没有这种优势的。**如果我们依然使用文本获取数据，那么不幸的是，为了获取树形结构，我们必须对整个文件进行读入、解析，最终回到了起点。所以我们还需要等待 HTTP API.  
  
我们的使用场景可能是这样的：在 100 个文档中，每个文档修改了 1 个错别字。如果为了一百个错别字，就要将 100 个文档都从头解析一遍，我觉得这是不可接受的。  
所以我的期望是这样：我们知道这一百个文档都修改了 1 个错别字，将这个错别字块找出来，然后寻找他的祖先，继承他祖先的所有属性，再回到它本身。  
这样一来，对于一个小修改，我们只需要 $O(h)$ 的时间复杂度，只需要遍历树高个节点。  
当然，这依然存在不足：如果节点们拥有同样的祖先，却依然反复查询获取，反而凭空增加了复杂度。自下而上的溯源会有这种弊端，但这是可以避免的。只需要在内存中做一个简单的缓存就能解决这个问题，虽然不是那么完美。  
  
对于 Anki，我们将思源笔记的块 ID 直接存放在 Anki 的卡片中。当思源中发生了内容变更，我们只需要寻找对应的卡片，进行相应的修改。如果找不到相应的卡片，就进行添加。而对于删除就困难一些，因为我不知道思源是否会保存被删除的块的历史。如果没有的话，删除这一块，我希望留给 Anki 端来做。  
  
还有一个困难，就是将思源格式解析为 Html，或者 Markdown 也可以。如果没有什么官方提供的方法，我可能又要手写一个 Parser...预计只会支持到与 Markdown 同等级的语法。  
  
做到这些之后，我们就可以实现思源笔记与 Anki 卡片的单向同步。只需要定期进行一次检查，看看有没有新增的块，再进行内容同步。这样，一个简单的单向同步就完成了。  
  
而 Anki 到思源就更繁琐一些，我们必须从 0 开始。更具体的，我们希望做到在修改卡片的时候，修改的卡片内容能够同步到思源。  
首先的想法是，通过 ID 很快可以找到卡片与思源对应的块，然后替换内容就可以了。但替换内容不是说说那么简单，将 Html 重新转换为思源格式，也并不是个简单的活。  
思源官方实际上已经做过这样的事情了，希望后续沟通一下，可以避免重复造轮子。具体地，我们可以将 Html 转化为 Markdown 再转化为思源格式。而后者，在思源 1.2 升级时已经有官方的轮子了。  
  
为了实现这个功能，必须使用 Anki 插件的形式进行开发。这触及到了我的知识盲区，但学习应该并不困难。我们的逻辑很简单：修改，同步，仅此而已。  
  
还有上文提及到的删除，如果不能在思源修改时删除，那么就在 Anki 打开时自检验即可。对于所有从思源导入的卡片，我们依次查询对应的 ID 是否存在。一下子又多了好多个查询，感觉性能会很差。但考虑到日常生活的数据量，$O(n \log n)$ 的时间复杂度应该还是游刃有余。但通过 HTTP API 进行查询的话，恐怕还是有些延迟。  
考虑到删除卡片是稀有操作，是否值得牺牲如此性能来实现，我觉得需要再次考虑。直接不写删除功能也是一种可能。  
  
---  
  
UPD: 事实上思源已经偷偷支持了简单的 HTTP API 了，见 https://github.com/88250/Stairway-To-Heaven/blob/main/index.html  
  
可以准备上手开发。
