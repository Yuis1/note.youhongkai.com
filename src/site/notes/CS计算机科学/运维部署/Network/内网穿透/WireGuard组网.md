---
{"dg-publish":true,"permalink":"/CS计算机科学/运维部署/Network/内网穿透/WireGuard组网/","noteIcon":"","created":"2026-01-06T18:27:42.110+08:00","updated":"2026-02-02T10:53:25.060+08:00"}
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

最终选择 Tailscale，原因是可以自建中转服务器，且配置相对方便，附加的实用功能很多。

比如域名+https，对前端开发就很友好。还有设备间文件传输。

### Mesh VPN 有以下几个杀手锏
- **真正的零暴露：** 在直连模式下，公网不开启任何监听端口，黑客完全无处下手。
- **隐私性：** 即使经过中转，流量依然是在 A 和 B 端点间进行 **WireGuard 端到端加密**的。
- **自带身份验证：** 需要用 GitHub / Google 账号登录 Tailscale。这意味着访问 3000 端口的前提是你已经通过了双重身份验证（2FA）。
- **性能：** 如果你是在公司连接家里的开发机，通常能触发 P2P 直连，速度接近你的宽带物理上限，比经过 frp 服务器中转要快得多。

### 自建中转服务器
- **Tailscale** 称之为 **DERP** 服务器。
- **ZeroTier** 称之为 **Planetary/Moon** 节点。

## Tailscale配置要点
### 与 v2ray 共存

防止路由冲突导致连接中断：

• 分流配置：在代理客户端（如 Clash/v2rayN）的路由配置中，将 Tailscale 网段（`100.64.0.0/10`）设为直连（Direct）。

• DNS 优化：停用 Tailscale DNS功能，使用IP访问，避免 MagicDNS 与 Fake-IP 模式冲突。

• UDP 放行：确保代理软件不拦截 41641 等 UDP 端口，以维持 P2P 直连。

#### 解决V2Ray DNS 冲突

这是最容易出问题的地方。Tailscale 默认会开启 **MagicDNS**，它会接管系统的 DNS 解析以便你通过机器名（如 `http://my-dev-machine:3000`）访问。

- **如果你使用 Clash/v2ray 的 Fake-IP 模式**：两者会抢夺 `/etc/resolv.conf` 或系统的 DNS 权限。
- **解决方案**：
    1. **推荐：** 在 Tailscale 控制台禁用 MagicDNS，直接通过 **Tailscale IP**（100.x.x.x）访问开发机。
    2. 或者在 v2ray 的 DNS 配置中，将 Tailscale 的域名后缀（通常是 `*.ts.net`）设为使用系统原生 DNS 解析。

v2ray dns 设置：

```json
  "servers": [
    {
      "address": "localhost",
      "domains": [
        "domain:ts.net", 
        "domain:tailscale.com", 
        "geosite:private"
      ]
    },
    ...
```

sing-box dns 设置：

```json
  "servers": [
    {
      "tag": "tailscale-dns",
      "address": "local",
      "detour": "direct"
    },
    ...

	"rules": [
    {
      "domain_suffix": [
        ".ts.net",
        ".tailscale.com",
        ".local"
      ],
      "server": "tailscale-dns"
    },
    
```


### 让WSL也能用
- WSL Setting中，Network一定要使用 Mirror 模式，才可以让组网的其它机器访问到本机WSL内部服务。不推荐使用NAT模式，因为还得手工做端口转发。
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

### Tailscale 网络诊断
#### 查看连接状态

`tailscale status`

- **direct**：表示**直连**。后面通常会跟着对端的公网 IP 和端口（如 `1.2.3.4:41641`）。这是最理想的状态，速度最快，延迟最低。
- **relay**：表示**转发**。后面会跟着一个 DERP 节点的简称（如 `relay "hkg"` 代表香港中转，`relay "tok"` 代表东京中转）。

#### 测试详细延迟

`tailscale ping`

- 前几次通常显示 `via DERP`，因为 Tailscale 在后台正在尝试“打洞”。
- 如果最后出现了 `via <公网IP>`，说明**打洞成功，已转为直连**，此时的 `15ms` 就是真实的 P2P 延迟。

#### 诊断网络环境

`tailscale netcheck`

**UDP: true**：这是直连的基础。如果为 `false`，说明 UDP 被防火墙拦截，几乎肯定只能走中转。

**MappingVariesByDestIP**：如果为 `true`，代表你是“对称型 NAT”（Hard NAT），打洞难度较大。

#### **不要开启 "Exit Node"**

Tailscale 默认只修改发往虚拟网段的路由，**不会**接管你的所有互联网流量。除非你明确想让开发机当你的加速器，否则不要在笔记本上选择开发机作为 Exit Node。这样，你访问开发机的 3000 端口走 Tailscale，访问其他网站依然走 v2ray。

### Tailscale 的 "Subnet Router"

支持“节点中转局域网流量”，充当一个“桥梁”，让其它机器可以能访问本机所在局域网的所有设备（比如NAS、路由器管理界面）。

**配置命令**： `tailscale up --advertise-routes=192.168.1.0/24`

### 中转服务器

Tailscale 不支持普通客户端做为中转服务器，需要单独安装**DERPer** 。Tailscale 官方并不直接提供 DERP 的镜像，但社区有非常成熟且轻量的实现（如 `fredliang/derper`）。

- **域名与 SSL**：DERP 必须工作在 HTTPS (443) 端口上（为了伪装流量并确保安全）。你需要一个指向该服务器 IP 的域名。
- **防火墙开放端口**：
    - **TCP 443**：HTTPS 流量。
    - **UDP 3478**：STUN 协议（用于协助其他节点进行 NAT 穿透打洞）。

为 DERP 分配一个二级域名（如 `derp.yourdomain.com`），让 Nginx 处理 SSL 证书，然后将流量转发给运行在非标准端口上的 DERP 容器。

docker-compose.yml

```yml
services:
  derper:
    image: fredliang/derper
    container_name: derper
    restart: always
    ports:
      - "3478:3478/udp" # STUN 端口保持不变
      - "127.0.0.1:8443:443" # 只监听本地回环，交给 Nginx 转发
    environment:
      - DERP_DOMAIN=derp.yourdomain.com
      - DERP_CERT_MODE=manual # 改为手动模式，不让它自己申请证书
      - DERP_ADDR=:443
      - DERP_VERIFY_CLIENTS=false # 如果是个人用，可以设为 false
```

部署好容器后，Tailscale 并不会自动使用它。你需要在 [Tailscale Admin Console](https://login.tailscale.com/admin/acls) 的 **Access Control** 中添加 `derpMap` 配置：

```
"derpMap": {
    "OmitDefaultRegions": false, // 是否禁用官方海外节点，建议先设为 false
    "Regions": {
        "901": {
            "RegionID": 901,
            "RegionCode": "my-derp",
            "Nodes": [
                {
                    "Name": "1",
                    "RegionID": 901,
                    "HostName": "你的域名.com",
                    "IPv4": "服务器公网IP",
                    "DERPPort": 443,
                    "STUNPort": 3478
                }
            ]
        }
    }
}
```

验证：

在其它机器运行 `tailscale netcheck`。如果输出中看到了 `my-derp` 且延迟极低，说明配置成功。

### Tailscale其它功能
#### Services (服务发现)
**Services** 是虚拟内网中**正在运行的具体业务**的实时清单。
- Tailscale 客户端（tailscaled）会自动扫描每台机器上正在监听的端口，它会自动把这些信息上报给控制台。
- **作用**：
    - **资产盘点**：让你一眼看到全网内开了哪些 Web 服务、数据库或 SSH 端口。
    - **快速访问**：控制台会列出这些服务的访问链接。
    - **安全审计**：如果你发现某台开发机莫名其妙多出了一个未知的监听端口，你可以迅速定位并排查，防止内网泄露。
        
- **注意点**：
    - Services 是**被动收集**的。
    - 它不代表权限，只是一个“目录”。能不能访问这些 Service，依然由你的 **ACL (访问控制列表)** 决定。
#### Apps (应用集成)
**Apps** 页面则代表了 Tailscale 与**外部第三方服务**或**身份验证系统**的深度绑定。
包含以下三个功能：
##### Auth Keys & OAuth Clients
- **Auth Keys**：生成用于自动化部署的密钥（如在 Docker 服务器上一键安装并登录 Tailscale，无需手动扫码）。
- **OAuth Clients**：用于让第三方工具通过 API 管理你的 Tailscale 网络。
##### App Connectors (访问公网 SaaS)

这是一个进阶功能。假设你的公司使用 **Salesforce** 或 **GitHub Enterprise**，且你希望员工只有在连接了 Tailscale 的情况下才能访问这些公网网站。

- 你可以将某台 Linux 服务器设为 **App Connector**。
- 它会把这些特定域名的流量强制导向 Tailscale 内网出口，实现对公网 App 的安全加固。
##### Tailscale Integrations (三方集成)

可以将 Tailscale 连接到 **GitHub**、**Okta** 或 **Microsoft Entra ID**。

- 例如，通过集成 GitHub App，你可以让 Tailscale 的访问权限直接关联到 GitHub 的团队成员，实现“GitHub 账号登录，自动拥有内网权限”。

### TailScale疑难杂症
TS安装在Windows宿主机，在WSL2为Mirror模式的网络环境下，有时候会引起WSL无法获取宿主机网卡。
解决方案：
1. 让 Tailscale 延迟启动（解决竞态）：把 Tailscale 服务改成 **Automatic (Delayed Start)**（服务管理器 `services.msc` 里改），让 WSL 先把 mirrored 网卡镜像出来，再让 Tailscale 改网络栈。
2. 如果还无法解决，在 Windows 的 Tailscale 里关掉 DNS 接管。
3. 终极方案：WSL使用NAT网络模式。

## 其它Mesh组网方案
### ZeroTier

ZeroTier 的节点（Peers）具备一定的“中转意识”。

- **Moon 节点**：虽然它也建议你自建 Moon（类似 DERP），但 ZeroTier 的协议允许节点之间进行更复杂的路径选择。
- **逻辑**：ZeroTier 将整个网络视为一个巨大的虚拟交换机。如果 A 和 B 无法直连，它们会尝试通过它们都能连接到的“最近”节点进行跳转。
- **缺点**：在特殊的网络环境下（UDP 大规模丢包），如果没有一个固定的物理服务器做 Moon，节点间的互相中转往往非常不稳定。

### NetBird (支持自建中转且架构更现代)

NetBird 同样基于 WireGuard，但它在“中转”这件事上比 Tailscale 更开放一点。

- 它使用 **ICE/STUN/TURN** 协议（WebRTC 常用的那一套）。可以指定网络中的某台性能较好的 Linux 机器作为 **Relay 节点**。
- **优点**：它的管理后台支持非常直观的路径查看。
