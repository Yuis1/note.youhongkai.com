---
{"dg-publish":true,"permalink":"/CS计算机科学/Drupal/Drupal多站点/","tags":["Drupal"],"noteIcon":"","created":"2024-03-04T00:17:44.729+08:00","updated":"2024-03-20T15:08:37.078+08:00"}
---


[Multisite Drupal | Getting started | Drupal Wiki guide on Drupal.org](https://www.drupal.org/docs/getting-started/multisite-drupal)

[Drupal 8多站点实现：使用一套Drupal代码运行多个不同网站](https://www.daweibro.com/node/167)

使用场景：

- 当所有站点都使用完全相同的模块和设置，只是主题外观不同，例如 大学有几十个系、或者是每个球队一个站点。这样当批量更新时，更省时间。

优点：

- 节省代码库的存储空间（大概500M）
- 节省数据库存储空间（大概40M）

弊端：

- 当其中一个站点的模块更新时，需要在所有站点都运行 update.php
- 当一个站点的代码出错时，影响所有站点