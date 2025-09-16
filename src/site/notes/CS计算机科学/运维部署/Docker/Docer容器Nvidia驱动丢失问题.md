---
{"dg-publish":true,"permalink":"/CS计算机科学/运维部署/Docker/Docer容器Nvidia驱动丢失问题/","noteIcon":"","created":"2025-09-16T11:40:07.676+08:00","updated":"2025-09-16T11:41:14.898+08:00"}
---


**问题描述：**Linux系统自带显卡驱动太老，从Nvidia官网安装了最新驱动。docker 容器运行一段时间后，频繁加载模型，几天后在容器内就无法访问nvidia-smi了。

```
# 容器内
user@sinomis-ai:~$ docker exec -it ollama bash 
root@487091d1a31c:/# nvidia-smi 
Failed to initialize NVML: Unknown Error

# 宿主机
user@sinomis-ai:~$ sudo systemctl restart nvidia-persistenced
Failed to restart nvidia-persistenced.service: Unit nvidia-persistenced.service not found.
```

**原因分析：**

NVIDIA驱动一直运行在非持久模式（Non-Persistent Mode）下。当初安装NVIDIA驱动时，没有正确地安装nvidia-persistenced守护进程。从Nvidia官网下载的 .run 文件，不会自动配置守护进程。

什么是非持久模式？ 在这种模式下，NVIDIA驱动只有在需要时（比如你启动了一个GPU应用）才会被加载到内核中。当最后一个使用GPU的应用退出后，驱动会随之卸载以节省少量内存。

为什么这会导致问题？ 这种频繁的加载/卸载过程，尤其是在有Docker这种中间层的复杂环境中，会增加驱动状态出错的风险。当容器长时间运行，而你又通过 docker exec 发起一个新的GPU请求（比如 nvidia-smi）时，驱动可能无法为这个新请求正确地重新初始化，从而导致 Failed to initialize NVML 错误。

持久模式 (Persistence Mode) 通过运行一个轻量级的 nvidia-persistenced 守护进程，强制让NVIDIA驱动始终保持加载和活动状态。这极大地提高了稳定性和响应速度，是所有服务器环境的推荐设置。

**解决方案：**手工创建守护进程服务：

```
# 先把 Persistence-M 状态设为打开
sudo nvidia-smi -pm 1

# 创建nvidia-persistenced守护进场，维持 Persistence-M 的状态
# 注意：nvidia-persistenced 并不会打开Persistence-M，只会维护Persistence-M的状态
sudo nano /etc/systemd/system/nvidia-persistenced.service
sudo systemctl daemon-reload
sudo systemctl enable nvidia-persistenced
sudo systemctl restart docker
```

```
[Unit]
Description=NVIDIA Persistence Daemon
Wants=syslog.target

[Service]
Type=forking
PIDFile=/var/run/nvidia-persistenced/nvidia-persistenced.pid
Restart=always
ExecStart=/usr/bin/nvidia-persistenced --verbose
ExecStopPost=/bin/rm -rf /var/run/nvidia-persistenced

[Install]
WantedBy=multi-user.target
```