---
{"dg-publish":true,"permalink":"/CS计算机科学/爬虫/爬虫库/Newspaper3k - 爬取并解析新闻/","noteIcon":"","created":"2024-06-22T22:28:04.871+08:00","updated":"2024-04-24T00:26:31.000+08:00"}
---

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?chksm=eb8d29c4dcfaa0d20d32ace2f94d4827440d4cd489a71ad2927e9666253f6dc8634cd008fe67&exptype=unsubscribed_card_recommend_article_u2i_mainprocess_coarse_sort_tlfeeds&ranksessionid=1706352213_2&mid=2247486312&sn=02cb77b561270542a3d3fe1f1cc4cca8&idx=1&__biz=MzI4MzMyNjQwMw%3D%3D&scene=169&subscene=200&sessionid=1706352213&flutter_pos=17&clicktime=1706352600&enterid=1706352600&finder_biz_enter_id=5&ascene=56&fasttmpl_type=0&fasttmpl_fullversion=7049308-en_US-zip&fasttmpl_flag=0&realreporttime=1706352600453&devicetype=android-31&version=28002d3d&nettype=WIFI&abtest_cookie=AAACAA%3D%3D&lang=en&session_us=gh_27ea27c8c859&countrycode=CN&exportkey=n_ChQIAhIQwcMTtm1rDV5Fioztua7ByRLfAQIE97dBBAEAAAAAAAZQC7iI5w0AAAAOpnltbLcz9gKNyK89dVj03KGKwWDTbYwHPFj25V8g%2BPAZdik9NXEmSIwJx45yKPUjM2xHQgGxGG8s9h4QO0v%2FiQk7fI%2BnknlDvPYe%2BBCVS3mMBprJjeMlw00i5inFVMtT3LcwzzGyvYf8syEbWMIzGTnpb6nCd95PoNImS5Gk2I3Lxu7M3J5%2B%2FJYcx68DWjzD1kqB1xzXWMJt4TxObu3y6rfYi82GVBw8NouxMnr%2BiXZCIJh6P%2BC9nRrr6axSmZ055h0kLYIB4%2FI%3D&pass_ticket=nYqjntOjFo953%2BU98t8xh9hGqc1I8pAOr0qdS%2Ff8Z%2Fr5wyUJ5q8LmuxNF3jhCIu9Tx5PhMZTdbmwZNBJssDGzg%3D%3D&wx_header=3)

* * *

Newspaper3k 不仅可以帮助你抓取文章，解析出干净的文本，而且还能自动提取关键信息，如作者、发布日期、关键字等。

更惊艳的是它还可以支持多种语言，从英文到中文，再到阿拉伯语等等，这一切都变得易如反掌。

Newspaper3k 启发自 requests 库的简单性，并借助 lxml 的速度优势，成为了处理新闻抓取任务的优选库。

虽然它支持 Python 2，但是强烈推荐您在 Python 3 环境下使用，以享受更稳定和更强大的功能。

Newspaper3k 是在 Python 环境中进行新闻文章抓取和内容解析的测试库。由 Lucas Ou-Yang 创建，并在 Github 上得到了社区的积极维护和更新。

项目地址：**https://github.com/codelucas/newspaper**

### 安装

通过 pip 可以轻松安装 Newspaper3k：

```
pip3 install newspaper3k
```

> **注意**：请确保使用 `pip3` 命令安装 `newspaper3k`，而不是 `newspaper`。

### 基本用法

#### **使用新闻源**

使用 Newspaper3k 搭建一个新闻源，我们可以抽取它的所有文章和分类链接：

```
import newspapercnn_paper = newspaper.build('http://cnn.com')for article in cnn_paper.articles:    print(article.url)for category in cnn_paper.category_urls():    print(category)
```

#### **处理单篇文章**

处理新闻文章主要涉及几个关键步骤：下载、解析和自然语言处理。以下是针对单个文章操作示例：

```
from newspaper import Articleurl = 'http://fox13now.com/2013/12/30/new-year-new-laws-obamacare-pot-guns-and-drones/'article = Article(url)# 下载文章 article.download()# 解析文章 article.parse()# 获取作者 print(article.authors)# 获取发布日期 print(article.publish_date)# 获取顶部图片 print(article.top_image)# 文章摘要 article.nlp()print(article.summary)
```

> **注意**：使用 nlp，需要下载 NLTK 数据，可能会比较慢
>
> 详细信息可参考：**https://www.nltk.org/data.html**

#### **多语言**

Newspaper3k 支持多种语言，包括但不限于中文、英文和阿拉伯语。它可以无缝提取和检测语言，如果没有指定语言，Newspaper3k 将尝试自动检测：

```
from newspaper import Articleurl = "https://news.sina.com.cn/c/2024-01-26/doc-inaewiyx7293475.shtml"a = Article(url, language='zh')  # 使用中文 a.download()a.parse()print(a.text[:30])print(a.title)
```

应该能输出：

```
1月26日，2024年春运正式启动。统筹冬季寒潮大风和海冰等

我国三大海域举行海空立体巡航及演练 全力保障海上安全
```

### 实践一下

试试自己抓取一个新闻网站的文章和相关信息。

首先，选择一个新闻网站，例如 BBC 中文网。

然后使用 Newspaper3k 的 API 来查看有哪些文章可用并提取它们的内容。

> **注意**：请遵守目标网站的爬虫限制政策

### 总结

Newspaper3k 是一个功能强大的 Python 库，能帮助您轻松地提取和分析网络新闻文章。

使用它，你不仅可以抓取新闻内容，而且能进行深度学习，提取文章的关键信息。

此外，它对多语言和并行处理的支持使它成为处理大规模新闻数据的一个理想工具。有了 Newspaper3k，世界新闻的门户轻轻松松就可以打开了。

比心！

* * *

**示例代码:**

**https://github.com/alisx/amazing_python_lib/  