---
{"dg-publish":true,"permalink":"/CS计算机科学/后端开发/FastAPI学习笔记/","noteIcon":"","created":"2024-03-13T08:55:08.950+08:00","updated":"2024-04-24T00:02:02.622+08:00"}
---


作者: 游鱼思

---
## 简介

FastAPI专注于前后端分离开发，并且异步性能优异。可以认为是面向API版本的Flask

FastAPI 基于 Starlette 构建，继承了 Starlette 的所有特性，并在此基础上增加了对 OpenAPI 规范的支持、自动数据验证等功能。

1. **路径操作装饰器**：如 `@app.get()`, `@app.post()`, `@app.put()`, `@app.delete()` 等，用于将函数声明为响应特定 HTTP 请求的路径操作函数。
2. **路径参数**：使用 `{参数}` 在路径中声明参数，例如 `/items/{item_id}`，然后在函数中作为参数使用，用于获取 URL 中的数据。
3. **查询参数**：直接在函数的参数中声明，不需要特殊装饰器，用于从 URL 的查询部分中获取数据。
4. **请求体**：通过 Pydantic 模型定义请求体的结构，然后在路径操作函数中作为参数接收 JSON 请求体数据。
5. **响应模型**：同样利用 Pydantic 模型，可以定义 API 响应的格式，用于数据序列化和输出的控制。
6. **依赖注入系统**：使用 `Depends` 实现依赖注入，可以用于共享逻辑（如身份验证、数据库连接等）和数据（如请求中的用户信息）。
7. **安全性和认证**：提供了多种安全和认证工具，包括 OAuth2、JWT、HTTP 基本/摘要认证等。
8. **环境配置**：通过 `Settings` 模型和 `python-dotenv` 库，可以方便地管理和加载环境变量，用于配置项目。
9. **跨域资源共享（CORS）**：通过 `CORSMiddleware` 中间件，可以轻松添加 CORS 支持，解决前后端分离时的跨域请求问题。
10. **静态文件**：使用 `StaticFiles` 组件，可以方便地提供静态文件（如图片、CSS、JavaScript 文件）服务。
11. **背景任务**：通过 `BackgroundTasks` 添加后台任务，可以在响应发送给客户端后执行长时间运行的任务。
12. **测试**：FastAPI 提供了测试工具，便于使用 `pytest` 对应用进行单元测试和集成测试。

## 后台管理系统

1. **FastAPI-Admin**: 这是专为 FastAPI 设计的一个后台管理系统框架，它提供了一个简单易用的界面，用于管理数据库中的内容。你可以使用它来创建用户管理、内容管理等管理界面。FastAPI-Admin 可以自动从你的数据库模型生成CRUD（创建、读取、更新、删除）界面，支持多种数据库。
2. **Starlette-Admin**: 类似于 FastAPI-Admin，这也是一个基于 Starlette（FastAPI 底层使用的框架）构建的后台管理系统。它提供了可定制的界面，用于管理数据库内容。
## 中间件

由于FastAPI 无法同时注册两个相同路径的路由处理函数，第二个同名的路由处理函数会覆盖第一个。

可以使用中间件，进行前处理、后处理。

```Python
from fastapi import FastAPI
app = FastAPI()

@app.middleware("http")
async def log_requests(request, call_next):
    # 请求处理之前的逻辑
    response = await call_next(request)
    # 请求处理之后的逻辑，比如记录日志
    return response

@app.get("/items/")
async def read_items():
    return {"message": "Items returned successfully"}
```

## 启动事件和关闭事件

在FastAPI框架中，启动事件（Startup Event）和关闭事件（Shutdown Event）是两种特殊的应用生命周期事件，它们允许你在应用启动和关闭时执行特定的代码逻辑。这些事件对于资源管理（如数据库连接的创建和释放）、初始化操作（如加载配置文件或外部资源）以及进行清理动作（如临时文件的删除）等场景非常有用。

### 启动事件（Startup Event）

启动事件在FastAPI应用启动并准备接受请求之前触发。这是执行初始化任务的理想时机，比如：

- 建立数据库连接。
- 加载配置文件或环境变量。
- 启动异步任务或后台任务。
- 检查外部依赖服务的可用性。

在FastAPI中，你可以使用`@app.on_event("startup")`装饰器来注册一个启动事件处理函数：

```python
from fastapi import FastAPI

app = FastAPI()

@app.on_event("startup")
async def startup_event():
    print("Application is starting...")
    # 这里可以放置启动时需要执行的代码，比如数据库连接
```

### 关闭事件（Shutdown Event）

关闭事件在FastAPI应用即将停止且不再接受新的HTTP请求时触发。这是执行清理和资源释放操作的理想时机，比如：

- 关闭数据库连接。
- 停止运行的异步任务或后台任务。
- 释放占用的外部资源，如临时文件、锁等。

在FastAPI中，使用`@app.on_event("shutdown")`装饰器来注册一个关闭事件处理函数：

```python
@app.on_event("shutdown")
async def shutdown_event():
    print("Application is shutting down...")
    # 这里可以放置关闭应用前需要执行的清理代码
```

### 应用

在实际应用中，启动和关闭事件的处理函数可以是异步的，这意味着你可以在其中执行异步操作，如异步数据库操作、发送异步HTTP请求等。这提供了极大的灵活性，允许你的应用在启动和关闭时与异步系统或服务进行高效的交互。

使用这些事件可以让应用的资源管理更加精细和可控，特别是在涉及到外部资源和需要进行重量级初始化操作的复杂应用中。它们帮助你确保应用启动时所有必要的条件都已满足，同时在应用停止时能够优雅地释放资源，避免资源泄露和其他潜在问题。

总之，通过恰当使用启动事件和关闭事件，你可以提高应用的健壮性和可维护性，确保应用在其整个生命周期内管理资源和执行任务的方式既有效又优雅。