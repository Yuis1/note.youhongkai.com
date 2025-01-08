---
{"dg-publish":true,"permalink":"/CS计算机科学/运维部署/Network/Cloudflare Tunnel 免费内网穿透/","noteIcon":"","created":"2024-02-03T17:47:59.631+08:00","updated":"2024-04-30T00:40:26.527+08:00"}
---

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [ednovas.xyz](https://ednovas.xyz/2023/02/24/cloudflaretunnel/#%E5%86%99%E5%9C%A8%E6%9C%80%E5%90%8E)

> Cloudflare Tunnelcloudflare 提供的免费加密通道，通过 cloudflare 的 cnd 访问部署在内网的服务，比如 web,ssh,smb,tcp 等。


[](#Cloudflare-Tunnel "Cloudflare Tunnel")Cloudflare Tunnel
-----------------------------------------------------------

cloudflare 提供的免费加密通道，通过 cloudflare 的 cnd 访问部署在内网的服务，比如 web,ssh,smb,tcp 等。

[](#准备 "准备")准备
--------------

那肯定必须需要一个 cloudflare 账号，以及开通 cloudflare zero trust。

cloudflare：[https://www.cloudflare.com/](https://www.cloudflare.com/)  
zero trust: [https://one.dash.cloudflare.com/](https://one.dash.cloudflare.com/)

[](#安装 "安装")安装
--------------

在你的需要内网穿透的设备上安装 cloudflared（cf 的本地 cli 客户端），这里博主以 raspberry pi 4 作为例子。

由于 raspberry pi 4 是 arm64 的，所以需要下载 arm 架构的文件：

```
curl -L 'https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64' -o /usr/bin/cloudflared
chmod +x /usr/bin/cloudflared
```

如果是普通 amd64 架构的，使用以下命令：

```
curl -L 'https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64' -o /usr/bin/cloudflared
chmod +x /usr/bin/cloudflared
```

然后登录 cloudflared

```
cloudflared tunnel login
```

终端会给出一个类似 `https://dash.cloudflare.com/argotunnel?callback=https%3A%2F%2Flogin.cloudflareaccess.org%xxxxxxxxxxxxxxxxxxxxxxx` 的登陆地址，复制黏贴该地址到浏览器中，然后按照要求登录即可。

如果是有二步验证的，需要登录了以后，再次黏贴该地址，因为二步验证结束后并不会自动跳转到该 cloudflared 验证登录页面。

授权的话，每次只能选择一个网站，当然，一个网站也就够用了，我们可以解析很多三级域名。

当然如果需要授权多个网站，授权完成后不要关闭网页！依次点击其他需要授权的域名，进行多次授权。

[](#创建隧道 "创建隧道")创建隧道
--------------------

授权完成后，就可以创建我们需要的隧道了。cloudflare 提供两种方式，一种是在 zero trust 页面用 web 添加授权，也可以在本地服务器（raspberry）上用 config 文件添加，这里用的是 config 文件添加，因为本地 config 感觉更方便操作一些。

然后开始创建隧道，建议一个服务器就创建一个隧道。

```
cloudflared tunnel create <你的隧道名字>
```

比如

```
cloudflared tunnel create ednovas
```

创建完以后，会把信息保存到 root 下的 .cloudflared 文件夹下，注意不要误删了。然后终端会输出如下的内容，隧道的 UUID（下图中的示例 12345-123-123-123-12345 就是 UUID），一定要记录下来，这个 UUID 后面配置的时候会经常用到。

```
Tunnel credentials written to /root/.cloudflared/12345-123-123-123-12345.json. cloudflared chose this file based on where your origin certificate was found. Keep this file secret. To revoke these credentials, delete the tunnel.

Created tunnel webserver-1 with id 12345-123-123-123-12345
```

[](#创建域名CNAME "创建域名CNAME")创建域名 CNAME
------------------------------------

然后我们就可以创建内网转发域名了。这里可以先添加一个域名

```
cloudflared tunnel route dns <隧道名字> <域名>
```

比如

```
cloudflared tunnel route dns ednovas 80.ednovas.xyz
```

然后 cloudflare 就会在你的域名下自动生成一个 CNAME，把 80.ednovas.xyz 指向 <隧道 UUID>.cfargotunnel.com

以此类推，我们可以如法炮制在面板上添加其他的需要的 CNAME

注意：如果之前添加的有多个不同的二级域名，比如 `ednovas.xyz` 和 `ednovas.blog`，只会对第一个添加的 `ednovas.xyz` 生效，后面的 `ednovas.blog` 域名下的 CNAME 记录需要自行去网站上手动添加了。当然如果只有 `ednovas.xyz` 一个域名就不影响了，可以依旧使用类似以下的命令来创建更多需要的域名解析（我是每个端口服务一个域名）。

```
cloudflared tunnel route dns ednovas ssh.ednovas.xyz
```

[![](/img/user/Z-attach/Snipaste_2023-02-24_03-24-23.png)](https://cdn.jsdelivr.net/gh/ednovas/CDN/New%20folder/Snipaste_2023-02-24_03-24-23.png)

[](#配置Config文件 "配置Config文件")配置 Config 文件
----------------------------------------

添加好想要穿透的域名后，我们就可以开始映射了。

```
vim ~/.cloudflared/config.yml
```

然后 i 输入内容，黏贴修改后的如下配置文件（自行按需修改）

```
tunnel: <隧道UUID>
credentials-file: /root/.cloudflared/<隧道UUID>.json
protocol: h2mux
ingress:
  
  - hostname: <域名1.com>
    service: http://localhost:80
  
  - hostname: <域名2.com>
    service: https://127.0.0.1:443
    originRequest:
      noTLSVerify: true
      originServerName: <域名2.com>
  
  - hostname: <*.域名3.com>
    service: http://localhost:8012
  
  - hostname: <mysql.域名4.com>
    service: unix:/tmp/mysql.sock
  
  - hostname: <ssh.域名5.com>
    service: ssh://localhost:22
  - service: http_status:404
```

需要把 <隧道 UUID> 替换成之前保存的 UUID，域名就是上一步解析的几个 CNAME 域名，比如刚刚解析的 `80.ednovas.xyz`，下面的地址，保持 localhost 不要更改，更改后面的端口。比如想要映射本地的 80 端口服务到 80.ednovas.xyz 域名，此外还需要开启 22 的 ssh 端口（需要上一步就已经把 `ssh.ednovas.xyz` 解析好），就可以这么修改

```
tunnel: <隧道UUID>
credentials-file: /root/.cloudflared/<隧道UUID>.json
protocol: h2mux
ingress:
  - hostname: 80.ednovas.xyz
    service: http://localhost:80
  - hostname: ssh.ednovas.xyz
    service: ssh://localhost:22
  - service: http_status:404
```

注意：最后的 `- service: http_status:404` 一定要加，这是兜底的规则。还有复制黏贴的时候确保格式没有错误，很可能会发生 mapping error。

如果需要更多的端口 / 协议支持，可以查看 cloudflare 官方文档：  
[https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/local/local-management/ingress/#supported-protocols](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/local/local-management/ingress/#supported-protocols)

配置完以后，可以测试下配置文件是否有问题。

```
cloudflared tunnel ingress validate
```

还可以测试规则是否命中，如果说并未找到该规则，就说明配置文件可能哪里填错了，或者终端输入错误问题（域名前的 https 不可省略，否则会说 no hostname）。

```
cloudflared tunnel ingress rule https://80.ednovas.xyz
```

[](#测试 "测试")测试
--------------

可以运行以下命令进行测试：

```
cloudflared --loglevel debug --transport-loglevel warn --config ~/.cloudflared/config.yml tunnel run <隧道UUID>
```

终端会输出一大堆 log，只要没有红色报错就是正常的（退出 log 的时候 ctrl+c 报的俩红色 error 不算）。

登陆 Zero Trust 的控制台 [https://one.dash.cloudflare.com/](https://one.dash.cloudflare.com/)

左边选择 `Access`-`Tunnels`，如果正常的话，创建的隧道的状态会是 Active。

[](#创建systemctl "创建systemctl")创建 systemctl
------------------------------------------

用 service install 即可自动创建 systemctl 启动服务。

```
cloudflared service install
systemctl start cloudflared
systemctl enable cloudflared
systemctl status cloudflared
```

配置文件会被拷贝到 `/etc/cloudflared/config.yml`，并且从该目录下启动，所以如果你需要更改文件的话，需要在 `/root/.cloudflared` 下修改完 `config.yml` 文件后：

```
cp /root/.cloudflared/config.yml /etc/cloudflared/config.yml
```

覆盖一下，然后再

```
systemctl restart cloudflared
```

[](#SSH协议 "SSH协议")SSH 协议
------------------------

[https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/use_cases/ssh/](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/use_cases/ssh/)

这是 cloudflare 的关于 SSH 协议的官方说明，我们会发现要用 SSH 协议必须要在用户端也要安装软件，稍微麻烦点了。我们可以直接用 zero trust 提供的 `self-hosted application` 来创建一个网页 WEB 的 SSH 终端登录。

在 zero trust 的控制面板创建一个新的 application

[![](/img/user/Z-attach/Snipaste_2023-02-24_03-44-17.png)](https://cdn.jsdelivr.net/gh/ednovas/CDN/New%20folder/Snipaste_2023-02-24_03-44-17.png)

类型选 `self-hosted`

[![](/img/user/Z-attach/Snipaste_2023-02-24_03-45-03.png)](https://cdn.jsdelivr.net/gh/ednovas/CDN/New%20folder/Snipaste_2023-02-24_03-45-03.png)

以下是两个必须要填的内容。application name 就随意，session duration 的话，安全起见，30 分钟 - 1 天比较好。

[![](/img/user/Z-attach/Snipaste_2023-02-24_03-45-49.png)](https://cdn.jsdelivr.net/gh/ednovas/CDN/New%20folder/Snipaste_2023-02-24_03-45-49.png)

然后输入之前自己解析到 ssh 端口的内网穿透域名。

[![](/img/user/Z-attach/Snipaste_2023-02-24_03-47-34.png)](https://cdn.jsdelivr.net/gh/ednovas/CDN/New%20folder/Snipaste_2023-02-24_03-47-34.png)

Policy 页面，名字还是随意，action 选择默认的 allow，session duration 选择和应用的 session timeout 一样就行（默认）。下面的规则，建议是选择 email，并且输入你的 cloudflare 的 email 邮箱地址来保证安全。

[![](/img/user/Z-attach/Snipaste_2023-02-24_03-48-47.png)](https://cdn.jsdelivr.net/gh/ednovas/CDN/New%20folder/Snipaste_2023-02-24_03-48-47.png)

在最后一页的最下面把 browser rendering 设置成 SSH。并且开启 Enable automatic cloudflared authentication。

[![](/img/user/Z-attach/Snipaste_2023-02-24_03-51-22.png)](https://cdn.jsdelivr.net/gh/ednovas/CDN/New%20folder/Snipaste_2023-02-24_03-51-22.png)

然后此时再登录你的 SSH 域名，就可以直接访问页面端的 SSH 了，输入你的 SSH 用户名和密码即可访问（支持密钥登录）。

[![](/img/user/Z-attach/Snipaste_2023-02-24_03-54-07.png)](https://cdn.jsdelivr.net/gh/ednovas/CDN/New%20folder/Snipaste_2023-02-24_03-54-07.png)

[](#RDP "RDP")RDP
-----------------

自行翻阅官网文档，应用价值较小，不再赘述

[https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/use_cases/rdp/](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/use_cases/rdp/)

[](#SMB "SMB")SMB
-----------------

自行翻阅官网文档，应用价值较小，不再赘述

[https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/use_cases/smb/](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/use_cases/smb/)

[](#写在最后 "写在最后")写在最后
--------------------

cloudflare tunnel 是一个十分优秀的免易操作的内网穿透方案，优点很明显，相较于 FRP 等内网穿透配置简单，完全免费，功能也比较强大（这里只是介绍了最基础的 tunnel 使用方法）。缺点也很明显，用的是 cloudflare cdn，国内不友好。

> 参考 [https://bra.live/setup-home-server-with-cloudflare-tunnel/](https://bra.live/setup-home-server-with-cloudflare-tunnel/)