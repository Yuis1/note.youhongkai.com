---
{"dg-publish":true,"permalink":"/CS计算机科学/运维部署/Docker/Docker Compose/","noteIcon":"","created":"2025-07-31T11:06:14.736+08:00","updated":"2025-02-13T17:00:37.000+08:00"}
---


1、docker-compose.yml 对缩进位置、缩进所采用的字符敏感，最好统一成一个缩进两个空格。否则会出现 yaml.parser.ParserError: while parsing a block mapping… 错误。

2、ports: 前一个端口是指本机端口，后一个端口是容器端口。

在Docker Compose中，可以通过在服务的配置中使用restart关键字来指定容器的重启策略。unless-stopped和always是两种不同的重启策略，它们的区别如下：

- always：表示容器在任何情况下都会自动重启。如果容器停止或出现故障，它会自动启动。这种策略适用于需要持续运行的服务，即使是在服务器重新启动或发生故障时。
- unless-stopped：表示容器只在容器停止后自动启动。如果容器已经在运行中，则不会自动重启。这种策略适用于需要在服务器启动时启动的服务，但不需要在服务器停止时保持运行的服务。

`extra_hosts` 允许你在容器内部的 `/etc/hosts` 文件中添加额外的条目，从而将宿主机的 IP 地址映射到一个或多个别名。

version: "3.9"

services:

  web:

    image: nginx:latest

    extra_hosts:

      - "host.docker.internal:host-gateway"

      - "my-host:192.168.1.100" # 将 my-host 映射到宿主机的 IP 地址

- `host.docker.internal:host-gateway`：这是一个特殊的 DNS 名称，它解析为 Docker 主机的内部 IP 地址。在 Docker Desktop for Mac、Docker Desktop for Windows 和 Linux（使用 Docker Engine 18.03+）上可用。
- `my-host:192.168.1.100`：这是一个自定义的映射，将 `my-host` 别名映射到宿主机的 IP 地址 `192.168.1.100`。你需要将 `192.168.1.100` 替换为你的宿主机的实际 IP 地址。