---
{"dg-publish":true,"permalink":"/CS计算机科学/数据/Tracking/utm_source, utm_medium 等参数的含义是什么？/","noteIcon":"","created":"2023-11-20T15:30:38.000+08:00","updated":"2024-04-27T01:22:17.000+08:00"}
---


Eution 小明 ， 原文地址： https://www.zhihu.com/question/48724061/answer/1358962571

## **什么是 UTM**

UTM 是什么呢？UTM 是 Urchin Tracking Module 的缩写，是 Google 用来追踪网址 (URL) 的格式。Urchin 其实是 Google 在 2004 年买下的一间软件公司，这个公司的产品 Urchin 软件随后也演变成今日的 Google Analytics。 现在 UTM 成了跟踪参数的标准，无论是 Google 还是百度统计都是支持的。

## **UTM 的参数设置**

在 UTM 中有 3 个参数是必须需要填写的：Source、Medium 还有 Campaign：

*   **Source**：表示你文章投放的渠道
*   **Medium**: 表示投放的渠道类型，常见的类型有：  
*   [direct](https://www.zhihu.com/search?q=direct&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1358962571%7D): 用户直接通过收藏夹或者地址栏输入网址访问，统计工具会自己识别；
*   organic：用户点击搜索引擎结果页的链接访问网站；
*   social：用户点击微博等[社交媒体](https://www.zhihu.com/search?q=%E7%A4%BE%E4%BA%A4%E5%AA%92%E4%BD%93&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1358962571%7D)的链接访问网站；
*   email：用户点击邮件中的链接点击访问网站；
*   [affiliates](https://www.zhihu.com/search?q=affiliates&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1358962571%7D)：用户点击合作联盟网站的链接访问网站；
*   [referral](https://www.zhihu.com/search?q=referral&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1358962571%7D)：用户通过其他网站推荐内容中的链接访问网站；
*   cpc：用户通过点击 cpc 广告访问的网站；
*   display：用户通过点击展示广告访问的网站；
*   other：用户可以自己自定义该参数，比如 wechat-group／qq-group 等等。
*   **Campaign**：表示具体为某次推广名称
*   **Term**：非必填项。
*   **Content**：非必填项，用于 A / B 测试和内容定位广告。使用 [utm_content](https://www.zhihu.com/search?q=utm_content&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1358962571%7D) 来区分指向同一网址的广告或链接。比如如果是文字广告链接，推荐使用 textlink，如果是图片广告链接，推荐使用 [logolink](https://www.zhihu.com/search?q=logolink&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1358962571%7D)。

## **UTM 的目的**

UTM 的主要目的是为了网站 / app / 小程序拥有者识别出访问我的用户的是从哪里来的，能更好的优化产品和广告投放策略。

张溪梦 Simon​

在这里详细介绍下 UTM 的使用和含义。

UTM 除了最基础的追踪流量来源外，还可以根据不同渠道、不同内容做精细化运营分析，帮你对比区分优质和劣质渠道，提高流量在产品内的转化。

先来看一个结果：添加 UTM 参数的链接的链接投放后，我们就可以看到这样的统计了：

![](/img/user/Z-attach/4a97c5556403190d36124b826b4c454b_r.jpg)

每一个渠道带来的流量都十分清晰，用户在产品内的行为也一目了然，是否注册了，是否最终购买了，都可以看到。我们可以看到讲述 heatmap 热图的这篇内容在渠道「微博 1」投放的链接，带来了 9992 个页面浏览量，2066 个注册用户量，以及 1614 个购买用户量。

而且不仅可以看到同一篇文章在不同渠道的流量情况，如 heatmap 热图这篇内容在微信、微博和其他渠道的推广情况；还可以看到同一个渠道不同文章带来的流量情况，如在微博渠道，heatmap 热图的文章的导流情况比 features 功能文章的导流情况更好。

用户在产品内的行为，有多少进行了注册，有多少完成了购买，清清楚楚，而且，我们还可以将不同渠道进行[分组](https://www.zhihu.com/search?q=%E5%88%86%E7%BB%84&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A122730629%7D)，查看不同渠道的用户留存和转化。

那么，我们先来看下，这样的 UTM 参数是怎样设置的呢？

  

**Part 1 | UTM 参数的设置**

通过 UTM 参数追踪外部流量的访问情况的原理是：把你投放在不同渠道的链接打上特定的标记，以监控各个链接的流量情况。

_1. 确定[目标链接](https://www.zhihu.com/search?q=%E7%9B%AE%E6%A0%87%E9%93%BE%E6%8E%A5&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A122730629%7D)_

首先，确定这个链接最终指向的目标网页是哪个？一般来说是你自己的网站的某个页面，然后这个页面需要加载过[数据统计分析](https://www.zhihu.com/search?q=%E6%95%B0%E6%8D%AE%E7%BB%9F%E8%AE%A1%E5%88%86%E6%9E%90&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A122730629%7D)工具的 SDK 。举个例子，如果使用

[GrowingIO](https://link.zhihu.com/?target=https%3A//www.growingio.com/%3Futm_source%3Dzhihu%26utm_medium%3Dqa%26utm_campaign%3Dq48724061%26utm_content%3D160919-utm%26utm_term%3Dtool)

进行接下来的拆解分析，就需要这个页面是加载过 Growing JS 代码的网址。不要以为在别人网站的链接后加上 UTM 参数，你就可以看到别人网站的点击情况了，这一切的前提是，链接最终指向加载了相应的[分析代码](https://www.zhihu.com/search?q=%E5%88%86%E6%9E%90%E4%BB%A3%E7%A0%81&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A122730629%7D)的你自己的网站。

_2. 添加自定义的参数_

接下来，我们需要设置 UTM 的参数，也就是在链接上添加规则，进行标记，投放链接后我们就可以知道是哪个来源带来的流量了。对于不同的活动或文章，我们要设置不同的 UTM 参数用来区分。

说白了，这里就是你用各种各样的内容来描述这条[链接](https://www.zhihu.com/search?q=%E9%93%BE%E6%8E%A5&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A122730629%7D)是放在哪个活动、哪个来源上的，我们来看一个例子进行理解。

![](/img/user/Z-attach/1f3cfbaad4660f74969cd45807dceed1_r.jpg)

  

以现在很常用的新媒体营销方式为例，我们在微信的阅读原文里放了一条引导流量的链接：

[https://www.growingio.com/?utm_source=zhihu&utm_medium=article&utm_campaign=product&utm_content=0811-tool&utm_term=tool](https://link.zhihu.com/?target=https%3A//www.growingio.com/%3Futm_source%3Dzhihu%26utm_medium%3Darticle%26utm_campaign%3Dproduct%26utm_content%3D0811-tool%26utm_term%3Dtool)  

这条链接的意思是什么呢？

1.  [https://www.growingio.com/](https://link.zhihu.com/?target=https%3A//www.growingio.com/%3Futm_source%3Dzhihu%26utm_medium%3Darticle%26utm_campaign%3Dproduct%26utm_content%3D0811-tool%26utm_term%3Dtool) 这条链接最终指向的地址；  
    
2.  [utm_source=zhihu](https://link.zhihu.com/?target=https%3A//www.growingio.com/%3Futm_source%3Dzhihu%26utm_medium%3Darticle%26utm_campaign%3Dproduct%26utm_content%3D0811-tool%26utm_term%3Dtool) 投放的渠道是知乎；  
    
3.  [utm_medium=article](https://link.zhihu.com/?target=https%3A//www.growingio.com/%3Futm_source%3Dzhihu%26utm_medium%3Darticle%26utm_campaign%3Dproduct%26utm_content%3D0811-tool%26utm_term%3Dtool) 媒介是一篇文章；  
    
4.  [utm_campaign=product](https://link.zhihu.com/?target=https%3A//www.growingio.com/%3Futm_source%3Dzhihu%26utm_medium%3Darticle%26utm_campaign%3Dproduct%26utm_content%3D0811-tool%26utm_term%3Dtool) 这篇文章是产品介绍系列的；  
    
5.  [utm_content=0811-tool](https://link.zhihu.com/?target=https%3A//www.growingio.com/%3Futm_source%3Dzhihu%26utm_medium%3Darticle%26utm_campaign%3Dproduct%26utm_content%3D0811-tool%26utm_term%3Dtool) 文章内容是「8.11 编辑，介绍工具」；  
    
6.  [utm_term=tool](https://link.zhihu.com/?target=https%3A//www.growingio.com/%3Futm_source%3Dzhihu%26utm_medium%3Darticle%26utm_campaign%3Dproduct%26utm_content%3D0811-tool%26utm_term%3Dtool) 文章的关键词是「tool」;  
    

你一定会问，这个 URL “?” 之后的参数都是什么？简单说，可以把 “ ? ” 之后的 UTM 参数理解为链接的名字，即为投放在不同渠道的每个链接起的分析工具能够识别的名字。

我们把这些信息连起来，这条 UTM 代表的含义就是：这个[指向](https://www.zhihu.com/search?q=%E6%8C%87%E5%90%91&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A122730629%7D)

[http://www.growingio.com/](https://link.zhihu.com/?target=http%3A//www.growingio.com)

的投放链接，是在 8 月 11 日 utm_content=0811-tool，知乎 utm_source=zhihu 的文章里 utm_medium=article 投放的，这篇文章是介绍工具 utm_term=tool 的产品文章 utm_campaign=product 。

当你在数据分析工具里做分析时，就可以像破解[密码](https://www.zhihu.com/search?q=%E5%AF%86%E7%A0%81&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A122730629%7D)一样读出它的意思了，知道它放在了哪个内容里，用在了哪个活动里。

当我们有很多内容同时在各个渠道投放时，这样的链接就十分有用了，我们知道每个渠道每条内容带来的流量，也可以按照不同的渠道将流量进行分组，分析不同渠道带量的效果和质量。

我们提供的 UTM 参数和自定义参数的方式采用的是目前市面上最常用的定义方式：

![](/img/user/Z-attach/53be54fc398bdf52c97b7c77fdef6b54_r.jpg)

我们可以根据需要，进行各种各样自定义的填充，因为 UTM 最初是用在广告监控上的，所以它的很多名称还是关于广告的，但是我们现在已经可以把它放在各个内容、活动、推广中，监控渠道的流量情况。

具体的填写参数的意义和方法，可以根据下面这些情景进行灵活的变通。

_1. 当这条链接用于付费推广时，可以这样定义：_

![](/img/user/Z-attach/457a27ca5b2aceb7b8003a8f91f95e06_r.jpg)

2. 当这条链接用于内容文章时，可以这样定义：_

![](/img/user/Z-attach/5f2af1e60a8a94bd9df0ff23a0026db3_r.jpg.png)

  

_3. 当这条链接用于活动时，可以这样定义：_

![](/img/user/Z-attach/aaee8514c40384b1fb4761b96c117a27_r.jpg.png)

  

如果是你自己看这个数据，只要设置你能看懂的内容就可以，涉及到团队协作时，最好统一下标准，以便后续的数据分析。

  

**Part 2 | UTM 使用的案例**

UTM 做好了之后，可以做哪些分析呢？我们就可以进行日常的监控和活动的监控了。

现在，我们知道哪些投放的渠道来的量高、哪些量低了，可以有的放矢地进行市场推广和渠道运营，我们可以用 UTM 里面的维度来制图，看一下这一周文章投放的效果：

![](/img/user/Z-attach/235999524bc591a9cef6f9bdfd60703c_r.jpg)

  

接下来，你可能想了解更多细节，这些人都访问了哪些页面呢？比如说他们是否最终[注册](https://www.zhihu.com/search?q=%E6%B3%A8%E5%86%8C&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A122730629%7D)完成了呢？我们可以加上注册页面的[指标](https://www.zhihu.com/search?q=%E6%8C%87%E6%A0%87&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A122730629%7D)来做图：

![](/img/user/Z-attach/8c4cdaf08d3bde47f3101e395e0deca5_r.jpg)

  

这些都只是一个开始，接下来我们还可以做更有价值的数据分析，在漏斗里，用 UTM 参数作为不同的维度，可以对比不同来源不同内容的[转化率](https://www.zhihu.com/search?q=%E8%BD%AC%E5%8C%96%E7%8E%87&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A122730629%7D)：

![](/img/user/Z-attach/832f3ffac07d108828142e07d1967e67_r.jpg)

  

借助 UTM，可以把流量来源、转化、ROI 都分析清楚。

  

注：文中单图、漏斗等产品截图截来自

[GrowingIO - 硅谷新一代数据分析产品](https://link.zhihu.com/?target=https%3A//www.growingio.com/%3Futm_source%3Dzhihu%26utm_medium%3Dqa%26utm_campaign%3Dq48724061%26utm_content%3D160919-utm%26utm_term%3Dtool)

![](/img/user/Z-attach/8c9895e1a2db9eba3c09b558affd2b00_l.jpg.png)

zhibo shan

最早是 Google analytics 用来定义链接来源 媒介等使用的参数以方便在分析用户站内行为时分析用户来源的效果。

utm_source 用来定义来源，如 baidu，weibo，wechat （obligation）

utm_medium 用来定义媒介方式 如 email，cpc，cpm

utm_campaign 用来定义 campaign 名称

utm_content 用来定义内容

utm_term 用来定义 campaign 的关键词

至于为什么要交 UTM 故事要从十几年前讲起，当时还没有 Google analytics，但是有一个叫 urchin 的网站分析工具，UTM 是 urchin tracking module 的缩写（悄悄地说我有个现在的同事当时在做这个产品）。后来 google 收购了 urchin 的这个产品并改名叫 Google analytics（我那个同事也成了 GA 的 PM）。

但是 UTM 这个名字因为大家已经习惯，就一直保留下来并被大量其它后起的网站监测工具使用（国内的如 Growing IO，百度统计等）