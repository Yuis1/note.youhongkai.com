---
{"dg-publish":true,"permalink":"/P生产力/PKM/给数字花园添加 Giscus 评论系统/","noteIcon":"","created":"2024-06-22T22:30:23.550+08:00","updated":"2024-04-27T01:21:35.000+08:00"}
---

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/681343213)

本文首发地址为：[https://blog.rahc.top/article/tech-share-garden-giscus-comment.html](https://link.zhihu.com/?target=https%3A//blog.rahc.top/article/tech-share-garden-giscus-comment.html).

前言
--

对于自己的数字花园，一开始并没有在评论区和自己的阅读者进行交流互动的打算，主要的原因是考虑到在数字花园中的想法并不成熟，加之数字花园现有的笔记基本是自己学习过程中的笔记，更多的是对自己所学知识的整理。直到昨天在完善自己博客的评论系统，突然想到一个评论系统更能拉近自己和读者之间的交流。面对同样的知识，读者可能会遇到各种各样的问题，希望能与作者进行交流，答疑。所以，决定也给自己数字花园加上一个评论系统。

评论系统的选择
-------

在之前写的《[利用 Obsidian 搭建自己的 Digital Garden](https://link.zhihu.com/?target=https%3A//blog.rahc.top/article/tech-share-mydigitalgarden.html)》中已经详细阐述了自己数字花园搭建的流程。数字花园的实现过程是采用了 oleeskild 开源的插件 [obsidian-digital-garden](https://link.zhihu.com/?target=https%3A//github.com/oleeskild/obsidian-digital-garden)。关于评论系统在它的[官方文档](https://link.zhihu.com/?target=https%3A//dg-docs.ole.dev/advanced/guides-and-how-tos/adding-comments/)中，推荐了 3 种方案，分别是 [giscus](https://link.zhihu.com/?target=https%3A//giscus.app/zh-CN)、[utterances](https://link.zhihu.com/?target=https%3A//utteranc.es/) 和 [disqus](https://link.zhihu.com/?target=https%3A//disqus.com/)。前两种评论都依赖于 github 账号，考虑到目前数字花园的笔记主要是后端开发和人工智能，受众是程序员，基本上都有 github 账号，所以选择了 giscus 进行配置和部署。

giscus 的部署
----------

giscus 是基于 Github Discussions 实现的评论系统，受 utterances(话语) 的启发。它会使用 Github Discussions 搜索 API 来搜索当前页面对应的 discussions，即对于当前页面的评论。

1.  点击以下链接在 github 创建新仓库。

![](/img/user/Z-attach/v2-8177b83e8b8c4becda9e4e003ef9c6df_r.jpg)

2. 在新建的仓库的 settings->general->Discussions ，中打开 Discussions 功能。

![](/img/user/Z-attach/v2-e79fd557cc20d5d935a8f1f9a7f965b3_r.jpg)

3. 点击该链接：[https(超文本传输协议安全（Hyper Text Transfer Protocol）)://github.com/apps/giscus](https://link.zhihu.com/?target=https%3A//github.com/apps/giscus)，在 github 安装 giscus 插件。
4. 点击该链接：[https://giscus.app/zh-CN](https://link.zhihu.com/?target=https%3A//giscus.app/zh-CN)，对 giscus 进行配置，把 giscus 关联到步骤 1 创建的新仓库

![](/img/user/Z-attach/v2-bdfa6bcdbdb5b5ede0c7f180be83814c_r.jpg)

5. 复制以下代码，等待后续步骤的使用。

![](/img/user/Z-attach/v2-8de69b5ebb57e0f49fa2e416f25f883b_r.jpg)

6. 配置数字花园仓库，这里我推荐用 [GitHub Desktop](https://link.zhihu.com/?target=https%3A//desktop.github.com/) 拉取数字花园仓库，再用 [vscode](https://link.zhihu.com/?target=https%3A//code.visualstudio.com/) 编辑，然后再用 GitHub Desktop 提交并推送到远程仓库的方式。

![](/img/user/Z-attach/v2-379844d00fb3adb2a0f464e2dc08eebd_r.jpg)

  

在`src/site/_includes/components/user` 目录下，我们可以给自己的数字花园添加自定义的组件。由于我们是给我们的每篇笔记页面都添加一个评论，所以在该目录下新建`notes/footer` 目录，在该目录下添加如下文件`001-giscus-comment.njk` ，文件名除了扩展名`.njk` 外，名字可以随意。文件内容为步骤 5 中的内容。

```
<script src="https://giscus.app/client.js"
 data-repo="github用户名/步骤1创建的仓库名"
 data-repo-id="你的date-repo-id"
 data-category="Announcements"
 data-category-id="你的data-category-id"
 data-mapping="pathname"
 data-strict="0"
 data-reactions-enabled="1"
 data-emit-metadata="0"
 data-input-position="bottom"
 data-theme="preferred_color_scheme"
 data-lang="zh-CN"
 crossorigin="anonymous"
 async>
</script>
```

7. 修改页面布局。编辑`src/_includes/layouts/note.njk` 文件。 默认的布局是如图所示，分为左、中、右：

![](/img/user/Z-attach/v2-9364b719097115ba2b1ca944a32cbd3e_r.jpg)

  

如果不修改，它会在最底层生成一个横跨左、中、右的一个评论区域，影响美观。而我只是想在中间的底部，加上一个评论区域。在`note.njk` 文件的`<main>...</main>` 之间添加如下代码：

```
<main>
 // 底部添加如下代码
 {% for imp in dynamics.notes.footer %}
   {% include imp %}
  {% endfor %}
</main>
```

  

注释掉 main 之后，body 内相同的代码：

```
<body>
 {# {% for imp in dynamics.notes.footer %}
   {% include imp %}
  {% endfor %} #}
  {%include "components/lucideIcons.njk"%}
</body>
```

  

现在的布局如图所示：

![](/img/user/Z-attach/v2-3c22b7b51100050e1b5ad57394f0645b_r.jpg)

8. 最终的效果如图所示：

![](/img/user/Z-attach/v2-17d7e8aba9e3306d4c50e82e43431042_r.jpg)

  

最后，欢迎您到[我的数字花园](https://link.zhihu.com/?target=https%3A//garden.rahc.top/)逛逛，一起进步成长，学有所成。

总结归纳
----

本文简单讲述了如何为自己的数字花园添加 giscus 评论系统。

引用或提到的资源
--------

*   Mr.Charley(查理（人名）). 利用 Obsidian(黑曜石) 搭建自己的 Digital Garden. [https(超文本传输协议安全（Hyper Text Transfer Protocol）)://blog.rahc.top/article/tech-share-mydigitalgarden](https://link.zhihu.com/?target=https%3A//blog.rahc.top/article/tech-share-mydigitalgarden).
*   oleeskild. obsidian(黑曜石)-digital-garden. [https(超文本传输协议安全（Hyper Text Transfer Protocol）)://github.com/oleeskild/obsidian-digital-garden](https://link.zhihu.com/?target=https%3A//github.com/oleeskild/obsidian-digital-garden).
*   giscus. [https(超文本传输协议安全（Hyper Text Transfer Protocol）)://giscus.app/zh-CN](https://link.zhihu.com/?target=https%3A//giscus.app/zh-CN).
*   utterances(话语). [https(超文本传输协议安全（Hyper Text Transfer Protocol）)://utteranc.es/](https://link.zhihu.com/?target=https%3A//utteranc.es/).
*   disqus. [https(超文本传输协议安全（Hyper Text Transfer Protocol）)://disqus.com/](https://link.zhihu.com/?target=https%3A//disqus.com/).
*   GitHub Desktop. [https://desktop.github.com/](https://link.zhihu.com/?target=https%3A//desktop.github.com/).
*   vscode. [https://code.visualstudio.com/](https://link.zhihu.com/?target=https%3A//code.visualstudio.com/).

​ 有关在数字花园中部署 giscus 上的问题，欢迎您在底部评论区留言，一起交流~