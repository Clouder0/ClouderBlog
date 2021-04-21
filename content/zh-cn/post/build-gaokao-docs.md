---
title: "搭建高考文档"
date: 2021-04-21T19:26:40+08:00
slug: "build-gaokao-docs"
type: post
tags:
  - gaokao
categories:
  - Other
---

最近，高考互助组成立并进入试运营阶段，产生了一定量的工作成果。

![image.png](https://b3logfile.com/siyuan/1609132319768/assets/image-20210421184912-nu7rch6.png)

HackMD 在很多方面令人满意，从手感良好的 markdown 编辑器到与 Github 手动同步，这个极客风格的多人文档协同工具给我留下了不错的印象。

然而，它有两个不足之处：

- 国内无法访问。由于天朝特殊的网络环境，服务器在国外的 HackMD 在中国完全无法访问。虽然编辑者可以科学上网，但读者未必有这个条件。
- 无法全文搜索。查重等工作需要通过全文搜索来开展，尽管可以借用本地编辑器来完成，但终究不够方便。

于是我花费一个下午的时间，搭建了[高考文档](https://b3logfile.com/siyuan/1609132319768/https://gaokao.codein.icu/)，以解决以上两个问题。

## 框架选取

最终选择了 Docsify，有如下优点：

- 方便快捷。单文件部署，直接渲染 markdown 文件，无需额外工作。
- 支持全文搜索。
- 静态化部署，无需服务器。
- PWA 支持，一定程度上缓解海外服务器访问缓慢的问题。

最大的痛点是没有开箱即用的 MathJax,只找到了一个 KaTeX 的插件，然而试了一下发现 mhchem 宏包需要额外配置。我发现我不会配置，于是作罢，干脆不渲染数学公式了。

相信各位读者一定能裸眼渲染 LaTeX 公式的。

## 部署

部署 Docsify 相对简单，尽管折腾各种问题消耗了一定的时间，但最后发现折腾不出什么来就直接轻装上阵了。

部署方法：在根目录下新建 `index.html`，将官方文档中的示例代码拷贝进去。

做了一些基础的配置：

- 更换主题，支持黑夜模式。
- 添加全文搜索插件。
- 添加折叠目录插件。
- 开启 PWA

最终大概长这样：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>高考文档</title>
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
    <meta name="description" content="Description">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0">
    <link rel="stylesheet" href="//cdn.jsdelivr.net/npm/docsify-darklight-theme@latest/dist/style.min.css" title="docsify-darklight-theme" type="text/css"/>
</head>
<body>
    <div id="app"></div>
    <script>
        window.$docsify = {
            name: '高考',
            repo: 'gaokao-team/GaokaoNote',
            loadSidebar: true,
            subMaxLevel: 2,
            search: 'auto',
        }
    </script>
    <!-- Docsify v4 -->
    <script src="//cdn.jsdelivr.net/npm/docsify@4"></script>
    <script src="//cdn.jsdelivr.net/npm/docsify-darklight-theme@latest/dist/index.min.js" type="text/javascript"></script>
    <script src="//cdn.jsdelivr.net/npm/docsify/lib/plugins/search.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/docsify-sidebar-collapse/dist/docsify-sidebar-collapse.min.js"></script>
    <script>
        if (typeof navigator.serviceWorker !== 'undefined') {
            navigator.serviceWorker.register('sw.js')
        }
</script>
</body>
</html>
```

### 侧边栏目录

Docsify 不能自动获取子文件夹下的文件来生成目录。

因此必须使用某些工具来手动生成 `_sidebar.md` 文件，才能得到可用的目录。
然而现有的工具都是由文件名生成目录，而我希望以文章中的一级标题作为名称来生成。

迫不得已，现学现卖，拿我破烂的 C++ 技术栈来做这个事情。

```cpp
#include <iostream>
#include <fstream>
#include <string>
#include <filesystem>
using std::endl;
using std::ofstream;
using std::ifstream;
using std::string;
using std::filesystem::directory_iterator;
int main() 
{
    string path = std::filesystem::current_path();
    ofstream output;
    output.open(path + "/_sidebar.md");
    for (const auto & sub : directory_iterator(path))
    {
      if (!sub.is_directory()) continue;
      if (sub.path().filename() == ".git" || sub.path().filename() == ".vscode") continue;
      output << "  - " << sub.path().filename().generic_string() << endl;
      for (const auto &file : directory_iterator(sub.path())) 
      {
          if(file.path().extension() == ".md")
          {
              ifstream reader;
              reader.open(file.path());
              string now;
              do { reader >> now; }while(now != "#");
              getline(reader,now);
              while(now.back() == ' ') now.pop_back();
              while(now.front() == ' ') now.erase(now.begin());
              //now is the title of the file
              output << "    - [" << now << "](/" << sub.path().filename().generic_string()
                     << '/' << file.path().filename().generic_string() << ')' << endl;
              reader.close();
          }
      }
    }
    return 0;
}
```

本来想集成到 Github Actions 里，后来发现我又不会操作，于是作罢。

## 完工

如果不算途中各种无效尝试的话，整体的耗时大概在一小时左右。

![image.png](https://b3logfile.com/siyuan/1609132319768/assets/image-20210421190517-sveh0v6.png)

目前来说，这依然只是一个非常简陋的项目，有如下的改进空间：

- 增加数学公式支持。
- 在新建文档时自动更新侧边栏

大体上感觉这是个吃灰项目，以后再说吧。