---
{"dg-publish":true,"permalink":"/CS计算机科学/知识图谱/LLM-KG/通过LLM辅助知识图谱构建/","noteIcon":"","created":"2024-03-27T17:10:37.209+08:00","updated":"2024-07-08T16:27:17.289+08:00"}
---


## 情况的变化

之前要实现这种信息抽取的话，一般是用 Bert + NER 来实现，要用到几千个样本，才能开发出一个效果相对不错的模型。而到了大语言模型时代，我们有了 few-shot 和 zero-shot 的能力。

有了大语言模型之后，用户对数据的需求会减少很多，对大多数人而言，你不需要那么多预算去搞数据了，大语言模型就能实现数据的简单抽取，满足你的业务基本需求，再辅助一些规则就可以。

[4种通过LLM进行文本知识图谱的构建方法对比介绍](https://mp.weixin.qq.com/s?src=11&timestamp=1711530523&ver=5164&signature=ScupIaFex6KHsmFdGtKIXM8620-YBMKenNGR9BeFQ2GwBwWZJ7cReFceMe0z1nxKS5bgtQLeANM3p0V1mDFQQn1APjFtXTka7yeO2qFOqDE5mveiDxxS7g-4qm2Tg7mI&new=1)  

总结：

1. 可以让LLM自己生成 自定义本体 微调所需要的提示词
2. 使用微调后的LLM，来提取知识图谱，可以节省token，提升性能
3. 建立文本到KG转换的“准确性”测试
4. 转换后进行实体对齐
