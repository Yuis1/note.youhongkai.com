---
{"dg-publish":true,"permalink":"/CS计算机科学/大模型/RAG/RAG产品对比/Grobid 专题/","noteIcon":"","created":"2024-06-22T22:30:29.391+08:00","updated":"2025-01-06T17:48:44.000+08:00"}
---


作者：游鱼思

---

Demo: https://kermitt2-grobid.hf.space/

GROBID 的核心优势在于文献管理和元数据提取，包括：

引文识别和链接

作者和附属机构的识别

头部和尾部（例如标题、摘要、关键词、参考文献）的识别和结构化

PDF 文档的内容提取和处理

对于版面恢复，GROBID 可以处理多列布局并提取文本内容，但它的主要目标不是提供精细的版面分析，如复杂图形、表格的布局恢复或详细的页面组织。

## 运行 Grobin
### 服务端

基于Java环境，最好是通过Docker运行：

https://grobid.readthedocs.io/en/latest/Run-Grobid/

拉取10G的完整版，含深度学习，精度更高：docker pull grobid/grobid:0.8.1-SNAPSHOT

不使用GPU：docker run --init --ulimit core=0 -p 8070:8070 --restart always grobid/grobid:0.8.1-SNAPSHOT

有个坑：由于作者没有设置 :latest ，所以通过 docker pull grobid/grobid 是获取不到的。

### 客户端

推荐用Python端

调用API： https://grobid.readthedocs.io/en/latest/Grobid-service/

PDF全文转TEI /api/processFulltextDocument

## 处理性能

https://github.com/kermitt2/grobid/issues/443#issuecomment-505208132

process with around 10.6 PDF per second (around 915,000 PDF per day, around 20M pages per day) with the node.js client listed above during one week on a 16 CPU machine (16 threads, 32GB RAM, no SDD).

一般设置的并发数=cpu线程数2倍，即可达到最快运行效率。

## 测试结论

目前Grobin对中文文献的支持并不好，许多论文读取的全文只有部分片段。

但是对英文文献支持较好。

优点：会读取doi、作者等元数据信息。