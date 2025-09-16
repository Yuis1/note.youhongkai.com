---
{"dg-publish":true,"permalink":"/CS计算机科学/大模型/RAG/理论和组件/Open Parse：构建 RAG 的开源利器，一站式解决复杂文档分块难题/","noteIcon":"","created":"2025-07-31T11:02:43.491+08:00","updated":"2025-01-08T23:57:17.000+08:00"}
---


原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/TSmgiyDWShvWEr3OL76UQg)

_**Open Parse 是一个开源工具，旨在解决复杂文档分块难题。它采用视觉化驱动的文档分析方式，能够保留文档的原始语义结构，并支持 Markdown 语法和高精度表格解析。Open Parse 可应用于各种场景，包括语义处理、信息提取、文档摘要和问答系统。**_

在文档处理的过程中，将复杂文档有效分块并不是一件容易的事。高质量的分块结果对 AI 应用程序的成功至关重要，但大多数开源工具在处理复杂文档时都存在一些缺陷。这就是 Open Parse 应运而生的原因 - 它旨在填补这一空白，为开发者提供一个灵活、易用且能有效分析文档布局的工具库。

**什么是 Open Parse?**

Open Parse 是一个旨在解决复杂文档分块难题的工具。与其他工具不同，它采用视觉化的方式驱动文档分析，而不是简单地将文档转换为纯文本再切分，因此能更好地保留原始文档的语义结构，如标题、章节和列表等。传统的文本分割则会丢失这些宝贵的信息。

![](/img/user/Z-attach/640.jpg)

此外，Open Parse 还支持解析 Markdown 语法和高精度提取表格内容，这是现有工具所欠缺的。总的来说，Open Parse 极大地增强了复杂文档处理的能力，为 AI 应用提供了高质量的输入数据源。

**Open Parse 的优势**

相比现有的文本分割、布局解析和商业解决方案，Open Parse 拥有以下明显优势：

**视觉化驱动的文档分析：** Open Parse 不是将文档简单地转换为纯文本再切分，而是先分析文档的视觉布局结构，因此能更好地保留语义信息。这种方式克服了文本分割的缺陷。

**保留原始语义结构：**由于采用视觉化分析，Open Parse 能够有效保留文档中的标题、章节、列表等结构信息，而不是将其视为扁平的文本块。

**支持 Markdown:** Open-Parse 支持基本的 Markdown 语法，如标题、粗体和斜体，方便用户处理 Markdown 文档，并将其转换为其他格式。

**高精度表格解析:** Open-Parse 的表格解析功能基于最先进的 Table Transformer (DETR) 模型，能够实现高精度的表格提取。Open-Parse 能够将表格准确地提取为 Markdown 格式，超越传统工具，有效解决复杂表格布局的解析难题。

![](/img/user/Z-attach/640-1.jpg)

**可扩展:** 用户可以轻松添加自定义的后处理步骤，以满足特定的需求，例如将解析后的内容用于信息提取、文档摘要等任务。

**易用性:** Open-Parse 提供良好的编辑器支持和完善的文档，易于使用和学习，降低了开发者的使用门槛。

**开源免费，优于商业解决方案：**与一些定价 10 美元 / 1000 页的商业解决方案不同，Open Parse 是一个完全开源和免费的工具。用户无需将数据共享给第三方供应商，降低了隐私风险。

**Open-Parse 应用场景**

Open-Parse 可以应用于各种场景，包括：

**语义处理:** 通过嵌入文本并进行聚类分析，将语义相似的节点分组，例如将同一主题的段落聚合在一起。

**信息提取:** 从文档中提取关键信息，如表格数据、标题和文本块，方便进行数据分析和知识图谱构建。

**文档摘要:** 生成文档的摘要，方便快速了解文档内容，提高信息获取效率。

**问答系统:** 将文档解析成语义单元，并与问答系统结合，实现对文档内容的精准问答。

****Open Parse** 核心代码**

**表格解析：openparse/tables/parse.py**

用于从 PDF 文档中提取表格数据的。它定义了多个函数和类，并使用不同的算法实现了表格解析功能。以下是代码的主要部分：

**1. 类定义:**

ParsingArgs: 基类，定义了基本的解析参数，包括 parsing_algorithm 和 table_output_format。

TableTransformersArgs: 继承自 ParsingArgs，用于 Table Transformer 模型的参数，包括置信度阈值和输出格式。

PyMuPDFArgs: 继承自 ParsingArgs，用于 PyMuPDF 库的参数，包括输出格式。

UnitableArgs: 继承自 ParsingArgs，用于 Unitable 模型的参数，包括置信度阈值和输出格式。

**2. 函数定义:**

_ingest_with_pymupdf: 使用 PyMuPDF 库解析表格。

_ingest_with_table_transformers: 使用 Table Transformer 模型解析表格。

_ingest_with_unitable: 使用 Unitable 模型解析表格。

ingest: 根据 parsing_args 选择不同的解析算法并调用相应的函数。

**3. 代码逻辑:**

ingest 函数是主要的入口点，它根据传入的 parsing_args 选择不同的解析算法。

每个解析函数都会将 PDF 文档转换为图片，并使用相应的算法识别表格区域和内容。

解析后的表格内容会被转换为指定的格式（字符串、Markdown 或 HTML），并以 TableElement 对象的形式返回。

**4. 技术细节:**

代码使用了 Pydantic 库进行数据验证和类型提示。

Table Transformer 和 Unitable 模型需要安装额外的依赖库，如 torch、torchvision 和 transformers。

代码使用了 PyMuPDF 库进行 PDF 处理和图像转换。

Open parse 提供了三种不同的方法来从 PDF 文档中提取表格内容, 分别基于 PyMuPDF、Table Transformer 和 Unitable 模型。用户可以根据需求选择合适的解析算法和输出格式。

**表格解析：openparse/pdf.py**

这个 Python 代码文件定义了 Pdf 类，用于处理 PDF 文件。它结合了 pypdf 和 pdfminer.six 库的功能，并提供了以下功能：

**1. PDF 文件操作:**

读取 PDF 文件。

保存修改后的 PDF 文件。

提取指定页面范围的 PDF 内容。

将 PDF 转换为 PyMuPDF (fitz) 文档对象。

**2. PDF 页面布局分析:**

使用 pdfminer.six 提取页面布局信息，包括文本块、图片等元素的位置和大小。

**3. 可视化和标注:**

在 PDF 页面上绘制边界框，以便直观地显示元素的位置。

支持使用 IPython 显示标注后的 PDF 页面。

可以将标注后的 PDF 导出到文件。

**4. 主要类和函数:**

Pdf: 主要的类，封装了 PDF 文件操作和布局分析功能。

_BboxWithColor: 用于存储边界框和颜色信息的辅助类。

_random_color: 生成随机颜色的函数。

_prepare_bboxes_for_drawing: 将边界框列表转换为 _BboxWithColor 对象列表的函数。

extract_layout_pages: 使用 pdfminer.six 提取页面布局信息的函数。

save: 保存 PDF 文件的函数。

extract_pages: 提取指定页面范围的函数。

to_pymupdf_doc: 将 PDF 转换为 PyMuPDF 文档对象的函数。

_draw_bboxes: 在 PDF 页面上绘制边界框的函数。

display_with_bboxes: 使用 IPython 显示标注后的 PDF 页面的函数。

export_with_bboxes: 将标注后的 PDF 导出到文件的函数。

_flip_coordinates: 转换坐标系的函数。

**5. 代码逻辑:**

Pdf 类初始化时，会读取 PDF 文件并创建 PdfReader 和 PdfWriter 对象。

extract_layout_pages 函数使用 pdfminer.six 提取页面布局信息。

display_with_bboxes 和 export_with_bboxes 函数使用 _draw_bboxes 函数在 PDF 页面上绘制边界框，并使用 PyMuPDF 库进行可视化和导出。

**6. 技术细节:**

代码使用了 pypdf 库进行 PDF 文件操作。

使用了 pdfminer.six 库进行页面布局分析。

使用了 PyMuPDF (fitz) 库进行可视化和导出。

使用了 IPython 库进行交互式显示。

**Open-Parse 的安装和使用**

**Open-Parse  安装**

 **核心库安装**

```
pip install openparse
```

**启用 OCR 支持**

要启用 OCR 功能，需要先安装 Tesseract OCR，并通过设置 TESSDATA_PREFIX 环境变量来指定 Tesseract 语言数据包的路径。

**安装机器学习表格检测模型  
**

如需使用高精度的表格检测功能，需要额外安装 Table Transformer 模型：  

```
pip install "openparse[ml]" 

openparse-download
```

**基本用例**

使用 Open Parse 非常简单，只需几行代码即可：

```
import openparse  

doc_path = "./sample-docs/doc.pdf"
parser = openparse.DocumentParser() 
parsed_doc = parser.parse(doc_path)

for node in parsed_doc.nodes:
    print(node)
```

‍

**语义分块处理**

如果需要根据语义对节点进行分组，可以添加语义处理管道：

```
from openparse import processing, DocumentParser

semantic_pipeline = processing.SemanticIngestionPipeline(
    openai_api_key=OPENAI_API_KEY,
    model="text-embedding-ada-002",
    min_tokens=64, 
    max_tokens=1024,
)
parser = DocumentParser(processing_pipeline=semantic_pipeline)
parsed_content = parser.parse(doc_path)
```

Open Parse 提供了详细的 Cookbooks 教程，帮助用户快速上手：  

https://github.com/Filimoa/open-parse/tree/main/src/cookbooks  

官方文档则覆盖了更多高级用法：

https://filimoa.github.io/open-parse/

![](/img/user/Z-attach/640-54.png)

Open-Parse 是一个具有潜力的开源项目，未来将会支持更多文档格式和功能，例如：  

支持更多语言的文档解析，满足不同地区和领域的需求。

支持更复杂的文档结构，例如嵌套表格，提升解析能力和适用范围。

提供更强大的语义处理功能，例如命名实体识别、关系抽取等，为下游任务提供更丰富的信息。

Open Parse 作为一个功能丰富的文档分块工具，可以极大地提高复杂文档处理的效率，并为构建高质量的 AI 应用提供有力支持。