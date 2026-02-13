---
{"dg-publish":true,"permalink":"/P生产力/OS/Windows/WSL2疑难杂症/","noteIcon":"","created":"2025-07-31T09:55:12.356+08:00","updated":"2026-02-09T11:29:14.326+08:00"}
---

## Mirrored镜像模式无法联网
### .wslconfig配置错误

使用ifconfig可以看到检测不到宿主机网卡了

需要在个人主目录 .wslconfig 文件中，移除或注释 localhostForwarding ，即使是 false 也不可以！

```toml
[wsl2]
networkingMode=mirrored
# 必须删除下面这一行，不能留 localhostForwarding=false 
# localhostForwarding=false
```

### Tailscale 竞态

先关闭 TS，启动WSL，再开启TS

```bash
# 在powershell中
wsl --shutdown
tailscale down
wsl
```

然后再开powershell窗口，执行：`tailscale up`

## WSL使用宿主机的代理

在 .bashrc 添加如下内容

``` bash
# 获取正确的 WSL 宿主机 IP 
export WINDOWS_HOST=$( \ ip route | grep '^default' | awk '{print $3}' | grep '^172\.' \ )

# 如果上述方法失败，使用备用方法（Powershell获取） 
if [ -z "$WINDOWS_HOST" ]; then 
	export WINDOWS_HOST=$( \ 
		powershell.exe -c "(Get-NetIPAddress | ? {\$_.IPAddress -like '172.*'}).IPAddress" | head -1 | tr -d '\r' \ 
		) 
fi 

# 设置代理，修改端口为实际代理端口 
export HTTP_PROXY="http://$WINDOWS_HOST:1087" 
export HTTPS_PROXY="http://$WINDOWS_HOST:1087" 
export ALL_PROXY="socks5://$WINDOWS_HOST:1086"
```

## VS-Code无法连上WSL
### 资源分配不足

主要原因是只分配了4G内存、2核CPU，而我运行了pg、qdrant、vs code server、claude code 等一系列服务，导致了资源占用高峰期很不稳定。

打开配置： C:\Users\你的用户名\.wslconfig

增加到8G内存、10核CPU、8G虚拟内存，重启wsl，现在没出问题了。

### VS Code Server 重新安装时的网络问题

VS Code Server 是wsl下**按需启动的进程**。

当 Windows VS Code 尝试连接到 WSL 时，它会远程执行脚本来启动这些进程；当关闭 VS Code 窗口时，这些进程通常会在一段时间后自动退出。

从 VS Code 的 Terminal可以看到，VS Code Server 卡在了下载环节。

于是我调整宿主机VPN到全局，让它可以顺利下载，问题解决。