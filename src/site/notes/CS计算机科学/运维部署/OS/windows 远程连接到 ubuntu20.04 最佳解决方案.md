---
{"dg-publish":true,"permalink":"/CS计算机科学/运维部署/OS/windows 远程连接到 ubuntu20.04 最佳解决方案/","noteIcon":"","created":"2024-06-22T22:30:29.022+08:00","updated":"2024-04-27T01:21:44.000+08:00"}
---

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/jaycethanks/p/14988794.html)

> 环境：  
> windows:20H2  
> ubuntu:20.04

### 1. 外网远程连接

目前已知外网访问最佳体验是通过向日葵远程访问；  
这里简单说明向日葵的安装：

**Step1: 下载 .deb 向日葵软件包**

到官网下载最新包 [传送门](https://sunlogin.oray.com/download/)

**Step2 : 授予执行权限，并执行安装**

```
# xxxxx.deb 包所在路径下sudo 权限执行$ chmod 777 xxxxx.deb$ dpkg -i xxxxx.deb
```

**Step4 :** 到这里就安装好了，这里主要是为了说明你可能会遇到的问题，“windows 远程连接后立即断开” , 解决方案如下：

```
$ sudo apt-get install lightdm
```

然后根据命令行图形界面引导提示选择 lightdm

> 注: lightdm 是一款桌面管理程序，和 gdm3 是一样的。 ubuntu20.04 默认的 gdm3 在运行花生壳时可能存在兼容性问题，所以换成 lightdm。

**Step5 : ** 重启电脑

```
$ systemctl reboot -i# 或者reboot, 如果遇到问题自己通过GUI图形界面重启。
```

Over， 此时应该就可以通过 windows 远程上去了。

> 初次使用向日葵，请自行百度使用教程。

### 2. 局域网连接

> 局域网的连接方案，不推荐使用 svn 的方式连接，windows vnc 连接 ubuntu 体验不好。安装也麻烦，容易遇到奇怪问题。压缩图传质量非常差。 所以这里采用 windows 下 RDP 的连接方式。

**Step1 : Ubuntu 下安装 Xrdp 服务**

```
$ sudo apt install xrdp
```

**Step2 : 通过 systemd 启动 xrdp 服务**

```
$ sudo systemctl enable --now xrdp
```

**Step3 : 开启远程访问 3389 端口白名单授予远程访问权限**

```
$ sudo ufw allow from any to any port 3389 proto tcp
```

**Step4 : 使用 windows 远程 RDP 连接试一试**

1.  回到 windows 桌面 : Win + D;
    
2.  运行指令 ： Win + R;
    
3.  启动远程访问程序，输入： `mstsc` + 回车
    
4.  输入 ubuntu 的局域网 ip 地址
    
    ```
    $ ip a s #linux ip 查看指令
    ```
    

    ![](/img/user/Z-attach/1735896-20210709005855839-1480958790.png)

    

    ![](/img/user/Z-attach/1735896-20210709005855619-1962313955.png)

    

    点击连接

    

    ![](/img/user/Z-attach/1735896-20210709005855419-106166450.png)

    

    是

    

    ![](/img/user/Z-attach/1735896-20210709005855218-2069211210.png)

    

    输入 root 账户

    

    ![](/img/user/Z-attach/1735896-20210709005854903-765060279.png)

    

    成功连接！

    
5.  此时就能够成功连接了， 而且非常的清晰， 但是有个问题，就是进行图形化操作的时候，会非常卡， 但是只是使用 terminal 几乎没有什么影响。 所以，这里不推荐使用 windows 自带的远程工具连接。 推荐使用 MobaXterm ， 以下是简单的使用教程。
    

### MobaXterm 的安装与 RDP 远程连接 Ubuntu

**Step1 . 下载并安装**

[官网传送门](https://mobaxterm.mobatek.net/download.html)

[社区版家庭安装版下载直链](https://download.mobatek.net/2122021051924233/MobaXterm_Installer_v21.2.zip)

> 如果比较卡的话，可能需要外网环境。此时建议自行百度找安装包。

安装：略

**Step2 . 使用说明**

![](/img/user/Z-attach/1735896-20210709005854586-1013508831.png)

![](/img/user/Z-attach/1735896-20210709005854359-29747664.png)

![](/img/user/Z-attach/1735896-20210709005854125-1677754279.png)

![](/img/user/Z-attach/1735896-20210709005853760-18688276.png)

Over

![](/img/user/Z-attach/1735896-20210709005853317-207298469.png)

**更多**  
额外的, 如果你想通过外网 DRP 远程 windows to windows 可以参考这里：  
[外网远程访问 windows to windows](https://www.cnblogs.com/jaycethanks/p/15084454.html)