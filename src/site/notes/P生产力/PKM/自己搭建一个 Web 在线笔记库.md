---
{"dg-publish":true,"permalink":"/P生产力/PKM/自己搭建一个 Web 在线笔记库/","noteIcon":"","created":"2024-01-28T00:20:14.842+08:00","updated":"2024-04-27T01:21:35.000+08:00"}
---

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzkwNzU4NTMyMA==&mid=2247485191&idx=1&sn=abdd1f46c015af850dbcf85fb269e07b&chksm=c177e2f0dd96a132a441aab0c30cac17f0167a72b1ca5c2149876f11047bb58dbfe866d514f5&xtrack=1&scene=90&subscene=93&sessionid=1706366589&flutter_pos=0&clicktime=1706366593&enterid=1706366593&finder_biz_enter_id=4&ranksessionid=1706366572&ascene=56&fasttmpl_type=0&fasttmpl_fullversion=7049308-en_US-zip&fasttmpl_flag=0&realreporttime=1706366593396&devicetype=android-31&version=28002d3d&nettype=WIFI&abtest_cookie=AAACAA%3D%3D&lang=en&session_us=gh_aaa63197a39e&countrycode=CN&exportkey=n_ChQIAhIQZkF2c7uRvb9opwp54MyA9RLfAQIE97dBBAEAAAAAAGkoF1H%2FveoAAAAOpnltbLcz9gKNyK89dVj0wgve8rdcu%2FffEcssARH8LSoiO9ODEv3bTrSg%2BEMbdCB8ZWFSwrIE27FNP3Q8TIoIu%2BgUj56Ws7Jo2CTF%2FEVhVbxvdv1suH8iwlqVuwHbLrpcqc%2FOJ1%2FaLS0s%2BEfuzYocukie%2BQaAc5N0fjgOwIevOgdlYFV%2FJvMn0hHj2nonr6zDTtCIdJxlm08ry%2BgIMdPI983SEKZ3lHtR%2BZ9DWl8pvDdMW2hI%2FVO2RsqZLiDFWQznsYRJUKshj7g%3D&pass_ticket=nYqjntOjFo953%2BU98t8xh9hGqc1I8pAOr0qdS%2Ff8Z%2FoJYZRP8x1j6J%2BPeKzxuJd5ypy5xBnK5eqwwoJsbuVWlQ%3D%3D&wx_header=3)

推荐阅读：

[《13.9K star！一款人人可用，免费开源的数据可视化工具！》](http://mp.weixin.qq.com/s?__biz=MzkwNzU4NTMyMA==&mid=2247484795&idx=1&sn=ff0c04a38136c6c85280ffbccd6b6399&chksm=c0d7b73cf7a03e2a1d4d503e1b6050c71763fa14c14d525bf96099f08b9f72aec27ce9fc4922&scene=21#wechat_redirect)

大家好，我是开源君！

到年底了，各种总结、报告又要开始准备起来了，看着硬盘里那一大堆的资料，真是头大!

不过还好，我发现了一个好用的 Web 笔记应用 - `BatNoter`，支持 markdown 语法，可托管同步到 Github，安全方便，非常适合咱们程序员使用。

![](/img/user/Z-attach/640-17.png)

### 简介

`BatNoter`，一个自托管的 Web 端的笔记程序，主要使用 react（typescript）、redux-toolkit 和 mui 组件构建的前端项目。

支持将 GitHub 仓库作为个人笔记存储库（这个好评），同时支持笔记分类、搜索、Markdown 快捷编辑与预览、书签管理等内容。

目前`BatNoter`在 github 上收获了 2.3K star。

![](/img/user/Z-attach/640.gif)

如果你还担心云笔记的内容不安全，或者还在为云笔记的托管费用而不开心的话，那么一定要试试这个项目，笔记内容都是云托管在自己的 Git 仓库，非常的香！！！

项目特点
----

*   使用 GitHub 登录。

    

*   使用漂亮干净的用户界面轻松创建、编辑、删除、组织和探索笔记。

    

*   支持 Markdown 格式，允许用户在笔记中添加超链接、表格、标题、代码块、块引用等。

    

*   编辑器允许预览。

    

*   使用复制到剪贴板按钮从代码部分快速复制代码。

    

*   将笔记直接存储在根目录或使用文件夹来组织它们（支持嵌套）。

    

*   通过单击浏览特定目录中的所有笔记。

    

*   所有笔记都存储在用户的 GitHub 存储库中。

    

*   缓存笔记以避免额外的 API 调用。

    

*   URL 可以加书签。

    

*   支持暗 / 亮模式。

    

效果预览
----

直接打开官网进行体验：

```
https://batnoter.com/
```

使用 Github 账号就可以登录。

新建笔记

![](/img/user/Z-attach/640-18.png)

设置笔记仓库

![](/img/user/Z-attach/640-18.png)

预览笔记

![](/img/user/Z-attach/640-18.png)

查看笔记内容

![](/img/user/Z-attach/640-18.png)

亮 / 暗模式切换

![](/img/user/Z-attach/640-18.png)

本地开发环境搭建
--------

本地需要搭建 Node 环境，并且注意，Node.js 版本 18 或以上。

### 启动服务

```
npm install
npm start
```

然后在浏览器输入 http://localhost:3000 查看运行的效果。

### 运行测试

```
npm test
```

另外，官方还贴心的准备了一些 FAQ，方便新人操作。

![](/img/user/Z-attach/640-18.png)

更多其他的使用功能、细节，感兴趣的小伙伴可以自行去项目地址寻找答案~~~

```
Github 地址：https://github.com/batnoter/batnoter
```