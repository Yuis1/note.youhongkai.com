---
{"dg-publish":true,"permalink":"/CS计算机科学/运维部署/Network/Https/Dify中的certbot服务/","title":"Dify中的certbot服务","tags":["clippings"],"noteIcon":"","created":"2025-08-13T20:16:37.068+08:00","updated":"2025-08-16T16:05:11.682+08:00"}
---


官方文档：

docker/certbot/README.md  https://github.com/langgenius/dify/blob/6e6389c930a4aa86d238e4e90c155a80a5548a02/docker/certbot/README.md

---

Original 扫地升 *2025年07月06日 17:43*

本文使用Dify v1.5.0版本，主要介绍了 `Dify` 中的 `certbot` 服务。Certbot 是一个自动获取和更新 HTTPS 证书的工具，通常与 Let's Encrypt 证书颁发机构配合使用。Let's Encrypt 的证书，有效期为 90天。可以在证书到期前更新，通常可以设置每 60天 自动更新一次，以确保证书不会过期。

同时，介绍了新方式（使用certbot容器自动申请、管理和更新Let's Encrypt证书）和旧方式（不使用certbot容器，完全手动管理证书，需要自行获取和更新证书文件）的区别。新方式提供了更现代化和自动化的SSL证书管理方案，而旧方式则保留了手动控制的灵活性和向后兼容性。

## 一.certbot服务配置

这部分配置定义了用于自动获取和管理HTTPS证书的Certbot服务。

![Image](/img/user/Z-attach/Image-8.webp)

### 1.基本配置

- 镜像：使用官方的 `certbot/certbot` 镜像，包含了Let's Encrypt客户端工具
- 启动方式：被定义在 `certbot` profile中，只有通过 `docker-compose --profile certbot up` 命令才会启动

说明：当执行 `docker-compose up` 命令时， `certbot` 服务并不会启动。

### 2.卷挂载

配置了6个卷挂载点：

- `/etc/letsencrypt` ：存储证书、密钥和账户信息
- `/var/www/html` ：用于HTTP验证时存放验证文件
- `/var/log/letsencrypt` ：保存Certbot操作日志
- `/etc/letsencrypt/live` ：包含最新的证书链接
- `/update-cert.template.txt` ：证书更新模板
- `/docker-entrypoint.sh` ：自定义的容器启动脚本

### 3.环境变量

- `CERTBOT_EMAIL` ：用于Let's Encrypt通知的邮箱地址
- `CERTBOT_DOMAIN` ：需要申请证书的域名
- `CERTBOT_OPTIONS` ：可选的额外Certbot参数

### 4.运行配置

- 入口点设置为自定义脚本 `/docker-entrypoint.sh`
- 容器启动后执行 `tail -f /dev/null` 命令保持容器运行而不退出

这种配置适合首次获取证书和自动续期，配合Nginx服务可实现HTTPS安全访问。

### 5..env中CERTBOT相关环境变量

- `CERTBOT_EMAIL` ：用于注册 Let's Encrypt 账户的电子邮件地址。这是获取证书的必要信息，Let's Encrypt 会用它来发送证书过期提醒和重要通知。
- `CERTBOT_DOMAIN` ：需要获取 SSL 证书的域名。应该设置为实际使用的域名，如 `example.com` 或 `dify.yourdomain.com` 。
- `CERTBOT_OPTIONS` ：可以添加额外的 Certbot 命令行选项，例如：
- `--force-renewal` ：强制更新证书
	- `--dry-run` ：测试模式，不实际颁发证书
	- `--test-cert` ：使用测试证书（开发环境使用）
	- `--debug` ：输出调试信息

这些设置需要配合 `NGINX_ENABLE_CERTBOT_CHALLENGE=true` 使用，这样 Nginx 才会接受 Let's Encrypt 验证域名所有权的请求。

> 注解：使用OpenSSL生成自签名证书（适用于测试环境）
>
> ```
> # 1. 生成私钥
> openssl genrsa -out dify.key 2048
> 
> # 2. 生成证书签名请求(CSR)
> openssl req -new -key dify.key -out dify.csr -subj "/CN=你的域名"
> 
> # 3. 生成自签名证书
> openssl x509 -req -days 365 -in dify.csr -signkey dify.key -out dify.crt
> ```
>
> 将生成的 `dify.crt` 和 `dify.key` 文件放到`./nginx/ssl/` 目录下，然后在`.env` 文件中设置：
>
> ```
> NGINX_HTTPS_ENABLED=true
> ```

## 二.dify\\docker\\certbot\\README.md

### 1.简要说明

使用 `docker compose` 的 certbot 配置，兼容旧版本（无需单独 certbot 容器）。使用以下命令启用此功能：

```
docker compose --profile certbot up
```

### 2.使用 SSL 证书启动新服务器的最简单方法

###### 步骤1：获取Let’s Encrypt证书

（1）设置`.env` 文件中的变量

```
NGINX_SSL_CERT_FILENAME=fullchain.pem
NGINX_SSL_CERT_KEY_FILENAME=privkey.pem
NGINX_ENABLE_CERTBOT_CHALLENGE=true
CERTBOT_DOMAIN=your_domain.com
CERTBOT_EMAIL=example@your_domain.com
```

（2）执行以下命令

```
docker network prune  # 删除所有未被任何容器使用的网络资源
docker compose --profile certbot up --force-recreate -d
```

（3）容器启动完成后执行

```
docker compose exec -it certbot /bin/sh /update-cert.sh
```

###### 步骤2：启用HTTPS

（1）在 `.env` 文件中添加如下配置

```
NGINX_HTTPS_ENABLED=true
```

（2）再次执行启动命令

```
docker compose --profile certbot up -d --no-deps --force-recreate nginx
```

说明： `--no-deps` （不启动依赖）：只启动指定的服务（这里是 `nginx` ），而不启动其依赖服务； `--force-recreate` （强制重建）：强制销毁并重新创建指定的容器。

（3）然后就可以通过 HTTPS 访问服务： `https://your_domain.com`

### 3.SSL证书续期

要续期 SSL 证书，执行以下命令：

```
docker compose exec -it certbot /bin/sh /update-cert.sh
docker compose exec nginx nginx -s reload
```

说明：这个命令用于优雅地重新加载 `nginx` 配置，而不会中断服务。它包含几个关键部分： `docker compose exec` \- 在运行中的容器内执行命令； `nginx` \- 目标容器名称； `nginx -s reload` \- 在容器内执行的实际命令。

### 4.Certbot可选项

可以通过 `CERTBOT_OPTIONS` 设置 certbot 的附加选项。例如用于测试的参数：

```
CERTBOT_OPTIONS=--dry-run
```

如果修改了 `CERTBOT_OPTIONS` ，请先重新生成 certbot 容器，再更新证书：

```
docker compose --profile certbot up -d --no-deps --force-recreate certbot
docker compose exec -it certbot /bin/sh /update-cert.sh
```

如有必要，再重载 nginx：

```
docker compose exec nginx nginx -s reload
```

### 5.兼容旧服务器方式

如果希望使用旧方式，即证书文件存放在 `nginx/ssl` 目录中，只需不加 `--profile certbot` 参数启动容器即可：

```
docker compose up -d
```

## 三.dify\\docker\\certbot\\update-cert.template.txt

这个Bash脚本用于自动管理Let's Encrypt SSL证书的获取和更新。此脚本设计用于Docker环境中运行，作为自动化SSL证书管理解决方案的一部分。

```
#!/bin/bash
set -e

DOMAIN="${CERTBOT_DOMAIN}"
EMAIL="${CERTBOT_EMAIL}"
OPTIONS="${CERTBOT_OPTIONS}"
CERT_NAME="${DOMAIN}"# 証明書名をドメイン名と同じにする

# Check if the certificate already exists
if [ -f "/etc/letsencrypt/renewal/${CERT_NAME}.conf" ]; then
echo"Certificate exists. Attempting to renew..."
  certbot renew --noninteractive --cert-name ${CERT_NAME} --webroot --webroot-path=/var/www/html --email ${EMAIL} --agree-tos --no-eff-email ${OPTIONS}
else
echo"Certificate does not exist. Obtaining a new certificate..."
  certbot certonly --noninteractive --webroot --webroot-path=/var/www/html --email ${EMAIL} --agree-tos --no-eff-email -d ${DOMAIN}${OPTIONS}
fi
echo"Certificate operation successful"
# Note: Nginx reload should be handled outside this container
echo"Please ensure to reload Nginx to apply any certificate changes."
```

### 1.变量设置

- `DOMAIN`: 从环境变量获取的域名
- `EMAIL`: 用于Let's Encrypt通知的邮箱地址
- `OPTIONS`: certbot的额外配置选项
- `CERT_NAME`: 证书名称（设为与域名相同）

### 2.工作流程

**（1）检查证书是否存在**

通过检查 `/etc/letsencrypt/renewal/${CERT_NAME}.conf` 文件判断。

**（2）证书管理逻辑**

- 如果证书已存在：执行更新操作
- 如果证书不存在：获取新证书

**（3）证书获取方式**

- 使用webroot方式验证域名所有权
- 指定web根目录为 `/var/www/html`

### 3.注意事项

- 脚本采用非交互模式运行
- 证书操作完成后，需要在容器外部重新加载Nginx以应用新证书
- 脚本设置了 `set -e` 标志，任何命令失败都会导致脚本立即退出

## 四.dify\\docker\\certbot\\docker-entrypoint.sh

这是一个 Docker 容器的入口点脚本，专门为 Certbot（Let's Encrypt 证书管理工具）设计。这个脚本是典型的 Docker 容器初始化流程：先进行环境准备和检查，然后将控制权交给实际的应用程序命令。

```
#!/bin/sh
set -e

printf'%s\n'"Docker entrypoint script is running"

printf'%s\n'"\nChecking specific environment variables:"
printf'%s\n'"CERTBOT_EMAIL: ${CERTBOT_EMAIL:-Not set}"
printf'%s\n'"CERTBOT_DOMAIN: ${CERTBOT_DOMAIN:-Not set}"
printf'%s\n'"CERTBOT_OPTIONS: ${CERTBOT_OPTIONS:-Not set}"

printf'%s\n'"\nChecking mounted directories:"
for dir in"/etc/letsencrypt""/var/www/html""/var/log/letsencrypt"; do
if [ -d "$dir" ]; then
printf'%s\n'"$dir exists. Contents:"
        ls -la "$dir"
else
printf'%s\n'"$dir does not exist."
fi
done

printf'%s\n'"\nGenerating update-cert.sh from template"
sed -e "s|\${CERTBOT_EMAIL}|$CERTBOT_EMAIL|g" \
    -e "s|\${CERTBOT_DOMAIN}|$CERTBOT_DOMAIN|g" \
    -e "s|\${CERTBOT_OPTIONS}|$CERTBOT_OPTIONS|g" \
    /update-cert.template.txt > /update-cert.sh

chmod +x /update-cert.sh

printf'%s\n'"\nExecuting command:""$@"
exec"$@"
```

### 1.基本结构

- 使用 `/bin/sh` 作为解释器
- 设置 `set -e` 确保任何命令失败时脚本立即停止

### 2.环境检查与调试

（1）打印运行状态消息

（2）检查并显示关键环境变量

- `CERTBOT_EMAIL` ：Let's Encrypt 账户邮箱
- `CERTBOT_DOMAIN` ：需要申请证书的域名
- `CERTBOT_OPTIONS` ：附加的 Certbot 选项

（3）检查必要的挂载目录

- `/etc/letsencrypt` ：证书和配置存储目录
- `/var/www/html` ：用于 HTTP 验证的 Web 根目录
- `/var/log/letsencrypt` ：日志目录

### 3.证书更新脚本生成

脚本根据模板文件 `/update-cert.template.txt` 动态生成 `/update-cert.sh` ：

- 使用 `sed` 命令将环境变量替换到模板中
- 添加执行权限到生成的脚本

### 4.命令执行

最后执行传递给容器的命令（通过 Docker 的 CMD 或运行时参数提供）。

## 五.Certbot配置新旧方式

### 1.Certbot配置新旧方式区别

新方式主要优势在于自动化程度更高，简化了SSL证书的获取和更新流程，特别适合新服务器的部署。旧方式的主要特点是所有证书管理流程都需要手动操作，没有自动化获取或更新证书的机制。

（1）新方式（使用certbot容器）

- 使用 `--profile certbot` 参数启动容器： `docker compose --profile certbot up`
- 通过环境变量配置证书设置（如 `CERTBOT_DOMAIN` 、 `CERTBOT_EMAIL` 等）
- 证书自动化获取流程：先启动容器，然后执行 `update-cert.sh` 脚本
- 支持通过 `CERTBOT_OPTIONS` 自定义选项（如 `--dry-run` 测试模式）
- 证书更新方便：只需执行两个命令即可更新并重新加载

（2）旧方式（不使用certbot容器的传统方式）

- 不使用 `--profile certbot` 参数： `docker compose up -d`
- 证书文件需手动放置在 `nginx/ssl` 目录中
- 没有自动化获取证书的流程
- 证书更新需手动处理

### 2.Certbot配置新方式

（1）配置`.env` 文件

```
NGINX_SSL_CERT_FILENAME=fullchain.pem
NGINX_SSL_CERT_KEY_FILENAME=privkey.pem
NGINX_ENABLE_CERTBOT_CHALLENGE=true
CERTBOT_DOMAIN=your_domain.com
CERTBOT_EMAIL=example@your_domain.com
```

（2）启动容器

```
docker network prune
docker compose --profile certbot up --force-recreate -d
```

（3）获取证书

```
docker compose exec -it certbot /bin/sh /update-cert.sh
```

（4）启用HTTPS

编辑`.env` 文件，添加以下配置：

```
NGINX_HTTPS_ENABLED=true
```

然后重新启动容器：

```
docker compose --profile certbot up -d --no-deps --force-recreate nginx
```

完成后即可通过HTTPS访问服务器。

### 3.Certbot配置旧方式

旧方式是指不使用certbot容器进行证书管理，需要手动获取和配置SSL证书。配置步骤如下：

（1）手动获取SSL证书

- 使用其它方式（如本地certbot工具、SSL提供商等）获取域名证书
- 准备好证书文件（通常包括 `fullchain.pem` 和 `privkey.pem` 或其它格式的证书文件）

（2）放置证书文件

将证书文件手动放入 `nginx/ssl` 目录。

```
# 例如
cp your_cert.pem ./nginx/ssl/dify.crt
cp your_key.pem ./nginx/ssl/dify.key
```

（3）配置环境变量

编辑`.env` 文件，设置以下参数：

```
NGINX_HTTPS_ENABLED=true
NGINX_SSL_CERT_FILENAME=dify.crt  # 你的证书文件名
NGINX_SSL_CERT_KEY_FILENAME=dify.key  # 你的私钥文件名
```

（4）启动容器

使用普通方式启动容器（不带certbot配置）。

```
docker compose up -d
```

（5）证书更新

- 证书到期前需要手动更新证书
- 获取新证书后，替换 `nginx/ssl` 目录中的文件
- 重新加载nginx配置
```
docker compose exec nginx nginx -s reload
```

### 4.Let's Encrypt 证书目录结构

Certbot容器中的目录结构为SSL证书管理提供了完整的基础设施。

（1） `/etc/letsencrypt` 目录

这是证书管理的核心目录，包含：

- accounts/ - 存储与Let's Encrypt的账户关联信息和注册数据
- archive/ - 保存所有版本的证书和私钥，包括历史版本
- renewal/ - 存储证书的自动续期配置信息
- keys/ - 备份的私钥存储
- csr/ - 存储证书签名请求(Certificate Signing Request)文件

（2） `/var/www/html` 目录

用于HTTP-01验证挑战：

- 当验证域名所有权时，Let's Encrypt会要求在特定URL路径放置验证文件
- Certbot自动在此目录创建`.well-known/acme-challenge/` 子目录并放置验证文件
- Nginx配置中有相应规则将这些验证请求指向此目录

（3） `/var/log/letsencrypt` 目录

- 记录所有Certbot操作的日志，包括证书申请、更新和错误信息
- 对排查证书问题和审计证书操作历史非常有用

（4） `/etc/letsencrypt/live/{domain-name}` 目录

这个特殊目录包含指向最新证书的符号链接：

- cert.pem - 服务器证书（仅包含域名证书部分）
- chain.pem - 中间证书链（CA的证书链）
- fullchain.pem - 完整证书链（域名证书+中间证书链），通常配置在Nginx中
- privkey.pem - 私钥文件，用于解密HTTPS流量

这种设计使Nginx可以始终引用相同的文件路径，而Certbot在更新证书时只需更新这些符号链接的指向。当证书自动更新时，新证书将存储在 `archive` 目录，而 `live` 目录中的链接会自动更新指向最新版本，实现无缝续期。

## 参考文献

\[0\] Dify中的certbot服务：https://z0yrmerhgi8.feishu.cn/wiki/Oj5ww6SWXixhIkksC9ucr0O0nne

\[1\] https://github.com/langgenius/dify/tree/main/docker/certbot  

  

---

知识星球：Dify源码剖析及答疑，Dify扩展系统源码，AI书籍课程|AI报告论文，公众号付费资料。加微信 `buxingtianxia21`

![[NLP工程化知识星球\|NLP工程化知识星球]](https://mp.weixin.qq.com/www.w3.org/2000/svg'%20xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg%20stroke='none'%20stroke-width='1'%20fill='none'%20fill-rule='evenodd'%20fill-opacity='0'%3E%3Cg%20transform='translate(-249.000000,%20-126.000000)'%20fill='%23FFFFFF'%3E%3Crect%20x='249'%20y='126'%20width='1'%20height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

\[NLP工程化知识星球\]

[Read more](https://mp.weixin.qq.com/)

继续滑动看下一个

NLP工程化

向上滑动看下一个