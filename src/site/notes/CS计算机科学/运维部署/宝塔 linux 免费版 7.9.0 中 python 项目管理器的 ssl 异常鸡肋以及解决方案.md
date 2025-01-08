---
{"dg-publish":true,"permalink":"/CS计算机科学/运维部署/宝塔 linux 免费版 7.9.0 中 python 项目管理器的 ssl 异常鸡肋以及解决方案/","noteIcon":"","created":"2024-04-11T20:33:18.570+08:00","updated":"2024-04-27T01:21:43.000+08:00"}
---

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.20m8.com](http://www.20m8.com/161.html)

> 要真正把宝塔的 python 项目管理器使用起来，需要经历一番复杂的操作才可以，否则这个东西就是个鸡肋。

        20220402 按照我的思路安装完会出现旧版本的 php 中 openssl 跟实际 openssl 版本不符的问题，需要重新编译 php，在编译的时候手动添加模块 --with-openssl=/usr/local/openssl111 才行。最近站长才发现其实新版本的宝塔是自带 openssl1.1.1 的，路径在 / usr/local/openssl111（已知宝塔 7.9.8 以其之后的版本就没有这个文件夹了，centos 的可以通过 [https://download.csdn.net/download/hehu158/87712580](https://download.csdn.net/download/hehu158/87712580) 下载），默认 1.0.2 的安装路径在 / usr/local/openssl，所以无需升级 openssl，只要重新编译 php 或者 python，指定 openssl 路径即可，也经过站长的测试确实可行，不过通过测试重新编译指定 openssl 路径是可以实现 php7.4\php7.1 的内置 openssl1.1.1 的。默认安装完宝塔是没有 openssl111 跟 openssl 文件夹的，需要编译安装一个 php8.0 及以上的 php 才会有的。

        首先要先在 python 项目管理器中安装一次 python3.10，然后下载官方的 python 源码到 / root 路径解压，执行如下命令指定 openssl111，这样完全可以不用升级默认的 openssl。  

        pushd Python-3.10.2

        ./configure -C --with-openssl=/usr/local/openssl111 --with-openssl-rpath=auto --prefix=/www/server/python_manager/versions/3.10.0 

        make -j8

        make altinstall

 **非官方教程，目前测试发现的问题如下，php8 以下版本，使用 curl 请求 https 会出现 502 Bad Gateway nginx，默认 php7 都是内置 openssl1.0.0 的，升级 openssl1.1.1 后，查看 phpinfo，出现的 openssl 版本不一致，所以导致 curl 请求 https 会失效，php8 内置的是 openssl1.1.1 的，所以不受影响。如介意请勿操作升级。可以实现看下宝塔官网团队针对 openssl 升级问题的回答：[https://www.bt.cn/bbs/forum.php?mod=viewthread&tid=79935&highlight=openssl](https://www.bt.cn/bbs/forum.php?mod=viewthread&tid=79935&highlight=openssl)，如下思路大家就不用看了哈。**

[![](/img/user/Z-attach/1646701662742766.png)](http://www.20m8.com/ueditor/php/upload/image/20220308/1646701662742766.png "1646701662742766.png")

        通过 python 项目管理器安装的 python，不管你升级没升级 openssl，他都是不会带 openssl。不装这个东西，sanic 跑不起来，通过 pip install xxx 模块基本都是失败的，因为现在的镜像源地址都是 https 的，没有这个东西，安装个模块需要这么写，还是要通过命令行来安装，那么这个 python 项目管理器就是个看看的，啥也用不了了，在 windows 上写好的代码想在自己服务器上跑一跑都不行，这个真的很打击人的，如果要测试环境，建议大家还是搞个虚拟机先玩一玩，别用正式的服务器。站长是一步一步测试了好几轮，建议将 sqlite3、openssl、python3.10.2 源码包下载过来上传到 root 目录，就是~ 目录下，进入每个目录执行一边 chmod 777 * 获取下所有权限。

        虚拟 python 的环境路径 / python -m pip install django -i [](http://pypi.douban.com/simple)[http://mirrors.aliyun.com/pypi/simple/](http://mirrors.aliyun.com/pypi/simple/)[](http://pypi.douban.com/simple) --trusted-host [mirrors.aliyun.com](http://mirrors.aliyun.com/pypi/simple/)

没有安装 openssl 时会报错

通过 python 管理器中的模块添加基本都是会提示安装失败的，这个东西也感觉不是很友好，有的高一点的 python 版本错误日志还没有，搞的人一头雾水。

测试支不支持 opensll 的方法很简单，命令行下执行 / www/server/python_manager/versions/3.10.0/bin/python3

然后输入 import ssl 回车

Traceback (most recent call last):

  File "

  File "/www/server/python_manager/versions/3.10.0/lib/python3.10/ssl.py", line 98, in

    import _ssl             # if we can't import it, let the error propagate

ModuleNotFoundError: No module named '_ssl'

centos7 宝塔安装完默认的 openssl 是 OpenSSL 1.0.2k-fips  26 Jan 2017，官网 [https://www.openssl.org/](https://www.openssl.org/) 上最新的是 3.x（这个还没仔细研究过，没安装成功），1.1.1m 版本的仅支持到 2023 年

下面给大家介绍下站长具体的解决方案：

1.  安装宝塔；
    
2.  升级 sqlite3，如果没有用到的话就算了，这个倒还是挺好的，你升级完再通过 python 项目管理器安装还是会将最版本的 sqlite3 带进来的；
    
3.  升级 openssl 到 1.1.1m，这个通过 python 项目管理器是带不动的；
    
4.  通过 python 项目管理器安装最高版本好了；
    
5.  最后偷梁换柱，从 python 官网下载源码对 python 重新编译重新安装，覆盖到 / www/server/python_manager/versions/3.10.0，最后你再使用 python 项目管理器你就会觉得真香了哈。参考网址 https://docs.python.org/zh-cn/3.10/using/unix.html#on-linux
    

[![](/img/user/Z-attach/1646705571373179.png)](http://www.20m8.com/ueditor/php/upload/image/20220308/1646705571373179.png "1646705571373179.png")