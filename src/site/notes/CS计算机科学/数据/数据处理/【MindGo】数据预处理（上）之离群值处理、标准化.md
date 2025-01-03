---
{"dg-publish":true,"permalink":"/CS计算机科学/数据/数据处理/【MindGo】数据预处理（上）之离群值处理、标准化/","noteIcon":"","created":"2024-08-28T17:27:38.000+08:00","updated":"2024-04-27T01:22:17.000+08:00"}
---


　一般的数据预处理中常提及到三类处理：去极值、标准化、中性化。这几个词想必大家都不陌生，也许存在疑问或有自己的一番见解，本文将先对前两个进行解释和总结，欢迎讨论和指正~ **一、离群值处理**　　因为过大或过小的数据可能会影响到分析结果，尤其是在做回归的时候，我们需要对那些离群值进行处理。实际上离群值和极值是有区别的，因为极值不代表异常，但实际处理中这两个所用方法差不多，所以这里也不强行区分了。　　

处理方法是调整因子值中的离群值至上下限（Winsorzation 处理），其中上下限由离群值判断的标准给出，从而减小离群值的影响力。离群值的判断标准有三种，分别为 MAD、 3σ、百分位法。下图是 2017-10-23 日的全市场 BP 原始数据。

![](/img/user/Z-attach/v2-fe47ca38c7a1421cfa98458a5580d9ce_r.jpg.png)

**1. MAD 法：**

MAD 又称为绝对值差中位数法（Median Absolute Deviation）。MAD 是一种先需计算所有因子与平均值之间的距离总和来检测离群值的方法　　

处理的逻辑：第一步，找出所有因子的中位数 Xmedian；第二步，得到每个因子与中位数的绝对偏差值 Xi−Xmedian；第三步，得到绝对偏差值的中位数 MAD；最后，确定参数 n，从而确定合理的范围为 [Xmedian−nMAD,Xmedian nMAD]，并针对超出合理范围的因子值做如下的调整：

![](/img/user/Z-attach/v2-73556e9f157495ca6c5da90dc48bfc03_r.jpg.png)

对全市场 BP 原始数据进行 MAD 处理后的结果：

![](/img/user/Z-attach/v2-0341898dcdfef75f1d1e15375f618073_r.jpg.png)

**2. 3σ法**

又称为标准差法。标准差本身可以体现因子的离散程度，是基于因子的平均值 Xmean 而定的。在离群值处理过程中，可通过用 Xmean±nσ来衡量因子与平均值的距离。　　

标准差法处理的逻辑与 MAD 法类似，首先计算出因子的平均值与标准差，其次确认参数 n（这里选定 n = 3），从而确认因子值的合理范围为 [Xmean−nσ,Xmean nσ]，并对因子值作如下的调整：

![](/img/user/Z-attach/v2-e8f5eb35b525071252ba117af4f6acd3_r.jpg.png)

对全市场 BP 原始数据进行 3σ法处理后的结果：

![](/img/user/Z-attach/v2-e0bba000b071ae1bea336210b56f3065_r.jpg.png)

**3. 百分位法：**

计算的逻辑是将因子值进行升序的排序，对排位百分位高于 97.5% 或排位百分位低于 2.5% 的因子值，进行类似于 MAD 、 3σ 的方法进行调整。　　

对全市场 BP 原始数据进行百分位法处理后的结果：

![](/img/user/Z-attach/v2-a18e7f476fb3caef03f34d84abf3214d_r.jpg.png)

**二、标准化**

标准化（standardization）在统计学中有一系列含义，一般使用 z-score 的方法。处理后的数据从有量纲转化为无量纲，从而使得数据更加集中，或者使得不同的指标能够进行比较和回归。　　

由此可见，标准化应该用于多个不同量级指标之间需要互相比较的时候。讲到这里，我们应该区分一下标准化和中性化。中性化的目的在于消除因子中的偏差和不需要的影响，详细的内容将会在下一个帖子总结~　　

对因子进行标准化处理的方法主要有以下两种：

**1、对原始因子值进行标准化；**　　方法一可以保留更多的因子分布信息，但是需要去掉极端值，否则会影响到回归结果。回归的方法一般使用 z-score，将因子值的均值调整为 0，标准差调整为 1。 标准化处理基于原始数据的均值和标准差，处理的逻辑是因子值减去均值后，再除以标准差。　　对已经过 3σ法去极值后的结果进行标准化：

![](/img/user/Z-attach/v2-19e71bc461532608e3959179c558dca5_r.jpg.png)

**2、用因子的排序值进行标准化。**

方法二只关注原始序列的相对排序关系，所以对原始变量的分布不做要求，属于非参数统计方法，可以适用于更多类型的数据。首先将原始数据的排序值作为参数，再将之带入方法一的标准化计算中。　　由于转为排序值之后的分布图像意义不大，就不在此贴出。

源代码及相应书籍请前往 MindGo 量化交易平台查看！

[主题 - MindGo](https://link.zhihu.com/?target=http%3A//quant.10jqka.com.cn/platform/html/article.html%23id/92877941)