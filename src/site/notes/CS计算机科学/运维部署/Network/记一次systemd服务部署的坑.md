---
{"dg-publish":true,"permalink":"/CS计算机科学/运维部署/Network/记一次systemd服务部署的坑/","noteIcon":"","created":"2024-06-22T22:30:29.089+08:00","updated":"2024-04-24T00:32:04.000+08:00"}
---


作者：游鱼思

---

通过 systemd 部署一个Python应用，其中 aiohttp 模块始终连不上网络。

找了好几天问题还是无解，直到ChatGPT在我的反复逼问下，说出了：  

*如果在命令行手动运行时一切正常，可能还与环境变量有关。确保通过 systemd 启动时，环境变量和手动运行时的环境一致。你可以在 systemd 服务文件中设置环境变量：*

```
[Service]
Environment="PATH=/usr/bin:/usr/local/bin"
Environment="OTHER_VAR=value"
```

并且，$Path 变量不能用，因为 $Path  在 systemd 配置文件中不会自动展开。

我的问题出在环境变量中需要指定证书：

```
Environment="SSL_CERT_FILE=/usr/local/lib/python3.11/site-packages/certifi/cacert.pem"
```

这个锅应该 宝塔面板 来背，宝塔自带的Python项目管理真的没用，因为SSL模块是老版本，安装依赖包都没法成功。我只能自己安装了一套Python环境，并配置了证书。