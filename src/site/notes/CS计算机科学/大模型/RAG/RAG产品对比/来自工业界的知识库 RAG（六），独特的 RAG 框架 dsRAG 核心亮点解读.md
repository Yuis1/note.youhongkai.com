---
{"dg-publish":true,"permalink":"/CS计算机科学/大模型/RAG/RAG产品对比/来自工业界的知识库 RAG（六），独特的 RAG 框架 dsRAG 核心亮点解读/","noteIcon":"","created":"2025-01-29T13:56:28.411+08:00","updated":"2025-01-13T17:33:24.000+08:00"}
---

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [hustyichi.github.io](https://hustyichi.github.io/2024/08/28/dsrag/)

> 来自易迟的博客，什么都会涉及，随意看看

背景介绍
----

在前面介绍了较多的开源 RAG 框架，比如主打 Rerank 的 [QAnything](https://zhuanlan.zhihu.com/p/697031773), 主打精细文件解析的 [RagFlow](https://zhuanlan.zhihu.com/p/697902937), 主打模块化灵活组合的 [GoMate](https://zhuanlan.zhihu.com/p/705218535)。这些库的设计除了少量的独特之处外，相似的部分很多。

最近有注意到一款另类的 RAG 框架 [dsRAG](https://github.com/D-Star-AI/dsRAG)，使用了较多独特的 RAG 优化方案，因此花了一些时间对其核心亮点进行了考察，整理相关内容在这边。

核心亮点
----

官方对 dsRAG 的介绍是一款高性能非结构化数据检索引擎，从 2024 年 4 月开源至现在，Github star 数量为 730。相对其他 RAG 项目动辄几千 star 的量级，dsRAG 属于比较冷门的了。

从实际了解的情况来看，这个现状也比较正常，dsRAG 确实不太适合直接用在生产环境。但是不可否认 dsRAG 的独特之处确实值得了解下：

1.  语义分块（Semantic sectioning）：相对其他框架大多采取 Level 2 级别的递归字符切片，部分实现的是 Level 3 级别的基于文件类型的切片方案，而 dsRAG 实现的是 Level 4 级别的语义分块，有兴趣了解切片的等级可以查看 [5_Levels_Of_Text_Splitting](https://github.com/FullStackRetrieval-com/RetrievalTutorials/blob/main/tutorials/LevelsOfTextSplitting/5_Levels_Of_Text_Splitting.ipynb) ;
2.  自动上下文（AutoContext）：这个是指在分块的内容前增加额外的上下文信息，这样可以提升单个块的内容的可检索性。目前大部分 RAG 框架都没有实现对应的能力；
3.  文本块智能组合（Relevant Segment Extraction）：为了避免分片信息不完整的问题，dsRAG 会将检索的文本块智能组合，这样在面对复杂问题时可以提供更完整的信息。目前大部分的 RAG 框架没有实现类似的功能；

下面就主要围绕上面提到的亮点，从原理出发，深入到实现层面，理解其中的技术细节。

功能细节
----

#### 语义分块

语义分块是指基于语义的相似性切分文本，[langchain](https://python.langchain.com/v0.1/docs/modules/data_connection/document_transformers/semantic-chunker/) 中就实现了类似的能力，langchain 会将原始文本切分为句子，之后比较句子向量的相似性，如果相似性超过一定的阈值，则进行切分。这样可以一定程度避免固定块切分带来的语义不完整的问题。

而 dsRAG 的语义分块是基于大模型实现，主要的流程如下所示：

1.  将原始的文本按照换行符 `\n` 切分为按行的句子；
2.  利用大模型生成分片方案，考虑到大模型的语义理解能力，大模型给出的分片可能比常规的向量模型更准确；

实际实现时基于大模型的结构化输出能力，因此目前只支持 OpenAI 和 Anthropic 的大模型：

```
system_prompt = """
Read the document below and extract a StructuredDocument object from it where each section of the document is centered around a single concept/topic. Whenever possible, your sections (and section titles) should match up with the natural sections of the document (i.e. Introduction, Conclusion, References, etc.). Sections can vary in length, but should generally be anywhere from a few paragraphs to a few pages long.
Each line of the document is marked with its line number in square brackets (e.g. [1], [2], [3], etc). Use the line numbers to indicate section start and end.
The start and end line numbers will be treated as inclusive. For example, if the first line of a section is line 5 and the last line is line 10, the start_index should be 5 and the end_index should be 10.
The first section must start at the first line number of the document ({start_line} in this case), and the last section must end at the last line of the document ({end_line} in this case). The sections MUST be non-overlapping and cover the entire document. In other words, they must form a partition of the document.
Section titles should be descriptive enough such that a person who is just skimming over the section titles and not actually reading the document can get a clear idea of what each section is about.
Note: the document provided to you may just be an excerpt from a larger document, rather than a complete document. Therefore, you can't always assume, for example, that the first line of the document is the beginning of the Introduction section and the last line is the end of the Conclusion section (if those section are even present).
"""

class Section(BaseModel):
    title: str = Field(description="main topic of this section of the document (very descriptive)")
    start_index: int = Field(description="line number where the section begins (inclusive)")
    end_index: int = Field(description="line number where the section ends (inclusive)")


class StructuredDocument(BaseModel):
    """obtains meaningful sections, each centered around a single concept/topic"""
    sections: List[Section] = Field(description="a list of sections of the document")

# 大模型输出结构化内容

client.chat.completions.create(
    model=model,
    response_model=StructuredDocument,
    max_tokens=4000,
    temperature=0.0,
    messages=[
        {
            "role": "system",
            "content": system_prompt.format(start_line=start_line, end_line=end_line),
        },
        {
            "role": "user",
            "content": document_with_line_numbers,
        },
    ],
)


```

可以看到就是一个简单的 ChatGPT 调用，就根据文本生成结构化的结果。后续根据分片的 `start_index` 和 `end_index` 确定分片的开始和结尾内容，从而实现了语义分片。

#### 自动上下文

自动上下文主要期望解决原始文本不容易检索的问题，实际在生产环境应用过 RAG 服务很容易发现，原始文档中总是会存在部分内容不太容易检索。

比如以一个疾病的论文为例，可能会存在一段介绍疾病对应的治疗方案，但是治疗方案的内容中可能没有相关疾病的描述，如果检索相关病例的治疗手段，可能会发现无法在大量的文本中精准命中对应的治疗手段。

dsRAG 补充的上下文信息目前支持文档标题，文档摘要，文档块的标题，文档块的摘要。而这些内容从何而来呢？答案与上面的分片一样：大模型。

dsRAG 为上面的四种类型的数据分别实现一个对应的 prompt，之后就可以借助大模型实现对应的功能了，举例来看，实现文档标题的对应的 prompt 如下所示：

```
DOCUMENT_TITLE_PROMPT = """
INSTRUCTIONS
What is the title of the following document?

Your response MUST be the title of the document, and nothing else. DO NOT respond with anything else.

{document_title_guidance}

{truncation_message}

DOCUMENT
{document_text}
""".strip()


```

具体的生成过程就是一个简单的大模型调用，这部分就不展示具体的实现了。

生成对应的上下文数据之后，将上下文与分片的内容组合在一起进行了向量化，这样就得到了信息更丰富，更容易检索的文本块了。

#### 文本块智能组合

文本智能组合（Relevant Segment Extraction, RSE）是将检索到的文本块进行聚类，并进行智能组合形成更长的文本。通过使用 RSE 方案，dsRAG 在回答复杂问题，特别是需要跨越多个文本的问题时就具备明显优势。

根据官方给出的测试数据来看，RSE 是对效果影响最大的策略，官方测试评分如下所示：

<table><thead><tr><th>&nbsp;</th><th>Top-k</th><th>RSE</th><th>CCH+Top-k</th><th>CCH+RSE</th></tr></thead><tbody><tr><td>AI Papers</td><td>4.5</td><td>7.9</td><td>4.7</td><td>7.9</td></tr><tr><td>BVP Cloud</td><td>2.6</td><td>4.4</td><td>6.3</td><td>7.8</td></tr><tr><td>Sourcegraph</td><td>5.7</td><td>6.6</td><td>5.8</td><td>9.4</td></tr><tr><td>Supreme Court Opinions</td><td>6.1</td><td>8.0</td><td>7.4</td><td>8.5</td></tr><tr><td>Average</td><td>4.72</td><td>6.73</td><td>6.04</td><td>8.42</td></tr></tbody></table>

可以看到，通过单纯的 RSE 策略就可以取得不错的分数，当然自动上下文（CCH） + 文本块智能组合（RSE） 的组合肯定能取得更高的分数。

RSE 实现的效果如下所示：

![](/img/user/Z-attach/rse.png)

在上面的图中，dsRAG 向量检索命中了 `chunk1`, `chunk3`, `chunk4`。如果不使用任何优化手段，大模型拿到的是独立的三个分片，信息都是碎片化的。遇到需要综合这些文本块的问题时，往往效果不佳。

通过 RSE 机制，实际可能组合生成的是一个包含 `chunk1`, `chunk2`, `chunk3`, `chunk4` 完整片段，从而可以提供更完整的信息用于问题回答，大模型返回的答案预期也会更好。

RSE 主要用于对检索后的文本进行组合，其中包含的步骤如下所示：

1.  计算文件中检索命中的第一块到最后一块文本的相关性评分，注意没有被检索到的文本块可能也需要评分。比如文本中包含 `chunk1` ~ `chunk10` 的文本块，但是检索时命中了 `chunk3`, `chunk4`, `chunk7`, 那么就需要计算 `chunk3` ~ `chunk7` 的文本块的相关性评分；
2.  计算文件中所有可能的连续组合文本块的评分，选择最高的评分的组合文本块进行使用；

**评分计算**

文本块的评分是基于文本块的排名，相似分计算出来，并使用文本长度进行缩放，计算的公式如下所示：

![](/img/user/Z-attach/score.png)

其中的变量如下所示：

*   `rank`: 为检索排名，如果没有检索到，那么排名为 1000;
*   `relevance_value`: 为检索相关性分数，没有检索到的情况下，分数为 0
*   `chunk_length`： 为文本长度，可以看到文本长度越大，评分是线性增加的；
*   `delay_rate`: 衰减率常量，可以决定排名的相对重要性；
*   `chunk_penalty`: 无关块的惩罚常量，作为一个阈值调整获得长文本块的倾向性，当值为 0.05 时，一般会生成 20~50 个的长文本块，当值为 0.4 时，一般给出会给出 1~3 个文本块的短文本块。

可以看到检索排名 `rank` 会对最终结果产生指数级的差异，检索相似分 `relevance_value` 会对结果产生线性差异，而最终的评分会被文本长度 `chunk_length` 进行线性缩放。

其实现如下所示：

```
# 计算原始评分

def get_chunk_value(chunk_info: dict, irrelevant_chunk_penalty: float, decay_rate: int):
    rank = chunk_info.get('rank', 1000)
    absolute_relevance_value = chunk_info.get('absolute_relevance_value', 0.0)
    v = np.exp(-rank / decay_rate)*absolute_relevance_value - irrelevant_chunk_penalty
    return v

# 基于文本长度进行评分缩放

def adjust_relevance_values_for_chunk_length(relevance_values: list[float], chunk_lengths: list[int], reference_length: int = 700):
    adjusted_relevance_values = []
    for relevance_value, chunk_length in zip(relevance_values, chunk_lengths):
        adjusted_relevance_values.append(relevance_value * (chunk_length / reference_length))
    return adjusted_relevance_values



```

**组合确定**

根据上面计算的各个文本块的评分，需要确定各个连续分块组合的最大值，熟悉算法的同学应该都想到经典的滑窗算法，但是 dsRAG 实现的是暴力求和，具体如下所示：

```
for start in range(len(relevance_values)):
    # 跳过开始为 0 情况

    if relevance_values[start] < 0:
        continue

    for end in range(start+1, min(start+max_length+1, len(relevance_values)+1)):
        # 计算连续块的得分，更新可能的最大得分

        segment_value = sum(relevance_values[start:end])
        if segment_value > best_value:
            best_value = segment_value
            best_segment = (start, end)


```

通过上面的实现可以容易理解，RSE 会尽可能扩张与问题相关的文本块。即使当前块没有被检索到，如果前后临近文本块都被检索到，那么同样有可能会被加入组合中。在实际的文本中，如果前后的文本块都与问题相关，那么当前块大概率也是与问题相关的。

熟悉滑动窗口的都知道，滑动窗口算法会尽可能扩张大于 0 的窗口值，这样就能理解为什么上面计算公式中的 `chunk_penalty` 可以直接快速影响最终组合块的长度了。

总结
--

dsRAG 是一个比较有意思的 RAG 库，提供了一些不同常规的 RAG 优化思路，在较多的地方大幅高频使用大模型可能会导致生产环境使用的成本过高，但是不妨碍其中的思路值得借鉴，特别是 RSE 机制，预期可以对 RAG 的效果有明显的改善。

* * *