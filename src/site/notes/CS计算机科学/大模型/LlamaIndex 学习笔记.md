---
{"dg-publish":true,"permalink":"/CS计算机科学/大模型/LlamaIndex 学习笔记/","tags":["LLM","RAG"],"noteIcon":"","created":"2024-06-22T22:30:29.224+08:00","updated":"2024-04-24T00:08:04.000+08:00"}
---


作者：游鱼思

---

Llama Index 专注于RAG环节。  

## RAG 业务流

Loading - Indexing - Storing - Querying - Evaluating

## Loading
### 概念

Document：文档，数据源的容器，例如PDF、API、网页、数据库检索。Document由Nodes构成。  
Document包含文本数据和在头部包含一些文件的属性，如元数据、关系数据。

Node：文档块，在LlamaIndex中是原子粒度的数据。

Document作为数据窗口，对数据分割解析，Node就是相应的抽象。Node也包含了和Document一样的数据和属性。

parser = SimpleNodeParser.from_defaults()

nodes = parser.get_nodes_from_documents(documents)

### 流程

Loader - Transformations - Add Metadata - Embeddings

### Loaders

*SimpleDirectoryReader*

支持Markdown, PDFs, Word documents, PowerPoint decks, images, audio and video.

*Llama Hub Data Loaders*

提供了各种类型文档的加载器。

问题：有哪些loader可以从科研网站上查询论文数据？

回答：semantic scholar

问题：复杂排版的论文PDF，最好的解析器是什么？

回答：估计是 unstructured.io

*直接创建Documents*

doc = Document(text="text")

### Transformations

包括 chunking, extracting metadata, and embedding each chunk

```
Python

text_splitter = SentenceSplitter(chunk_size=512, chunk_overlap=10)

service_context = ServiceContext.from_defaults(text_splitter=text_splitter)

index = VectorStoreIndex.from_documents(

    documents, service_context=service_context

)

index.as_query_engine()

```

*metadata*

关于如何定义Metadata：

https://docs.llamaindex.ai/en/stable/module_guides/loading/documents_and_nodes/usage_documents.html

```
document = Document(

    text="text",

    metadata={"filename": "<doc_file_name>", "category": "<category>"},

)
```

#### Node Parser


## Indexing
### 概念

Indexes：索引。将数据进行向量化索引，同时也可存储元数据。  
Embedding：嵌入。当按相关性过滤数据时，LlamaIndex将查询语句转化为嵌入，向量存储就可以按嵌入查询的相似性来查找数据。  

索引方式包括：  

- List Index：Node顺序存储，可用关键字过滤Node  
- Vector Store Index：每个Node一个向量，查询的时候取top-k相似  
- Tree Index：树形Node，从树根向叶子查询，可单边查询，或者双边查询合并。  
- Keyword(关键字) Table Index：每个Node有很多个Keywords(关键字)链接，通过查Keyword(关键字)能查询对应Node。  

回复合成方式包括：  

- 创建并提纯（Create and Refine)，即线性依次迭代；  
- 树形总结（Tree Summarize）：自底向上，两两合并，最终合并成一个回复。  


### ServiceContext 组件

服务上下文。用来索引和检索阶段的公用资源包。  

text_splitter：文本切分器。将文本进行切块。

*PromptHelper*：提示词助手。帮助删减、重新打包文本块，以适应LLM的上下文窗口大小。  

- context_window: LLM上下文窗口。通常已基于模型元数据自动设置好，但也可在此调整。
- num_output: 模型的最大输出数。通常已基于模型元数据自动设置好。不真正限制模型输出，只在计算可用的上下文窗口时，影响我们节省的空间量。  

综合起来示例：  

```Python

llm = OpenAI(model="text-davinci-003", temperature=0, max_tokens=256)

embed_model = OpenAIEmbedding()

text_splitter = SentenceSplitter(chunk_size=1024, chunk_overlap=20)

prompt_helper = PromptHelper(

    context_window=4096,

    num_output=256,

    chunk_overlap_ratio=0.1,

    chunk_size_limit=None,

)

service_context = ServiceContext.from_defaults(

    llm=llm,

    embed_model=embed_model,

    text_splitter=text_splitter,

    prompt_helper=prompt_helper,

)

```

## Storing

提供了和各类向量数据库的存储交互。

### Storage context

存储上下文容器是存储节点、切片、向量的实用容器，包含了：  

- docstore: BaseDocumentStore
- index_store: BaseIndexStore
- vector_store: VectorStore
- graph_store: GraphStore

示例：

```Python

if not os.path.exists(persist_dir):

    # Load data from Semantic Scholar

    documents = s2reader.load_data(query_space, total_papers, full_text=full_text)

    index = VectorStoreIndex.from_documents(documents, service_context=service_context)

    index.storage_context.persist(persist_dir=persist_dir)

else:

    index = load_index_from_storage(

        StorageContext.from_defaults(persist_dir=persist_dir),

        service_context=service_context,)

```

## Querying
### 概念

Retrivers：检索器，定义了如何高效的从索引中检索相关的上下文。  
Routers：路由。决定了使用那个检索器，来从相关知识库中查找上下文。特别指出，RouterRetriever类，负责选择1~多个候选的检索器，以用于执行查询。  
Node Postprocessors：节点后置处理器。对检索到的节点，进行变换、过滤、重排等处理。  
Response Synthesizers：响应合成器。基于用户查询和检索到的文本块，使用LLM生成回应。  

### Query Engines

对数据进行单次的查询。面向过程。  

#### Citation query engine

检索结果包含原文的引用，这是个好东西，写论文必备。

对比Langchain，还要自己实现： https://python.langchain.com/docs/use_cases/question_answering/citations

Args:

retriever (BaseRetriever): A retriever object.

response_synthesizer (Optional[BaseSynthesizer]): A BaseSynthesizer object.

citation_chunk_size (int): Size of citation chunks, default=512. Useful for controlling granularity of sources.

citation_chunk_overlap (int): Overlap of citation nodes, default=20.

text_splitter (Optional[TextSplitter]): A text splitter for creating citation source nodes. Default is a SentenceSplitter.

callback_manager (Optional[CallbackManager]): A callback manager.

metadata_mode (MetadataMode): A MetadataMode object that controls how metadata is included in the citation prompt.

### Chat Engines

对数据进行交互式对话。面向过程。  

### Agents Engines

Agent：代理。面向目标。基于LLM构建的，可以自动决定采用哪个工具来和世界进行交互。代理可以使用任意步骤，来完成一个特定的任务，并对行为路线进行动态决策。LlamaIndex的许多组件是面向代理的，可以对特殊用例和数据自动决策。 另一方面，LlamaIndex也能作为其他代理框架的核心工具。  

## Evaluating

## Llama Hub LlamaPack 组件

基于预定义的接口，LlamaPack包预封装特定的组件与模版，帮助开发者加速构建大模型应用。

## 用例



### LangChain集成




