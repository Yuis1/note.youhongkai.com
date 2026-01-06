---
{"dg-publish":true,"permalink":"/CS计算机科学/运维部署/Network/内网穿透/WireGuard组网/","noteIcon":"","created":"2026-01-06T18:27:37.222+08:00","updated":"2026-01-06T18:58:23.993+08:00"}
---


为了让几台内网的开发机器可以互相访问，权衡 端口映射、FRP、FRPS、CF Tunnel 多种方案：

- 端口映射：国内大部分情况不支持，一般需要IPv6。家用路由器对IPv6的防火墙设置很粗糙，完全开放端口又带来了新的安全问题。
- FRP：虽然可以通过 Nginx 反向代理 + Basic Auth（账号密码） 增加安全性，由 Nginx 负责对外提供 HTTPS 和 密码验证。但是禁不住长期被扫描，并且配置很繁琐，并且wss等其它协议也很麻烦。
- FRPS：足够安全。frp 的 `stcp` (Secret TCP) 模式，类似于VPN。比较推荐。
- WireGuard Mesh组网：基于WireGuard协议的生态丰富，
- VSCode Tunnel：仅支持vscode，通用性不足。且卡顿。
- CF Tunnel：卡顿。

## WireGuard Mesh组网介绍
### 产品
|**方案**|**大陆流畅度**|**部署难度**|**安全性**|**适用场景**|
|---|---|---|---|---|
|**Tailscale + 私有 DERP**|极高 (依赖国内服务器)|中等|极高 (2FA+端到端)|**最推荐**，最稳健，体验最好|
|**ZeroTier + Moon**|高|简单|高|适合不想折腾复杂配置的用户|
|**Headscale**|极高|较高|极高 (完全私有)|追求完全开源、隐私、无限制的极客|
|**NetBird (私有化)**|高|较高|极高|需要复杂权限管理的团队/个人|

### Mesh VPN 有以下几个杀手锏
- **真正的零暴露：** 在直连模式下，公网不开启任何监听端口，黑客完全无处下手。
- **隐私性：** 即使经过中转，流量依然是在 A 和 B 端点间进行 **WireGuard 端到端加密**的。
- **自带身份验证：** 需要用 GitHub / Google 账号登录 Tailscale。这意味着访问 3000 端口的前提是你已经通过了双重身份验证（2FA）。
- **性能：** 如果你是在公司连接家里的开发机，通常能触发 P2P 直连，速度接近你的宽带物理上限，比经过 frp 服务器中转要快得多。

### 自建中转服务器
- **Tailscale** 称之为 **DERP** 服务器。
- **ZeroTier** 称之为 **Planetary/Moon** 节点。

## 配置要点
### 与 v2ray 共存
防止路由冲突导致连接中断：
• 分流配置：在代理客户端（如 Clash/v2rayN）的路由配置中，将 Tailscale 网段（`100.64.0.0/10`）设为直连（Direct）。
• DNS 优化：停用 Tailscale DNS功能，使用IP访问，避免 MagicDNS 与 Fake-IP 模式冲突。
• UDP 放行：确保代理软件不拦截 41641 等 UDP 端口，以维持 P2P 直连。

### 让WSL也能用
- WSL Setting中，network一定要使用 Mirror 模式，才可以让组网的其它机器访问到本机WSL内部服务。不推荐使用NAT模式，因为还得手工做端口转发。
- 检查Windows防火墙，不要拦截WSL提供的服务端口

### WSL中快速启用代理且Mesh VPN网段不走V2ray
将以下脚本存为  /usr/local/proxy
```
#!/bin/bash
# 自动检测 WSL 网络模式并获取主机 IP
_gateway_ip=$(ip route | grep default | awk '{print $3}')
# 判断是否为 NAT 模式（默认网关是 WSL 内部网关）
if [[ $_gateway_ip =~ ^172\.(2[89]|3[01])\. ]]; then
  host_ip=$_gateway_ip
else
  # Mirror 模式使用本地回环地址
  host_ip="127.0.0.1"
fi

# 获取本机局域网 IP（用于 no_proxy）
_local_ip=$(ip route | grep -E "^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+/[0-9]+ dev [^ ]+" | \
            grep -v "docker\|br-" | \
            awk '{print $1}' | head -1 | cut -d'/' -f1)

# 构建 no_proxy 列表（本地、私有和局域网地址）
_no_proxy_entries="localhost,127.0.0.1,::1"
if [ -n "$_local_ip" ]; then
  _no_proxy_entries="$_no_proxy_entries,$_local_ip"
  # 添加同网段 (例如 192.168.1.*)
  _local_subnet=$(echo $_local_ip | cut -d'.' -f1-3).*
  _no_proxy_entries="$_no_proxy_entries,$_local_subnet"
fi
# 添加私有和 VPN 地址范围
_no_proxy_entries="$_no_proxy_entries,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16,100.64.0.0/10"

if [ "$1" = "on" ]; then
  export http_proxy="http://$host_ip:1087"
  export https_proxy="http://$host_ip:1087"
  export HTTP_PROXY="http://$host_ip:1087"
  export HTTPS_PROXY="http://$host_ip:1087"
  export no_proxy="$_no_proxy_entries"
  export NO_PROXY="$_no_proxy_entries"
  echo "Proxy is now ON (using $host_ip:1087)."
  echo "no_proxy: $no_proxy"
elif [ "$1" = "off" ]; then
  unset http_proxy
  unset https_proxy
  unset HTTP_PROXY
  unset HTTPS_PROXY
  unset no_proxy
  unset NO_PROXY
  echo "Proxy is now OFF."
elif [ "$1" = "status" ]; then
  if [ -n "$http_proxy" ]; then
    echo "Proxy is ON: $http_proxy"
    [ -n "$no_proxy" ] && echo "no_proxy: $no_proxy"
  else
    echo "Proxy is OFF"
  fi
else
  echo "Usage: proxy [on|off|status]"
fi
```

执行  `chmod +x /usr/local/proxy`
今后开启代理：`proxy on`
关闭代理：`proxy off`