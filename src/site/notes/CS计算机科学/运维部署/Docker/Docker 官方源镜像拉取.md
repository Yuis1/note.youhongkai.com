---
{"dg-publish":true,"permalink":"/CS计算机科学/运维部署/Docker/Docker 官方源镜像拉取/","noteIcon":"","created":"2025-01-31T18:26:56.001+08:00","updated":"2025-04-29T11:14:22.982+08:00"}
---


作者：游鱼思

---
## 问题

设置三方镜像站的方法：[Docker Hub 镜像加速器](https://gist.github.com/y0ngb1n/7e8f16af3242c7815e7ca2f0833d3ea6#docker-hub-%E9%95%9C%E5%83%8F%E5%8A%A0%E9%80%9F%E5%99%A8)

采用镜像站的问题：

1. 大型、稳定的越来越少，生命期也变短了
2. 更新存在滞后，有些镜像没有
3. 代码安全性未知

最优方案还是通过代理访问官方源。

但是这里存在一个坑，新手往往混淆了Docker使用代理的三类场景：

1. docker pull 拉取镜像时用代理
2. docker build 构建镜像时用代理
3. docker 容器访问外网时用代理

参考： [ 配置 HTTP/HTTPS 网络代理](https://yeasy.gitbook.io/docker_practice/advanced_network/http_https_proxy)

## docker pull 时设置代理
### 新手陷阱
**docker pull 设置代理的新手陷阱** ：此时由docker daemon管理镜像拉取，无论是直接挂系统代理（除非开启Tun模式）、设置环境变量 http_proxy、https_proxy ，还是在 docker desktop 的 Settings - Resources - Proxies 里设置，统统都没用。

在官方文档 [Daemon proxy configuration](https://docs.docker.com/engine/daemon/proxy/)给出两种方法：

1. 通过 daemon.json 来配置
2. 通过环境变量来设置 —— 仅限于docker通过 systemd 服务来运行的场景，Mac端、Windows端

我再补充两种方法：

1. 网关透明代理：在路由器网关加代理，比较容易实现。如果路由器不支持，也可以让另一台电脑或手机挂代理后临时当一下网关。
2. 本机Tun模式：在操作系统中创建虚拟网络接口（虚拟网卡）来实现对网络流量进行捕获和重定向的代理模式。Windows端 v2rayN 支持开启，Mac端V2RayU Routing 选Global应该也可以。

### 通过 daemon.json 来配置docker pull时的代理

参考 [Docker daemon configuration overview](https://docs.docker.com/engine/daemon/)  ， daemon.json 位置如下。

| OS and configuration | File location                              |
| -------------------- | ------------------------------------------ |
| Linux, regular setup | /etc/docker/daemon.json                    |
| Linux, rootless mode | /etc/docker/daemon.json                    |
| Windows              | `C:\ProgramData\docker\config\daemon.json` |

注意， /etc/docker/daemon.json 是docker读取配置的默认地址，和运行docker的用户无关。

Mac下的 daemon.json，可以通过桌面端的  Settings - Docker Engine 来设置。

在 deamon.json 中，加入代理地址：

```
{
  "proxies": {
    "http-proxy": "http://proxy.example.com:3128",
    "https-proxy": "https://proxy.example.com:3129",
    "no-proxy": "*.test.example.com,.example.org,localhost,127.0.0.0/8"
  }
}
```

注意：  

- 如果对应客户端没有http代理，并提示错误 malformed HTTP request ，则 `"http-proxy": "http://proxy.example.com:3128", "https-proxy": "https://proxy.example.com:3129",`  换成 `"http-proxy": "sockt5://proxy.example.com:3128", "https-proxy": "sockt5://proxy.example.com:3129",`  。
- "no-proxy": "localhost,127.0.0.0/8" 不要省略，否则会把本地请求也代理。

Docker desktop 中也能设置：

![](/img/user/Z-attach/Screenshot 2025-01-31 at 11.52.25.png)

[http-proxy 和 https-proxy 的区别](../Network/http-proxy%20和%20https-proxy%20的区别.md)

### Mac端疑难杂症

以上都设置了，docker pull 时还是报错：

```shell
docker pull qdrant/qdrant:latest
Error response from daemon: Get "https://registry-1.docker.io/v2/": proxyconnect tcp: dial tcp [::1]:1087: connect: connection refused
```

直接用新版 docker.app 覆盖，期待能保留容器。

结果倒好，容器页面 Error，运行 docker pull 提示：

```shell
docker pull qdrant/qdrant:latest
request returned Internal Server Error for API route and version http://%2FUsers%2Fyuis%2F.docker%2Frun%2Fdocker.sock/v1.47/images/create?fromImage=qdrant%2Fqdrant&tag=latest, check if the server supports the requested API version
```

没办法，只能建脚本 MacOS_Uninstall_Docker.sh ，彻底删除docker程序和文件：

```shell
#!/bin/bash

# 确保脚本以管理员权限运行
if [[ $EUID -ne 0 ]]; then
   echo "该脚本需要管理员权限，请使用sudo运行。" 1>&2
   exit 1
fi

# 停止Docker Desktop
echo "停止Docker Desktop..."
osascript -e 'quit app "Docker"'

# 等待Docker完全退出
sleep 5

# 卸载Docker Desktop
echo "卸载Docker Desktop..."
rm -rf /Applications/Docker.app

# 删除Docker相关文件和配置
echo "删除Docker配置文件和数据..."
rm -rf ~/Library/Application\ Support/Docker
rm -rf ~/Library/Containers/com.docker.docker
rm -rf ~/Library/Containers/com.docker.helper
rm -rf ~/Library/Preferences/com.docker.docker.plist
rm -rf ~/Library/Preferences/com.docker.helper.plist
rm -rf ~/.docker
rm -rf ~/Library/Logs/Docker
rm -rf ~/Library/Caches/com.docker.docker
rm -rf ~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux

# 清空废纸篓
# echo "清空废纸篓..."
# osascript -e 'tell app "Finder" to empty trash'

# 验证Docker是否已卸载
echo "验证Docker是否已卸载..."
if ! command -v docker &> /dev/null; then
    echo "Docker已成功卸载。"
else
    echo "Docker未完全卸载，请手动检查残留文件。"
fi

echo "Docker Desktop彻底卸载完成。"
```

再次挂代理，docker pull ，就治好了。

## Linux下通过环境变量来配置docker pull时的代理

注意：由于docker通过systemd来启动，此时的环境变量要在  systemd drop-in file 里设置：

```
sudo mkdir -p /etc/systemd/system/docker.service.d
```

Create a file named `/etc/systemd/system/docker.service.d/http-proxy.conf` that adds the `HTTP_PROXY` environment variable:

```systemd
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:3128"
```

If you are behind an HTTPS proxy server, set the `HTTPS_PROXY` environment variable:

```systemd
[Service]
Environment="HTTPS_PROXY=https://proxy.example.com:3129"
```

Multiple environment variables can be set; to set both a non-HTTPS and a HTTPs proxy;

```systemd
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:3128"
Environment="HTTPS_PROXY=https://proxy.example.com:3129"
```

记得重新载入服务配置

```
sudo systemctl daemon-reload
sudo systemctl restart docker
```
