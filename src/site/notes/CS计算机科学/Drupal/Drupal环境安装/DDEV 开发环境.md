---
{"dg-publish":true,"permalink":"/CS计算机科学/Drupal/Drupal环境安装/DDEV 开发环境/","noteIcon":"","created":"2024-07-03T13:23:09.000+08:00","updated":"2024-07-05T17:18:50.000+08:00"}
---


作者：游鱼思

---
## DDEV介绍

ddev 是一个开源的开发工具，用于简化本地开发环境的设置和管理。它利用 Docker 容器技术来创建隔离的、可重复的开发环境，使得在本地机器上开发和部署复杂的 web 应用变得更加容易。以下是 ddev 的主要机制和功能介绍：

### 核心机制

[How DDEV Works - DDEV Docs](https://ddev.readthedocs.io/en/stable/users/usage/architecture/#container-architecture)

![](/img/user/Z-attach/Pasted image 20240704030650.png)

1. **Docker 容器化**：
    
    - ddev 使用 Docker 来创建独立的开发环境，每个项目都有自己的容器。这些容器可以运行不同版本的 PHP、数据库和其他服务，而不会相互干扰。
2. **简单的配置**：
    
    - ddev 通过简单的配置文件（如 `.ddev/config.yaml`）来定义项目的设置。这个文件包含了项目的基本信息，如项目名称、类型（Drupal、WordPress 等）、PHP 版本、数据库类型等。
3. **自动化**：
    
    - ddev 提供了一系列命令和钩子（hooks），允许开发者在特定的生命周期事件（如启动、停止、重启等）时运行自定义脚本。这使得复杂的配置和安装任务可以自动化处理。

# 安装DDEV

[DDEV Installation - DDEV Docs](https://ddev.readthedocs.io/en/stable/users/install/ddev-installation/)

[WSL 上的 Docker 容器入门 | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/wsl/tutorials/wsl-containers)

介绍了安装 DDEV，再安装、设置 Next for Drupal 模块

Windows系统注意事项：

- WSL2 offers a faster, smoother experience.  
    It’s vastly more performant, and you’re less likely to have obscure Windows problems.
    Windows系统一定要用WSL2

- Execute DDEV commands inside WSL2.  
    You’ll want to run DDEV commands inside Ubuntu, for example, and never on the Windows side in PowerShell or Git Bash.
    DDEV包括在Windows下的 ddev.exe 和 Linux子系统中的 ddev。一定要在WSL2 里运行 ddev，而不是在Windows层面。
    
- Projects should live under the home directory of the Linux filesystem.  
    WSL2’s Linux filesystem (e.g. `/home/<your_username>`) is much faster and has proper permissions, so keep your projects there and **not** in the slower Windows filesystem (`/mnt/c`).
    一定要在子系统Linux的文件系统下建立项目，比在Windows下快很多
    
- Custom hostnames are managed via the Windows hosts file, not within WSL2.  
    DDEV attempts to manage custom hostnames via the Windows-side hosts file—usually at `C:\Windows\system32\drivers\etc\hosts`—and it can only do this if it’s installed on the Windows side. (DDEV inside WSL2 uses `ddev.exe` on the Windows side as a proxy to update the Windows hosts file.) If `ddev.exe --version` shows the same version as `ddev --version` you’re all set up. Otherwise, install DDEV on Windows using `choco upgrade -y ddev` or by downloading and running the Windows installer. (The WSL2 scripts below install DDEV on the Windows side, taking care of that for you.) If you frequently run into Windows UAC Escalation, you can calm it down by running `gsudo.exe cache on` and `gsudo.exe config CacheMode auto`, see [gsudo docs](https://github.com/gerardog/gsudo#credentials-cache).
    
- WSL2 is not the same as Docker Desktop’s WSL2 engine.  
    Using WSL2 to install and run DDEV is not the same as using Docker Desktop’s WSL2 engine, which itself runs in WSL2, but can serve applications running in both traditional Windows and inside WSL2.

建立Drupal项目：[CMS Quickstarts - DDEV Docs](https://ddev.readthedocs.io/en/stable/users/quickstart/#drupal)

### 本地证书

如果证书不被浏览器信任，重新执行以下步骤：

1、In an administrative PowerShell, run `$env:CAROOT="$(mkcert -CAROOT)"; setx CAROOT $env:CAROOT; If ($Env:WSLENV -notlike "*CAROOT/up:*") { $env:WSLENV="CAROOT/up:$env:WSLENV"; setx WSLENV $Env:WSLENV }`   . This will set WSL2 to use the Certificate Authority installed on the Windows side. In some cases it takes a reboot to work correctly.

2、Double-check in Ubuntu (or your distro): `echo $CAROOT` should show something like `/mnt/c/Users/<you>/AppData/Local/mkcert`

3、In WSL2, run `mkcert -install`

If you get the prompt `Installing to the system store is not yet supported on this Linux`, you may need to add `/usr/sbin` to the `$PATH` so that `/usr/sbin/update-ca-certificates` can be found.

如果还是不行，手工检查ddev的全局配置文件  ~/.ddev/global_config.yaml

确保配置项 mkcert_caroot 的路径指向和在WSL下 `echo $CAROOT` 的一致。

### 本地域名

DDEV会为项目生成并解析本地域名。

1、通过自动修改系统hosts文件实现

全局配置项：wsl2_no_windows_hosts_mgt: false

On WSL2 by default the Windows-side hosts file (normally C:\Windows\system32\drivers\etc\hosts) is used for hosts file management, but doing that requires running sudo and ddev.exe on Windows side; you may not want this if you're running your browser in WSL2 or for various other reasons.

在 WSL2 中运行 ddev 命令时，确保使用 `sudo` 权限来修改 Windows 的 hosts 文件。

需要在 WSL2 中安装 Windows 版本的 ddev 工具 (`ddev.exe`)，并确保它在WSL的PATH 中。

这个配置自相矛盾，可能是历史遗留，ddev无法通过sudo来运行。

2、通过本地路由来实现

在项目目录下 .ddev/config.yaml 中，配置项 use_dns_when_possible: true

If the host has internet access and the domain configured can successfully be looked up, DNS will be used for hostname resolution instead of editing /etc/hosts

避坑：

当本机开启v2ray类VPN时，会造成本地路由失效，浏览时提示：The provided host name is not valid for this server.

目前办法是要么关闭v2ray，要么v2ray设置绕过这些域名。

## DDEV使用

在 WSL 环境中，同一个目录下运行 `npm install` 和 `ddev npm install` 的主要区别在于运行环境和环境管理的方式。`ddev npm install` 提供了一个隔离且一致的容器环境，有助于避免依赖冲突和环境差异问题，确保团队协作和项目管理的高效性和一致性。而 `npm install` 则依赖于本地 WSL 环境，适合单人开发或需要直接访问本地文件系统的场景。

WSL默认添加Windows下所有的 $PATH ，这对于docker ddev 等是必要的。

但是也带来了不兼容和困扰，就比如 npm  命令。

彻底移除Windows下所有的 $PATH（不推荐）：

[Automatically Configuring WSL - Windows Command Line (microsoft.com)](https://devblogs.microsoft.com/commandline/automatically-configuring-wsl/)

WSL下执行：sudo vim /etc/wsl.conf

```
# 不加载Windows中的PATH内容
[interop]
appendWindowsPath = false

# 不自动挂载Windows系统所有磁盘分区
#[automount]
#enabled = false
```

保存关闭后，需要重启 wsl，在 win 命令行中输入

wsl --shutdown

之后再次打开 wsl 就可以了

Ubuntu 下安装node npm

官方最新版本都太老了，需要从 NodeSource 源安装。

[nodesource/distributions: NodeSource Node.js Binary Distributions (github.com)](https://github.com/nodesource/distributions?tab=readme-ov-file#ubuntu-versions)

安装完成后，就可以使用 `ddev npm install` 来在容器环境安装运行node项目了。
