---
{"dg-publish":true,"permalink":"/CS计算机科学/运维部署/Network/Https/Dify不使用默认端口且绑定域名/","noteIcon":"","created":"2025-08-13T04:02:46.027+08:00","updated":"2025-08-13T03:23:18.000+08:00"}
---


Dify自带了Certbot，可以自动申请SSL。详见：[Dify中的certbot服务](Dify中的certbot服务.md)

但是，以下场景不用开启dify的Certbot：

dify通过docker部署，由于宿主机80端口还有其他Web服务，只能将dify自带NGINX的端口绑定到宿主机的 1080, 1443。

并且，还要绑定一个域名，让域名不加端口号可访问，并开通SSL（否则浏览器会提示不安全）。

DS R1 回答：

基于您的场景，我会推荐采用"​**​宿主机Nginx作为反向代理​**​"的解决方案。以下是详细配置步骤和示意图：

### 网络架构示意图

```
用户请求 HTTPS(443) 
       ↓
宿主机Nginx（处理SSL证书） 
       ↓ 代理到内部HTTP
宿主机:1080 → Docker容器:80 (Dify自带Nginx)
```

### 完整配置流程

#### 1. 修改Dify容器的.env配置

```
# 关闭Dify内部HTTPS（由宿主机统一处理）
NGINX_HTTPS_ENABLED=false

# 不使用Certbot（由宿主机统一管理）
# 注释或删除以下Certbot相关配置
# CERTBOT_EMAIL=...
# CERTBOT_DOMAIN=...
```

#### 2. 修改Docker Compose端口映射

```
# docker-compose.yml
services:
  nginx:
    ports:
      - "1080:80"    # 只暴露HTTP端口
    # 移除443端口映射
    # - "1443:443"   ← 删除这行
```

#### 3. 配置宿主机Nginx (关键步骤)

```
# /etc/nginx/conf.d/dify.conf
server {
    listen 80;
    server_name dify.your-domain.com;
    return 301 https://$host$request_uri; # HTTP强制跳转HTTPS
}

server {
    listen 443 ssl;
    server_name dify.your-domain.com;
    
    # SSL证书配置（使用Certbot自动生成）
    ssl_certificate /etc/letsencrypt/live/dify.your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/dify.your-domain.com/privkey.pem;
    
    # 安全增强配置
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    
    # 反向代理到Dify容器
    location / {
        proxy_pass http://localhost:1080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket支持
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
    
    # Certbot证书续期验证路径
    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }
}
```

#### 4. 宿主机配置Certbot

```
# 创建证书验证目录
sudo mkdir -p /var/www/certbot

# 申请证书（使用webroot验证方式）
sudo certbot certonly --webroot -w /var/www/certbot -d dify.your-domain.com

# 设置自动续期（添加到crontab）
sudo crontab -e
# 添加以下内容：
0 3 * * * certbot renew --quiet --post-hook "systemctl reload nginx"
```

#### 5. 完成配置并重启服务

```
# 重启宿主机Nginx
sudo systemctl restart nginx

# 重启Dify容器（在dify/docker目录）
docker compose down
docker compose up -d
```

### 验证与排查指南

1. ​**​证书验证测试​**​：
    

```
sudo certbot renew --dry-run
```

2. ​**​端口连通性检查​**​：
    

```
# 检查宿主机1080端口
curl -I http://localhost:1080

# 检查HTTPS访问
curl -Ik https://dify.your-domain.com
```

3. ​**​Nginx配置测试​**​：
    

```
sudo nginx -t
```

4. ​**​日志排查​**​：
    

```
# Nginx错误日志
tail -f /var/log/nginx/error.log

# Dify容器日志
docker compose logs -f nginx
```

### 故障处理方案

|故障现象|解决方案|
|---|---|
|HTTPS访问显示不安全|1. 检查证书路径是否正确  <br>2. 确保证书包含完整链  <br>3. 执行`sudo certbot certificates`验证|
|502 Bad Gateway|1. 检查Dify容器是否运行  <br>2. 验证`proxy_pass http://localhost:1080`是否可达  <br>3. 检查防火墙设置|
|证书续期失败|1. 验证`/var/www/certbot`写入权限  <br>2. 检查`.well-known`路径是否被其他配置覆盖|
|WebSocket连接失败|1. 确认已添加`Upgrade`和`Connection`头  <br>2. 检查Nginx版本是否支持websocket|

> ​**​关键点总结​**​：通过宿主机Nginx统一管理SSL证书和443端口流量，将HTTPS请求反向代理到Dify容器的HTTP 1080端口，既解决了端口冲突问题，又简化了证书管理。此架构下，Dify容器只需关注应用逻辑，无需处理HTTPS相关配置。