---
{"dg-publish":true,"permalink":"/CS计算机科学/前端开发/WWW仍然属于 URL 吗？它可以消失吗/","noteIcon":"","created":"2024-06-22T22:30:30.079+08:00","updated":"2024-03-20T15:07:16.000+08:00"}
---

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [developer.aliyun.com](https://developer.aliyun.com/article/1295331)

> “WWW” 属于 URL 吗？一些开发人员对此主题持有强烈的意见。在了解了一些历史之后，我们将探讨支持和反对它的论据。

> 多年来，我们的地址栏上一直在进行着一场小小的较真战。也就是 [Google](http://www.google.com/)、[Instagram](http://www.instagram.com/) 和 [Facebook 等](http://www.facebook.com/)品牌。该群组已选择重定向 `example.com` 至 `www.example.com`。相反：[GitHub](http://github.com/)、[DuckDuckGo](http://duckduckgo.com/) 和 [Discord](http://discord.com/)。该组织选择执行相反的操作并重定向 `www.example.com` 到 `example.com`

**“WWW” 属于 URL 吗？一些开发人员对此主题持有强烈的意见。在了解了一些历史之后，我们将探讨支持和反对它的论据。**

WWW 是什么？
--------

WWW 代表`"World Wide Web"`，是上世纪 80 年代晚期的一个发明，引入了浏览器和网站。使用 "WWW" 的习惯源于给子域名命名的传统：

*   一个位于`www.example.com`的 Web 服务器
*   一个位于`ftp.example.com`的 FTP 服务器
*   一个位于`irc.example.com`的 IRC 服务器

如果没有 WWW 会发生什么问题？
-----------------

### 1. 向子域名泄露 cookies

反对`"没有WWW"`的域名的批评者指出，在某些情况下，`subdomain.example.com`可以读取`example.com`设置的`cookies`。如果你是一个允许客户在你的域名上运营子域名的 Web 托管提供商，这可能是不希望看到的。

然而，这种行为只存在于 Internet Explorer 中。

RFC 6265 标准化了浏览器对 cookies 的处理，并明确指出这种行为是错误的。

另一个潜在的泄露源是 example.com 设置的 cookies 的 Domain 值。如果 [Domain](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie) 值明确设置为 example.com，那么这些 cookies 也将被其子域名所访问。

<table><thead><tr><th>Cookie 值</th><th>暴露于 example.com</th><th>暴露于 subdomain.example.com</th></tr></thead><tbody><tr><td>secret=data</td><td>✅</td><td>❌</td></tr><tr><td>secret=data; Domain=example.com</td><td>✅</td><td>✅</td></tr></tbody></table>

总之，只要你不明确设置`Domain值`，而且你的用户`不使用Internet Explorer`，就不会发生 cookie 泄露。

### 2. DNS 的困扰

有时，"没有 WWW" 的域名可能会使你的域名系统 (DNS) 设置复杂化。

当用户在浏览器的地址栏中输入 example.com 时，浏览器需要知道他们想访问的 Web 服务器的 Internet 协议 (IP) 地址。浏览器通过你的域名的域名服务器向其 DNS 服务器（通常间接通过用户的互联网服务提供商 (ISP) 的 DNS 服务器）请求 IP 地址。如果你的域名服务器配置为响应包含 IP 地址的 [A 记录](https://wizardzines.com/comics/dns-record-types/)，那么 "没有 WWW" 的域名将正常工作。

在某些情况下，你可能希望使用[规范名称 (CNAME)](https://wizardzines.com/comics/cname/) 记录来代替为你的网站设置。这样的记录可以声明`www.example.com`是`example123.somecdnprovider.com`的别名，这会告诉用户的浏览器去查找`example123.somecdnprovider.com`的 IP 地址，并将 HTTP 请求发送到那里。

请注意，上面的示例使用了一个`WWW子域名`。对于`example.com`，不可能定义一个 CNAME 记录。根据 [RFC 1912](https://www.ietf.org/rfc/rfc1912.html#section-2.4)，CNAME 记录不能与其他记录共存。如果你尝试为`example.com`定义 CNAME 记录，`example.com`上的 MX（邮件交换）记录将无法存在。因此，就`不可能在@example.com上接收邮件`。

一些 DNS 提供商可以让你绕过这个限制。Cloudflare 称其解决方案为 [CNAME 解析](https://blog.cloudflare.com/introducing-cname-flattening-rfc-compliant-cnames-at-a-domains-root/)。通过这种技术，域名管理员配置一个 CNAME 记录，但他们的域名服务器将暴露一个 A 记录。

例如，如果管理员为`example.com`配置了指向`example123.somecdnprovider.com`的 CNAME 记录，并且存在一个指向`1.2.3.4`的`example123.somecdnprovider.com`的 A 记录，那`么Cloudflare`就会暴露一个指向`1.2.3.4`的 example.com 的 A 记录。

总之，虽然这个问题对希望使用`CNAME`记录的域名所有者来说是有效的，但现在有一些 DNS 提供商提供了合适的解决办法。

没有 WWW 的好处
----------

大部分反对 WWW 的论点是实用性或外观方面的。"无 WWW" 的支持者认为`example.com`比`www.example.com`更容易说和输入（对于不那么精通技术的用户可能更不容易混淆）。

反对 WWW 子域名的人还指出，去掉它会带来一种谦虚的性能优势。网站所有者可以通过这样做每个 HTTP 请求节省 4 个字节。虽然这些节省对于像 Facebook 这样的高流量网站可能会累积起来，但带宽通常并不是一种紧缺的资源。

有 "WWW" 的好处
-----------

支持 WWW 的一个实际论点适用于使用较新顶级域的情况。例如，`www.example.miami`在`example.miami`无法立即被识别为 Web 地址。对于具有诸如. com 这样的可识别顶级域的网站，这不是一个太大的问题。

### 对搜索引擎排名的影响

目前的共识是你的选择不会影响你的搜索引擎表现。如果你希望从一个 URL 迁移到另一个 URL，你需要配置永久重定向（HTTP 301）而不是临时重定向（HTTP 302）。永久重定向确保你旧的 URL 的 SEO 价值转移到新的 URL。

### 同时支持两者的技巧

网站通常会选择`example.com`或`www.example.com`作为官方网站，并为另一个配置 HTTP 301 重定向。理论上，可以支持`www.example.com和example.com两者`。但实际上，成本可能会超过效益。

从技术角度来看，你需要验证你的技术栈是否能够处理。你的内容管理系统 (CMS) 或静态生成的网站需要将内部链接输出为相对 URL 以保留访问者的首选主机名。除非你可以将主机名配置为别名，否则你的分析工具可能会将流量分别记录在两个主机名上。

最后，你需要采取额外的措施来保护你的搜索引擎表现。谷歌将把 URL 的`"WWW"`和`"非WWW"`版本视为重复内容。为了在其搜索索引中去重复内容，谷歌将显示它认为用户更喜欢的那个版本——不论是好是坏。

为了在谷歌中保持对自己的控制，建议插入规范链接标签。首先，决定哪个主机名将成为官方（规范）主机名。

例如，如果你选择了 www.example.com，则必须在 `https://example.com/my-article`里的 `<head>` 上的标记 中插入以下代码段：

```
<link href="<https://www.example.com/my-article>" rel="canonical">
```

这个代码片段告诉谷歌 "无 WWW" 变体代表着相同的内容。通常情况下，谷歌会在搜索结果中偏好你标记为规范的版本，也就是在这个例子中的 "WWW" 变体。

总结
--

对于是否在 URL 中加入 "WWW"，人们有不同的观点。下面是支持和反对的论点：

支持 "WWW" 的论点：

1.  存在子域名的安全性问题：某些情况下，子域名可以读取主域名设置的 cookies。虽然这个问题只存在于 Internet Explorer 浏览器中，并且已经被 RFC 6265 标准化修复，但仍有人认为使用 "WWW" 可以避免潜在的安全风险。
2.  DNS 配置的复杂性：如果你的域名系统 (DNS) 配置为响应包含 IP 地址的 A 记录，那么 "没有 WWW" 的域名将正常工作。但如果你想使用 CNAME 记录来设置规范名称，那么 "没有 WWW" 的域名可能会导致一些限制，例如无法同时定义 CNAME 记录和 MX（邮件交换）记录。
3.  对搜索引擎排名的影响：对于使用较新顶级域的网站，使用 "WWW" 可以帮助识别网址，而不是依赖可识别的顶级域名。然而，目前的共识是选择是否使用 "WWW" 对搜索引擎表现没有直接影响。

支持去除 "WWW" 的论点：

1.  实用性和外观：去除 "WWW" 可以使域名更简洁和易于输入，减少了用户可能混淆的机会。
2.  节省字节：去除 "WWW" 可以每个 HTTP 请求节省 4 个字节。虽然这对于高流量网站来说可能是一个可累积的优势，但对于大多数网站来说，带宽通常不是一个紧缺的资源。

最佳实践：  
一般来说，网站会选择将 example.com 或 www.example.com 作为官方网址，并对另一个进行重定向。你可以通过使用 HTTP 301 永久重定向来确保旧 URL 的 SEO 价值转移到新 URL。同时，你还可以在页面的

标签中插入规范链接标签，告诉搜索引擎两个 URL 代表相同的内容，以避免重复内容问题。

需要注意的是，在做决策时要考虑到技术栈的支持能力、DNS 配置的限制和谷歌对搜索排名的处理方式。

[本文同步我的技术文档](https://docs.zcsuper.cn/)