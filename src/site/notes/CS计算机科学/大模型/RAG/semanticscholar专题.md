---
{"dg-publish":true,"permalink":"/CS计算机科学/大模型/RAG/semanticscholar专题/","noteIcon":"","created":"2024-06-22T22:30:29.361+08:00","updated":"2024-04-24T00:06:09.000+08:00"}
---


作者：游鱼思

---
## 简介

https://www.semanticscholar.org/

提供了2亿篇论文的智能检索，有摘要和元数据，大部分可以下载全文。

库中最重要的部分：对文档进行分区。 文档分区的目标是读取源文档，将文档拆分为多个部分，对这些部分进行分类， 并提取与这些部分关联的文本。根据文档类型，非结构化使用不同的方法 对文档进行分区。

## LlamaIndex集成

https://llamahub.ai/l/semanticscholar

LlamaIndex的 SemanticScholarReader 类，实现逻辑：

- 初始化：arxiv、base_dir 用来存全文
- _download_pdf()：只下载pdf，并切片成1k，保存到bash_dir。
- _get_full_text_docs()：根据元数据中的id、url，下载全文。如果id包含ArXiv，也会从ArXiv下载。返回全文列表，其中包括全文文本、元数据。
- load_data()：输入query，返回命中的文档的元数据（title, abstract, authos 等）