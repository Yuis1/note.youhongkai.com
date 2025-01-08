---
{"dg-publish":true,"permalink":"/CS计算机科学/Drupal/数据交互/Drupal8 中 jsonApi 的使用拓展/","noteIcon":"","created":"2024-03-24T16:57:03.990+08:00","updated":"2024-04-30T00:41:36.618+08:00"}
---

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6844904191266291720?from=search-suggest)

概述
--

笔者在[《Drupal8 网站的 API 接口调用与权限控制》](https://juejin.cn/post/6844904190657953800 "https://juejin.cn/post/6844904190657953800")一文中，介绍了如何用 Drupal8 作为后端数据管理平台，前端通过 API 调用数据接口。实现 Drupal8 的数据解耦，与小程序和移动 APP 进行无缝衔接。

本文进一步介绍 jsonapi 的拓展功能。包括如何添加相关的扩展模块，实现 jsonapi 接口文档的自动生成和优雅展示。

相关模块介绍
------

### 1.JSON:API Extras

*   [JSON：API 模块](https://link.juejin.cn?target=https%3A%2F%2Fwww.drupal.org%2Fproject%2Fjsonapi "https://www.drupal.org/project/jsonapi")提供零配置开箱。**使用 JSON：API Extras 自定义您的 API**。JSON：API Extras 提供了一种方法来覆盖 JSON：API 模块提供的默认零配置实现并为其提供有限的配置。
*   使用 JSON：API Extras 自定义 [JSON：API](https://link.juejin.cn?target=https%3A%2F%2Fwww.drupal.org%2Fproject%2Fjsonapi "https://www.drupal.org/project/jsonapi") 模块生成的 [API](https://link.juejin.cn?target=https%3A%2F%2Fwww.drupal.org%2Fproject%2Fjsonapi "https://www.drupal.org/project/jsonapi")。
*   以下是 JSON：API Extras 模块的当前功能：

    1.  启用和禁用资源。
    2.  覆盖资源的名称。示例：`article`代替`node--article`
    3.  覆盖资源的路径：示例：`api/articles`代替`jsonapi/node/article`
    4.  禁用字段。
    5.  使用字段别名。示例：`tags`代替`field_tags`。
    6.  增强现场输出。示例：使用`date_time`字段增强器格式化日期字段。

### 2. OpenAPI for JSON:API

*   与基本的 Open API 模块集成以生成 JSON：API 的定义。如果您需要由 JSON：API 驱动的 API 的文档，则需要启用此模块和文档生成器，例如 [Open API UI ReDoc](https://link.juejin.cn?target=https%3A%2F%2Fwww.drupal.org%2Fproject%2Fopenapi_ui_redoc "https://www.drupal.org/project/openapi_ui_redoc")。请查看 [Open API 项目](https://link.juejin.cn?target=https%3A%2F%2Fwww.drupal.org%2Fproject%2Fopenapi "https://www.drupal.org/project/openapi")，以获取有关替代 UI 生成器的更多信息。
*   此模块需要 Open API 2.x 或更高版本。
*   该模块确实为 Open API 提供了必要的插件，以服务于 JSON：API 定义文档。为此，它至少需要 **[Open API（8.x-2.x）](https://link.juejin.cn?target=https%3A%2F%2Fwww.drupal.org%2Fproject%2Fopenapi "https://www.drupal.org/project/openapi")**。

    
### 3. ReDoc for OpenAPI UI

*   [ReDoc](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FRebilly%2FReDoc "https://github.com/Rebilly/ReDoc") 是一个 JavaScript 库，允许用户浏览 Web 服务 API 的 api 文档。该模块提供 ReDoc 库，用于在 Drupal 站点中显示 OpenAPI 规范。使用该项目需要 [OpenAPI UI](https://link.juejin.cn?target=https%3A%2F%2Fwww.drupal.org%2Fproject%2Fopenapi_ui "https://www.drupal.org/project/openapi_ui")，因为它提供了基础架构。请访问 OpenAPI 项目页面以获取更多信息。
*   该模块需要 [OpenAPI UI 模块](https://link.juejin.cn?target=https%3A%2F%2Fwww.drupal.org%2Fproject%2Fopenapi_ui "https://www.drupal.org/project/openapi_ui")。

### 4.OpenAPI UI

*   OpenAPI UI 模块围绕在 Drupal 站点内显示 OpenAPI 规范实现了一个 API。该库实现了一个插件库，可用于在您的网站中初始化 API 资源管理器 UI。
*   该模块没有任何已实现的插件，但是 [Swagger UI](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fswagger-api%2Fswagger-ui "https://github.com/swagger-api/swagger-ui") 和 [ReDoc](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FRebilly%2FReDoc "https://github.com/Rebilly/ReDoc") 的实现可以作为 Drupal 模块使用。
*   请访问 [Swagger for OpenAPI UI 模块](https://link.juejin.cn?target=https%3A%2F%2Fwww.drupal.org%2Fproject%2Fopenapi_ui_swagger "https://www.drupal.org/project/openapi_ui_swagger")和 [ReDoc for OpenAPI UI 模块](https://link.juejin.cn?target=https%3A%2F%2Fwww.drupal.org%2Fproject%2Fopenapi_ui_redoc "https://www.drupal.org/project/openapi_ui_redoc")页面，以获取有关使用这些库的信息。
*   [OpenAPI](https://link.juejin.cn?target=https%3A%2F%2Fwww.openapis.org%2F "https://www.openapis.org/") 是用于记录 Web 服务 api 的规范，该规范是一致的，开发人员可以重复使用。使用一致的格式可以重用 api 客户端库，并获得学习新 api 的一致体验。
*   该模块通过在 Drupal 和库之间提供集成以显示 api 规范供开发人员和最终用户使用，从而适合 OpenAPI 生态系统。有关 Drupal api 的 OpenAPI 规范以及其他贡献模块的 API 的信息，请查看 [OpenAPI 模块](https://link.juejin.cn?target=https%3A%2F%2Fwww.drupal.org%2Fproject%2Fopenapi "https://www.drupal.org/project/openapi")。

示例
--

### 安装并启用扩展模块

```
composer require drupal/jsonapi_extras drupal/openapi_jsonapi drupal/openapi_ui_redoc drupal/openapi_ui
```

```
drush en -y jsonapi_extras openapi_jsonapi openapi_ui_redoc 
drush cr
```

### 自动生成 API 文档

*   来到`/admin/config/services/openapi`页面。

![](/img/user/Z-attach/172b58fd3fa6cb54~tplv-t2oaga2asx-jj-mark!3024!0!0!0!q75.png)

*   点击`View/Download`按钮，可以查看和下载自动生成的 API 文档。

    

*   点击`Explore with ReDoc`按钮，可以看到以 ReDoc 方式显示的在线 API 文档，看起来更加美观。

    

    ![](/img/user/Z-attach/172b58fd40dadb0e~tplv-t2oaga2asx-jj-mark!3024!0!0!0!q75.png)

*   需要自定义 API，通过`/admin/config/services/jsonapi/extras`页面来设置。需要进行资源覆写点击`Resource overrides`按钮。

例如，覆写 article 资源的 API：

1. 在过滤框中输入 article。

![](/img/user/Z-attach/172b58fd4100da97~tplv-t2oaga2asx-jj-mark!3024!0!0!0!q75.png)

2. 点击 “覆写” 按钮。

![](/img/user/Z-attach/172b58fd412bcce2~tplv-t2oaga2asx-jj-mark!3024!0!0!0!q75.png)

*   选择禁用资源，则相关 article 的资源 API 不可以访问。
*   可以自定义资源路径。如，将`node/artcle`改为`article`。 ![](/img/user/Z-attach/172b58fd447e35c6~tplv-t2oaga2asx-jj-mark!3024!0!0!0!q75.png)

![](/img/user/Z-attach/172b58fd4572c84d~tplv-t2oaga2asx-jj-mark!3024!0!0!0!q75.png)

*   可以禁用不需要 API 返回的资源字段，只需在字段左边勾选。

![](/img/user/Z-attach/172b58fd790ff9ac~tplv-t2oaga2asx-jj-mark!3024!0!0!0!q75.png)

3. 保存。回到 API 文档查看页面`/admin/config/services/openapi`，点击`Explore with ReDoc`按钮，找到`article`资源。

![](/img/user/Z-attach/172b58fd7aa00930~tplv-t2oaga2asx-jj-mark!3024!0!0!0!q75.png)