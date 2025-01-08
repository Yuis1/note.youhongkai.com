---
{"dg-publish":true,"permalink":"/CS计算机科学/Drupal/数据交互/Drupal 对外提供API/","tags":["Drupal","API"],"noteIcon":"","created":"2024-03-19T15:09:10.702+08:00","updated":"2024-05-07T15:10:07.110+08:00"}
---


作者：游鱼思

感悟：编程找Bug就像做侦探，思维得严丝合缝，不能放过一点蛛丝马迹。

---
## 重要参考

[JSON:API — A specification for building APIs in JSON (jsonapi.org)](https://jsonapi.org/)

视频教程：[JSON API Drupal - YouTube](https://www.youtube.com/playlist?list=PLZOQ_ZMpYrZsyO-3IstImK1okrpfAjuMZ)

演示了通过Restful进行 注册、登录等流程：[What JSON:API DOESN'T do | JSON:API module | Drupal Wiki guide on Drupal.org](https://www.drupal.org/docs/core-modules-and-themes/core-modules/jsonapi-module/what-jsonapi-doesnt-do)

[Register a User with Drupal 8 REST API | AreaType LLC](https://areatype.com/blog/register-user-drupal-8-rest-api)

## Drupal API 相关模块

[JSON:API module | Core modules | Drupal Wiki guide on Drupal.org](https://www.drupal.org/docs/core-modules-and-themes/core-modules/jsonapi-module)

[JSON API模块 与核心的 REST 模块](https://www.drupal.org/docs/core-modules-and-themes/core-modules/jsonapi-module/jsonapi-vs-cores-rest-module)

**Json API ：** 不支持注册、登录等流程，只支持增删改查。但是却支持按 Node Type、Entity Type 来查询对应的列表。不使用Drupal的nid，却使用另一套唯一id来查询Node、Entity。

Json API权限：

Json API权限 是跟着Drupal系统走的。Drupal系统的权限有个坑：View unpublished ECK entities ，会查出其它人创建且未发布的Entity。

错误的做法：在权限里勾选 View unpublished ECK entities ，会查出其它人创建且未发布的Entity，引发权限漏洞。

正确的做法：不勾选 View unpublished ECK entities 。此时又无法直接查看自己未发布的 Entity，但是可以通过views检索出条目，以及可以编辑详情。

[RESTful Web Services API | Drupal APIs | Drupal Wiki guide on Drupal.org](https://www.drupal.org/docs/drupal-apis/restful-web-services-api)

**Restful API：** 和 Json API 以上几点特性正好相反。

结论：两个都得开启用 😤

###  Drupal REST & JSON API Authentication

收费版本功能还挺多的，免费版好像比较鸡肋

[Drupal REST & JSON API Authentication | Drupal.org](https://www.drupal.org/project/rest_api_authentication)

说明文档：[Drupal API Authentication | Drupal Rest API Authentication (miniorange.com)](https://plugins.miniorange.com/drupal-api-authentication)

## 使用技巧

### 通过Views可以新建 Rest导出 视图

Rest导出 视图 就是 Rest API

这样导出的内容可以做到很简洁，Views查询也可以很灵活

比较遗憾的是不支持模糊匹配搜索（理论上通过覆写Views生成的SQL是可以的，我还没这么试）

### json api 通过include参数可以查询关联实体

通过 include 参数，可以加载关联实体的字段信息，避免了二次查询，或者再去建Views进行关联查询。示例：

```
http(s)://sourcesite.com/jsonapi/node/article?include=field_image,uid.author
```

In JSON API you can follow the related information down through as many levels as necessary.

多个平行的关联关系用 , 分隔，多个嵌套的关联关系用 . 分隔。

## 问题排查
### 通过Restful API注册的用户，默认是阻止状态

通过restful api注册的新用户，默认状态是 阻止，导致了无法通过账户激活链接来激活。

尝试了以下方案都无效：

- format=hal_json、 format=json
- 从/session/token 获取 Session Token，在注册接口header中增加 X-CSRF-Token
- 卸载各插件、清空缓存等

这是drupal的bug：[User created via /user/register?_format=json get blocked [#3055807] | Drupal.org](https://www.drupal.org/project/drupal/issues/3055807)

修复方案：通过Rules规则，让新用户立即启用。

- event: After Saving a new User  
- action: unBlock User ，数据选择器填：user

另外，通过HTTP请求restful api，得到的新用户激活链接也是http。

### 登录表单流程优化模块对API不生效

如 [Mail Login | Drupal.org](https://www.drupal.org/project/mail_login)   [Email Registration | Drupal.org](https://www.drupal.org/project/email_registration)  等，一般没有对Restful API调用进行覆写，对API不生效。

### Nginx 的伪静态规则需要调整

问题：请求API后提示：

```
"title": "Bad Request",
"status": "400",
"detail": "The following query parameters violate the JSON:API spec: 'q'."
```

解决方法：Nginx 的伪静态规则

```nginx
if (!-e $request_filename) {
      rewrite ^/(.*)$ /index.php?q=$1 last;
    }
```

需要调整成：

```nginx
if (!-e $request_filename) {
  rewrite ^/(.*)$ /index.php last;
}
```



## 接口调用示例

以下是配置好的Postman文件，包括了 获取session、注册、登录、登录状态查询、注销、用户详情查询、用户更新、node列表、node详情等一系列接口配置，含要点和文档链接。打开Postman导入即可使用。

```json
{
	"info": {
		"_postman_id": "ccab7818-ac4e-4a44-9cd8-7473e4dc8032",
		"name": "示例网站",
		"description": "## 接口文档\n\nrest：[OpenAPI Documentation | 示例网站 (example.com)](https://example.com/admin/config/services/openapi/swagger/rest)\n\njsonapi：[OpenAPI Documentation | 示例网站 (example.com)](https://example.com/admin/config/services/openapi/swagger/jsonapi)\n\n---\n\n## 进阶说明\n\nJSONAPI 文档： [JSON:API module | Core modules | Drupal Wiki guide on Drupal.org](https://www.drupal.org/docs/core-modules-and-themes/core-modules/jsonapi-module)\n\nRestful API 文档：[RESTful Web Services module | Core modules | Drupal Wiki guide on Drupal.org](https://www.drupal.org/docs/8/core/modules/rest)",
		"schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json",
		"_exporter_id": "13674700"
	},
	"item": [
		{
			"name": "00 获取Session Token",
			"request": {
				"method": "GET",
				"header": [],
				"url": {
					"raw": "https://example.com/session/token",
					"protocol": "https",
					"host": [
						"example",
						"com"
					],
					"path": [
						"session",
						"token"
					]
				},
				"description": "when performing **non-read-only requests**, that token is required.  \nSuch a token can be retrieved at `/session/token`."
			},
			"response": []
		},
		{
			"name": "用户注册_hal_json",
			"protocolProfileBehavior": {
				"disabledSystemHeaders": {}
			},
			"request": {
				"auth": {
					"type": "noauth"
				},
				"method": "POST",
				"header": [
					{
						"key": "Content-Type",
						"value": "application/hal+json",
						"type": "text"
					},
					{
						"key": "Accept",
						"value": "application/hal+json",
						"type": "text"
					},
					{
						"key": "X-CSRF-Token",
						"value": "h1oK8A0dx6LVY6IGiKDSMI-29Y8y2HaZcHgxSaSR_Bg",
						"type": "text"
					}
				],
				"body": {
					"mode": "raw",
					"raw": "{\n    \"_links\": {\n        \"type\": {\n            \"href\": \"https://example.com/rest/type/user/user\"\n        }\n    },\n    \"name\": [\n        {\n            \"value\": \"user\"\n        }\n    ],\n    \"mail\": [\n        {\n            \"value\": \"93054555@qq.com\"\n        }\n    ]\n}"
				},
				"url": {
					"raw": "https://example.com/user/register?_format=hal_json",
					"protocol": "https",
					"host": [
						"example",
						"com"
					],
					"path": [
						"user",
						"register"
					],
					"query": [
						{
							"key": "_format",
							"value": "hal_json"
						}
					]
				},
				"description": "### User registration\n\nIn order to allow users to register an account, the user_registration REST resource must be enabled (see the REST UI example below).\n\n``` php\ncurl \\\n--header \"Content-Type: application/json\" \\\n--header \"X-CSRF-Token: 57sTS-KS7UoYAWAPyzt0iJmo300CFct3jdKyWM-UiiQ\" \\\n--request POST \"https://drupal.d9/user/register?_format=json\" \\\n--data '{\"name\": {\"value\": \"thename123\"}, \"pass\": {\"value\": \"thepass\"}, \"mail\": {\"value\": \"someone@example.com\"}}'\n\n ```\n\nUse the session token value with the X-CSRF-Token header. A successful response should contain some user field values, including the UUID of the newly created user:\n\n``` php\n{\n   \"uuid\" : [ { \"value\" : \"3e75b757-831e-4bf7-bbb6-25b8c50c7ac0\" } ]\n}\n\n ```\n\nAlso note that the response doesn't have \"Set-Cookie\" headers, even if visitors are allowed to create an account by themselves. So, if no approval or confirmation required, you can log a user in after successful registration using same name and pass values."
			},
			"response": []
		},
		{
			"name": "用户注册_json",
			"protocolProfileBehavior": {
				"disabledSystemHeaders": {}
			},
			"request": {
				"auth": {
					"type": "noauth"
				},
				"method": "POST",
				"header": [
					{
						"key": "Content-Type",
						"value": "application/json",
						"type": "text"
					},
					{
						"key": "Accept",
						"value": "application/json",
						"type": "text"
					},
					{
						"key": "X-CSRF-Token",
						"value": "7AHNAUDcTLWlAMZp-W-OOZ-zbuwznKbwzDmQNUy3PGw",
						"type": "text",
						"disabled": true
					}
				],
				"body": {
					"mode": "raw",
					"raw": "{\n  \"name\": [{\"value\": \"user1\"}],\n  \"mail\": [{\"value\": \"y.h.k@live.com\"}]\n}\n"
				},
				"url": {
					"raw": "https://example.com/user/register?_format=json",
					"protocol": "https",
					"host": [
						"example",
						"com"
					],
					"path": [
						"user",
						"register"
					],
					"query": [
						{
							"key": "_format",
							"value": "json"
						}
					]
				},
				"description": "### User registration\n\nIn order to allow users to register an account, the user_registration REST resource must be enabled (see the REST UI example below).\n\n``` php\ncurl \\\n--header \"Content-Type: application/json\" \\\n--header \"X-CSRF-Token: 57sTS-KS7UoYAWAPyzt0iJmo300CFct3jdKyWM-UiiQ\" \\\n--request POST \"https://drupal.d9/user/register?_format=json\" \\\n--data '{\"name\": {\"value\": \"thename123\"}, \"pass\": {\"value\": \"thepass\"}, \"mail\": {\"value\": \"someone@example.com\"}}'\n\n ```\n\nUse the session token value with the X-CSRF-Token header. A successful response should contain some user field values, including the UUID of the newly created user:\n\n``` php\n{\n   \"uuid\" : [ { \"value\" : \"3e75b757-831e-4bf7-bbb6-25b8c50c7ac0\" } ]\n}\n\n ```\n\nAlso note that the response doesn't have \"Set-Cookie\" headers, even if visitors are allowed to create an account by themselves. So, if no approval or confirmation required, you can log a user in after successful registration using same name and pass values."
			},
			"response": []
		},
		{
			"name": "用户登录",
			"protocolProfileBehavior": {
				"disabledSystemHeaders": {}
			},
			"request": {
				"auth": {
					"type": "noauth"
				},
				"method": "POST",
				"header": [
					{
						"key": "Content-Type",
						"value": "application/json",
						"type": "text"
					},
					{
						"key": "Accept",
						"value": "application/json",
						"type": "text",
						"disabled": true
					},
					{
						"key": "X-CSRF-Token",
						"value": "KnNanbA7KEd_hEQ09eiom5tUexxDQSZ2aL5Tv31yNts",
						"type": "text",
						"disabled": true
					}
				],
				"body": {
					"mode": "raw",
					"raw": "{\"name\":\"user1\", \"pass\":\"1\"}"
				},
				"url": {
					"raw": "https://example.com/user/login?_format=json",
					"protocol": "https",
					"host": [
						"example",
						"com"
					],
					"path": [
						"user",
						"login"
					],
					"query": [
						{
							"key": "_format",
							"value": "json"
						}
					]
				},
				"description": "### User login\n\n``` php\ncurl \\\n  --header \"Content-type: application/json\" \\\n  -c cookie.txt \\\n  --request POST \"http://drupal.d8/user/login?_format=json\" \\\n  --data '{\"name\":\"admin\", \"pass\":\"admin\"}'\n\n ```\n\nthe `-c cookie.txt` tells curl to save a cookie. Your response should look something like this:\n\n``` php\n{\n   \"csrf_token\" : \"57sTS-KS7UoYAWAPyzt0iJmo300CFct3jdKyWM-UiiQ\",\n   \"logout_token\" : \"zzRaD8ZgLT1TkG804mYpVVTyM-pgoDm4h9XZ9JHSoCw\",\n   \"current_user\" : {\n      \"roles\" : [\n         \"authenticated\",\n         \"administrator\"\n      ],\n      \"name\" : \"admin\",\n      \"uid\" : \"1\"\n   }\n}\n\n ```"
			},
			"response": []
		},
		{
			"name": "用户注销登录",
			"protocolProfileBehavior": {
				"disabledSystemHeaders": {}
			},
			"request": {
				"auth": {
					"type": "noauth"
				},
				"method": "POST",
				"header": [
					{
						"key": "Content-Type",
						"value": "application/json",
						"type": "text"
					},
					{
						"key": "Accept",
						"value": "application/json",
						"type": "text"
					},
					{
						"key": "csrf_token",
						"value": "48BY_Tahw8K6digDa1pSyfIVzJyLSIJzIgkCCl-aAMg",
						"type": "text",
						"disabled": true
					}
				],
				"url": {
					"raw": "https://example.com/user/logout?_format=json&token=AhwpLwXPe0IXZSqbrK9Cd_XzI5X6UkKPIYk8y5PsjOo",
					"protocol": "https",
					"host": [
						"example",
						"com"
					],
					"path": [
						"user",
						"logout"
					],
					"query": [
						{
							"key": "_format",
							"value": "json"
						},
						{
							"key": "token",
							"value": "AhwpLwXPe0IXZSqbrK9Cd_XzI5X6UkKPIYk8y5PsjOo",
							"description": "登录时给的 \"logout_token\""
						}
					]
				},
				"description": "### User logout\n\n``` php\ncurl \\\n  --header \"Content-type: application/json\" \\\n  -b cookie.txt \\\n  --request POST \"http://drupal.d8/user/logout?_format=json&token=zzRaD8ZgLT1TkG804mYpVVTyM-pgoDm4h9XZ9JHSoCw\"\n\n ```\n\nWill log the user authenticated by `cookie.txt` out. Use the logout_token value with the token query parameter.\n\nHeader 中的token，也即是登录时给的 \"logout_token\"\n\n如果没存 \"logout_token\" ，也可以通过清除cookies来注销。"
			},
			"response": []
		},
		{
			"name": "用户登录状态查询",
			"protocolProfileBehavior": {
				"disableBodyPruning": true,
				"disabledSystemHeaders": {}
			},
			"request": {
				"auth": {
					"type": "noauth"
				},
				"method": "GET",
				"header": [
					{
						"key": "Content-Type",
						"value": "application/json",
						"type": "text"
					},
					{
						"key": "Accept",
						"value": "application/json",
						"type": "text"
					},
					{
						"key": "X-CSRF-Token",
						"value": "7AHNAUDcTLWlAMZp-W-OOZ-zbuwznKbwzDmQNUy3PGw",
						"type": "text"
					}
				],
				"body": {
					"mode": "raw",
					"raw": ""
				},
				"url": {
					"raw": "https://example.com/user/login_status?_format=json",
					"protocol": "https",
					"host": [
						"example",
						"com"
					],
					"path": [
						"user",
						"login_status"
					],
					"query": [
						{
							"key": "_format",
							"value": "json"
						}
					]
				},
				"description": "### User status\n\n``` php\ncurl \\\n  --header \"Content-type: application/json\" \\\n  -b cookie.txt \\\n  --request GET \"http://drupal.d8/user/login_status?_format=json\"\n\n ```\n\n`-b cookie.txt` tells curl to _send_ (not save) the cookie from the last request. If you're logged in, this will return `1` in the response body in plain text format (not JSON), otherwise `0`."
			},
			"response": []
		},
		{
			"name": "用户详情查询",
			"protocolProfileBehavior": {
				"disabledSystemHeaders": {
					"accept": true
				}
			},
			"request": {
				"auth": {
					"type": "noauth"
				},
				"method": "GET",
				"header": [
					{
						"key": "Accept",
						"value": "accept: application/json",
						"type": "text"
					}
				],
				"url": {
					"raw": "https://example.com/user/27?_format=json",
					"protocol": "https",
					"host": [
						"example",
						"com"
					],
					"path": [
						"user",
						"27"
					],
					"query": [
						{
							"key": "_format",
							"value": "json"
						}
					]
				},
				"description": "/user/{user}\n\n{user} 是uid。\n\n认证：basic auth ，提供用户名、密码；或者该用户登录之后的cookies认证。\n\n权限控制：有用户管理权限的角色，可以查询所有用户的详情。普通用户，只能查询自己的用户详情。"
			},
			"response": []
		},
		{
			"name": "用户更新",
			"protocolProfileBehavior": {
				"disabledSystemHeaders": {
					"accept": true
				}
			},
			"request": {
				"auth": {
					"type": "noauth"
				},
				"method": "GET",
				"header": [
					{
						"key": "Accept",
						"value": "accept: application/json",
						"type": "text"
					}
				],
				"url": {
					"raw": "https://example.com/user/27?_format=json",
					"protocol": "https",
					"host": [
						"example",
						"com"
					],
					"path": [
						"user",
						"27"
					],
					"query": [
						{
							"key": "_format",
							"value": "json"
						}
					]
				},
				"description": "/user/{user}?_format=json\n\n{user} 是uid。\n\n认证：basic auth ，提供用户名、密码；或者该用户登录之后的cookies认证。 另外，还需要在 header提供 X-CSRF-Token 。值是登录接口提供的 csrf_token。\n\n权限控制：有用户管理权限的角色，可以更新所有用户的所有字段。普通用户，只能更新自己的用户的部分字段（用户名 不允许修改）。"
			},
			"response": []
		},
		{
			"name": "文章列表",
			"protocolProfileBehavior": {
				"disabledSystemHeaders": {
					"accept": true
				}
			},
			"request": {
				"method": "GET",
				"header": [
					{
						"key": "Accept",
						"value": "application/vnd.api+json",
						"type": "text"
					}
				],
				"url": {
					"raw": "https://example.com/jsonapi/node/article",
					"protocol": "https",
					"host": [
						"example",
						"com"
					],
					"path": [
						"jsonapi",
						"node",
						"article"
					]
				},
				"description": "认证：basic auth ，提供用户名、密码；或者该用户登录之后的cookies认证。\n\n权限控制：只有登录用户才有权限查询文章列表。"
			},
			"response": []
		},
		{
			"name": "文章详情查询",
			"request": {
				"method": "GET",
				"header": [],
				"url": {
					"raw": "http://example.com/node?_format=json",
					"protocol": "http",
					"host": [
						"example",
						"com"
					],
					"path": [
						"node"
					],
					"query": [
						{
							"key": "_format",
							"value": "json"
						}
					]
				},
				"description": "/jsonapi/node/article/{entity}\n\n{entity} 是通过列表查到的内容id\n\n认证：basic auth ，提供用户名、密码；或者该用户登录之后的cookies认证。"
			},
			"response": []
		}
	]
}
```

### Drupal Entity类接口调用

没找到文档，我自己试出来的，以下是Postman配置：

```json
{
	"info": {
		"_postman_id": "5016425d-3dca-4413-a05f-1c2d7258d113",
		"name": "Drupal Entity 调用示例",
		"description": "接口如果调不通，几个原因：\n\n1、权限不够。是否已登录了，或者经过 simple_auth ，或者对应用户组无权限。写类的接口还需要在header中传入 X-CSRF-Token ，从 获取Session Token 接口获得。\n\n2、注意header中对 Accept、Content_type 的要求，目前不统一。\n\n3、路径、id是否正确",
		"schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json",
		"_exporter_id": "13674700"
	},
	"item": [
		{
			"name": "用户-交易所配置列表查询",
			"protocolProfileBehavior": {
				"disabledSystemHeaders": {
					"accept": true
				}
			},
			"request": {
				"auth": {
					"type": "noauth"
				},
				"method": "GET",
				"header": [
					{
						"key": "Accept",
						"value": "application/vnd.api+json",
						"type": "text"
					}
				],
				"url": {
					"raw": "https://example.com/jsonapi/rel_conf/user_ex_conf",
					"protocol": "https",
					"host": [
						"example",
						"top"
					],
					"path": [
						"jsonapi",
						"rel_conf",
						"user_ex_conf"
					]
				},
				"description": "认证：basic auth ，提供用户名、密码；或者该用户登录之后的cookies认证。\n\n权限控制：每个用户可查得自己的交易所配置列表；管理员用户可得查询所有用户交易所配置列表。\n\ndata.id 本配置的id-用于jsonapi查询\n\n\"attributes\": {\n\n\"drupal_internal__id\": 3, 本配置的id-用于url访问\n\n\"langcode\": \"zh-hans\",\n\n\"created\": \"2024-03-21T13:16:23+00:00\", 配置创建时间\n\n\"changed\": \"2024-03-21T13:16:23+00:00\", 配置更新时间\n\n\"status\": true, 配置是否启用\n\n\"field_copy_total_amt\": \"7832.32\", 最大跟单金额（USDT）\n\n\"field_ex_apikey\": \"yuis1 test key\", 交易所 API Key\n\n\"field_ex_pass_phrase\": \"yuis1 test pass phrase\", 交易所 Secret Key\n\n\"field_ex_secretkey\": \"yuis1 test secret key\" 交易所 Pass Phrase\n\nrelationships.uid.data.id 用户的id-用于jsonapi查询\n\nrelationships.uid.data.meta.drupal_internal__target_id 用户的id-用于url访问\n\nrelationships.field_ex.data.id 交易所的id-用于jsonapi查询\n\nrelationships.field_ex.data.meta.drupal_internal__target_id 交易所的id-用于url访问"
			},
			"response": []
		},
		{
			"name": "用户-交易所配置新增",
			"protocolProfileBehavior": {
				"disabledSystemHeaders": {
					"accept": true
				}
			},
			"request": {
				"auth": {
					"type": "noauth"
				},
				"method": "POST",
				"header": [
					{
						"key": "Accept",
						"value": "application/vnd.api+json",
						"type": "text"
					},
					{
						"key": "Content-Type",
						"value": "application/vnd.api+json",
						"type": "text"
					},
					{
						"key": "X-CSRF-Token",
						"value": "x5wwSCIt6nuimAkL64L7a1RA631Iwao0hD-_Ew91l3E",
						"type": "text"
					}
				],
				"body": {
					"mode": "raw",
					"raw": " {\"data\": {\n\t\"type\": \"rel_conf--user_ex_conf\",\n\t\"attributes\": {\n\t\t\"status\": true,\n\t\t\"default_langcode\": true,\n\t\t\"field_copy_total_amt\": \"7832.32\",\n\t\t\"field_ex_apikey\": \"yuis1 test key333\",\n\t\t\"field_ex_pass_phrase\": \"yuis1 test pass phrase333\",\n\t\t\"field_ex_secretkey\": \"yuis1 test secret  key333\"\n\t},\n\t\"relationships\": {\n\t\t\"rel_conf_type\": {\n\t\t\t\"data\": {\n\t\t\t\t\"type\": \"rel_conf_type--rel_conf_type\",\n\t\t\t\t\"id\": \"b53fa5f8-8d15-4da3-83e1-7099813c360c\", //这个id是关联类型id，从entity详情中查得\n\t\t\t\t\"meta\": {\n\t\t\t\t\t\"drupal_internal__target_id\": \"user_ex_conf\"\n\t\t\t\t}\n\t\t\t}\n\t\t},\n\t\t\"uid\": {\n\t\t\t\"data\": {\n\t\t\t\t\"type\": \"user--user\",\n\t\t\t\t\"id\": \"a151b65e-ab8a-45af-888d-820010271317\" //需要配置的用户id\n\t\t\t}\n\t\t},\n\t\t\"field_ex\": {\n\t\t\t\"data\": {\n\t\t\t\t\"type\": \"node--ex\",\n\t\t\t\t\"id\": \"8318eab4-8cc6-4f57-a042-53b341211b44\" //配置的交易所id\n\t\t\t}\n\t\t}\n\t}\n}\n}\n",
					"options": {
						"raw": {
							"language": "json"
						}
					}
				},
				"url": {
					"raw": "https://example.com/jsonapi/rel_conf/user_ex_conf",
					"protocol": "https",
					"host": [
						"example",
						"top"
					],
					"path": [
						"jsonapi",
						"rel_conf",
						"user_ex_conf"
					]
				},
				"description": "认证：basic auth ，提供用户名、密码；或者该用户登录之后的cookies认证。另外，还需要在 header提供 X-CSRF-Token ，值是登录接口提供的 csrf_token。\n\n权限控制：每个用户可新增自己的交易所配置；管理员用户可新增所有用户交易所配置。\n\n注意：由于目前没有对 用户-交易所 进行唯一性约束，如果存在多套相同的 用户-交易所 ，约定以最新的记录为准。尽量只创建并更新唯一的 用户-交易所 。\n\ndata.id 本配置的id-用于jsonapi查询\n\n\"attributes\": {\n\n\"drupal_internal__id\": 3, 本配置的id-用于url访问\n\n\"langcode\": \"zh-hans\",\n\n\"created\": \"2024-03-21T13:16:23+00:00\", 配置创建时间\n\n\"changed\": \"2024-03-21T13:16:23+00:00\", 配置更新时间\n\n\"status\": true, 配置是否启用\n\n\"field_copy_total_amt\": \"7832.32\", 最大跟单金额（USDT）\n\n\"field_ex_apikey\": \"yuis1 test key\", 交易所 API Key\n\n\"field_ex_pass_phrase\": \"yuis1 test pass phrase\", 交易所 Secret Key\n\n\"field_ex_secretkey\": \"yuis1 test secret key\" 交易所 Pass Phrase\n\nrelationships.uid.data.id 用户的id-用于jsonapi查询\n\nrelationships.uid.data.meta.drupal_internal__target_id 用户的id-用于url访问\n\nrelationships.field_ex.data.id 交易所的id-用于jsonapi查询\n\nrelationships.field_ex.data.meta.drupal_internal__target_id 交易所的id-用于url访问"
			},
			"response": []
		},
		{
			"name": "用户-交易所配置更新",
			"protocolProfileBehavior": {
				"disabledSystemHeaders": {
					"accept": true
				}
			},
			"request": {
				"auth": {
					"type": "noauth"
				},
				"method": "PATCH",
				"header": [
					{
						"key": "Accept",
						"value": "application/vnd.api+json",
						"type": "text"
					},
					{
						"key": "Content-Type",
						"value": "application/vnd.api+json",
						"type": "text"
					},
					{
						"key": "X-CSRF-Token",
						"value": "x5wwSCIt6nuimAkL64L7a1RA631Iwao0hD-_Ew91l3E",
						"type": "text"
					}
				],
				"body": {
					"mode": "raw",
					"raw": " {\"data\": {\n\t\"type\": \"rel_conf--user_ex_conf\",\n    \"id\": \"a824a797-5495-4838-a58e-8fb1934e4c1c\", // 这里还需要再传一遍配置id\n\t\"attributes\": {\n\t\t\"status\": true,\n\t\t\"default_langcode\": true,\n\t\t\"field_copy_total_amt\": \"555.55\",\n\t\t\"field_ex_apikey\": \"yuis1 test key222111\",\n\t\t\"field_ex_pass_phrase\": \"yuis1 test pass phrase222111\",\n\t\t\"field_ex_secretkey\": \"yuis1 test secret  key222111\"\n\t},\n\t\"relationships\": {\n\t\t\"rel_conf_type\": {\n\t\t\t\"data\": {\n\t\t\t\t\"type\": \"rel_conf_type--rel_conf_type\",\n\t\t\t\t\"id\": \"b53fa5f8-8d15-4da3-83e1-7099813c360c\", //关联类型id，从entity详情中查得\n\t\t\t\t\"meta\": {\n\t\t\t\t\t\"drupal_internal__target_id\": \"user_ex_conf\"\n\t\t\t\t}\n\t\t\t}\n\t\t},\n\t\t\"uid\": {\n\t\t\t\"data\": {\n\t\t\t\t\"type\": \"user--user\",\n\t\t\t\t\"id\": \"a151b65e-ab8a-45af-888d-820010271317\" //需要配置的用户id\n\t\t\t}\n\t\t},\n\t\t\"field_ex\": {\n\t\t\t\"data\": {\n\t\t\t\t\"type\": \"node--ex\",\n\t\t\t\t\"id\": \"8318eab4-8cc6-4f57-a042-53b341211b44\" //配置的交易所id\n\t\t\t}\n\t\t}\n\t}\n}\n}\n",
					"options": {
						"raw": {
							"language": "json"
						}
					}
				},
				"url": {
					"raw": "https://example.com/jsonapi/rel_conf/user_ex_conf/a824a797-5495-4838-a58e-8fb1934e4c1c",
					"protocol": "https",
					"host": [
						"example",
						"top"
					],
					"path": [
						"jsonapi",
						"rel_conf",
						"user_ex_conf",
						"a824a797-5495-4838-a58e-8fb1934e4c1c"
					]
				},
				"description": "/jsonapi/rel_conf/user_ex_conf/{配置id}\n\n认证：basic auth ，提供用户名、密码；或者该用户登录之后的cookies认证。另外，还需要在 header提供 X-CSRF-Token ，值是登录接口提供的 csrf_token。\n\n权限控制：每个用户可新增自己的交易所配置；管理员用户可新增所有用户交易所配置。\n\n注意：由于目前没有对 用户-交易所 进行唯一性约束，如果存在多套相同的 用户-交易所 ，约定以最新的记录为准。尽量只创建并更新唯一的 用户-交易所 。\n\ndata.id 本配置的id-用于jsonapi查询\n\n\"attributes\": {\n\n\"drupal_internal__id\": 3, 本配置的id-用于url访问\n\n\"langcode\": \"zh-hans\",\n\n\"created\": \"2024-03-21T13:16:23+00:00\", 配置创建时间\n\n\"changed\": \"2024-03-21T13:16:23+00:00\", 配置更新时间\n\n\"status\": true, 配置是否启用\n\n\"field_copy_total_amt\": \"7832.32\", 最大跟单金额（USDT）\n\n\"field_ex_apikey\": \"yuis1 test key\", 交易所 API Key\n\n\"field_ex_pass_phrase\": \"yuis1 test pass phrase\", 交易所 Secret Key\n\n\"field_ex_secretkey\": \"yuis1 test secret key\" 交易所 Pass Phrase\n\nrelationships.uid.data.id 用户的id-用于jsonapi查询\n\nrelationships.uid.data.meta.drupal_internal__target_id 用户的id-用于url访问\n\nrelationships.field_ex.data.id 交易所的id-用于jsonapi查询\n\nrelationships.field_ex.data.meta.drupal_internal__target_id 交易所的id-用于url访问"
			},
			"response": []
		},
		{
			"name": "用户-交易所配置详情查询",
			"protocolProfileBehavior": {
				"disabledSystemHeaders": {
					"accept": true
				}
			},
			"request": {
				"auth": {
					"type": "noauth"
				},
				"method": "GET",
				"header": [
					{
						"key": "Accept",
						"value": "application/vnd.api+json",
						"type": "text"
					}
				],
				"url": {
					"raw": "https://example.com/jsonapi/rel_conf/user_ex_conf/a824a797-5495-4838-a58e-8fb1934e4c1c",
					"protocol": "https",
					"host": [
						"example",
						"top"
					],
					"path": [
						"jsonapi",
						"rel_conf",
						"user_ex_conf",
						"a824a797-5495-4838-a58e-8fb1934e4c1c"
					]
				},
				"description": "/jsonapi/rel_conf/user_ex_conf/{配置id}\n\n认证：basic auth ，提供用户名、密码；或者该用户登录之后的cookies认证。\n\n权限控制：每个用户可查得自己的交易所配置；管理员用户可得查询所有用户交易所配置。\n\ndata.id 本配置的id-用于jsonapi查询\n\n\"attributes\": {\n\n\"drupal_internal__id\": 3, 本配置的id-用于url访问\n\n\"langcode\": \"zh-hans\",\n\n\"created\": \"2024-03-21T13:16:23+00:00\", 配置创建时间\n\n\"changed\": \"2024-03-21T13:16:23+00:00\", 配置更新时间\n\n\"status\": true, 配置是否启用\n\n\"field_copy_total_amt\": \"7832.32\", 最大跟单金额（USDT）\n\n\"field_ex_apikey\": \"yuis1 test key\", 交易所 API Key\n\n\"field_ex_pass_phrase\": \"yuis1 test pass phrase\", 交易所 Secret Key\n\n\"field_ex_secretkey\": \"yuis1 test secret key\" 交易所 Pass Phrase\n\nrelationships.uid.data.id 用户的id-用于jsonapi查询\n\nrelationships.uid.data.meta.drupal_internal__target_id 用户的id-用于url访问\n\nrelationships.field_ex.data.id 交易所的id-用于jsonapi查询\n\nrelationships.field_ex.data.meta.drupal_internal__target_id 交易所的id-用于url访问"
			},
			"response": []
		},
		{
			"name": "用户-交易所配置删除",
			"protocolProfileBehavior": {
				"disabledSystemHeaders": {
					"accept": true
				}
			},
			"request": {
				"auth": {
					"type": "noauth"
				},
				"method": "DELETE",
				"header": [
					{
						"key": "Accept",
						"value": "application/vnd.api+json",
						"type": "text"
					},
					{
						"key": "X-CSRF-Token",
						"value": "x5wwSCIt6nuimAkL64L7a1RA631Iwao0hD-_Ew91l3E",
						"type": "text"
					}
				],
				"url": {
					"raw": "https://example.com/jsonapi/rel_conf/user_ex_conf/88a94d08-133f-40fb-9669-7f0a220e8b01",
					"protocol": "https",
					"host": [
						"example",
						"top"
					],
					"path": [
						"jsonapi",
						"rel_conf",
						"user_ex_conf",
						"88a94d08-133f-40fb-9669-7f0a220e8b01"
					]
				},
				"description": "/jsonapi/rel_conf/user_ex_conf/{配置id}\n\n认证：basic auth ，提供用户名、密码；或者该用户登录之后的cookies认证。另外，还需要在 header提供 X-CSRF-Token ，值是登录接口提供的 csrf_token。\n\n权限控制：每个用户可删除自己的交易所配置；管理员用户可删除所有用户交易所配置。\n\ndata.id 本配置的id-用于jsonapi查询\n\n\"attributes\": {\n\n\"drupal_internal__id\": 3, 本配置的id-用于url访问\n\n\"langcode\": \"zh-hans\",\n\n\"created\": \"2024-03-21T13:16:23+00:00\", 配置创建时间\n\n\"changed\": \"2024-03-21T13:16:23+00:00\", 配置更新时间\n\n\"status\": true, 配置是否启用\n\n\"field_copy_total_amt\": \"7832.32\", 最大跟单金额（USDT）\n\n\"field_ex_apikey\": \"yuis1 test key\", 交易所 API Key\n\n\"field_ex_pass_phrase\": \"yuis1 test pass phrase\", 交易所 Secret Key\n\n\"field_ex_secretkey\": \"yuis1 test secret key\" 交易所 Pass Phrase\n\nrelationships.uid.data.id 用户的id-用于jsonapi查询\n\nrelationships.uid.data.meta.drupal_internal__target_id 用户的id-用于url访问\n\nrelationships.field_ex.data.id 交易所的id-用于jsonapi查询\n\nrelationships.field_ex.data.meta.drupal_internal__target_id 交易所的id-用于url访问"
			},
			"response": []
		}
	]
}
```

## 实现搜索API功能

在Drupal中，实现通过API根据实体（entity）字段的条件进行搜索，并支持模糊匹配，通常涉及到创建一个自定义的RESTful Web服务或使用Views模块提供的REST导出功能。这里，我将介绍两种方法：

### 方法 1: 使用Views REST导出

如果你不想编写代码，可以使用Views模块提供的REST导出功能。这不需要自定义模块，但功能上可能有一定的限制。

1. **创建一个新视图**（View）：选择要展示的实体类型和字段。在"显示格式"中选择"REST导出"。
2. **配置过滤器**（Filter criteria）：添加你需要作为搜索条件的字段。对于模糊匹配，可以使用"搜索词"或"包含"这样的操作符。
3. **配置路径**：为你的REST导出视图指定一个路径，如`/rest/search`。
4. **测试**：通过构造URL并附上相应的查询参数，比如`/rest/search/{搜索条件1}/{搜索条件2}`，来测试你的视图。

这种方法适用于简单的场景，且不需要编写额外的代码。但是，不支持模糊匹配。

### 方法 2: 创建自定义REST资源

对于需要高度定制的场景，Drupal 8及以上版本允许你通过创建自定义模块来定义RESTful Web服务。下面是一个简单的例子：

1. **创建自定义模块**：假设模块名称为`custom_search_api`。
2. **定义一个REST资源插件**：在`custom_search_api/src/Plugin/rest/resource`目录下创建一个PHP文件，比如`CustomSearchResource.php`。

```php
namespace Drupal\custom_search_api\Plugin\rest\resource;

use Drupal\rest\Plugin\ResourceBase;
use Drupal\rest\ResourceResponse;
use Symfony\Component\HttpFoundation\Request;

/**
 * Provides a resource for custom entity searches.
 *
 * @RestResource(
 *   id = "custom_search",
 *   label = @Translation("Custom Search"),
 *   uri_paths = {
 *     "canonical" = "/api/custom-search"
 *   }
 * )
 */
class CustomSearchResource extends ResourceBase {

  public function get(Request $request) {
    $query = \Drupal::entityQuery('node'); // 或者其他实体类型
    $keyword = $request->query->get('keyword');

    // 添加模糊匹配的条件
    $query->condition('title', '%' . $keyword . '%', 'LIKE');

    // 可以添加其他条件
    // $query->condition('field_your_field', 'value', 'OPERATOR');

    $ids = $query->execute();
    $nodes = \Drupal\node\Entity\Node::loadMultiple($ids);

    $data = [];
    foreach ($nodes as $node) {
      // 根据需要自定义返回的数据结构
      $data[] = [
        'nid' => $node->id(),
        'title' => $node->getTitle(),
      ];
    }

    return new ResourceResponse($data);
  }
}
```

3. **清除缓存**：让Drupal发现你的新插件。
4. **配置权限**：在`custom_search_api/config/install`目录下创建`rest.resource.custom_search.yml`，为你的REST资源配置权限。
5. **测试**：使用HTTP GET请求访问`/api/custom-search?keyword=test`，根据`keyword`参数进行模糊搜索。

这种方法提供了最大的灵活性，允许你根据需求定制搜索逻辑和结果的返回格式。需要注意的是，这里的代码仅供示例，根据实际情况可能需要作出相应的调整。