---
{"dg-publish":true,"permalink":"/CS计算机科学/运维部署/OS/CentOS 7 升级 git 版本到 2.x/","noteIcon":"","created":"2024-06-22T22:30:29.006+08:00","updated":"2024-04-27T01:21:44.000+08:00"}
---

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7071910670056292389)

CentOS 7 升级 git 版本
==================

首先确认在 `CentOS 7` 服务器已安装的 `git` 版本。

```
git --version
```

```
git version 1.8.3.1
```

使用 yum 安装
---------

### 移除旧版本

```
sudo yum remove git

sudo yum remove git-*
```

### 添加 End Point Package Repository(存储库)

在 `CentOS 7` 上安装新版本 `Git` 最快的方法是通过 `End Point` 库。

```
sudo yum install https://packages.endpointdev.com/rhel/7/os/x86_64/endpoint-repo.x86_64.rpm
```

其他版本的 `CentOS` 替换成对应版本安装，可参考：[packages.endpointdev.com/](https://link.juejin.cn?target=https%3A%2F%2Fpackages.endpointdev.com%2F "https://packages.endpointdev.com/")

### 安装

添加了 `end point` 库，就可以直接安装 `git 2.x`

```
sudo yum install git
```

安装完成之后，检查 `git` 版本：

```
git --version
```

```
git version 2.34.1
```

编译安装
----

### 移除旧版本

```
sudo yum remove git

sudo yum remove git-*
```

### 安装必需依赖

具体依赖说明可以参考 `github` 库：[github.com/git/git/blo…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fgit%2Fgit%2Fblob%2Fmaster%2FINSTALL "https://github.com/git/git/blob/master/INSTALL")

安装过程可能需要 `root` 权限：

```
sudo yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel

sudo yum install gcc perl-ExtUtils-MakeMaker
```

### 下载源码

如果没有 `wget` 需要安装：

```
sudo yum -y install wget
```

进入保存下载文件的目录，下载源码：

```
# 进入下载文件保存目录，根据实际情况替换
cd ~/downloads

wget https://github.com/git/git/archive/v2.34.1.tar.gz
```

> 这里可以替换成我们想要下载的版本，只需将 `v2.34.1` 改成对应的版本号即可。

```
export VERSION="2.34.1"

wget https://github.com/git/git/archive/v${VERSION}.tar.gz
```

具体版本可以看 `git` 代码库中的 `tags`:

![](/img/user/Z-attach/0ca4e24ebe25483ab3c00eb2ff9d3fec~tplv-k3u1fbpfcp-zoom-in-crop-mark!1512!0!0!0.awebp.webp)

### 解压安装

解压之后，删除压缩包，并进入代码目录：

```
tar -xvf v2.34.1.tar.gz

rm -f v2.34.1.tar.gz

cd git-2.34.1
```

然后编译安装：

```
make configure

sudo ./configure --prefix=/usr

sudo make

sudo make install
```

安装完成之后，检查 `git` 版本：

```
$ git --version
```

```
git version 2.34.1
```

可能遇到的依赖问题
---------

问题：`fatal error: expat.h: No such file or directory`

解决办法：

```
yum install expat-devel
```

问题：`fatal error: expat.h: No such file or directory`

解决办法：

```
yum install libcurl-dev libcurl-devel
```

参考资料
----

1.  [github.com/git/git](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fgit%2Fgit "https://github.com/git/git")
    
2.  [github.com/git/git/blo…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fgit%2Fgit%2Fblob%2Fmaster%2FINSTALL "https://github.com/git/git/blob/master/INSTALL")
    
3.  [how-to-install-latest-version-of-git-on-centos-8-x-7-x-6-x](https://link.juejin.cn?target=https%3A%2F%2Fstackoverflow.com%2Fquestions%2F21820715%2Fhow-to-install-latest-version-of-git-on-centos-8-x-7-x-6-x "https://stackoverflow.com/questions/21820715/how-to-install-latest-version-of-git-on-centos-8-x-7-x-6-x")
    
4.  [how-to-install-latest-version-of-git-git-2-x-on-centos-7](https://link.juejin.cn?target=https%3A%2F%2Fcomputingforgeeks.com%2Fhow-to-install-latest-version-of-git-git-2-x-on-centos-7%2F "https://computingforgeeks.com/how-to-install-latest-version-of-git-git-2-x-on-centos-7/")