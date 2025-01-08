---
{"dg-publish":true,"permalink":"/CS计算机科学/知识图谱/LLM-KG/再看大模型遇见知识图谱：大模型用于知识图谱补全方案回顾及 MPIKGC 实现思路/","noteIcon":"","created":"2024-04-20T15:32:28.000+08:00","updated":"2024-07-08T16:27:17.290+08:00"}
---

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Ohvdt97dPTC-BTE4nOmeXw)

今天是 2024 年 3 月 12 日，星期二。北京，天气晴。

我们今天来看看大模型用于知识图谱推理的一些话题。

知识图谱与大模型进行融合，我们已经有过多个工作介绍，我们在之前的文章《**大模型用于知识图谱推理常用范式：兼论基于 TransE 模型的知识推理实践》，https://mp.weixin.qq.com/s/qTRi3CY6kHTx_hUYx4aNYw** 中系统介绍了关于知识图谱补全的一些工作。

![](/img/user/Z-attach/640-42.png)

在工作**《Text-Augmented Open Knowledge Graph Completion via Pre-Trained Language Models》** 一文将大模型应用于知识图谱推理，这个我们在之前的文章《预训语言模型在知识图谱补全任务上的探索：TAGREAL 高质量提示自动化生成与 KG 补全框架工作解读》**https://mp.weixin.qq.com/s/UmCbanlUpJQnj4A1i7m0mg，(https://arxiv.org/pdf/2305.15597.pdf**) 中已有介绍，该方法自动生成高质量的查询 prompt 提示，并从大型文本语料库中检索支持信息，以探测 PLM 中的知识，从而完成 KG 补全。

![](/img/user/Z-attach/640-35.png)

我们来看看基于大模型的多角度知识图谱补全框架 MPIKGC 这一工作，

最近的工作：**《Multi-perspective Improvement of Knowledge Graph Completion with Large Language Models》(https://arxiv.org/pdf/2403.01972.pdf，https://github.com/quqxui/MPIKGC)** 提出了 MPIKGC 框架，通过从各个角度查询大型语言模型（LLM）来弥补上下文知识的不足，包括利用 LLM 的推理、解释和摘要功能来分别扩展实体描述、理解关系和提取结构，其旨在回答**如何有效地利用语言模型的能力和知识来改进图形学习？**的经典问题。

其核心解决的问题是，**从三个角度研究通过查询 LLM 来提高 KGs 的质量**，通过设计思想链提示来扩展实体描述，通过设计全局、局部和反向提示来增强对关系的理解，以及通过关键词摘要和匹配来提取结构数据这一工作。

供大家一起参考并思考。

一、关于大模型用于知识图谱补全的回顾
------------------

知识图补全（KGC）通过对缺失链接进行预测来解决知识图中的不完备问题，KGC 的链接预测任务是基于已知的文本和结构数据来推断遗漏的事实，它包括两个部分，即当给定（h，r，？）时预测尾部实体，当提供（？，r，t）时预测头部实体，通常的实现方案是**通过计算正三元组和负三元组的得分函数来对 E 中的所有实体进行排名。**

**1）基于基于结构的 KGC 方案**

基于结构的 KGC，例如 TransE、Dist-Mult、RotatE、ConvE 等只考虑来自观察到的三元组事实的图结构信息，并将每个实体和关系分别嵌入到可训练的索引嵌入中，例如：《Knowledge Graph Embedding: A Survey of Approaches and Applications，https://arxiv.org/abs/2211.03536》的一些论述。

![](/img/user/Z-attach/640-37.png)

这类方法的核心思想是是嵌入 KG，包括将实体和关系转化为连续的向量空间，从而简化操作，同时保留 KG 的原有的结构。那些实体和关系嵌入能进一步应用于各种任务中，如 KG 补全、关系提取、实体分类和实体解析。

在实现上，其使用连续向量空间表示实体关系，关系通常被视为向量空间的运算，然后定义评分函数，用来测量事实的合理性，最后，学习实体关系的表示，优化问题：最大化全局观测事实的合理性。

其中，平移距离模型利用基于距离的评分函数。通常是在通过关系进行翻译之后，用两个实体之间的距离来衡量一个事实的合理性，TransE 是一个典型代表：

![](/img/user/Z-attach/640-32.png)

实体和关系都在同一空间，对于每一个三元组（h,r,t），TransE 希望：h+r=t

关于这一块的工作，可以看看

**2）基于基于结构的 KGC 方案**

与基于结构的 KGC 不同，基于**描述的 KGC 方法使用预先训练的语言模型将实体和关系的文本编码到语义空间中，通过计算头部实体、关系和尾部实体之间的三元组或匹配语义相似性的评分函数来预测事实的合理性。**

![](/img/user/Z-attach/640-31.png)

例如：

DKRL（《Representation Learning of Knowledge Graphs with Entity Descriptions》,http://nlp.csai.tsinghua.edu.cn/~lzy/publications/aaai2016_dkrl.pdf）使用卷积神经网络对实体描述进行编码；

![](/img/user/Z-attach/640-30.png)

KG-BERT（《KG-BERT: BERT for Knowledge Graph Completion》,https://arxiv.org/pdf/1909.03193.pdf）使用预先训练的 BERT 模型来学习实体和关系的嵌入；

![](/img/user/Z-attach/640-33.png)

KEPLER（《KEPLER: A Unified Model for Knowledge Embedding and Pre-trained Language Representation》，https://arxiv.org/pdf/1911.06136.pdf）进一步微调预先训练的语言模型，以同时优化知识嵌入和语言建模目标。MMRNS 利用描述的相似性对硬样本进行负采样。

![](/img/user/Z-attach/640-34.png)

LMKE(《Language Models as Knowledge Embeddings》，https://arxiv.org/pdf/2206.12617.pdf) 利用语言模型和文本信息为实体，特别是长尾实体生成知识嵌入。

![](/img/user/Z-attach/640-32.png)

CSProm-KG(《Dipping PLMs Sauce: Bridging Structure and Text for Effective Knowledge Graph Completion via Conditional Soft Prompting》，https://arxiv.org/pdf/2307.01709.pdf，https://github.com/chenchens190009/CSProm-KG) 认为经过微调的 PLM 往往过分关注文本信息而忽视结构知识。为了解决这个问题，提出 CSProm-KG（KGC 的条件软提示），它保持了结构信息和文本知识之间的平衡，在训练过程中，CSProm-KG 仅调整实体和关系表示生成的附加软提示的参数。

![](/img/user/Z-attach/640-44.png)

SimKGC(《SimKGC: Simple Contrastive Knowledge Graph Completion with Pre-trained Language Models》,https://arxiv.org/pdf/2203.02167.pdf) 引入对比学习任务来增强嵌入学习。

通过这种方式，文本编码器有助于将模型简单地概括为看不见的图实体，从而比实体嵌入具有更好的可伸缩性。

然而，尽管基于描述的 KGC 在学习文本和结构知识方面取得了显著的成功，但其有效性仍然受到互联网抓取文本质量和不完整结构的限制。

例如，在图 1 中，对《伊恩 · 布莱斯》和《变形金刚：黑暗的月亮》的简短描述是没有信息的。在这种情况下，仅依赖 “producer” 关系的名称可能会导致对实体类型的理解不明确

同时，从已知图中学习结构模式对长尾实体来说是一项挑战。这些限制使得 KGC 很难在涉及不充分和不完整知识图的现实应用中实现高性能。

![](/img/user/Z-attach/640-43.png)

基于描述的 KGC 利用预训模型来学习具有名称或描述的实体和关系表示，但其仍然受到文本质量和不完整结构的限制，因为它缺乏足够的实体描述，并且仅依赖于关系名称。

二、基于大模型的多角度知识图谱补全框架 MPIKGC
--------------------------

鉴于当前大模型的能力，最近的工作：**《Multi-perspective Improvement of Knowledge Graph Completion with Large Language Models》(https://arxiv.org/pdf/2403.01972.pdf，https://github.com/quqxui/MPIKGC)** 提出了 MPIKGC 框架，通过从各个角度查询大型语言模型（LLM）来弥补上下文知识的不足，包括利用 LLM 的推理、解释和摘要功能来分别扩展实体描述、理解关系和提取结构，其旨在回答**如何有效地利用语言模型的能力和知识来改进图形学习？**的经典问题。

其核心解决的问题是，**从三个角度研究通过查询 LLM 来提高 KGs 的质量**，通过设计思想链提示来扩展实体描述，通过设计全局、局部和反向提示来增强对关系的理解，以及通过关键词摘要和匹配来提取结构数据。我们来看看这个工作。

**1、基本架构**

为了解决实体信息不完整的问题，使用 LLM 获取的知识来扩展实体描述，并通过设计思想链来实现，该提示允许 LLM 将查询分解为不同的方面，并逐步生成描述。

除了解决关系歧义问题外，采用一种提高 KGC 模型对关系含义理解的解决方案，包括使用三种精心策划的提示策略（即全局、局部和反向提示）查询 LLM，以捕捉关系之间的关联并促进更好的反向预测。

![](/img/user/Z-attach/640-40.png)

为了解决稀疏图链接的问题，特别是对于长尾实体，通过查询 LLM 来提取额外的结构信息，使用 LLM 总结的关键字来测量实体之间的相似性，从而丰富知识图谱，以及创建新的三元组，这些三元组构建相关实体之间的关联，并能够在 KGC 模型中形成新的结构模式。

![](/img/user/Z-attach/640-37.png)

**2、描述扩展：Description Expansion**

从 LLM 中归一化一个实体的全面知识并非易事，因为很难确定 LLM 是否生成并包含了该实体的所有信息。如下图所示，其设计一种思想链（CoT）提示策略，使 LLM 将复杂的查询分解为不同的方向，并逐步生成描述，而不需要明确的手动输入。它指示 LLM 自己隐式地查询相关信息，从而产生更高效、更广泛的回复。

![](/img/user/Z-attach/640-29.png)

对应的模板为 MPIKGC-E，要求 LLM 在回答之前提供全面的实体描述并提供基本理由，这可以作为回答的依据，并提高 KGC 模型的召回率。

例如，图 2 展示了 “Michael Bay” 的例子，LLM 生成了一个描述，其中包含了个人的各种职业和个人详细信息，并附有每个回应的理由，以增强 LLM 的陈述。

![](/img/user/Z-attach/640-39.png)

**3、关系理解 Relation Understanding**

知识图中异构关系的存在在区分两个实体方面起着至关重要的作用。然而，仅依赖关系名称可能会导致歧义解释，尤其是对于复杂的关系类别（如多对多和多对一），更重要的是，链接预测任务需要额外的反向预测，即预测给定（？，r，t）的头部实体，而反向的性能 多对一关系的预测显著低于正向预测。

基于结构的 KGC 方法试图通过为每个正向关系添加反向关系来解决这个问题，从而使关系的可训练嵌入加倍。相反，基于描述的 KGC 方法，如 SimKGC，在关系名称后附加一个字符串 “reverse”，但这种方法不能使模型完全理解关系的含义，导致性能较差。

因此，我们提出了三种提示策略，即全局 (MPIKGC-R Global)、局部(MPIKGC-R Local) 和反向(MPIKGC-R Reverse，对应的 prompt 如下所示：

![](/img/user/Z-attach/640-31.png)

MPIKGC-R Global 从整个 KG 的角度推断一种关系的意义，从而促进两种关系之间更好的关联。例如，“制片方”“produced by”和 “导演”“director” 都与电影业有关，而“上映地区”“release region” 和“出生地”“place of birth” 则与国家或地区的名称有关。

![](/img/user/Z-attach/640-36.png)

MPIKGC-R Local 从三元组的角度推断关系的含义，从而增强理解，并在预测缺失事实的同时提出可能的头 / 尾实体类型。例如，当查询 “(head entity, release region, tail entity)（头部实体、发布区域、尾部实体）” 的含义时，LLM 认为这种关系可能与电影和区域有关。

此外，MPIKGC-R Reverse 反向需要 LLM 将关系表示为动词，并将其转换为被动语态。例如，“product” 可以转换为 “producted by”，从而增强压缩并实现更好的反向预测。生成的文本被附加到关系名称，并根据每个 KGC 模型处理关系名称的工作流进行处理。

**4、结构提取 Structure Extraction**

KGC 模型能够从训练三元组中学习结构模式，并推广到测试三元组中缺失的环节。例如，拥有制片人或导演职业的个人实体可能与电影实体有关。然而，从图结构中进行模式学习仅限于稀疏链接，特别是对于长尾实体。

为了解决这个问题，通过 MPIKGC-S 查询 LLM 以生成额外的结构信息来丰富 KGs。为了将 LLM 的生成文本转换为基于图的数据，利用 LLM 的汇总能力从描述中提取相关关键词，对应的 prompt 如下：

![](/img/user/Z-attach/640-27.png)

然后根据匹配关键词的数量计算实体之间的匹配分数 s：

![](/img/user/Z-attach/640-28.png)

其中，kh 和 kt 分别表示头 / 尾实体的关键词，m 是 kh 和 kt 的交集。在对匹配分数进行排序后，选择了前 k 对，并以（head，SameAs，tail）的形式创建新的三元组，然后将其附加到训练集。

除了这些基于相似性的三元组之外，还考虑为每个实体添加一个具有关系 “SameAs” 的自循环三元组：（head，SameAs，head），以加强 KGC 模型对 “SameAs” 关系的学习。这些额外的三元组构建了相关实体之间的关联，并允许在 KGC 模型中形成新的结构模式。

![](/img/user/Z-attach/640-38.png)

例如，通过在 “IanBryce” 和“MichaelBay”之间添加 “SameAs” 关系，“IanBlyce”可以通过明确的路径到达“Transformers: Dark of the Moon” 实体，从而成为 KGC 模型学习过程的一个补充。

**5、送入模型进行建模**

在后续阶段使用四个基于描述的 KGC 模型将改进的 KGs 与原始 KGs 进行了比较：KG-BERT、SimKGC、LMKE 和 CSProm-KG，并在大模型基座上使用 Calma-2（Calma-2-7b-Chat）、ChatGLM2-6b、ChatGPT、GPT4，效果如下：

![](/img/user/Z-attach/640-41.png)

可以看到，从实体、关系和结构的角度进行增强后，KGC 模型在所有四个指标上都实现了改进。此外，MPIKGC-E&R 将生成的实体描述与关系的描述性文本相结合，可以使对单独使用任何一种方法的轻微改进，这表明了这两种方法的兼容性。

总结
--

本文主要介绍了利用大模型进行知识图谱补全的一些工作，其核心问题还是如何利用好大模型这种参数化知识。

参考文献
----

1、https://arxiv.org/pdf/2403.01972.pdf

2、https://github.com/quqxui/MPIKGC

3、https://mp.weixin.qq.com/s/qTRi3CY6kHTx_hUYx4aNYw

关于该工作
-----

老刘，刘焕勇，NLP 开源爱好者与践行者，主页：https://liuhuanyong.github.io。

老刘说 NLP，将定期发布语言资源、工程实践、技术总结等内容，欢迎关注。

**对于想加入更优质的知识图谱、事件图谱、大模型 AIGC 实践、相关分享的，可关注公众号，在后台菜单栏中点击会员社区 -> 会员入群加入。**