---
{"dg-publish":true,"permalink":"/P生产力/PKM/Obsidian手把手教你快速上手（附核心配置）/","noteIcon":"","created":"2023-11-20T15:47:55.000+08:00","updated":"2024-04-24T00:44:15.503+08:00"}
---

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/2def06958af6)

[Obsidian&Zettelkasten&LYT 系列文章的总目录](https://www.jianshu.com/p/6be9cbb134de)

      上一篇我介绍了 obsidian 的概况和自己的部分使用体验，有网友反馈说希望把 obsidian 的使用说得更具体些。这次我就从如何具体使用 obsidian 来做进一步解读。

      使用 obsidian 一个最重要的内容就是理解 Zettelkasten 笔记的操作方法。一个被广为流传的例子就是 Niklas Lumann（尼古拉斯 · 卢曼），一位著名的社会学家应用了这套方法，在其一生中，做出了令人惊叹的学术成就，其生活和工作效率无人能够望其项背，可谓是当代的生产力大师。有一本非常牛逼的书专门介绍了整套方法，《how to make smart notes》，中文版地址我放在了文章的后面，大家一定要看看，很超值！

![](/img/user/Z-attach/4565391-cdc348915f77457e.png) 尼古拉斯 · 卢曼

      这套方法的精髓上篇文章中做过介绍，大家可以翻看：[构建属于自己的第二大脑 obsidian](https://www.jianshu.com/p/d7bd0cb9d257)。简单的讲就是四个步骤：第一，随时随地记下自己的想法或者灵感。第二，对有价值的内容，比如文章、视频、教程、研讨等，把产生共鸣的内容快速记在 ZK 卡片上，正面记名称和出处，背面用自己理解的话，简明扼要复述你理解的内容，记住！一定要用自己的话。第三，每天或者遇到了合适的时机就把上述两类内容稍微整理下，与内容前后关联的卡片或内容，用前后关联的方式连接起来，形成思维链，并且确保用到的时候随时能检索到。第四，遇到具体事情的时候，不要从零开始，而是把思维链调取出来，继续在此基础上迭代。这是一个大杀器，我可以保证当你一旦用上之后便会迷上它。来吧，我们开始这个精彩之旅吧!

      上干货：以下分六个步骤讲解，并附上各个步骤对应的关键配置文件。

**Step1：先上手 obsidian，找找感觉**

      如果你已经下载好 obsidian，现在点击打开它。映入眼帘的是它的默认界面风格，我们马上就要改掉它，因为它不够快捷、直接。如果你曾有过开发经验，在一些开发 IDE 编辑器里写过代码，那 obsidian 的使用体验会让你如鱼得水，如果你没有开发经验也无妨，因为 obsidian 脱胎于 markdown 文本编辑器，你只要写过 markdown 文件，obsidian 对你就是零门槛。即便你也从未用过 markdown 文件，也没关系，obsidian 的操作及其简单，你只需要对着范例稍微熟悉最多半小时就可以轻松上手。点开范例随时开启 obsidian 之旅：点击 help，在 how to 下面点击 Format your note，看懂这一篇，在你自己的笔记里把内容练习一遍 obsidian 就可以上手了。

![](/img/user/Z-attach/4565391-c8cc1bc81a2530b8.png) obsidian 的帮助系统

**Step2：修改配置文件 obsidian.css**

      obsidian 好不好用很大程度上取决于它的界面风格和交互体验，所以我们首先就来需改它的界面风格，在 obsidian 的安装目录下，把 obsidian.css 文件替换掉，需要三个步骤：第一，打开配置 css 的开关选项，如下图所示：

![](/img/user/Z-attach/4565391-a088a3fa64598eb8.png) 打开 css 配置开关

第二，下载 obsidian.css 文件，链接：https://pan.baidu.com/s/1e9g4rjU_NRxa-VkRADt-hg 密码: pdht

第三，把下载的文件复制到如下目录中，替换掉原有文件：

cc

![](/img/user/Z-attach/4565391-8b4280eae344a8b4.png) obsidian 的目录

替换之后的效果如下图：

![](/img/user/Z-attach/4565391-878e703a23bd2ab5.png) css 文件更新后的 obsidian 风格效果

**Step3：创建目录树**

      这个每个人理解不同，习惯也不同，不做要求，但目录结构尽可能清晰一些，好的分类结构可以提升后续直接点击目录树查找文件的速度。不过这对 obsidian 来说不太重要，obsidian 的关联方式更多依赖前后向链接来做更精细的思维链接。所以这里不多强调，这是目录结构是大多数人做传统笔记的一个要点而已。我把自己的目录树放到这里，仅做参考吧：

![](/img/user/Z-attach/4565391-2dd718a3d06e96bc.png) 我的 obsidian 目录树结构

**Step4：效率倍增的模板设置**

      到了比较重要的地方了——模板设置。平时要做的事情都可以通过模板设置来加速，比如每日笔记、每天的任务速记、每天的灵感卡片等，还有一些微操作，比如插入时间戳、块引用命名、文章内锚点设置等高频操作。配置好了，可以起到事半功倍的效果，用起来特别爽。我把我最近最常用的模板附上，大家可以参考，别忘了需要先指定好模板文件所在的文件夹位置，我的叫 template 文件夹，然后把模板 ***.md 复制到 template 文件夹下面。

      比如，任务模板效果如下：

![](/img/user/Z-attach/4565391-ba4ff149cf43182b.png) 任务模板效果

      配合上每日笔记模板，就可以实现每日日程安排和时间要点记录的效果，而且可以通过搜索快速定位到到你想要的任何一个每日笔记里，效果非常赞，可以瞬间秒杀几乎所有时间日程管理软件。

![](/img/user/Z-attach/4565391-73be29c9b945b575.png) 每日日程模板效果

      把上面两者结合的效果如下：

![](/img/user/Z-attach/4565391-ec98c5e085a9941b.png) 每日日程和事件跟踪记录

      类似的模板可以制作很多，根据自己的需要来制作，效率会提升很多。

**Step5：Zettelkasten 模板即 ZK 模板**

      这个要结合 ZK 方法论来设计，我们另外写一篇专门介绍，它最重要的就是滑箱系统的设计，这也是 obsidian 的精髓所在。

**Step6：快捷键设置**

      所有重要操作最好都要设置好快捷键，这样操作的效率又会有成倍的提升。常见的快捷键设置如下：

![](/img/user/Z-attach/4565391-2b16e859fd51322c.png) obsidian 快捷键一览（1） ![](/img/user/Z-attach/4565391-d64ba9f5bba6d071.png) obsidian 快捷键一览（2） ![](/img/user/Z-attach/4565391-abfdea62aeb6fb05.png) obsidian 快捷键一览（3） ![](/img/user/Z-attach/4565391-3f912681e66777bd.png) obsidian 快捷键一览（4） ![](/img/user/Z-attach/4565391-d5ba311b00782a74.png) obsidian 快捷键一览（5）

**Step7：帮助与参考资料**

      掌握上面的操作，obsidian 就可以快速帮助到你的工作和生活，但要想发挥这个大杀器的威力，还需要自己持续摸索和学习实践，以下资料是目前网络上比较好的学习素材，分享给大家：

（1）obsidian 中文帮助网站：[obsidian 中文帮助网站](https://links.jianshu.com/go?to=https%3A%2F%2Fpublish.obsidian.md%2Fchinesehelp%2F00%2B%25E6%2596%25B0%25E6%2589%258B%25E5%2585%25A5%25E9%2597%25A8%2F00%2B%25E6%2596%25B0%25E6%2589%258B%25E5%2585%25A5%25E9%2597%25A8)

（2）ZK 方法论书籍：《how to take smart note 中文版》[学习骇客的翻译版](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.ershicimi.com%2Fp%2F0c79b26c2717b3dd55b4ed8f30607625)

（3）youtube 学习 obsidian 的优秀视频：[youtube 上的 obsidan 资源](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3Dr7WbgQeZnk4)

https://www.youtube.com/watch?v=r7WbgQeZnk4

      obsidian 应该说是一个跨时代的笔记产品，我还会写几篇文章继续介绍 obsidian 的最佳实践，希望大家支持喜欢，大家的支持也是我写作的动力！