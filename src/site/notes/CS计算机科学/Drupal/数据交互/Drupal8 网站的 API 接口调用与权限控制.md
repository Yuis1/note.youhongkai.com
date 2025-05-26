---
{"dg-publish":true,"permalink":"/CS计算机科学/Drupal/数据交互/Drupal8 网站的 API 接口调用与权限控制/","noteIcon":"","created":"2024-06-22T22:30:28.603+08:00","updated":"2024-04-27T01:26:38.000+08:00"}
---

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6844904190657953800)

概述
--

用 Drupal8 作为后端数据管理平台，前端通过 API 调用数据接口。实现 Drupal8 的数据解耦，与小程序和移动 APP 进行无缝衔接。

![](/img/user/Z-attach/172b0991d0bbd3f9~tplv-t2oaga2asx-jj-mark!3024!0!0!0!q75.png)

### API 规范：REST 与 JSON：API 与 GraphQL

有一个比较强大的最常见的技术规范的 REST，对于 JSON：API 来说，结果是非常肯定的。

![](/img/user/Z-attach/172b0991d22192bd~tplv-t2oaga2asx-jj-mark!3024!0!0!0!q75.png)

以上信息可帮助我们选择 JSON：API 作为具有有用文档的基本规范。

### 身份验证方法：基本身份验证与 OAuth

> 除非与某些外部安全系统（例如 TLS，Transport Layer Security，[RFC5246]）结合使用，否则此方案不被视为用户身份验证的安全方法，因为用户 ID 和密码以明文形式通过网络传递。

> [“基本” HTTP 身份验证方案文档](https://link.juejin.cn?target=https%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc7617%23page-3 "https://tools.ietf.org/html/rfc7617#page-3")

> 在 OAuth 中，客户端请求访问由资源所有者控制并由资源服务器托管的资源，并向客户端颁发与资源所有者的凭据不同的一组凭据。 客户端无需使用资源所有者的凭据来访问受保护的资源，而是获取访问令牌 - 表示特定范围，生存期和其他访问属性的字符串。

> [OAuth 2.0 授权框架文档](https://link.juejin.cn?target=https%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc6749%23page-5 "https://tools.ietf.org/html/rfc6749#page-5")

根据这些方法的官方规范，我们可以看到有关它们在工作中使用的一些结论。因此，明确选择支持 OAuth！但是我们可以快速设置并使用它吗？是的，`Simple OAuth`模块将为我们提供帮助。

安装 Drush
--------

```
composer require drush/drush
```
配置 drush 命令目录到 $PATH 中。
安装 devel 模块生成测试文章内容
-------------------

```
composer require drupal/devel
```

*   启动内容生成器模块, 勾选后保存。

    

    ![](/img/user/Z-attach/172b0991d6517f91~tplv-t2oaga2asx-jj-mark!3024!0!0!0!q75.png)

    

*   生成测试内容。

    

使用 simple oauth([网络] 开放授权) 模块控制权限
---------------------------------

*   安装扩展模块

```
composer require drupal/simple_oauth
```

*   启用模块

```
drush en -y simple_oauth
```

在使用它之前，我们必须限制默认 _Authenticated user_ 角色的权限。但是，最好在 “角色” 页面（_/admin/people/roles_）上创建一个新角色以与 API 一起使用。

### 生成密钥

*   来到`/admin/config/people/simple_oauth`页面设置。

    

*   点击右下角 “生成密钥” 按钮。

    

*   填入密钥存放路径。（建议放在网站目录之外）

    

    ![](/img/user/Z-attach/172b0991d3d2492b~tplv-t2oaga2asx-jj-mark!3024!0!0!0!q75.png)

    

*   点击 “生成” 按钮。

    

    ![](/img/user/Z-attach/172b0991d13d3bb7~tplv-t2oaga2asx-jj-mark!3024!0!0!0!q75.png)

    

*   点击 “保存配置” 按钮。

    

### 添加访问角色、用户和权限

*   添加角色。如：oauthapi。

    

    ![](/img/user/Z-attach/172b0991ee563d7d~tplv-t2oaga2asx-jj-mark!3024!0!0!0!q75.png)

    

*   添加一个用户，并赋予 api 角色。保存。

    

    ![](/img/user/Z-attach/172b0992007e5363~tplv-t2oaga2asx-jj-mark!3024!0!0!0!q75.png)

    

*   给 api 角色赋予相应的访问权限。如：创建文章、查看已发布内容等等。

    

    ![](/img/user/Z-attach/172b099202c4cf44~tplv-t2oaga2asx-jj-mark!3024!0!0!0!q75.png)

    

创建客户端服务
-------

*   来到`/admin/config/services/consumer`页面客户列表。

    

    ![](/img/user/Z-attach/172b099203d7aa96~tplv-t2oaga2asx-jj-mark!3024!0!0!0!q75.png)

    

*   点击 “添加客户” 按钮。

    

*   填写标签名、设置新密码、勾选 scopes。保存。

    

    ![](/img/user/Z-attach/172b099203f89397~tplv-t2oaga2asx-jj-mark!3024!0!0!0!q75.png)

    

用 postman 测试接口
--------------

*   创建和验证访问令牌

现在，我们已经配置了简单 OAuth 模块，我们需要从位于的资源请求访问令牌 / oauth/token。OAuth 2.0 规范规定，授权服务器上的 OAuth 令牌资源必须仅接受 POST 其主体格式为 form-data 或的请求 x-www-form-urlencoded，并且所有 JSON 格式的主体都将被拒绝。

*   您需要在请求正文中出现以下参数。

grant_type：大多数情况下，password。 client_id：上一部分中使用者的 UUID。 client_secret：添加使用者时提供的客户机密。 username：与使用者相关联的帐户的用户名。 password：与使用者相关联的帐户的密码。 scope: 指定的访问者角色。

![](/img/user/Z-attach/172b099204102065~tplv-t2oaga2asx-jj-mark!3024!0!0!0!q75.png)

*   获取 token。

```
curl --location --request POST 'http://d890.com/oauth/token' \
--form 'grant_type=password' \
--form 'client_id=93d8965b-d8df-4ef0-b887-97a675f968f1' \
--form 'client_secret=abc123' \
--form 'username=test' \
--form 'password=1' \
--form 'scope=oauthapi'
```

*   返回 token。

```
{
    "token_type": "Bearer",
    "expires_in": 869999,
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImp0aSI6IjE5MzdmMWM1YTU2OWJkZDIxZmJhMDJlYjhiOWQyYTFlMWY5ZGEzNzEwOGFmNTQ4YzNjYTg1ZDY5YjVkNGJlMmI0NGIzYzZjMTdlNGM0Mzk5In0.eyJhdWQiOiI5M2Q4OTY1Yi1kOGRmLTRlZjAtYjg4Ny05N2E2NzVmOTY4ZjEiLCJqdGkiOiIxOTM3ZjFjNWE1NjliZGQyMWZiYTAyZWI4YjlkMmExZTFmOWRhMzcxMDhhZjU0OGMzY2E4NWQ2OWI1ZDRiZTJiNDRiM2M2YzE3ZTRjNDM5OSIsImlhdCI6MTU5MjA5NDUzMiwibmJmIjoxNTkyMDk0NTMyLCJleHAiOjE1OTI5NjQ1MzEsInN1YiI6IjUiLCJzY29wZXMiOlsib2F1dGhhcGkiLCJhdXRoZW50aWNhdGVkIl19.J29rBtkl4pwde6b3Xf3bYiyv_w2pwMEQdtDI1YztqNZFm-rTvLFu_MkV3CDsfOi9YHp13d1RhMlz7JbYXylTlaPET9U6IjAI1SeEFV_ZqKxHC1z9LTXR-bxgzGToi7QqoVKa6ZiFPJgGdfhFsPxi6KD-zK1RUPmW5ueGpHM6U9JMPuh-ZL-mO69If36HJOhBR5eld0VDX_SMBq_4a-WJSOQmrzFxICo6Tqh34xLmNeSk3eiCE5hUij_MjcFHJ3jWnCNRh-8U5CaT5O_f5JB_PPu_sHww5nU8gaEXvHpwTLFT34T8ZobmaZD1WnFV836-eE238PNWUPELWCD4560Dz56sNlxRZg2BWAYLtrlKwoGBsgBGT_TSgYW9s2nfMW1_bAjuCgG80uBg5ZBYO-1snK7YxhS9oVERUXksZElSV09upyggQOWrKAxG__jEccejgc1j3muV4m5NNv_Z59qFgzuTBUKJg7aE7ZK5vrydvbgzEaL-X49H1lWyfPy5-HmsOVfRFafQIehvxBvMRQlXkJGIeRn2EVeQhjJx8sqe_PJw-GUe6ADPiC68KO6qPjVAfJZs5BMWjuSa8N_CN9Uokt7dZlgPHL5lG7ZCkeolQ5LKiWsphFHxsLjGooDREMOvWxlfPY8Q3Q4W9rQ-5QcGT7JQJBIxbZy_PGctJkCdwbU",
    "refresh_token": "def502007daab048cb4ff8bb3dc896fa1524587d23782f5a79593f82c2483f1e80bbaa3b0afcce9c2aff190c4614eaef08de601eddd429a94ae4169194cbf9e89a6680cee74f87343a9b20e96d7c2d149f7cf0fd518043554a3115aeceef957723c3143ca371f40734ed2a178a5b7958f39f3e16d4c02628bfb2ef425f123d8afec5ebd34226b2bd9553b20cb8213b82ae382351eef3a5c372a273905791b5b9b348b8f8a2630408b9d000f6721a56c350d87b32ebb582fc7efdbc72a6f1e4a85c2d8ff8c29bd06d6f5b635f48d1614b7c1caf65f04071261a9db20e1310f2844935f32f605218beab57542c18e5906a1afe57c43f1061274abb807bc29664239628e949d8d3b35da7dba7cba042ecc92b968b7614cf2d76496c8df1951d7899d9fb7fc6eab51354ed7e3bb725da7e5c537e28c95218b4d32daf4412acdc57b70cf07ba1ce2008804bd5c7b5ae8cefb94a79f153cb5113c3408584bfd72eba16f73d19ded9113d52061e9869cb4a23cb31cf6de41511e510e9dc40f1a1c82db0918ff661f05ca47dc031e16f34ee483af9e808fe755c4095d19be3cab2ab62b4"
}
```

*   发送请求。
    *   在权限中设置了未登录用户不能查看已发布内容。
    *   如果请求中不带 token，网站首页会拒绝访问。
    *   携带刚刚获取的 token，可以正常获取网站内容。

```
curl --location --request GET 'http://d890.com' \
--header 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImp0aSI6IjE5MzdmMWM1YTU2OWJkZDIxZmJhMDJlYjhiOWQyYTFlMWY5ZGEzNzEwOGFmNTQ4YzNjYTg1ZDY5YjVkNGJlMmI0NGIzYzZjMTdlNGM0Mzk5In0.eyJhdWQiOiI5M2Q4OTY1Yi1kOGRmLTRlZjAtYjg4Ny05N2E2NzVmOTY4ZjEiLCJqdGkiOiIxOTM3ZjFjNWE1NjliZGQyMWZiYTAyZWI4YjlkMmExZTFmOWRhMzcxMDhhZjU0OGMzY2E4NWQ2OWI1ZDRiZTJiNDRiM2M2YzE3ZTRjNDM5OSIsImlhdCI6MTU5MjA5NDUzMiwibmJmIjoxNTkyMDk0NTMyLCJleHAiOjE1OTI5NjQ1MzEsInN1YiI6IjUiLCJzY29wZXMiOlsib2F1dGhhcGkiLCJhdXRoZW50aWNhdGVkIl19.J29rBtkl4pwde6b3Xf3bYiyv_w2pwMEQdtDI1YztqNZFm-rTvLFu_MkV3CDsfOi9YHp13d1RhMlz7JbYXylTlaPET9U6IjAI1SeEFV_ZqKxHC1z9LTXR-bxgzGToi7QqoVKa6ZiFPJgGdfhFsPxi6KD-zK1RUPmW5ueGpHM6U9JMPuh-ZL-mO69If36HJOhBR5eld0VDX_SMBq_4a-WJSOQmrzFxICo6Tqh34xLmNeSk3eiCE5hUij_MjcFHJ3jWnCNRh-8U5CaT5O_f5JB_PPu_sHww5nU8gaEXvHpwTLFT34T8ZobmaZD1WnFV836-eE238PNWUPELWCD4560Dz56sNlxRZg2BWAYLtrlKwoGBsgBGT_TSgYW9s2nfMW1_bAjuCgG80uBg5ZBYO-1snK7YxhS9oVERUXksZElSV09upyggQOWrKAxG__jEccejgc1j3muV4m5NNv_Z59qFgzuTBUKJg7aE7ZK5vrydvbgzEaL-X49H1lWyfPy5-HmsOVfRFafQIehvxBvMRQlXkJGIeRn2EVeQhjJx8sqe_PJw-GUe6ADPiC68KO6qPjVAfJZs5BMWjuSa8N_CN9Uokt7dZlgPHL5lG7ZCkeolQ5LKiWsphFHxsLjGooDREMOvWxlfPY8Q3Q4W9rQ-5QcGT7JQJBIxbZy_PGctJkCdwbU'
```

使用 jsonapi 模块创建 api。
--------------------

### 安装 jsonapi 扩展模块

```
composer require drupal/jsonapi_extras
```

### 启用模块

```
drush en -y jsonapi jsonapi_extras
```

### 设置 jsonapi

JSON：API 是一个零配置模块，该模块提供对所有 Drupal 实体的访问权限，以进行即用型读取。但是，您不仅应考虑读取数据，还应考虑创建或更新数据的可能性。 启用后，您需要转到模块设置页面（_/admin/config/services/jsonapi_），并将 “允许的操作” 字段更改为 “ 接受所有 JSON：API 创建，读取，更新和删除操作 ” 值。

![](/img/user/Z-attach/172b09921cf0db1c~tplv-t2oaga2asx-jj-mark!3024!0!0!0!q75.png)

保存配置。

### 自定义 API

另外，出于安全原因，必须自定义 API，您可以使用 JSON：API Extras 模块来进行此操作。该模块已经通过 composer 安装，应在 “扩展” 页面（_/admin/modules_）上启用。

转到设置页面（_/admin/config/services/jsonapi/extras_），选中 “在集合查询中包括计数” 选项，然后将我们的 API 的 “ 路径前缀” 从 */jsonapi _更改为_ /api/json*。

稍后，您还可以在设置页面的 “资源替代” 标签上禁用无法通过 API 使用的额外资源。

![](/img/user/Z-attach/172b099226ce5dff~tplv-t2oaga2asx-jj-mark!3024!0!0!0!q75.png)

请记住清除 Drupal 缓存（_/admin(管理员)/config/development/performance_）以应用路由更改。

### 用 postman 进行接口测试

首先获取 token。必须携带 token 才能成功请求数据。

*   创建第一篇文章

```
curl --location --request POST 'http://d890.com/api/json/node/article' \
--header 'Accept: application/vnd.api+json' \
--header 'Content-Type: application/vnd.api+json' \
--header 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImp0aSI6IjE5MzdmMWM1YTU2OWJkZDIxZmJhMDJlYjhiOWQyYTFlMWY5ZGEzNzEwOGFmNTQ4YzNjYTg1ZDY5YjVkNGJlMmI0NGIzYzZjMTdlNGM0Mzk5In0.eyJhdWQiOiI5M2Q4OTY1Yi1kOGRmLTRlZjAtYjg4Ny05N2E2NzVmOTY4ZjEiLCJqdGkiOiIxOTM3ZjFjNWE1NjliZGQyMWZiYTAyZWI4YjlkMmExZTFmOWRhMzcxMDhhZjU0OGMzY2E4NWQ2OWI1ZDRiZTJiNDRiM2M2YzE3ZTRjNDM5OSIsImlhdCI6MTU5MjA5NDUzMiwibmJmIjoxNTkyMDk0NTMyLCJleHAiOjE1OTI5NjQ1MzEsInN1YiI6IjUiLCJzY29wZXMiOlsib2F1dGhhcGkiLCJhdXRoZW50aWNhdGVkIl19.J29rBtkl4pwde6b3Xf3bYiyv_w2pwMEQdtDI1YztqNZFm-rTvLFu_MkV3CDsfOi9YHp13d1RhMlz7JbYXylTlaPET9U6IjAI1SeEFV_ZqKxHC1z9LTXR-bxgzGToi7QqoVKa6ZiFPJgGdfhFsPxi6KD-zK1RUPmW5ueGpHM6U9JMPuh-ZL-mO69If36HJOhBR5eld0VDX_SMBq_4a-WJSOQmrzFxICo6Tqh34xLmNeSk3eiCE5hUij_MjcFHJ3jWnCNRh-8U5CaT5O_f5JB_PPu_sHww5nU8gaEXvHpwTLFT34T8ZobmaZD1WnFV836-eE238PNWUPELWCD4560Dz56sNlxRZg2BWAYLtrlKwoGBsgBGT_TSgYW9s2nfMW1_bAjuCgG80uBg5ZBYO-1snK7YxhS9oVERUXksZElSV09upyggQOWrKAxG__jEccejgc1j3muV4m5NNv_Z59qFgzuTBUKJg7aE7ZK5vrydvbgzEaL-X49H1lWyfPy5-HmsOVfRFafQIehvxBvMRQlXkJGIeRn2EVeQhjJx8sqe_PJw-GUe6ADPiC68KO6qPjVAfJZs5BMWjuSa8N_CN9Uokt7dZlgPHL5lG7ZCkeolQ5LKiWsphFHxsLjGooDREMOvWxlfPY8Q3Q4W9rQ-5QcGT7JQJBIxbZy_PGctJkCdwbU' \
--data-raw '{
  "data": {
    "type": "node--article",
    "attributes": {
      "title": "我的第一篇文章",
      "body": {
        "value": "这是我用jsonapi创建的第一篇文章",
        "format": "plain_text"
      }
    }
  }
}'
```

刷新网站首页看到我们新增的文章。

![](/img/user/Z-attach/172b099227960d90~tplv-t2oaga2asx-jj-mark!3024!0!0!0!q75.png)

*   获取文章列表

```
curl --location --request GET 'http://d890.com/api/json/node/article' \
--header 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImp0aSI6IjE5MzdmMWM1YTU2OWJkZDIxZmJhMDJlYjhiOWQyYTFlMWY5ZGEzNzEwOGFmNTQ4YzNjYTg1ZDY5YjVkNGJlMmI0NGIzYzZjMTdlNGM0Mzk5In0.eyJhdWQiOiI5M2Q4OTY1Yi1kOGRmLTRlZjAtYjg4Ny05N2E2NzVmOTY4ZjEiLCJqdGkiOiIxOTM3ZjFjNWE1NjliZGQyMWZiYTAyZWI4YjlkMmExZTFmOWRhMzcxMDhhZjU0OGMzY2E4NWQ2OWI1ZDRiZTJiNDRiM2M2YzE3ZTRjNDM5OSIsImlhdCI6MTU5MjA5NDUzMiwibmJmIjoxNTkyMDk0NTMyLCJleHAiOjE1OTI5NjQ1MzEsInN1YiI6IjUiLCJzY29wZXMiOlsib2F1dGhhcGkiLCJhdXRoZW50aWNhdGVkIl19.J29rBtkl4pwde6b3Xf3bYiyv_w2pwMEQdtDI1YztqNZFm-rTvLFu_MkV3CDsfOi9YHp13d1RhMlz7JbYXylTlaPET9U6IjAI1SeEFV_ZqKxHC1z9LTXR-bxgzGToi7QqoVKa6ZiFPJgGdfhFsPxi6KD-zK1RUPmW5ueGpHM6U9JMPuh-ZL-mO69If36HJOhBR5eld0VDX_SMBq_4a-WJSOQmrzFxICo6Tqh34xLmNeSk3eiCE5hUij_MjcFHJ3jWnCNRh-8U5CaT5O_f5JB_PPu_sHww5nU8gaEXvHpwTLFT34T8ZobmaZD1WnFV836-eE238PNWUPELWCD4560Dz56sNlxRZg2BWAYLtrlKwoGBsgBGT_TSgYW9s2nfMW1_bAjuCgG80uBg5ZBYO-1snK7YxhS9oVERUXksZElSV09upyggQOWrKAxG__jEccejgc1j3muV4m5NNv_Z59qFgzuTBUKJg7aE7ZK5vrydvbgzEaL-X49H1lWyfPy5-HmsOVfRFafQIehvxBvMRQlXkJGIeRn2EVeQhjJx8sqe_PJw-GUe6ADPiC68KO6qPjVAfJZs5BMWjuSa8N_CN9Uokt7dZlgPHL5lG7ZCkeolQ5LKiWsphFHxsLjGooDREMOvWxlfPY8Q3Q4W9rQ-5QcGT7JQJBIxbZy_PGctJkCdwbU'
```

![](/img/user/Z-attach/172b09922a0d9c7d~tplv-t2oaga2asx-jj-mark!3024!0!0!0!q75.png)

*   获取指定文章

```
curl --location --request GET 'http://d890.com/api/json/node/article/0de2ce4b-1f3b-4147-8315-6f15d0314b00' \
--header 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImp0aSI6IjE5MzdmMWM1YTU2OWJkZDIxZmJhMDJlYjhiOWQyYTFlMWY5ZGEzNzEwOGFmNTQ4YzNjYTg1ZDY5YjVkNGJlMmI0NGIzYzZjMTdlNGM0Mzk5In0.eyJhdWQiOiI5M2Q4OTY1Yi1kOGRmLTRlZjAtYjg4Ny05N2E2NzVmOTY4ZjEiLCJqdGkiOiIxOTM3ZjFjNWE1NjliZGQyMWZiYTAyZWI4YjlkMmExZTFmOWRhMzcxMDhhZjU0OGMzY2E4NWQ2OWI1ZDRiZTJiNDRiM2M2YzE3ZTRjNDM5OSIsImlhdCI6MTU5MjA5NDUzMiwibmJmIjoxNTkyMDk0NTMyLCJleHAiOjE1OTI5NjQ1MzEsInN1YiI6IjUiLCJzY29wZXMiOlsib2F1dGhhcGkiLCJhdXRoZW50aWNhdGVkIl19.J29rBtkl4pwde6b3Xf3bYiyv_w2pwMEQdtDI1YztqNZFm-rTvLFu_MkV3CDsfOi9YHp13d1RhMlz7JbYXylTlaPET9U6IjAI1SeEFV_ZqKxHC1z9LTXR-bxgzGToi7QqoVKa6ZiFPJgGdfhFsPxi6KD-zK1RUPmW5ueGpHM6U9JMPuh-ZL-mO69If36HJOhBR5eld0VDX_SMBq_4a-WJSOQmrzFxICo6Tqh34xLmNeSk3eiCE5hUij_MjcFHJ3jWnCNRh-8U5CaT5O_f5JB_PPu_sHww5nU8gaEXvHpwTLFT34T8ZobmaZD1WnFV836-eE238PNWUPELWCD4560Dz56sNlxRZg2BWAYLtrlKwoGBsgBGT_TSgYW9s2nfMW1_bAjuCgG80uBg5ZBYO-1snK7YxhS9oVERUXksZElSV09upyggQOWrKAxG__jEccejgc1j3muV4m5NNv_Z59qFgzuTBUKJg7aE7ZK5vrydvbgzEaL-X49H1lWyfPy5-HmsOVfRFafQIehvxBvMRQlXkJGIeRn2EVeQhjJx8sqe_PJw-GUe6ADPiC68KO6qPjVAfJZs5BMWjuSa8N_CN9Uokt7dZlgPHL5lG7ZCkeolQ5LKiWsphFHxsLjGooDREMOvWxlfPY8Q3Q4W9rQ-5QcGT7JQJBIxbZy_PGctJkCdwbU'
```

![](/img/user/Z-attach/172b09922a17b79e~tplv-t2oaga2asx-jj-mark!3024!0!0!0!q75.png)

*   修改文章内容

```
curl --location --request PATCH 'http://d890.com/api/json/node/article/1bbfa4af-c902-4253-8b4f-fdfb7b4e6d1f' \
--header 'Accept: application/vnd.api+json' \
--header 'Content-Type: application/vnd.api+json' \
--header 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImp0aSI6IjE5MzdmMWM1YTU2OWJkZDIxZmJhMDJlYjhiOWQyYTFlMWY5ZGEzNzEwOGFmNTQ4YzNjYTg1ZDY5YjVkNGJlMmI0NGIzYzZjMTdlNGM0Mzk5In0.eyJhdWQiOiI5M2Q4OTY1Yi1kOGRmLTRlZjAtYjg4Ny05N2E2NzVmOTY4ZjEiLCJqdGkiOiIxOTM3ZjFjNWE1NjliZGQyMWZiYTAyZWI4YjlkMmExZTFmOWRhMzcxMDhhZjU0OGMzY2E4NWQ2OWI1ZDRiZTJiNDRiM2M2YzE3ZTRjNDM5OSIsImlhdCI6MTU5MjA5NDUzMiwibmJmIjoxNTkyMDk0NTMyLCJleHAiOjE1OTI5NjQ1MzEsInN1YiI6IjUiLCJzY29wZXMiOlsib2F1dGhhcGkiLCJhdXRoZW50aWNhdGVkIl19.J29rBtkl4pwde6b3Xf3bYiyv_w2pwMEQdtDI1YztqNZFm-rTvLFu_MkV3CDsfOi9YHp13d1RhMlz7JbYXylTlaPET9U6IjAI1SeEFV_ZqKxHC1z9LTXR-bxgzGToi7QqoVKa6ZiFPJgGdfhFsPxi6KD-zK1RUPmW5ueGpHM6U9JMPuh-ZL-mO69If36HJOhBR5eld0VDX_SMBq_4a-WJSOQmrzFxICo6Tqh34xLmNeSk3eiCE5hUij_MjcFHJ3jWnCNRh-8U5CaT5O_f5JB_PPu_sHww5nU8gaEXvHpwTLFT34T8ZobmaZD1WnFV836-eE238PNWUPELWCD4560Dz56sNlxRZg2BWAYLtrlKwoGBsgBGT_TSgYW9s2nfMW1_bAjuCgG80uBg5ZBYO-1snK7YxhS9oVERUXksZElSV09upyggQOWrKAxG__jEccejgc1j3muV4m5NNv_Z59qFgzuTBUKJg7aE7ZK5vrydvbgzEaL-X49H1lWyfPy5-HmsOVfRFafQIehvxBvMRQlXkJGIeRn2EVeQhjJx8sqe_PJw-GUe6ADPiC68KO6qPjVAfJZs5BMWjuSa8N_CN9Uokt7dZlgPHL5lG7ZCkeolQ5LKiWsphFHxsLjGooDREMOvWxlfPY8Q3Q4W9rQ-5QcGT7JQJBIxbZy_PGctJkCdwbU' \
--data-raw '{
  "data": {
    "type": "node--article",
    "id": "1bbfa4af-c902-4253-8b4f-fdfb7b4e6d1f",
    "attributes": {
      "title": "修改我的第一篇文章"
    }
  }
}'
```

刷新网站首页看到我们修改成功的文章。

![](/img/user/Z-attach/172b09922a37b19a~tplv-t2oaga2asx-jj-mark!3024!0!0!0!q75.png)

*   删除一篇文章

```
curl --location --request DELETE 'http://d890.com/api/json/node/article/1bbfa4af-c902-4253-8b4f-fdfb7b4e6d1f' \
--header 'Content-Type: application/vnd.api+json' \
--header 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImp0aSI6IjE5MzdmMWM1YTU2OWJkZDIxZmJhMDJlYjhiOWQyYTFlMWY5ZGEzNzEwOGFmNTQ4YzNjYTg1ZDY5YjVkNGJlMmI0NGIzYzZjMTdlNGM0Mzk5In0.eyJhdWQiOiI5M2Q4OTY1Yi1kOGRmLTRlZjAtYjg4Ny05N2E2NzVmOTY4ZjEiLCJqdGkiOiIxOTM3ZjFjNWE1NjliZGQyMWZiYTAyZWI4YjlkMmExZTFmOWRhMzcxMDhhZjU0OGMzY2E4NWQ2OWI1ZDRiZTJiNDRiM2M2YzE3ZTRjNDM5OSIsImlhdCI6MTU5MjA5NDUzMiwibmJmIjoxNTkyMDk0NTMyLCJleHAiOjE1OTI5NjQ1MzEsInN1YiI6IjUiLCJzY29wZXMiOlsib2F1dGhhcGkiLCJhdXRoZW50aWNhdGVkIl19.J29rBtkl4pwde6b3Xf3bYiyv_w2pwMEQdtDI1YztqNZFm-rTvLFu_MkV3CDsfOi9YHp13d1RhMlz7JbYXylTlaPET9U6IjAI1SeEFV_ZqKxHC1z9LTXR-bxgzGToi7QqoVKa6ZiFPJgGdfhFsPxi6KD-zK1RUPmW5ueGpHM6U9JMPuh-ZL-mO69If36HJOhBR5eld0VDX_SMBq_4a-WJSOQmrzFxICo6Tqh34xLmNeSk3eiCE5hUij_MjcFHJ3jWnCNRh-8U5CaT5O_f5JB_PPu_sHww5nU8gaEXvHpwTLFT34T8ZobmaZD1WnFV836-eE238PNWUPELWCD4560Dz56sNlxRZg2BWAYLtrlKwoGBsgBGT_TSgYW9s2nfMW1_bAjuCgG80uBg5ZBYO-1snK7YxhS9oVERUXksZElSV09upyggQOWrKAxG__jEccejgc1j3muV4m5NNv_Z59qFgzuTBUKJg7aE7ZK5vrydvbgzEaL-X49H1lWyfPy5-HmsOVfRFafQIehvxBvMRQlXkJGIeRn2EVeQhjJx8sqe_PJw-GUe6ADPiC68KO6qPjVAfJZs5BMWjuSa8N_CN9Uokt7dZlgPHL5lG7ZCkeolQ5LKiWsphFHxsLjGooDREMOvWxlfPY8Q3Q4W9rQ-5QcGT7JQJBIxbZy_PGctJkCdwbU' \
--data-raw ''
```

回到首页刷新，看到我们的刚刚新增的文章已经被删除。

> 有关 jsonapi 的进一步使用，包括过滤、关联、筛选等操作参看[官方文档](https://link.juejin.cn?target=https%3A%2F%2Fwww.drupal.org%2Fdocs%2Fcore-modules-and-themes%2Fcore-modules%2Fjsonapi-module "https://www.drupal.org/docs/core-modules-and-themes/core-modules/jsonapi-module")。

参考文献：

*   [Decoupled(解耦) Drupal Authentication with OAuth([网络] 开放授权) 2.0](https://link.juejin.cn?target=https%3A%2F%2Fdev.acquia.com%2Fblog%2Fdecoupled-drupal-authentication-oauth-20 "https://dev.acquia.com/blog/decoupled-drupal-authentication-oauth-20")
*   [How to quickly configure Drupal as a decoupled(解耦) API-first system](https://link.juejin.cn?target=https%3A%2F%2Fthebrainfiles.wearebrain.com%2Fhow-to-quickly-configure-drupal-as-a-decoupled-api-first-system-8730a3623388 "https://thebrainfiles.wearebrain.com/how-to-quickly-configure-drupal-as-a-decoupled-api-first-system-8730a3623388")