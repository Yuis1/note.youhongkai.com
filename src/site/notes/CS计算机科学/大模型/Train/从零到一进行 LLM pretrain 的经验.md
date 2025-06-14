---
{"dg-publish":true,"permalink":"/CS计算机科学/大模型/Train/从零到一进行 LLM pretrain 的经验/","noteIcon":"","created":"2025-01-29T13:58:01.256+08:00","updated":"2025-01-27T13:16:14.000+08:00"}
---


> 作者: ybq，nlp 码农，中国科学院大学 信号与信息处理硕士  
> 原文: https://zhuanlan.zhihu.com/p/718354385

这篇文章介绍下如何从零到一进行 pretrain 工作。

类似的文章应该有很多，不同的地方可能在于，我并不会去分析 pretrain 阶段的核心技术，而是用比较朴素的语言来描述这个大工程的每一块砖瓦。我的介绍偏方法论一些，主要目的是普及每个环节有哪些必须要做的琐碎工作、有哪些坑、以及有哪些避坑技巧。为了避免老板开了我，文中有一些内容的具体做法不会展开细说，请大家见谅。作为替代，我会推荐一些比较好的开源做法。

背景篇
===

时至今日，dense 模型有 qwen，MOE 模型有 deepseek，小尺寸模型有 minicpm。无论是个人还是大厂，都很难训出同 size 下更优秀的模型，大模型 pretrain 阶段全面拥抱开源的日子感觉不太远了。那么，在这个时代大背景下，自研 pretrain 模型的意义又有哪些呢？

### 正经答案

*   • 各公司仅仅是开源了模型参数，但并没有开源训练框架、训练数据等更核心的内容，其实本质上还是闭源。在这种情况下，每一个 qwen 模型的使用者都无法为下一版 qwen 模型的迭代做出贡献，qwen 团队也仅仅是收获了口碑，甚至因为自己的模型已经开源可以自行部署，买他们服务的客户可能都会变少。因此，在 llm 真正走向全面开源之前（大厂公开训练代码、配比数据，任何人都可以通过提 CR 来帮助大厂优化训练效率、炼丹技巧），掌握 pretrain 的技术能力依然是有意义的；
    
*   • 通用模型的变现能力远不如 domain 模型，continue-pretrain 的需求在日益增长，而 continue-pretrain 的技术栈和 pretrain 的技术栈并没有本质区别；
    
*   • 不是自己做的 pretrain，必然无法得知这个模型在 pretrain 阶段到底喂了什么数据。各种数据的精确配比、各种 knowledge 的掌握程度，并不是靠评估能准确衡量的。而如果不知道这些数据细节，那么 alignment 阶段就无法对症下药，就无法最大限度的开发模型潜力。举个简单的例子，你在 sft 阶段，让一个没训过唐诗宋词的通用模型学习作诗，它不出幻觉谁出幻觉？
    
*   • 使用开源模型的话，tokenizer 不可控，进而导致解码速度不可控。这里也举个例子，如果我们用 llama 模型来做意图识别任务，有个意图叫 ai.listen.music，会被映射成 5 个 token，但如果使用自己训练的大模型，便会在一开始就设置成 1 个 token，极大节省了生成速度。虽然扩词表已经是一个比较成熟的技术了，但不仅需要花费算力来恢复效果，而且不管训多少新语料，也很难做到模型效果完全不掉点。
    

### 不正经答案

*   • 有一个自己的模型很 cool，公司可以拿来做宣传，证明自己的科研能力，个人也会很有成就感；
    
*   • 可以埋彩蛋，在 pretrain 阶段给模型悄悄塞一点自己喜欢的知识或价值观。比如，每隔 100B token 就让模型学一次 “在 XXX 眼里，YYY 是最好看的女孩子”， 等模型上线了，拿去向 YYY 表白（被老板开了别说是我这篇文章怂恿的）。
    

数据篇
===

### 数据爬取

pretrain 大模型的第一件事：先找个 10T 左右的训练数据吧。也可以少找一些，等模型开始训了，在训练的同时，不断去收集更多的新数据。

至于怎么获取数据，爬网页、逛淘宝、联系数据贩子，等等等等。算法同学往往搞不定这个事情，你敢爬他就敢封你 IP，你爬得起劲他甚至还可以起诉你，所以这个工作最好还是让专业的数据团队同学来做。

有些高质量的数据，比如论文书籍，往往还都是 pdf 格式，这时候还需要去调用效果较好的 pdf 服务。不要指望着靠 python 库来解析，稍微涉及一点公式、表格的 pdf，解析效果都一塌糊涂。用 GPT4 等大模型进行解析，大概率价格会远高于 pdf 解析服务。当然，自己训一个 OCR 模型也是可用的候选方案，前提是你有足够高质量的 pdf - text 对齐数据。

好在，世上还是好人多！今年再做 pretrain 工作，网上的开源数据集已经很多了。FineWeb、pile、Skypile、RedPajama，凑合着差不多能当启动资金来用。但从另一个角度讲，世界上没有免费的午餐，所有开源出来的中文大模型数据集，我不认为是他们最干净的数据，质量多少都有点问题。

（即使是下载 huggingface 数据，也不是动动嘴皮子这么简单的。如果你实操了，就会发现：服务器没连外网，只能换成 hf_mirror 的链接；下载速度太慢，下完 1T 开源数据得好几天，得手动 split 要下载的数据集合，起多个进程在多台服务器上下载；下载完之后，文件量多的你执行 ls 都会卡死，你得用大数据集群技术来处理）

准备数据还要懂得一个基础概念：数据的知识密度是有差异的。“唐诗三百首” 的知识量要远远大于 “中国新闻网的三百篇新闻”。而这种高知识密度的训练数据，往往都是需要花钱的。最近，一种新的数据趋势是 “合成高知识密度数据”，把几千字的新闻概括成几百字喂给模型，四舍五入也等于训练速度提高了十倍。

总之，如果要认真做 pretrain 工作，我建议要组建数据团队，爬虫或购买是必须的，否则网上那几个翻来覆去的数据集在清洗之后根本不够用。

### 数据清洗

**“清洗” 是数据环节最最核心的工作，没有之一！**

目前，利用模型对 pretrain 数据的质量进行打分，已经成了数据清洗工作的标配，llama3、qwen2 的技术报告都有提及。需要注意的是，基本上大家都认同：同等 size 下，BERT 结构的模型的表征能力是强于 transformer-decoder 模型的，因此打分模型最好还是从 BERT 家族中选一个来训，效果好、速度还快。至于训练数据怎么搞，还是老一套，让 GPT4 标注一下，或者是利用 “某个源的数据是高质量数据，某个源的数据是低质量数据” 这种规则生产一些。特别强调，任何打分器，都会给 code、markdown、latex 等格式数据打很低的分数，我们必须把这些数据摘出来，免得被直接洗没了。（FineWeb 这个数据集，就洗的基本没有 code 数据了）

训打分器这个工作的难点是要学会放低心态，别那么执拗，不要执着于打分器 100% 的准确率，凑合能用就行了，有打分器总比没打分器强，但你要花一个月来训打分器，那就还不如没打分器。此外要学会变通，你有 32K 的语料不代表你要训 32K 的打分器，训个 4K 就差不多了，你非要纠结存在 “前 4K 低质量，后 28K 高质量” 这种特殊情况，我只能说算你牛逼。

打分器结果只是众多数据特征中的一个特征，并不一定要完全依赖它来洗数据，可以和其他特征结合使用。这也引出了数据清洗的另一个大杀器：规则。

不要瞧不起规则！不要瞧不起规则！不要瞧不起规则！

数据长度是否少于某个值，数据中某个 token 的比例超过某个阈值，数据的 zh 占比、en 占比、数字占比，数据是否有 “http” 字段，数据是否包含了 “新冠”、“疫情” 等低质量关键词，数据是否包含某些反动词汇，数据是否包含某些黄色字眼，等等等等。用启发式的规则过滤数据并不丢人，洗不干净数据才丢人。

但同时，必须注意到，用规则清洗或者过滤数据的时候，一定不要把数据搞成分布有偏的数据。比如、你觉着：“包含网址的数据质量低，而网址的英文占比高”，所以你把英文占比高的数据都去掉了。整挺好，模型成了单语模型。因此，用规则的时候，一定要多 check 下被滤出去的数据长什么样子，勤 vim 一下！

另外，数据脱敏也是数据清洗环节必须要做的一个工作。我们要尽可能的把训练数据中涉及到的人名、电话号码、邮箱等剔除出去，一旦被模型说出来，就构成了隐私侵犯，公司被罚的钱足够雇人把数据脱敏 N 遍了。更广义的，把数据的 “转载自……” 删掉，黄色信息、反动信息，references 等剔除出去，都可以视作数据脱敏工作的一部分。这个工作好像没任何奇淫巧技，老老实实的写正则匹配吧。

### 数据去重

数据环节最考研工程能力的环节到了：对 T 级别的数据进行去重。

不要心存任何幻想：能不能不做数据去重。答案肯定是不行的！网上基本所有的开源数据，都是来自 common crawl，你不去重如何混合使用呢。就算你只使用单一数据源或者自己爬取数据，也应该注意到：网页 A 引用了 网页 B，网页 B 引用了 网页 C……，网页 Z 又引用了网页 A。这种 url 循环调用的现象，在互联网屡见不鲜，你的训练数据集大概率会把一个网页翻来覆去的使用。即使能确保是不同的网页，一篇文章也会被知乎、CSDN、博客、微信公众号、小红书等不同软件反复转载。

去重工作唯一可以让步的地方是：是做 sentence 去重还是做 document 去重，这个我也不好断定，我的建议是量力而为。能做 sentence 去重，谁不愿意呢？可是数据量和工作难度也会陡增。

那么如何去重呢？首先，你一定要有一个大数据处理集群，hadoop 也好、spark 也罢，只要是一个 map / reduce 的框架就都可以。这个属于汽车的轮子，想要靠 python 写 for 循环完成这个工作，确实是勇气可嘉。

然后，就去实现一个简单的 minhash 代码，没啥难度，ChatGPT 一定会写。

数据去重工作有一个比较重要的意识：要先确定需要多少训练数据，再确定去重的粒度。去重工作是没有尽头的，任何时候你都能把数据继续洗下去，所以必须明确自己需要多少训练数据。需要 10T 训练数据，就卡相似度在 80% 的阈值进行去重；需要 5T 的训练数据，就卡相似度在 90% 的阈值进行去重；以此类推。

目前没有任工作能证明，一条数据在 pretrain 阶段训多少遍对模型是最友好的。因此，大胆的按需去重，即使去重粒度小，导致一篇文档出现多次，也可以通过让两篇相似文档之间隔尽量多的 token 来降低影响。

### 数据配比

前面提到，**我们要在数据清洗的时候把 code 等格式化数据摘出来，怎么实现呢？训练一个数据分类器**！对每一个 document 进行类别判断，不用特别精准，把数据划分成新闻、百科、代码、markdown、等类目即可，分类器模型依然可以选择使用 BERT 家族。

不同的数据源，在上文中介绍清洗和去重的时候也要有不同的阈值：

*   • 清洗的时候，“代码”和 “知识类文本” 当然要使用不同的阈值来决定是否是高质量；
    
*   • 去重的时候，“新闻” 类可能 70% 的重复度就不要，“知识” 类则可以 85% 的相似度才丢弃，在丢去重复文档的时候，优先保留数据打分器比较高的数据。
    

好了，引子环节说完了，默认大家已经都给自己的数据打好了类别，我们继续往下讲配比工作。

大部分的技术报告里，应该都提及了自己的数据是如何配比的，基本上都是 “知识 + 代码 + 逻辑” 三个大类目，其中知识数据分文中文知识和英文知识，逻辑数据则可以认为是 math 数据和 cot 数据的混合体。整体上，大部分中文模型的配比都在这个区间左右：中：英：code = 4:4:2（逻辑数据的比例我没有写进去，加入多少取决于你能收集多少，其他三类数据应该是要多少有多少的存在）。

我们可以根据自己的实际情况调整配比，但英文的比例一定不能太低。目前中文数据的质量不如英文数据质量基本已经成功共识，导致这个现象可能有两个原因：

*   • 中文确实比英文难学，语言空间的复杂度更高；
    
*   • 中文语料无论是干净程度还是数量级，都无法与英文语料相比较。
    

### 数据顺序

pretrain 的本质是一个教模型学知识的过程，既然是学习，那么知识的顺序就显得很重要，总不能先学微积分，再学数字加减法吧。这也就是 “课程学习” 的核心思想。

课程学习的内容很宽泛，无论是先学难知识、再学脏知识，还是先学好数据、再学脏数据，都可以视为是课程学习。其本质就是在阐述一件事情：“同样 1 个 T 的训练数据，通过调整训练顺序得到的不同模型，能力是不同的。” 这个观点基本已经被很多团队论证多次了，因此课程学习目前也可以认为是 pretrain 的标配。

虽然 next_token 的训练方法，基本不存在模型学不会某条数据的情况。但从另外一个角度来分析，灾难性遗忘可能始终在发生，A + B 的学习顺序可能导致 A 知识遗忘了 30%，B + A 的学习顺序可能导致 B 知识遗忘了 20%，那后者忘得少自然能力更强啊。而且，如果 B 是一个简单的知识，那就代表 B 在训练语料中会出现非常多的次数，即使遗忘了后续也会被重新捡起来，困难知识在全部训练数据中出现的次数自然也会小很多。（全局训练语料中，蜀道难全文出现的次数一定比静夜思全文出现的次数少）。

说了这么多，只是为了强调一件事：数据顺序真的很重要，那么如何敲定呢？

这里我推荐的 llama 的 In context pretrain 工作：利用语义相似度，优先将最相似的 document 进行拼接，从而构成语义更加连贯流畅的上下文，详见论文 https://arxiv.org/pdf/2310.10638。

需要强调的一个地方是，llama 坚定的认为：在同一条 pretrain 语料中，无关文档之间不能相互看见。具体来说，sentenceA + "" + sentenceB 这样一条训练语料，llama 认为 sentenceB 看不见 sentenceA 非常重要且效果很好，它在这篇论文和 llama3.1 技术报告中均有提及。

但在实操中，除了 llama，我没听说过还有哪个团队在 pretrain 阶段做 attention_mask，大家的实验结论基本都是做不做 mask 没什么区别。而且，我个人认为，pretrain 阶段应该要培养模型切换 topic 的能力，在 llm 的实际应用场景中，我们也不会每切换一个新话题，就起一个新的聊天窗口，模型需要有判断上文信息和当前信息是否相关的能力。因此，如果使用了 In context pretrain 这篇工作的论文，要不要做 attention_mask 还是要做实验去斟酌的。

### 数据流水线

首先要明确一个概念，pretrain 模型一定是动态加载数据的，读 1B 、训 1B、再读 1B 、再训 1B…… 原因很简单，你不知道你要训多少数据，即使知道你也没那么大的内存空间一下子读取好几 T 的数据。

再明确一个概念，pretrain 阶段模型获取的是 token_id，而不是 token 本身，我们的 tokenization、concatenation 操作肯定是要提前做好的。当机器读取了一个新数据块之后，如果不能直接去训练，而是还要花时间去转 token，去 concat、去 pad，这简直是对 GPU 的一种侮辱。

明确这两个概念之后，我们就应该知道，pretrain 的两个进程是独立的：“数据处理进程” 和 “模型训练进程”。前者要保证后者始终有最新的数据可用，除了 save_checkpoint 的时候，GPU 的空闲是一种极大的浪费。

pretrain 阶段的数据是可以复用的，高质量数据训多遍对模型并没有坏处。因此，数据处理进程在生产 part-00000.jsonl 的同时，它也应该标记清楚每一条原始的 document 数据被使用了多少次，被标记次数多的数据，后续要降低它再被选中的概率。

每个数据块不要太大，因为我们训练的时候，经常有烧卡、loss 炸、数据配错了，等不可控的天灾人祸，所以回退到上个数据块进行续训是一个很频繁的操作。较大的数据块自然会导致模型版本回退时损失的算力也较多。这里，我推荐每个数据块都以 B 为单位，正好是 1B、2B、4B 等。

每个数据块在训练代码中，自然会对应着一个 save_checkpoint 的操作，原因也是为了便于训练回退。这里可以分享一个以前的小技巧，曾经因为 warmup 阶段，数据块大小是动态增长的，阴差阳错地导致模型的保存逻辑始终为 False。我们眼睁睁看着 tensorboard 美丽的符合预期的 loss 曲线，但就是没办法让它 save，浪费了好一通算力。汲取教训之后，组里大佬就发明了一个机制，在训练代码加了个逻辑：如果检测到某个文件夹下存在一个叫 “save” 的文件，则立刻 save_checkpoint，我们将其称为模型动态保存机制（一脸骄傲）。

### 数据实验

当把以上的所有环节都串起来后，不要盲目的去开始训练，一定要先在小模型上做好实验，把 sacaling_law 的这个理念先搞懂。具体的实验内容，可以根据自己的时间、人力来三个阶段走：

*   • 粗糙一点的工作：在小模型上起多个数据配比、数据顺序，训练 500B 左右的数据量，然后选择 loss 曲线最完美，或者 loss 下降最低的那个模型（这个阶段刷 benchmark 意义不大，模型小，训得少，大概率都是瞎蒙）；
    
*   • 专业一点的工作：额外起多个 size 的小模型，跑出 loss 结果，结合 scaling_law 公式，去推算大模型最适合的数据配比、学习率、训练 token 量等参数；
    
*   • 创新一点的工作：像 llama 和 deepseek 技术报告里提到的一样，去绘制出 loss 到 benchmark 的 scaling_law，提前预知模型训多少 token 量能在某个 benchmark 达到什么样的能力。这个地方展开说的话，能说巨多的东西，但不方便细说，感兴趣的同学还是多读读各公司的技术报告吧。scaling_law 只是放缓了，不是死了，在没有新的技术指引的情况下，scaling_law 你不信也得信，它毕竟是用某种规则在做训练，按照自己的直觉来做训练基本等于 “random”。
    

**开弓没有回头箭，pretrain 的实验阶段一定要做的鲁棒一些**。

训练篇
===

### Tokenizer

我的同事提醒过我：扩词表容易把词表扩错，这和字典树的逻辑有关。简单来说，就是你加入 “中华人民” 这个新 token，并且引入相对应的 merge token 的逻辑，就可能导致 “中华人民共和国” 这个旧 token 永远不会被 encode 出来，那 “中华人民共和国” 这个 token 对应的知识也就丢失了。

因此，提前准备好自己的 tokenizer 真的非常重要，这就是一个打地基的工作。你如果想着后期可以扩词表解决，那和房子歪了你再加一根柱子撑着有啥区别呢？llama + 扩中文词表 + continue pretrain 训出来过效果惊艳的中文模型吗？

至于怎么训 tokenizer：找一个内存空间很大的 cpu 机器，再找一份很大的 common 数据集，然后利用 BPE / BBPE 算法去跑（ChatGPT 会写），这里只提醒一些细节。

*   • **数字切分**（虽然不知道 OpenAI 为什么不做了，但我们还是做吧，避免 9.9 > 9.11 的问题回答不正确)；
    
*   • **控制压缩率**，1 个 token 对应多少个汉字：压缩率太低，那就是字太多、词太少，很影响解码效率；压缩率太大，也就是词太多，又会影响模型的知识能力。通常，压缩率越低的模型，loss 也会低，大部份中文大模型的 1 个 token 会映射成 1.5 个汉字左右；
    
*   • **手动移除脏 token**，之前 GPT4o 的 token 词表泄漏， 就被发现有很多中文的色情、赌博 token；
    
*   • 如果提前知道自己的业务场景，那就应该**补充业务场景对应的 token**，增加业务场景文本的压缩率，比如医疗场景，就提前把阿莫西林、青霉素等作为一个 token；
    
*   • **词表的中、英覆盖率要足够大**，至于其他小语种是否要加，则看业务需求；
    
*   • tokenizer 的 vocab_size 和模型的 embedding_size 之间，要有一千个左右的 buffer，后续的 alignment 环节，需要大量在 pretrain 阶段没见过的全新 token 来做训练。
    

（说句题外话，我不太理解 strawberry 有几个 “r” 这种问题有啥好讨论的。这不是典型的训过就会，没训过就不会的问题嘛。你让模型背一下 “strawberry 的字母组成是 s, t, r, a, w, b, e, r, ,r , y" 不就解决了，这种 tokenizer 天生解决不了的问题，模型答出来了只能说明它训过，模型答不出来才是常态。希望这种噱头问题以后少点吧，不要天天拿着这种 case 吸引外行人，让路人都觉着 llm 实际上蠢的要命。）

### 模型结构

一个原则：能抄 llama 的结构就不要随便创新，就 rope + gqa + rms_norm + swiglu，**少创新 = 少踩坑，创新的前提是大量鲁棒的实验**。如果是 1B 左右很小的模型，那么 embedding 和 lm_head 还需要共享参数，目的是让 layer 的参数占全局参数的比例大一些，大一点的模型则没有这个必要。

**pretrain 是个成本极高的工作，一切都要以稳健为主**。假设你是 pretrain 负责人，你为了写论文，为了强行宣传自己的模型比 llama 更先进，做了一两周实验，在 0.5B 这种 size 发现新结构的 loss 下降更快。你大喜，也没去找两个数学系博士做做理论证明或推导，就去草草的去更改 llama 结构。等训练一个多月之后发现这个新结构有某个致命缺陷，几千万的算力资金已经投进去了。老板问你什么情况，你说当初在小模型上改结构没出问题啊！老板给你打了低绩效，自己心里又一百个不服气。

我觉着，国内 99% 的大模型技术岗，并不鼓励创新和试错，老板们只鼓励用最快的速度、最少的钱去追赶 OpenAI。除非你的老板真的支持你积极创新，否则不考虑试错成本盲目追求创新的人真的有点大病。

### 模型参数

OpenAI / Google / Meta 的早期论文，包含为什么模型要选定这 7、13、33、72 这几个 size 的模型，里面的各种观点看看懂个大概就好，也不能全信，我只从实际应用的角度出发来推荐参数敲定。

#### 模型的 size

我建议不要根据自己的场景需求来敲定模型 size（除非是 Math 等复杂任务必须是接近千亿参数的模型），小模型的极限在哪里目前仍然是个未知数，qwen2.5 也贴了一张图，在 benchmark 上能达到某个分数的模型 size，在持续变小。因为模型的 size 选小了，导致后续应用的时候业务场景扛不住，这种现象其实并不常见，alignment 基本都能救回来，无非就是 sft 的时候在业务能力上过拟合一些罢了。

因此，选模型 size 主要看两个因素，训练算力和推理算力：

*   • **训练算力**：手头有多少机器，能做多久 pretrain，有多少训练数据，使用的训练框架一天能吃多少 token，这些事情都是提前能确定的。假设要在 2 个月后训完模型，目标训 2T 数据，那么便可以计算出自己该训什么 size 的模型。尽量和大厂的模型 size 保持一致，不瞎创新就不会踩奇奇怪怪的坑，而且模型效果对比的时候也有说服力；
    
*   • **推理算力**：实操过模型的同学都知道，用 AutoModelForCausalLM 加载 33B 模型基本是卡着一张 80G 显存的极限在推理，加载 72B 模型基本上是卡着两张 80G 显存的极限在推理，稍微多一点 seq_len，立刻 OOM（暂不讨论量化等因素）。换句话也就是说，假设你有个 40B 的模型，他和 72B 模型效果一样，那又怎样呢？他还是得用两张卡推理，在部署成本上和 72B 模型还是一个量级的。所以，敲定模型 size 的时候，我们需要知道自己的推理机器是什么，不要出现 1 张推理卡刚好装不下模型的尴尬现象。适当给模型增大 1B、减小 1B 参数是可以的，这也能解释了为什么不是所有公司都用 70B，而是 65B、70B、72B、75B 等 size 都有的现象。
    

#### 模型的超参数 size

目前学界都有一个共识：一个好模型是 “身材匀称” 的。也就是说标准的 llama 结构应该是横向和纵向呈某种比例的，所以在增大模型 size 的时候，layer_num 和 hidden_size 要一起递增。具体如何递增，llama 论文和李牧老师视频里都有介绍，这里不在赘述了。我的观点依然是普通人没必要研究这个，超参数 size 的量级就和 llama 保持一致即可，少创新就等于少犯错。

但具体到该使用什么值的参数，还是有说法的：尽量能被 2 / 4 / 8 / 64 / 128 等数整除。不是有什么理论证明，而是训练框架要求你这样。

*   • **layer_num**：有尽量多的质因数，它与 pipeline_parallel 有关。pipeline_parallel 的要求是：assert layer_num % pipeline_size == 0。如果你的 layer_num = 30，那它就不能支持 pipeline_size = 4。当然，你可以修改训练代码，让 pipeline_parallel 的时候，不同的卡放置不同数量的 layer，但这不就又增加开发成本了；
    
*   • **num_head**：8 的整数倍，它与 tensor_parallel 有关。tensor_parallel 的极限一般是 8，因为大于 8 就引入了机间通讯，训练效率就低了。tensor_parallel 的效率很高，能开大就尽量开大一点；
    
*   • **hidden_states**：128 的整数倍，目前没有理由，保不齐以后有；
    
*   • **vocab_size**：128 的整数倍，目前没有理由，保不齐以后有。
    

另外一个比较重要的超参数是 seq_len 的选取：无论你的业务场景需不需要长文本，都不要一开始就使用 32K / 200K 这种夸张的数据 seq_len，算力根本承受不起，attention 的 seq_len^2 的计算量实在可怕。

rope 的 NTK 外推方法已经是各大厂标配的方案：4K/8K + rope 小 base + 90% 数据量 --> 32K/64K + rope 大 base + 10% 数据量。

### 训练框架

megatron 和 deepspeed 该怎么选？直接说结论：**从零开始的 pretrain 必须选 megatron，continue-pretrain 可以考虑使用 deepspeed**， 换句话说，**T 级别的 token 训练量必须是 megatron，B 的级别 token 训练量无所谓**。

#### megatron 的优点

*   • **训练速度快**：tensor_parallel 和 pipeline_parallel 被优化的炉火纯青，rope 已经被开发成了 apex 算子，速度远高于 llama 里的实现方案，据说 mlp 层的 apex 算子也正在开发。
    
*   • **参数清晰而明确**：argument.py 里，你能看见上百个参数配置，哪些层使用 dropout 等细节，训练框架早就帮你考虑好了，可微操的空间很大。
    
*   • **启动训练的时候模型加载速度快**，千亿级别的模型一分钟就加载完毕了，**debug 十分方便**。
    

#### megatron 的缺点

*   • **上手成本很高**，新手学习起来比较吃力，对 torch 的多机通讯需要从头开始学。而且基建工作比较多，trans_megatron_to_hf.py，trans_hf_to_megatron.py 对新人也挺折磨的。
    
*   • Nvidia 的官方代码库，**全是 bug**，不预留个一周时间进行 debug，很难直接跑起来。我有一个同事，非常迷信官方，隔两天就要 git pull megatron 的官方代码库，只能说工作量一言难尽。
    

#### deepspeed 的优点

*   • **代码简单**，超出想象的简单，而且有 model.trainer 这种大杀器框架。
    
*   • **用户群体多**，alignment 阶段的开源代码 90% 都是 deepspeed 实现。
    

#### deepspeed 的缺点

*   • **训练速度**慢，相比于 megatron 可能有 10% ～30% 左右的算力损失。当然，现在有很多工作已经实现了 deepspeed 的 pipeline_parallel 和 tensor_parallel，但相对的，用起来也就没那么简单了。
    
*   • **加载速度无敌超级十分很非常慢**！！！！！从 bash train.sh 到 看见 loss 出现，小模型得十几分钟，大模型得半个小时，如果你需要 debug，一个下午只够 debug 个三五次。
    
*   • 代码简单的代价是 “**微操很难**”，尤其是使用 model.trainer 训练，对源码几乎无修改空间。
    
*   • 微软的开源代码库在让人失望这件事上从不让人失望。megatron 有 bug 只是让你跑不起来，你修了就好了。deepspeed 的 bug 或者不合理的设置，并不影响你把代码跑起来，只有在你遇见解释不了的现象的时候才可能暴露。我的同事送给过我一句箴言：“遇见不合理的地方，不要质疑自己，无脑质疑 deepspeed 官方！”
    

哦对，无论是用哪个训练框架，都要记得把 attention 的默认方式换成 flash_attention。

### 训练技巧

#### 训练效率优化

在模型训练中，要始终谨记一条大原则，一条训练数据的训练速度：卡内通迅 > 卡间通讯 > 机间通讯。

能减小模型的通讯量就去减小，能避免机间通讯就去避免。在显存够用的情况下，能不引入 tensor_parallel，pipeline_parallel，sequence_parallel 就不要去引入，同样地、能不开 offload 就不要开 offload，能不开重算就不开重算。总结下来就是：

*   • data_parallel 几乎是唯一不牺牲算力的并行方式；
    
*   • 让数据在显存和内存之间切换，也是很耗时的，这种操作能避免就避免；
    
*   • 同样的结果多次计算，傻子都知道速度会变慢，因此能存储就别重算。
    

#### 训练 loss 分析

pretrain 训练环节中，最最最最最重要的中间产出，就是 tensorboard 的那条 loss 曲线。记住几条：

*   • 一定要有 channel_loss，至少，中文知识，英文知识，代码这三类数据的 loss 得分开观察；
    
*   • loss_spike 不可忽视，虽然目前还没有工作去证明说 loss_spike 会对模型造成不可逆的效果伤害，但不出现 loss_spike 总是好的啊。无论是 loss 突然激增还是突然激减，都要重点留意，大概率是数据问题（脏数据不仅 loss 高，也会 loss 低，全是乱码的数据 loss 很高，全是换行符的数据 loss 很低）。如果数据过脏，就回退到上个 checkpoint 进行训练；
    
*   • 即使数据没有问题，也是有可能出现 loss_spike 的，这时候要留意下数据的 grad 有没有异常，大概率和优化器有关，参考 llama 等的技术报告，好好调整 adamw 优化器的  这两个参数，基本上能解决大部分训练初期的 loss_spkie 问题。一旦熬过了训练初期，loss 平稳下降到了 2 ～3 左右，后续的训练也基本不会再有幺蛾子了。
    

### 训练流程

pretrain 训练流程目前基本是固定的，当训练数据和训练代码都准备就绪后，按照以下四个流程来设置学习率和超参数就可以啦：

*   • 开局 warmup，学习率缓慢上升到最大；
    
*   • 中间 cos / cos_decay / constant / constant_decay ，学习率比较大，是否 decay 自己决定，多翻翻别人的技术报告，或者在小模型上做实验决定；
    
*   • 后期改变 rope base + seq_len，开始让模型适应长文本；
    
*   • 收尾 anneal，用高精数据、IFT 数据等来给强化模型的考试能力，做完退火就该上考场刷 benchmark 了。
    

一般 pretrain 只要启动了，大家就可以去做别的工作了，只要模型训练没停，一般都不需要再有人为干预。烧卡了， loss 爆炸了，loss 陡然下降了（数据大概率重复了），就回退到上一个 checkpoint 重启。

评估篇
===

pretrain 的评估相对来说是 llm 全链路评估工作中最简单的环节，因为它并不需要考察模型的指令 follow 能力、安全能力、幻觉现象等，只需要看模型整体的知识掌握程度即可。

### PPL

机器学习的基本课：看测试集的 loss 来衡量模型效果。

准备一些百科、逻辑、code 等数据，每天观察模型在这些测试集合上的 loss 表现，偶尔浮动是正常的，但整体趋势肯定是持续下降，最后趋于稳定的。只要训练集中没这些语料，和训练曲线应该是基本一致的。

特别地、ppl 只能是自己的模型和自己比，前面说过，由于 tokenizer 的压缩率不同，不同模型的 loss 没有明确可比性，全是字没有词的 tokenizer，loss 一定是最低的。不过另一方面，不管你的 tokenizer 压缩率多少，在通用知识测试集上的 loss 也是应该能降低到 2 以下的，否则就说明有点训练不充分。

### Benchmark

如果 pretrain 阶段全程是自己在负责，那么 benchmark 还是有一定可信程度的。但无论是继承同事的 checkpoint，还是下载开源模型的 checkpoint，一律视为它们刷过榜。除了模型完全给自己用的公司，只要是有宣发需求的 pretrain 团队就一定会刷榜，否则根本和别的模型比不了。这种情况下，benchmark 考察的就是谁遗忘的少，而不是谁具有的知识多，训的越多模型得分就越低

即使排除了刷榜问题，现在最主流的 benchmark，形式也都实在是有点单一了。全是选择题，而且是没有 cot 环节的选择题，衡量模型的能力肯定是有偏颇的。cot 就是模型的草稿纸，就算是我们人，拿到一个选择题，不去演算一下就直接说出 A、B、C、D 该选哪一个，也是很匪夷所思的。

可是话说回来，benchmark 毕竟是现成的具有高密度知识的 question-answer 的高质量语料，不使用着实可惜。这里我的建议是改造 benchmark，无论是评估自己的模型还是评估别人的刷过榜的模型，尽量用生成式的方法去使用 benchmark，而不是看 A、B、C、D 哪个概率高。

改造可以是任意形式的，我举几个例子

*   • Question + Answer_A，Question + Answer_B，Question + Answer_C，Question + Answer_D。请结合上述文本，回答 Question；
    
*   • 把 Question 的正确答案丢弃，假设 B 选项是正确的答案，那就把它改成 “B. 其他答案全错”，看模型还能不能把 B 选出来；
    
*   • 把原始 benchmark 的 A / B / C / D 改成 一 / 二 / 三 / 四；
    
*   • 把多选题改造成单选题；
    
*   • Question + A / B / C / D，先把这个知识让模型不知道答案的情况下训一遍，然后让模型直接说出 Question 的正确答案；
    
*   • ……
    

如果担心模型不 follow 格式，那就用 few_shot 的方式让模型去抄袭格式，通常都能 follow 住；如果担心模型停止不了，就限制 max_new_token，或者是使用 StoppingCriteria，让模型见到换行符就停止。

总之，我们要灵活善变的来利用 benchmark，既然你训过了，那我就各种花式改造 benchmark 的使用方法，来跳过你训过的数据，最起码让模型背过的背答案失效——**考纲不变，考卷换了**。

改造方式大家可以自己多多揣测，找到自己最喜欢的那一款，反正是用来证明模型能力的，不是打榜的，得分高低没有意义，得分是否持续升高才有意义。还有，不管改造成什么形式，一定要用 ACC 来衡量评估结果。BLEU 和 Rouge 的得分还是太抽象了，连做 case 分析都不行。

### 概率探针

从概率的角度来监控模型的知识能力有没有遗忘或者提升，适用于我们要观察模型的某一项具体能力。思路也非常简单，我们就观察某个 token 的概率是否增加，某个句子的概率是否增加，唯一麻烦的地方是探针测试集往往需要训练者一条一条亲自去构造，而不能批量生成。

比较抽象，我举几个例子：

*   • Prob('北京'｜'中国的首都是')，就看一下这个概率值随着 pretrain 推进是否持续在增大；
    
*   • PPL('台湾属于中国')，这个句子的 ppl 是否持续在增大；PPL('台湾不属于中国')，这个句子的 ppl 是否持续在减小；
    
*   • 对比探针，PPL('尊重同性恋') > PPL('反对同性恋') 是否成立；
    
*   • 指令 follow 能力，Prob('{'| '以 json 输出')；
    
*   • ……
    

和 benchmark 改造类似，概率探针是可以根据自己的喜好任意构造的。我们重点观察的也是指标的变化趋势，而不是指标的绝对大小。

总结篇
===

pretrain 的全环节大抵如此，我列出来的每个环节我认为都是同等重要的。之前看见有种说法说洗数据是脏简历的工作，恕我不能认同。如果 infra 团队已经帮忙调通了 megatron 的训练代码，那么训练才是真的最没技术含量的工作，改几个参数，然后 bash train.sh，训练挂了就重启，这些工作谁做不来呢？反倒是洗数据时的灵光一现，往往能大大提升模型的效果。因此，“数据篇” 也是我笔墨最多的篇章。

希望这篇文章能帮到对 pretrain 感兴趣的同学，至于细节就不要再追问了，更多东西是真不能说了。另外，本文章由我的饭搭子赞助完成，文章的大多数内容都是和他一起讨论、实践所得。
