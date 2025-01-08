---
{"dg-publish":true,"permalink":"/CS计算机科学/WordPress/WordPress多语言/","noteIcon":"","created":"2024-07-10T02:35:07.000+08:00","updated":"2024-10-31T19:29:30.000+08:00"}
---

## 我的需求

1、不同的语言，在URL中有对应的子目录。例如 cn、en 等。

2、界面元素有直接可用的多语言 .po 文件，因为要面向全球推广，避免再手工翻译。

3、要求内容可以发布多个语言版本。

4、需要可以调用大模型自动翻译，且可以指定手工修正过的可以不自动更新翻译。

5、URL可以翻译，这样不会被谷歌认为是重复内容。

6、需要支持每种语言设置不同的域名。

## Gtranslate

https://gtranslate.io/

WP、Drupal、Shopify 都可用。

翻译好的内容存储在gtranslate的服务器上，不续费，不能再使用已翻译的内容。

如果要支持多域名，年付400$。

## Ploylang

不支持自动翻译，运营工作量增大了很多。

支持二级目录、二级域名、不同域名。

## TranslatePress

支持自动翻译，但是在页面加载的时候每次调用API翻译，成本巨高。

仅支持二级目录。

## Weglot

按字数付费， 20万字 5种语言  790欧/年。 790/(20*5) = 7.9欧/万字。

不支持多货币转换。

不考虑了。  

## WPML

x c

基于字段翻译，可以翻译WP系统字段、后台，支持扫描字段翻译。

支持自动翻译：Azure\Google\Deepl

一次翻译，永久保存。取消Pro版授权后，已翻译内容可继续使用。

按字数收费，google/deepl 是 1万字*4种语言，47欧。 47/(1*4) = 11.75欧/万字。

支持多语言多货币

支持二级目录、二级域名、不同域名，也支持多站点模式，并且 二级目录、二级域名、不同域名 可以和 多站点模式结合一起用。

和其它插件兼容性最好（超过3000个插件），有很多WPML兼容的主题可供选择（在WPML官网上 ）。

## 结论

WPML 就是事实上的标准。

## 参考资料

比较全面深度的测评对比：

https://www.bilibili.com/video/BV1cj411H7Z8/

https://www.bilibili.com/video/BV1xb4y1V7yo/

https://www.bilibili.com/video/BV1vb4y1V7HX/

https://www.bilibili.com/video/BV1ci4y1Y7j4/
