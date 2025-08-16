---
{"dg-publish":true,"permalink":"/CS计算机科学/前端开发/Json 查询语法/","noteIcon":"","created":"2024-06-22T22:30:30.063+08:00","updated":"2024-03-21T09:31:38.000+08:00"}
---

## JSONPath

一种用于在 JSON 数据中定位和提取特定信息的查询语言。它类似于 XPath 用于 XML 的功能。JSONPath 允许您以简洁的方式指定 JSON 结构中的路径，以访问特定的属性或元素。使用 JSONPath，您可以轻松地从复杂的 JSON 数据结构中提取所需的信息，而不需要手动编写复杂的代码来遍历和提取数据。

以下是 JSONPath 的一些示例用法：

- `$`：代表整个 JSON 结构。
- `$.store.book[0].title`：表示从 JSON 结构中提取第一本书的标题。
- `$.store.book[*].author`：表示提取所有书籍的作者。
- `$.store.book[?(@.price < 10)]`：表示查找价格低于 10 的所有书籍。

JSON & JSON Lines parser 是用于解析 JSON 数据和 JSON Lines 格式的工具或库。JSON (JavaScript Object Notation) 是一种常见的数据交换格式，用于在不同系统之间传输和存储数据。JSON 格式简洁清晰，易于阅读和编写，因此在 Web 开发和 API 设计中广泛应用。

在Drupal Feeds模块的Context定义中，需要用到这种语法。例如：[Setting Context for Extendable feeds | Feeds extensible parsers | Drupal Wiki guide on Drupal.org](https://www.drupal.org/node/3227985)

## JSON Lines

一种存储每行都是一个独立 JSON 对象的格式。与传统的 JSON 格式相比，JSON Lines 格式更适合处理大型数据集，特别是当数据以行为单位进行处理时。JSON Lines 格式在处理日志文件、数据流等场景中特别有用。

JSON & JSON Lines parser 的作用是将 JSON 格式的数据解析为程序可以处理的数据结构，例如对象或数组。它们可以将 JSON 数据转换为各种编程语言中的数据类型，使开发人员能够轻松地在其应用程序中使用 JSON 数据。

## JSONPath 和 JSON JMESPath 区别

都是用于在 JSON 数据中定位和提取特定信息的查询语言，但它们之间存在一些区别：

1. **语法差异**：
   - JSONPath 使用类似于 XPath 的语法，使用一系列简单的操作符来指定 JSON 结构中的路径。
   - JSON JMESPath 使用自定义的查询语法，该语法更简洁和灵活，并且具有更多高级功能，如过滤、投影、排序等。

2. **功能差异**：
   - JSONPath 通常用于简单的 JSON 结构的查询，支持基本的路径定位和提取操作。
   - JSON JMESPath 提供了更丰富的功能，包括嵌套查询、过滤、投影、排序等，使其更适用于复杂的数据查询和转换任务。

3. **支持范围**：
   - JSONPath 在不同编程语言中有不同的实现，但并不是所有的编程语言都有原生支持。
   - JSON JMESPath 是一个标准，具有多种编程语言的实现，并且有一致的语法和功能，因此更具有跨平台和跨语言的可移植性。

总的来说，JSON JMESPath 在功能上更加强大和灵活，语法更简洁，而且具有更好的跨语言支持。如果您需要进行复杂的 JSON 数据处理和转换操作，JSON JMESPath 可能是更好的选择。如果您只需要进行简单的 JSON 数据提取，JSONPath 可能会更加简单直观。