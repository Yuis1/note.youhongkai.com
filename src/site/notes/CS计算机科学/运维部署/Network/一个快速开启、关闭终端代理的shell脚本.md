---
{"dg-publish":true,"permalink":"/CS计算机科学/运维部署/Network/一个快速开启、关闭终端代理的shell脚本/","noteIcon":"","created":"2024-03-04T00:41:54.855+08:00","updated":"2024-04-24T00:36:26.704+08:00"}
---


作者：游鱼思

为了对终端也能用上代理（ [ssh命令走代理的方式](ssh命令走代理的方式.md) ），并快速设置，写了这个脚本。

---

步骤如下：

新建 set_proxy.sh，内容如下

`#!/bin/bash`

`export http_proxy="http://f128.kdltps.com:15818"`

`export https_proxy="https://f128.kdltps.com:15818"`

`export HTTP_PROXY="http://f128.kdltps.com:15818"`

`export HTTPS_PROXY="https://f128.kdltps.com:15818"`

`export http_proxy=socks5://localhost:1080`

`export https_proxy=socks5://localhost:1080``

`

注意：这里仅是示例。快代理境内代理只能访问境内服务器，境外代理只能由境外地址发起访问，反正就是不让翻墙。所以不能用于github、composer Drupal 源的加速。

如果是运行了docker，且docker需要通过宿主机代理，则应该在容器中设置：

`export http_proxy=socks5://host.docker.internal:1080`

`export https_proxy=socks5://host.docker.internal:1080`

Git的代理设置：

`git config --global http.proxy socks5://localhost:1080`

确认设置是否生效

`git config --global --get http.proxy`

坑来了，以下命令不对：

`sh set_proxy.sh`

`~/set_proxy.sh`

因为在Unix和Linux系统中，当你运行一个脚本时，它通常会在一个新的子shell中执行。这意味着在脚本中设置的环境变量仅在该子shell中有效，当脚本执行完毕，子shell关闭，这些变量设置就会消失。为了让环境变量设置在当前shell中生效，你需要使用source或.命令来执行脚本，而不是直接运行脚本。

正确的方式：

`source set_proxy.sh`

`. set_proxy.sh`

这时候再用 env 命令检查，发现已经添加到环境变量。

也可以用 `curl ipinfo.io` ，检查返回的是不是已经是代理的IP。

## Python 中使用系统代理

 "Could not install packages due to an OSError: Missing dependencies for SOCKS support" 通常出现在尝试使用支持 SOCKS 代理的 pip 命令时，但缺少必要的依赖项。这个问题通常可以通过在新终端安装 PySocks 解决。

`pip install PySocks`

注意：需要先关闭代理，或者在新的shell中安装，否则这个错误会阻塞安装。

## 阿里云通不了drupal的安装源

解决方案：

- 把阻塞的域名，映射到解析出来的其它IP上
- 用拨测工具[https://boce.aliyun.com/detect/ping](https://boce.aliyun.com/detect/ping?spm=5176.2020520104.0.0.30833f1boTVNpM)测试packages.drupal.org
- 在/etc/hosts文件中设置packages.drupal.org指向能ping通的IP
- 终极方案：只能用梯子，使用 V2Raya

