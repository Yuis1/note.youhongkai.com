---
{"dg-publish":true,"permalink":"/CS计算机科学/运维部署/Network/ssh命令走代理的方式/","noteIcon":"","created":"2024-04-20T15:32:28.000+08:00","updated":"2024-04-24T00:35:57.476+08:00"}
---


在 macOS 或任何 Unix-like 系统中，`http_proxy`, `https_proxy`, 和 `ALL_PROXY` 环境变量通常被用来指定命令行程序如 `curl` 和 `wget` 的代理设置。然而，`ssh` 命令本身并不直接支持通过这些环境变量配置的代理服务器进行连接。`ssh` 需要不同的配置方法来使用代理服务器。

如果你想让 `ssh` 通过一个代理服务器连接到远程服务器，你可以使用几种方法：

### 1. 使用 `ProxyCommand` 在 SSH 配置中设置代理

你可以在你的 `~/.ssh/config` 文件中为特定的主机或所有主机设置代理。这里是一个使用 SOCKS5 代理的例子：

```bash
Host *
  ProxyCommand nc -X 5 -x 127.0.0.1:1080 %h %p
```

这里，-X 5 指定了代理类型为 SOCKS5，-x 127.0.0.1:1080 指定了代理地址和端口。

这个配置会让所有的 `ssh` 连接都通过在 `127.0.0.1:1080` 上运行的 SOCKS5 代理。确保你已经安装了 `nc` (netcat)，它是这个命令需要的。

### 2. 使用 `-o` 选项直接在 SSH 命令中指定 `ProxyCommand`

如果你不想修改 `~/.ssh/config` 文件，也可以在命令行中直接使用 `-o` 选项来指定 `ProxyCommand`：

```bash
ssh -o 'ProxyCommand=nc -x 127.0.0.1:1080 %h %p' username@hostname
```

### 3. 使用第三方工具如 `proxychains`

`proxychains` 是一个 Unix 下的工具，允许你通过代理（如 SOCKS5、SOCKS4 或 HTTP）运行任何 TCP/IP 应用。首先安装 `proxychains`（如果还未安装），然后在其配置文件中设置代理，通常位于 `/etc/proxychains.conf`：

```bash
[ProxyList]
socks5 127.0.0.1 1080
```

然后你可以这样运行 `ssh`：

```bash
proxychains ssh username@hostname
```

### 注意

- 确保你安装了必要的工具，如 `nc` (netcat) 或 `proxychains`，根据你选择的方法。
- 修改 SSH 配置或其他系统配置文件时，需要具有相应的权限。

这些方法应该能帮助你使 `ssh` 命令通过代理连接，即使它不直接支持读取 `http_proxy` 或 `https_proxy` 环境变量。