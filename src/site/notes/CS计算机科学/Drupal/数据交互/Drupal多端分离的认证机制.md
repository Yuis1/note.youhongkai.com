---
{"dg-publish":true,"permalink":"/CS计算机科学/Drupal/数据交互/Drupal多端分离的认证机制/","tags":["JWT"],"noteIcon":"","created":"2024-03-23T08:45:35.236+08:00","updated":"2024-04-23T23:01:36.714+08:00"}
---


作者: 游鱼思

---
## 问题描述

前端：使用react或vue，用户在此登录。

Python端：没有用户界面，只负责交易对接。

Drupal端：负责内容体系、用户体系、登录认证。

React可以请求drupal端进行身份认证。

但是react在请求python端进行交易时，python端要如何鉴权当前请求的是哪个用户？

并且，不希望把一套密钥硬编码在react应用中，那样会有安全风险。

## 方案

背景知识：[JWT(JSON Web Token)](../../前端开发/认证/JWT(JSON%20Web%20Token).md)

文档：[JWT Authentication | API Authentication | Drupal Wiki guide on Drupal.org](https://www.drupal.org/docs/contributed-modules/api-authentication/jwt-authentication)

在这种多端应用场景下，一个可行的解决方案是采用基于Token的认证机制，具体来说是使用JWT（JSON Web Tokens）。这种方式可以在React前端、Python后端和Drupal之间安全地管理用户认证和授权。

1. 用户在React端通过Drupal认证登录成功后,Drupal服务端会生成一个JWT token,包含用户ID、角色权限等Claims信息,并返回给React端。
2. React端将该JWT token保存下来(如localStorage),在后续向Python服务端发起请求时,都需要在HTTP请求头中携带该JWT token。
3. Python服务端需要有一个JWT认证中间件,对于每个API请求,都需要先验证请求头中携带的JWT token是否合法有效,其中会验证Token的完整性(是否被篡改)、Token的签名是否由合法密钥签发、Token是否过期等。
4. 如果Token验证通过,就能从Token的Payload中解析出请求用户的ID、角色权限等Claims信息,Python端就知道当前请求的发起者身份了。
5. 对于不同的API,Python端可以根据解析出的用户身份信息,判断该用户是否有权限访问该API,从而实现授权控制。
6. JWT token的签发密钥可以是Drupal与Python服务共享的秘钥,或由可信的授权服务器(如Keycloak)统一签发和管理。React端不需要知道这个签名密钥。

### 步骤 1：用户在React应用中登录

1. 用户在React前端应用中输入登录凭据（用户名和密码）。
2. React应用将登录凭据发送到Drupal后端进行身份验证。

### 步骤 2：Drupal生成JWT

1. Drupal验证用户凭据。如果凭据有效，Drupal生成一个JWT，里面包含了用户的身份信息和权限数据。
2. Drupal将这个JWT返回给React应用。

### 步骤 3：React应用存储JWT

1. React应用接收到JWT后，将它存储在客户端的安全存储中，比如`localStorage`或`sessionStorage`（具体选择取决于您的安全需求和应用场景）。

### 步骤 4：React应用使用JWT请求Python端

1. 当React应用需要请求Python端进行交易对接时，它会从安全存储中获取JWT，并将这个Token作为请求的一部分发送给Python后端，通常是作为HTTP请求头中的`Authorization`字段。

### 步骤 5：Python端验证JWT

1. Python后端接收到请求后，首先解析`Authorization`字段中的JWT。
2. Python端需要有逻辑来验证JWT的有效性。这通常涉及到检查Token的签名、有效期等。为了验证签名，Python端需要有一个与Drupal生成JWT时相同的密钥或者公钥（如果使用RSA签名）。
3. 一旦JWT被验证为有效，Python后端就可以从Token中提取用户信息，然后根据用户信息处理请求。

### 关键考虑：

- **安全存储Token**：确保在客户端安全地存储JWT，避免跨站脚本攻击（XSS）。
- **避免密钥泄露**：确保密钥（用于签名JWT）的安全，不要在React应用中硬编码密钥，密钥应只在Drupal和Python后端之间安全共享。
- **Token刷新机制**：可能需要实现Token刷新机制，在JWT快过期时自动获取新的Token，以提升用户体验，避免用户频繁登录。
- **跨域资源共享（CORS）**：如果React应用和后端服务部署在不同的域下，需要正确配置CORS政策，确保前端应用能够安全地调用后端服务。
-

这种做法的优点是:

- 无状态,不需要在服务器端保存用户会话
- JWT自身包含了身份认证所需信息,可跨多个服务传递
- 通过签名可以防止Token被篡改
- React端只是形同"持token者"而已,不会暴露核心的认证秘钥

缺点是JWT token一旦签发就无法主动废止,通常需要设置一定的过期时间,过期则重新认证。

所以使用JWT可以实现你所描述的多服务跨域的用户鉴权需求,避免将核心密钥暴露给前端,也避免前后端直接共享用户凭证。这是目前很通用的做法。

## 需要启用的模块

在上述场景中,如果使用Drupal的JWT模块来实现JWT认证,需要启用以下子模块:

1. Key
**依赖模块**：JWT模块通常依赖于Key模块来管理用于签名和验证JWT的密钥。Key模块允许您安全地存储和管理密钥，而不是将密钥硬编码在配置文件中。
2. JSON Web Token (jwt)
这是JWT模块的核心模块,提供了基本的JWT生成、验证和管理功能。它是必须启用的。

### 使用要点
1. Key：不是所有Key都被JWT支持，我测了 RSA Key：RS 256 可以支持
2. 需要开启 JWT Auth issuer 模块 ，这样在登录接口才能拿到 "access_token"
3. 在其它需要认证的接口中，插入  --header 'Authorization: Bearer {access_token}'

### JWT Token过期问题

JWT Token 1 小时后过期

可以通过 /jwt/token ，认证方式用 cookies、simple_auth、jwt_token，获取更新的jwt_token