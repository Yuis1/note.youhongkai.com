---
{"dg-publish":true,"permalink":"/CS计算机科学/知识图谱/基础知识/KG 开发概述/","noteIcon":"","created":"2024-04-20T15:32:28.000+08:00","updated":"2024-04-24T00:29:04.347+08:00"}
---


作者：游鱼思

---
## 库
1. RDFLib：Python中广泛使用的RDF(资源描述框架)工具包,它提供了一个用于处理RDF数据的核心基础架构。RDFLib可用于解析、创建和操作RDF数据,构建本体和知识图谱模型。支持解析和序列化RDF/XML、N3、NTriples、N-Quads、Turtle等格式。
2. OWL-RL：用于设计和操作基于OWL(Web Ontology Language)本体和知识库的Python框架。它提供了一个高性能和内存高效的推理引擎,支持OWL的规则语言子集。
3. Owlready2：用于本体管理和本体导向编程的库，特别适合处理OWL本体。面向对象的Python模块,用于加载和保存本体,执行推理,并支持SWRL规则。它以Python语法访问OWL本体,使得操作本体和知识库变得更加方便。
2. **Py2neo**：是一个客户端库，用于在Python应用程序中工作和操作Neo4j图形数据库。
3. **SPARQLWrapper**：用于包装对SPARQL服务的访问，使得在Python中执行SPARQL查询变得简单。
4. **rdflib-jsonld**：用于在RDFlib库中处理JSON-LD数据的插件。

## 框架

[zjunlp/DeepKE: [EMNLP 2022] An Open Toolkit for Knowledge Graph Extraction and Construction (github.com)](https://github.com/zjunlp/DeepKE)

[quqxui/MPIKGC: [LREC-COLING 2024] Multi-perspective Improvement of Knowledge Graph Completion with Large Language Models (github.com)](https://github.com/quqxui/MPIKGC)

通过从各个角度查询大型语言模型（LLM）来弥补上下文知识的不足，包括利用LLM的推理、解释和摘要功能来分别扩展实体描述、理解关系和提取结构，其旨在回答**如何有效地利用语言模型的能力和知识来改进图形学习？**的经典问题。

其核心解决的问题是，**从三个角度研究通过查询LLM来提高KGs的质量**，通过设计思想链提示来扩展实体描述，通过设计全局、局部和反向提示来增强对关系的理解，以及通过关键词摘要和匹配来提取结构数据。