---
{"dg-publish":true,"permalink":"/P生产力/OS/Linux_Desktop/在 Ubuntu 安装配置 Fcitx 5 中文输入法/","noteIcon":"","created":"2024-06-29T00:48:53.000+08:00","updated":"2024-06-30T00:45:59.000+08:00"}
---

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [muzing.top](https://muzing.top/posts/3fc249cf/)

> 搞定 Linux 桌面环境下中文输入

[](#前言)前言[](#前言)
----------------

书接上回，一时兴起将主力机的 Ubuntu 20.04 LTS 升级至了刚刚发布的 22.04 LTS。从 `X` 切换到 [`Wayland`](https://wayland.freedesktop.org/)、[GNOME](https://www.gnome.org/) 从 3.36 升级至 42、Python 默认为 3.10 等等…… 使用太新的软件包反而暂时带来了麻烦，部分原有的软件和插件都不可用了。这其中就包括已经很久没有更新的[百度输入法](https://srf.baidu.com/site/guanwang_linux/index.html)。故需要寻找新的中文拼音输入法。经简单浏览对比，选择了 [Fcitx 5](https://fcitx-im.org/wiki/Fcitx_5/zh-cn)。

[![](/img/user/Z-attach/fcitx_logo.png)](https://oss.muzing.top/image/fcitx_logo.png "小企鹅输入法")

小企鹅输入法

[](#安装)安装[](#安装)
----------------

### [](#检查系统中文环境)检查系统中文环境[](#检查系统中文环境)

在 Ubuntu 设置中打开「区域与语言」—— 「管理已安装的语言」，然后会自动检查已安装语言是否完整。若不完整，根据提示安装即可。

[![](/img/user/Z-attach/20220503150712.png)](https://oss.muzing.top/image/20220503150712.png "检查可用的语言支持")

检查可用的语言支持

### [](#最小安装)最小安装[](#最小安装)

为使用 Fcitx 5，需要安装三部分基本内容：

1.  Fcitx 5 主程序
2.  中文输入法引擎
3.  图形界面相关

按照这个思路，可以直接使用 `apt` 进行安装：

```
sudo apt install fcitx5 \
fcitx5-chinese-addons \
fcitx5-frontend-gtk4 fcitx5-frontend-gtk3 fcitx5-frontend-gtk2 \
fcitx5-frontend-qt5


SHELL
```

### [](#安装中文词库)安装中文词库[](#安装中文词库)

在 GitHub 打开[维基百科中文拼音词库](https://github.com/felixonmars/fcitx5-pinyin-zhwiki)的 [Releases](https://github.com/felixonmars/fcitx5-pinyin-zhwiki/releases) 界面，下载最新版的 `.dict` 文件。按照 README 的指导，将其复制到 `~/.local/share/fcitx5/pinyin/dictionaries/` 文件夹下即可。

```
# 下载词库文件
wget https://github.com/felixonmars/fcitx5-pinyin-zhwiki/releases/download/0.2.4/zhwiki-20220416.dict
# 创建存储目录
mkdir -p ~/.local/share/fcitx5/pinyin/dictionaries/
# 移动词库文件至该目录
mv zhwiki-20220416.dict ~/.local/share/fcitx5/pinyin/dictionaries/


SHELL
```

[](#配置)配置[](#配置)
----------------

### [](#设置为默认输入法)设置为默认输入法[](#设置为默认输入法)

使用 im-config 工具可以配置首选输入法，在任意命令行输入：

```
im-config


SHELL
```

根据弹出窗口的提示，将首选输入法设置为 Fcitx 5 即可。

### [](#环境变量)环境变量[](#环境变量)

需要为桌面会话设置环境变量，即将以下配置项写入某一配置文件中：

```
export XMODIFIERS=@im=fcitx
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx


SHELL
```

如果使用 Bash 作为 shell，则建议写入至 `~/.bash_profile`，这样只对当前用户生效，而不影响其他用户。

另一个可以写入此配置的文件为系统级的 `/etc/profile`。

[![](/img/user/Z-attach/20220503154637.png)](https://oss.muzing.top/image/20220503154637.png "将配置写入到 /etc/profile 文件末尾")

将配置写入到 /etc/profile 文件末尾

### [](#开机自启动)开机自启动[](#开机自启动)

安装 Fcitx 5 后并没有自动添加到开机自启动中，每次开机后需要手动在应用程序中找到并启动，非常繁琐。

解决方案非常简单，在 Tweaks（`sudo apt install gnome-tweaks`）中将 Fcitx 5 添加到「开机启动程序」列表中即可。

[![](/img/user/Z-attach/20220502211650.png)](https://oss.muzing.top/image/20220502211650.png "将 Fcitx 5 添加到开机启动程序列表中")

将 Fcitx 5 添加到开机启动程序列表中

### [](#fcitx-配置)Fcitx 配置[](#fcitx-配置)

Fcitx 5 提供了一个基于 Qt 的强大易用的 GUI 配置工具，可以对输入法功能进行配置。有多种启动该配置工具的方法：

1.  在应用程序列表中打开「Fcitx 配置」
2.  在 Fcitx 托盘上右键打开「设置」
3.  命令行命令 `fcitx5-configtool`

根据个人偏好进行设置即可。需要注意的是「输入法」标签页下，应将「键盘 - 英语」放在首位，拼音（或其他中文输入法）放在后面的位置。

[![](/img/user/Z-attach/20220503150346.png)](https://oss.muzing.top/image/20220503150346.png "Fcitx5 configtool")

Fcitx5 configtool

### [](#自定义主题)自定义主题[](#自定义主题)

Fcitx 5 默认的外观比较朴素，用户可以根据喜好使用自定义主题。

第一种方式为使用[经典用户界面](https://fcitx-im.org/wiki/Theme_Customization/zh-cn#%E7%BB%8F%E5%85%B8%E7%94%A8%E6%88%B7%E7%95%8C%E9%9D%A2)，可以[在 GitHub 搜索主题](https://github.com/search?q=fcitx5+theme&type=Repositories)，然后在 [Fcitx5 configtool](#fcitx-%E9%85%8D%E7%BD%AE) —— 「附加组件」 —— 「经典用户界面」中设置即可。

第二种方式为使用 [Kim 面板](https://fcitx-im.org/wiki/Theme_Customization/zh-cn#kim%E9%9D%A2%E6%9D%BF)，一种基于 DBus 接口的用户界面。此处安装了 [Input Method Panel](https://extensions.gnome.org/extension/261/kimpanel/) 这个 GNOME 扩展，黑色的风格与正在使用的 GNOME 主题 [Orchis-dark](https://www.gnome-look.org/p/1357889) 非常搭配。

[![](/img/user/Z-attach/20220503125515.png)](https://oss.muzing.top/image/20220503125515.png "Input Method Panel 效果")

Input Method Panel 效果

[](#已知问题)已知问题[](#已知问题)
----------------------

### [](#修复-jetbrains-系-ide-显示问题)修复 JetBrains 系 IDE 显示问题[](#修复-jetbrains-系-ide-显示问题)

在 JetBrains 系 IDE（如 PyCharm）中，输入法选择框的位置始终固定于屏幕左下角，而非随输入光标移动，在中文输入很不方便。该问题为 IDE 的 [JetBrainsRuntime](https://github.com/JetBrains/JetBrainsRuntime) 缺陷所致。可尝试使用 [RikudouPatrickstar/JetBrainsRuntime-for-Linux-x64](https://github.com/RikudouPatrickstar/JetBrainsRuntime-for-Linux-x64) 这个仓库[发布](https://github.com/RikudouPatrickstar/JetBrainsRuntime-for-Linux-x64/releases)的 JBR 文件解决。

### [](#卸载-ibus-影响-fcitx-5-正常使用)卸载 iBus 影响 Fcitx 5 正常使用[](#卸载-ibus-影响-fcitx-5-正常使用)

出于精简空间和减少冲突干扰之考虑，使用 `sudo apt remove ibus` 卸载了 iBus，但重启（使生效）之后发现 Fcitx 5 受到了影响。具体表现为：除在终端中之外，其他输入场景无法切换至中文输入。使用 `apt` 装回 iBus，再次重启即又恢复正常。

检查包依赖关系，卸载 ibus 包后会自动移除 ibus-data、ibus-gtk4、python3-ibus-1.0 三个包，似乎都只是与 iBus 紧密联系的。暂为不解之谜。

> 本文的[知乎发布版](https://zhuanlan.zhihu.com/p/508797663)评论区有关于此话题的讨论，其中有可能的解决方案，供参考。

[](#参考)参考[](#参考)
----------------

[官方 Wiki —— 安装 Fcitx5](https://fcitx-im.org/wiki/Install_Fcitx_5/zh-cn)

[官方 Wiki —— Setup Fcitx 5](https://fcitx-im.org/wiki/Setup_Fcitx_5)

[官方 Wiki —— 定制主题](https://fcitx-im.org/wiki/Theme_Customization/zh-cn)

[Ubuntu20.04 安装 fcitx5 输入法](https://ouyen.github.io/fcitx5-ubuntu/)

[](#碎碎念)碎碎念[](#碎碎念)
-------------------

使用 Linux 桌面作为日常主力操作系统真的是一件有趣，但也很可能复杂而折腾的事情。对于 Windows 用户，既不会遇到升级操作系统就出现许多软件不可用的问题；又可以简简单单通过一个安装包搞定中文输入法等常用软件。但以 Ubuntu 为代表的 Linux 发行版的许多特性（尤其是强大无所不能的命令行）深深吸引着我，每次折腾时逐渐学到新知识、解决问题又充满了快乐，所以还会继续使用。

本次折腾中的几点感慨：

1.  谨慎缓慢更新：像主力机操作系统这样重要的东西，应至少等待数月再跟进
2.  善用官方文档，中文博客仅供参考：每台机器的环境都有所不同，每个用户的喜好也有差异，其他人的安装记录帖可以参考（包括本文），但还是要以官方文档（大概率是英文，阅读稍累一些）为准

5 月 17 日更新：由于 Ubuntu 22.04 的诸多问题已经严重影响了日常使用，目前已将主力系统切换至 Arch Linux。

2023 年 2 月更新：纠正 apt 安装列表中的错误。