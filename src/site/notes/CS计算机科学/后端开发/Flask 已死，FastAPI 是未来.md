---
{"dg-publish":true,"permalink":"/CS计算机科学/后端开发/Flask 已死，FastAPI 是未来/","noteIcon":"","created":"2024-06-22T22:30:28.791+08:00","updated":"2024-04-27T01:22:45.000+08:00"}
---


原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/672806587)

周末搜索一些信息时发现 2023 年还有不少人推荐使用 Flask 作为 Python 的 Web 框架，而我已经认为「Flask 已死，[FastAPI](https://link.zhihu.com/?target=https%3A//github.com/tiangolo/fastapi) 是未来」，所以写下本文，欢迎讨论和反驳。

### Flask vs FastAPI

Flask 是一个在 Python 开发者心目中十分重要的项目，如果你是一个 Web 开发者，我相信你一定用过 Flask，但是可能没有用过 FastAPI。这个和国外相比可能会更明显一点，我觉得主要是因为国内已经没有多少人在写技术文章推荐，教大家分辨趋势或者主流，而一些老手即便知道它们的优缺点可能也懒得写对比文章，结果造成很多新人不了解怎么选，只能看一些老的文章、大 V 或者书籍的推荐作为选择的依据。

但是如果你有兴趣，你可以考证 2 点:

1.  最近 1-2 年，Python 相关的知名新项目中只要有 Web 的，基本都使用 FastAPI。
2.  今天 (2023 年 12 月 18 日) 在 Github 上，FastAPI 的 star 数 (66k) 已经超过了 Flask(65.2)。

接着看一下 Python 官方的开发者调查中 Web 框架占比的变化:

![](/img/user/Z-attach/v2-2f4795862e9ab1e3a336acb2c749b36c_r.jpg)

可以看到 19 年 FastAPI 甚至不是一个选项，而 22 年已经占到 25% 的比率。这个开发者调查需要年中才会统计出上一年的，所以 2023 年的得明天第三季度才能知道，我预计 FastAPI 可以达到 35% 左右。

需要注意这个占比数据包含了现有系统，所以 Django 和 Flask 不容易跌得很快，但是趋势是很明显的。

我们知道 Web 框架这个领域是很高产的，基本每年都有新的框架诞生，大部分框架昙花一现，但有的框架却能够长青。FastAPI 诞生于 2018 年底，在 2019 年年底左右开始崭露头角，那么为什么它可以在短短的 5 年里让关注度超过在 2010 年底诞生的 Flask 呢？接着我们顺着时间线捋捋 Python Web 框架和相关解决方案的发展来理解吧。

### Web 框架 (插件、工具) 发展史

FastAPI 作者是一个非常关注 Python 生态发展的开发者，延伸阅读链接 2 是一篇作者写的，详细的介绍了 FastAPI 出各个库中的借鉴或者启发的细节，而发展史这部分我也参考了这篇文章。大家应该看看原文，里面包含了为什么会出现 FastAPI 以及作者的一些设计哲学。

### Django

Django 是 Python 社区里最早做到统治级别的 Web 框架，当时知名的 Instagram 就是用它构建的。它的特点是大而全，各种 web 开发需要用到的细节它基本都有相关的模块，尤其和关系数据库、管理功能等方面耦合度非常高。对于熟悉的老手，这是一个生产级别的框架，但即便有完整的文档，对于初学者非常不友好，学习曲线中后期很陡峭，它的复杂度和定制成本非常高。

其实我这篇文章，也可以叫做「Django 和 Flask 之死」，但是在框架应用场景角度来说 Flask 和 FastAPI 对比更合适，而 Django 直接和 FastAPI 相比有一些差别。在一些商业场景 (例如 CMS)Django 依然是首选，而 Flask(甚至 FastAPI) 看起来更像个「玩具」，所以 Django 短时间不会被取代(Flask 这些年不是也是没取代的了嘛)。

### Django REST Framework(DRF)

Django 主旨是为了在后端生成 HTML，而不是创建现代前端（如 React、Vue.js ）或与其通信的其他系统使用的 API。所以 FastAPI 其实和 Django REST Framework 直接对标，它们主要场景都是构建 Web API，但是名字上也可以看出来，DRF 还是依托于 Django 框架，所以缺点一样。

### Flask

Flask 是一个「微」框架（Micro framework），和 Django 截然相反，它仅保留小部分的核心，还为了解耦把功能拆分成几个库 (如 Jinja2，Werkzeug 等)，所以开发者有足够的自由，你可以很容易写出相关功能的第三方插件。它里面的蓝图、上下文、装饰器表示路由、信号等等设计在当时是非常先进的，再加上文档写的很完整，可以说对新手极为友好。

### Flask REST Frameworks

由于 Flask 的简单性，它非常适合构建 API，不过由于 Flask 什么也不带，我们需要专门的 REST 框架。所以相继出现了 flask-restful 、Flask-RESTPlus、flask-api 等框架，另外在 Rest 服务中，会需要数据验证、解析和规范等需要，也出现了 Marshmallow、Webargs 和 APISpec，一直到 Flask-apispec。但是整个发展过程中没有出现一个足够好的能够对标 DRF 的 Flask REST Framework。

而到这个阶段，Flask 的缺点其实也暴露了。

Flask 本来的优点是灵活性和极简主义，但这意味着很多很多组件需要自己造，这个要不然需要公司够大专门有开发者开发和维护，要不然得个人能力极强，否则插件很难达到生产级别，这就造成 Flask 的第三方插件质量层次不齐，且无法保证长期维护。就像我上面说的那些库，现在来看，其中已经很多不再维护了。

所以即便是今天，你想用 Flask 构建一个 API 服务，还是要东拼西凑各个组件，其中某些组件有些没及时更新的地方要自己动手解决，这对于老手来说还好，对于新人来说很痛苦，尤其是你想要应用现在最新的实践方案和理念，只能望而兴叹。

### Asyncio 生态

从 Python3.5 开始，asyncio 已经是未来的趋势了。所以开始出现了一些原生支持 asyncio 的 Web 框架，如 aiohttp、Starlette 和 sanic。

这个时候 Flask 并不想接受改变，社区迟迟的不加入 aio 的支持，另外 Flask 原作者也去写 rust 了，把项目交给了 2 个维护者（现在只剩一个人了)。

而用于构架 API 服务的项目例如 apistar，molten。它们都给 FastAPI 的带来了设计灵感。

### FastAPI

接着就是诞生 FastAPI 了。作者本来也是想找到一个好的解决方案，但是以上种种都是自己的问题或者说局限性，所以作者就造了这个轮子。

### FastAPI 为什么能成为统治者

这篇文章的核心部分啦，这些理由也是为什么 FastAPI 会替代 Flask 的原因。

### 异步设计

在 Flask 的那个时代，代码执行是单线程、同步的，这意味着要依次处理请求，前一个请求完成前，其他请求消耗在等待 I/O 操作上。而 asyncio 是最好的解决方案，它让 I/O 成为异步操作，无需等待任务完成即可获取任务结果并继续处理其他任务请求。

而 FastAPI 天生支持并发和异步代码，代码写对了就可以在效率上达到极致，所以它被认为是目前最快的 Python 框架，效率和 NodeJS 或 Go 接近。当速度和性能至关重要时，FastAPI 是最佳选择。

### 使用 Pydantic 做用户数据验证

API 开发中数据格式的验证、解析和序列化是很常用的，在这些年的发展过程中出了很多种方案，而目前最好的方案就是 [Pydantic](https://link.zhihu.com/?target=https%3A//docs.pydantic.dev/latest/):

```
from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None


app = FastAPI()


@app.post("/items/")
async def create_item(item: Item):
    return item
```

这个代码初看是 ORM/dataclass 的写法，其实是基于 Python 原生的 Type Hints 的语法给字段做类型注解，例如上例的`/items/`请求中的 Item 的 schema 已经定义好了，各个字段的值类型非常明确。和过去用 schema 描述甚至硬代码的方式相比很干净很 pythonic，对于 IDE 支持也更好。

在现阶段，Pydantic 在用户数据验证等领域是统治级别的，而 FastAPI 内置了它，这可以大大简化了验证过程，它还可以减少错误，所以 FastAPI 官网介绍优点中就提到这个方案可以将开发人员错误减少高达 40%。对于 Python 这种动态语言来说，如果不是用 mypy 强制类型检查，应用 Pydantic 是非常有必要的。

另外由于集成了 Pydantic，所以非常容易在项目中添加 ORM（如 [SQLAlchemy](https://link.zhihu.com/?target=https%3A//www.sqlalchemy.org/)），从请求中获得的对象可以直接传递到数据库，因为已经做过数据验证。反之亦然，可以将从数据库获取的对象直接返回。

相对的，Flask 这方面的缺失的。

### 原生支持 ASGI

先提一下 WSGI，他的全称是「Python Web Server Gateway Interface」，具体可以看延伸阅读链接三的《PEP 3333》，它是专门为 Web 应用程序和服务器相互交互而编写的 Python 标准。如果你使用过 PHP 或 Ruby 的人会更容易理解它。Flask 的依赖 Werkzeug 就是 WSGI 套件，所以 Flask 支持的是这个老的 WSGI，不支持 ASGI。

WSGI 的问题是无法利用异步达到性能和效率提升，所以 Django 组织首先提出了 ASGI。它的全称是「Asynchronous Server Gateway Interface」，它是一个迭代但基本的重新设计，它提供了异步服务器 / 应用程序接口，并支持 HTTP、HTTP/2 和 WebSocket。与 WSGI 不同，ASGI 允许每个应用程序有多个异步事件。另外，ASGI 支持同步和异步应用程序。你可以将旧的同步 WSGI Web 应用程序迁移到 ASGI，也可以使用 ASGI 构建新的异步 Web 应用程序。

出结论之前，先稍微补充 5 个名词解释:

1.  ASGI 工具集。实现了 ASGI 相关功能 (例如 URL 路由、Request/Response 对象、模板引擎等) 的库，在本文中主要指 [Starlette](https://link.zhihu.com/?target=https%3A//github.com/encode/starlette)，它对标的是 Flask 的依赖 Werkzeug。
2.  ASGI Web 框架。实现了 ASGI 规范的 Web 框架 (例如 FastAPI），而 Flask、Django 是实现了 WSGI 的 Web 框架，它是面向开发者编写应用程序，接口非常易用，开发者按照需求填业务逻辑就行。早期的框架都是内部实现了 ASGI 工具集，而后来的框架通常是组合合适的工具集来用，例如 Flask 用 Werkzeug(自己家的)，FastAPI 用 Starlette(别家的)。
3.  Web 应用。使用了 Web 框架创建的应用程序就是一个 Web 应用，而通常 Web 框架自带一个测试服务器，可以启动起来用于开发调试，如果不考虑性能和稳定性，你已经可以在浏览器里访问开发地址访问这个应用了。
4.  Web 服务器 (Web Server)。现实世界要比预想的复杂，Web 应用部署到生产环境后需要考虑请求负载均衡、服务静态文件、访问控制、反向代理等等需求，同时对于性能也很有要求，Web 框架内置的 Web 服务器时远远达不到这个要求的，所以就需要有专门的 Web 服务器，现在主流的就是 Nginx。
5.  ASGI 服务器。ASGI 服务器是 Web 服务器和 Web 应用之间的桥梁，ASGI 服务器也会遵守 ASGI 的规范，但是它的主要任务是满足转发请求的性能要求，所以主要做的是 ASGI 里面的 G 这部分，也就是网关，它的代码对于开发者写 Web 应用并不友好，不会有开发者直接使用 ASGI 写业务和路由逻辑。目前最知名的 ASGI 服务器是 [Uvicorn](https://link.zhihu.com/?target=https%3A//www.uvicorn.org/)，另外使用本来是 WSGI 服务器的 Gunicorn 的`uvicorn.workers.UvicornWorker`也可以。都是生产环境推荐的用法。

过去一般 WSGI 的生产环境方案是 `Nginx+Gunicorn+Flask(Django)`，而现在 ASGI 的生产环境方案是 `Nginx+Uvicorn+FastAPI`。

再补充一点。FastAPI 无论看项目名字还是介绍都能感觉出来它是用于构建 API 服务的，事实上 FastAPI 自己的核心代码也确实是这样的，可以说它不是一个传统的、完全自己实现的框架，它更像是一个集各家之长的框架，从一个空壳开始，把需要的、适合的组件组装起来。例如它没有模版引擎，如果你确实需要用它实现一个 web 应用要渲染模版，你可以再组合你想要的选择，当然还可以用 Starlette 内置的 Jinja2（是的，也是 Flask 内置的）。

### 为什么说 Flask 已死

上面提的是 FastAPI 的优势，但是也不能说明 Flask 已死，我为什么会这么觉得呢? 主要还是看开发者和用户的人气。

这个「人气」是一个很主观的感受，我能想到的有如下指标:

1.  社区活跃度。例如项目的 Issue 和 Pull Request 数量，Flask 都是个位数，和 FastAPI 相比根本不是一个水平。这其实侧面证明项目不再活跃，因为如果活跃的话，老的用户会有更多需求，它们不来说明已经离开，而新的用户意味着有各种问题，文档即便足够全，其实我们要知道还是会有很多用户来提问和贡献代码的，没有就说明用得少。
2.  讨论度。也就是博客文章，Stackoverflow 等网站咨询和讨论的热度，其实可以感受到，已经没什么人写 Flask 相关的内容了。
3.  开发迭代的频率。翻一下 commits，可以看到 Flask 只有一个维护者会偶尔修修 bug，没有什么大的功能特性开发。
4.  灵魂人物的影响力。Flask 的灵魂人物，也就是项目的发起人其实早就不再参与项目了，搜一下项目贡献记录可以看到 Armin Ronacher 上次参与开发已经是 6 年前了。

以上种种在我这里其实都说明 Flask 已死，大家应该考虑替代品。

哎，有一点伤感，感谢 Flask 带我入了 Web 开发的坑。

### 后记

是的，我认为现在是抛弃 Flask 的时候了，如果你是一个 Python Web 服务的初学者请直接使用 FastAPI，如果你是一个有一定工作经验的人，我建议下一个项目请使用 FastAPI 甚至替代现在的项目，欢迎讨论~

### 延伸阅读

1.  [https://lp.jetbrains.com/python-developers-survey-2022](https://link.zhihu.com/?target=https%3A//lp.jetbrains.com/python-developers-survey-2022)
2.  [https://fastapi.tiangolo.com/alternatives/](https://link.zhihu.com/?target=https%3A//fastapi.tiangolo.com/alternatives/)
3.  [https://peps.python.org/pep-3333/](https://link.zhihu.com/?target=https%3A//peps.python.org/pep-3333/)
4.  [https://asgi.readthedocs.io/en/latest/introduction.html](https://link.zhihu.com/?target=https%3A//asgi.readthedocs.io/en/latest/introduction.html)
5.  [https://www.encode.io/articles/hello-asgi](https://link.zhihu.com/?target=https%3A//www.encode.io/articles/hello-asgi)
6.  [https://fastapi.tiangolo.com/deployment/server-workers/](https://link.zhihu.com/?target=https%3A//fastapi.tiangolo.com/deployment/server-workers/)