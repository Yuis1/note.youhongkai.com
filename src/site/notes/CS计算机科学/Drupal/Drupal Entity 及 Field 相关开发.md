---
{"dg-publish":true,"permalink":"/CS计算机科学/Drupal/Drupal Entity 及 Field 相关开发/","noteIcon":"","created":"2024-03-20T18:57:52.951+08:00","updated":"2024-04-23T23:03:43.305+08:00"}
---


作者: 游鱼思

---
## 字段校验

### Field Validation

[Field Validation | Drupal.org](https://www.drupal.org/project/field_validation)  D10 ，可以支持Unique values on multiple fields

配置页面：/admin/structure/field_validation

配置示例：[Module Monday: Field Validation | Lullabot](https://www.lullabot.com/articles/module-monday-field-validation)

但遗憾的是，Unique values on multiple fields 不支持 D10，变通的实现方案：先通过 Computed Field 计算出联合后的字段，再对这个字段进行唯一性约束。

### Rules » Unique Field Value  

比较遗憾，好像只支持单个字段校验，不支持多个字段联合校验。

以上是截止2024年，D10 可用的模块。

## 字段

### [Computed Field](https://www.drupal.org/project/computed_field)

Allows for fields to be populated through the use of PHP code, including other other values. This module is intended to be used by PHP developers to insert & execute PHP code on a Drupal website.

[README.md · 4.0.x · project / computed_field · GitLab (drupalcode.org)](https://git.drupalcode.org/project/computed_field/-/blob/4.0.x/README.md)

不好用，因为得编写模块和代码才能实现自定义计算字段的功能。

如果有这个功夫，我自己写个完整的模块不香吗？为什么要去开个模块写一个代码片段？

### [Field Token Value](https://www.drupal.org/project/field_token_value)

通过在字段中插入Token来自定义内容。