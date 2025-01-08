---
{"dg-publish":true,"permalink":"/CS计算机科学/前端开发/javascript - http 请求头 referer 详解/","noteIcon":"","created":"2022-09-20T16:32:08.173+08:00","updated":"2024-04-27T01:26:34.000+08:00"}
---


原文： https://segmentfault.com/a/1190000024540592/

## Referer(引荐者) 请求头

### referer 与 origin 的区别？

1.  origin 是协议、域名；referer 可以根据携带路径和参数

### referer 是什么？什么时候携带？什么时候不携带？

referrer 是 HTTP 请求 header 的报文头，用于指明当前流量的来源参考页面。通过这个信息，我们可以知道访客是怎么来到当前页面的。这对于 Web Analytics 非常重要，可以用于分析不同渠道流量分布、用户搜索的关键词等。  
`**Referer**` 请求头包含了当前请求页面的来源页面的地址，即表示当前页面是通过此来源页面里的链接进入的。服务端一般使用 `Referer` 请求头识别访问来源，可能会以此进行统计分析、日志记录以及缓存优化等。

#### referer 的两种形式（个人认为）

1.  网页的 referer，即请求 html 文件接口请求头携带的 Referer；可以通过 document.refer**r**er 获取；

1.1 从 a 标签跳转的页面，从百度搜索页跳转的页面（百度自家网站，即同域名跳转），**刷新，document.referrer 不变！**  
1.2 从百度搜索页跳转的页面（其他网站的页面），**刷新，document.referrer 变为空！！** 因为涉及到百度引导流量的统计，所以这个再次刷新后 document.referrer 变为空，感觉是合理的, 但不知道是怎么实现的。？？

1.  其他请求静态资源或是交互 api 请求头的 Referer

#### 常见不携带的情况：

1.  来源页面采用的协议为表示本地文件的 "file" 或者 "data" URI；
2.  当前请求页面采用的是非安全协议，而来源页面采用的是安全协议（HTTPS）; 即从 https 到 http；与 referrer policy 的默认值一个意思。
3.  用户在地址栏输入网址，或者选中浏览器书签，也不发送`Referer`字段。
4.  使用 http 重定向

### 控制 referer 的方式

#### meta 方式

```
<meta name="referrer" content="origin">
```

```
<meta name="referrer" content="always">
```

#### html 标签方式

1.  通过`referrerpolicy`在内容属性 [a](https://link.segmentfault.com/?enc=ZymnEbte6zKH03PEpCE7fw%3D%3D.Fup3eK3IKgeOLRFCW1qyqc%2Bu4L5OSWEyCaPTz9BYjhba8iCryYz%2Fp0OP2sQaCPm4hhs50Oqk994UKAZWIcO9fGBWt6325thsFvN3%2FdcQKHk%3D)，[area](https://link.segmentfault.com/?enc=EyCgLnAFOEWlaIckso%2Fzvw%3D%3D.i2qE7yW8uj7YPexUw6mvTRwao5UL4M321uhKS96Jip5GYcbCea9s9Il8pAYgnkrq2cmS%2B2X9sEqzm%2BBO5DdZL25eJrW6LWTbuPK8rhZSmSM%3D)`，`[img](https://link.segmentfault.com/?enc=p%2Fx3Vvba7ANwm8sw9XMNsA%3D%3D.PviJkFoV9yKmRooz0MY%2F4xEN5l4li%2Fkr0lT9mplQ5bRcIyXgWxtBjMMouyLa%2B4PCMK%2F6mP%2Fesvrhd9YTpnw3L5rTjkacWj%2BDc%2FPmC5kBB%2Bo%3D)`，`[iframe](https://link.segmentfault.com/?enc=iW1cqOuRXmPcS0b35ML4PQ%3D%3D.k9uiY%2BNmTgNASRWjb7UMNkjU1aC5ynosvq9%2F3hz44eUhziN4AgN6MbdEMtajfT9vQKY3RK2CF5y2x2F0tfpJu6Ut2zikzP198zBREymSWyI1ldsph53jkUxYD3rGvRIn)`，`[link](https://link.segmentfault.com/?enc=JpHdTHg%2BIkdFX8nC0LqQoQ%3D%3D.ZhxHfzGj23P0ye0cU9n5Z4616MTSDcBpDrvUpFQeGBDobRzif6UqxpGM75%2By8t86r4LpCWHVDgMhoO2TboHQuuYvAnNhLcKdu4lWaisJbDQ%3D)`，或`[script](https://link.segmentfault.com/?enc=Jo8In9ciXv0Ox7BYN64YsQ%3D%3D.Q1kZ5jK6fyS8egFAVG7jdrx9ta0cnA3RHvQXbpFOOOb3kKf9R3kueVwC5CGYt4fnbFX1TdWulF9fCW2WrwElAA%3D%3D) 元素

1.1 例如

```
<a href="http://example.com" referrerpolicy="origin">
```

1.  通过元素 rel 属性控制。 [a](https://link.segmentfault.com/?enc=GH5OA%2BHr7UWi3jXlsVY9ZA%3D%3D.KKTfO%2BAv0DeZecHBaWgi%2FKKkSP3WkTRxRwwbfL%2BRX5578bIIw8A1mdisFE%2F81lOHoNcnODxbKQ%2FfSmDcRXFv8zeaclP%2BSYy4myhMA0SChw4%3D)`，`[area](https://link.segmentfault.com/?enc=qQusTEJ%2FKWpfUOOP8hxeuQ%3D%3D.EkwhoX0rSgYZa6u62Rxy9ICzH4RD6HcAHXi4BRd1MVjwjOhm8IXyQexN6obJcSQ%2F1TyEx8a8iqgAvnoybgP7OkhtIBsIC0zFWVvxT274M6o%3D)、form

```
<a href="..." rel="noreferrer" target="_blank">xxx</a>
```

#### 服务器发送网页的时候，通过 HTTP 头信息的`Referrer-Policy`告诉浏览器。

像这样:

```
Referrer-Policy: origin
```

## Referrer policy 控制 Referer 内容

![](/img/user/Z-attach/bVbO76u.png)

### referrer policy 各值的含义

**（1）no-referrer**

不发送`Referer`字段。

**（2）no-referrer-when-downgrade**

如果从 HTTPS 网址链接到 HTTP 网址，不发送`Referer`字段，其他情况发送（包括 HTTP 网址链接到 HTTP 网址）。这是浏览器的默认行为。

**（3）same-origin**

链接到同源网址（协议 + 域名 + 端口 都相同）时发送，否则不发送。注意，`https://foo.com`链接到`http://foo.com`也属于跨域。

**（4）origin**

`Referer`字段一律只发送源信息（协议 + 域名 + 端口），不管是否跨域。

**（5）strict-origin**

如果从 HTTPS 网址链接到 HTTP 网址，不发送`Referer`字段，其他情况只发送源信息。

**（6）origin-when-cross-origin**

同源时，发送完整的`Referer`字段，跨域时发送源信息。

**（7）strict-origin-when-cross-origin**

同源时，发送完整的`Referer`字段；跨域时，如果 HTTPS 网址链接到 HTTP 网址，不发送`Referer`字段，否则发送源信息。

**（8）unsafe-url**

`Referer`字段包含源信息、路径和查询字符串，不包含锚点、用户名和密码。

## 遗留未知内容

## 解决的问题

### 如何让浏览器在访问链接时不要带上 referer

1.  基于 [HTML 标准](https://link.segmentfault.com/?enc=s%2FrdGEOvPvY3fE6hfJz8jw%3D%3D.Av0BL9ODSaXsV3B2vyn59ERs7oCCdBQNxp3FmlyZQudigvBvLA%2B22cN%2FSmEe3AVOOgoYPm3jkWBRi%2FmyDlhFa9UcXnlyVwg3vquWuGXQQ%2B%2FGzA%2BA4WXNVcS2NOHwgFhl)，可以在 a 标签内使用`rel="noreferrer"`来达到这一目的。目前大部分基于 Webkit 的浏览器已经支持。Opera 并没有提供可以实现不发送 referrer 的方法，Firefox 也不支持。
2.  提供跨浏览器支持的更好的办法是使用一个第三方的库 [noreferrer.js](https://link.segmentfault.com/?enc=Soi1%2BxG84NOnuo6jerSvSw%3D%3D.%2Fzh0pwM4jYNpVmeLSBbAMTVwp47UsfccWvYdCNGN1p2spNHfzVpvfwwivm0sn6a%2B)，它可以自动识别浏览器并选择最优方案。实现方式利用 google url 中转，国内环境不行。
3.  跨浏览器解决方案，做一个自己网站的中转页。例如百度，搜索结果页跳转都会经过一个中转页。

#### 百度中转页做了哪些事

![](/img/user/Z-attach/bVbO8sh.png)

1.  **从 referer 剥离参数 wd**，保护用户隐私
2.  **新窗口打开中间页面**
3.  **验证点击真实性**

百度在这个中间页调用了 window.opener.bds.pdc.sendLinkLog(); 即在百度搜索结果页调用 sendLinkLog() 统计方法，验证是不是真实点击，因为如果不是从搜索结果页进入，是不存在这个方法的。虽然不能杜绝虚假模拟点击，这很大程度上过滤掉了许多爬虫和低级的虚假点击，为点击数据真实性加了 “双保险”。

1.  **剥离 Opener，保障安全**
2.  **跳转到目标页面**

## 参考内容

1.  [https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Referer](https://link.segmentfault.com/?enc=%2Bj%2Fc8tRB2hifQRC7Ja62eA%3D%3D.%2BcV5Bt8Mg16pfzZKanTe7g9hj5sxGQNZw55iYbRggsNP1QFh895zSo9vPx8rVK8kPh4yjUY9%2FOkpQo6D3Wj5Y65kgV9cybJRixVS4xOmLaI%3D)
2.  [http://www.ruanyifeng.com/blog/2019/06/http-referer.html](https://link.segmentfault.com/?enc=SmfPdUn%2Bzbd%2B24Xb8ZZCtA%3D%3D.qlmlMdDFZBzgzsYjLfScNuUORB99L%2FxEfoTND5A%2BhgReRKd4%2FIZEgudd3572QZrbr2aURsC9FeZjB9IPUIG0aw%3D%3D)
3.  [https://w3c.github.io/webappsec-referrer-policy/](https://link.segmentfault.com/?enc=vNDurfQ1nzIXCFHCYQWDKw%3D%3D.7gZtx451U21oGnQgc6IWTMhj%2FlF%2F0r3oRVkDc%2BbphAuvFMi1e1uuRIYg3ay0N2ep9grabPhpc0jUADngfwliCQ%3D%3D)
4.  [https://www.sohu.com/a/271128047_100258515](https://link.segmentfault.com/?enc=jk9opaR9Ow2JIMqsJX8csQ%3D%3D.iI5U4pTQ4gCjRF7dHo8%2BrJ%2FKoYSjDWZPwUHNqeQYZDZ0Bckr%2B5v7PKxW%2BGGrBAai)
5.  [http://www.imaiko.com/baidu-thunder-math-bug-test.html](https://link.segmentfault.com/?enc=Z5mw1JaOrQhk5SZ%2BUezK6A%3D%3D.s5vJ79KtnCeHF7T3vGQG2w9dpbn6Komay4ptE3l8qn4KD9gbiP%2BsgeXwMZr90LMSBVBc2qPAoH4JZ4EIgdZqNQ%3D%3D)