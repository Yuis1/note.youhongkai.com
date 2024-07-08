---
{"dg-publish":true,"permalink":"/CS计算机科学/运维部署/Network/正确使用 acme.sh， 让你的网站永久免费使用 ssl 证书/","noteIcon":"","created":"2024-06-22T22:30:29.127+08:00","updated":"2024-04-27T01:21:54.000+08:00"}
---

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/670418258)

![](/img/user/Z-attach/v2-d029b8c8efe89ca7322eb9eeea6f25b7_r.jpg.png)

[Let's Encrypt(加密) - 免费的 SSL/TLS 证书 (letsencrypt.org)](https://link.zhihu.com/?target=https%3A//letsencrypt.org/zh-cn/)

**acme.sh** 实现了 `acme` 协议, 可以从 letsencrypt 生成免费的证书.

主要步骤:

1.  安装 **acme(极致).sh**
2.  生成证书
3.  copy 证书到 nginx([网络] 服务器)/apache 或者其他服务
4.  更新证书
5.  配置服务器 nginx
6.  更新 **acme(极致).sh**
7.  出错怎么办, 如何调试

下面详细介绍.

1. 安装 **acme(极致).sh**
---------------------

安装很简单, 一个命令:

```
curl https://get.acme.sh | sh -s email=my@example.com
```

普通用户和 root 用户都可以安装使用. 安装过程进行了以下几步:

1.  把 acme.sh 安装到你的 **home** 目录下:

```
~/.acme.sh/
```

并创建 一个 shell 的 alias, 例如 .bashrc，方便你的使用: `alias acme.sh=~/.acme.sh/acme.sh`

1.  自动为你创建 cronjob, 每天 0:00 点自动检测所有的证书, 如果快过期了, 需要更新, 则会自动更新证书.

更高级的安装选项请参考: [https(超文本传输协议安全（Hyper Te...)://github.com/Neilpang/acme.sh/wiki(维基)/How-to-install](https://link.zhihu.com/?target=https%3A//github.com/Neilpang/acme.sh/wiki/How-to-install)

**安装过程不会污染已有的系统任何功能和文件**, 所有的修改都限制在安装目录中: `~/.acme.sh/`

2. 生成证书
-------

**acme.sh** 实现了 **acme** 协议支持的所有验证协议. 一般有两种方式验证: http 和 dns 验证.

### 1. http 方式需要在你的网站根目录下放置一个文件, 来验证你的域名所有权, 完成验证. 然后就可以生成证书了.

```
acme.sh --issue -d mydomain.com -d www.mydomain.com --webroot /home/wwwroot/mydomain.com/
```

只需要指定域名, 并指定域名所在的网站根目录. **acme.sh** 会全自动的生成验证文件, 并放到网站的根目录, 然后自动完成验证. 最后会聪明的删除验证文件. 整个过程没有任何副作用.

如果你用的 **apache** 服务器, **acme.sh** 还可以智能的从 **apache** 的配置中自动完成验证, 你不需要指定网站根目录:

```
acme.sh --issue -d mydomain.com --apache
```

如果你用的 **nginx** 服务器, 或者反代, **acme.sh** 还可以智能的从 **nginx** 的配置中自动完成验证, 你不需要指定网站根目录:

```
acme.sh --issue -d mydomain.com --nginx
```

**注意, 无论是 apache 还是 nginx 模式, acme.sh 在完成验证之后, 会恢复到之前的状态, 都不会私自更改你本身的配置. 好处是你不用担心配置被搞坏, 也有一个缺点, 你需要自己配置 ssl 的配置, 否则只能成功生成证书, 你的网站还是无法访问 https. 但是为了安全, 你还是自己手动改配置吧.**

如果你还没有运行任何 web 服务, **80** 端口是空闲的, 那么 **acme.sh** 还能假装自己是一个 webserver, 临时听在 **80** 端口, 完成验证:

```
acme.sh --issue -d mydomain.com --standalone
```

更高级的用法请参考: [https(超文本传输协议安全（Hyper Te...)://github.com/Neilpang/acme.sh/wiki(维基)/How-to-issue-a-cert(证书)](https://link.zhihu.com/?target=https%3A//github.com/Neilpang/acme.sh/wiki/How-to-issue-a-cert)

### 2. 手动 dns 方式, 手动在域名上添加一条 txt 解析记录, 验证域名所有权.

这种方式的好处是, 你不需要任何服务器, 不需要任何公网 ip, 只需要 dns 的解析记录即可完成验证. 坏处是，如果不同时配置 Automatic DNS API，使用这种方式 acme.sh 将无法自动更新证书，每次都需要手动再次重新解析验证域名所有权。

```
acme.sh --issue --dns -d mydomain.com \
 --yes-I-know-dns-manual-mode-enough-go-ahead-please
```

然后, **acme.sh** 会生成相应的解析记录显示出来, 你只需要在你的域名管理面板中添加这条 txt 记录即可.

等待解析完成之后, 重新生成证书:

```
acme.sh --renew -d mydomain.com \
  --yes-I-know-dns-manual-mode-enough-go-ahead-please
```

注意第二次这里用的是 `--renew`

dns 方式的真正强大之处在于可以使用域名解析商提供的 api 自动添加 txt 记录完成验证.

**acme(极致).sh** 目前支持 cloudflare([网络] 云端的服务), dnspod, cloudxns, godaddy([网络] 狗爹) 以及 ovh 等数十种解析商的自动集成.

以 dnspod 为例, 你需要先登录到 dnspod 账号, 生成你的 api id 和 api key, 都是免费的. 然后:

```
export DP_Id="1234"

export DP_Key="sADDsdasdgdsf"

acme.sh --issue --dns dns_dp -d aa.com -d www.aa.com
```

证书就会自动生成了. 这里给出的 api id 和 api key 会被自动记录下来, 将来你在使用 dnspod api 的时候, 就不需要再次指定了. 直接生成就好了:

```
acme.sh --issue -d mydomain2.com --dns  dns_dp
```

更详细的 api 用法: [https(超文本传输协议安全（Hyper Te...)://github.com/Neilpang/acme.sh/blob(大块头)/master/dnsapi/README.md](https://link.zhihu.com/?target=https%3A//github.com/Neilpang/acme.sh/blob/master/dnsapi/README.md)

3. copy / 安装 证书
---------------

前面证书生成以后, 接下来需要把证书 copy 到真正需要用它的地方.

注意, 默认生成的证书都放在安装目录下: `~/.acme.sh/`, 请不要直接使用此目录下的文件, 例如: 不要直接让 nginx/apache 的配置文件使用这下面的文件. 这里面的文件都是内部使用, 而且目录结构可能会变化.

正确的使用方法是使用 `--install-cert` 命令, 并指定目标位置, 然后证书文件会被 copy 到相应的位置, 例如:

### Apache example:

```
acme.sh --install-cert -d example.com \
--cert-file      /path/to/certfile/in/apache/cert.pem  \
--key-file       /path/to/keyfile/in/apache/cert.key  \
--fullchain-file /path/to/fullchain/certfile/apache/fullchain.pem \
--reloadcmd     "service apache2 force-reload"
```

### Nginx([网络] 服务器) example:

```
acme.sh --install-cert -d example.com \
--key-file       /path/to/keyfile/in/nginx/key.pem  \
--fullchain-file /path/to/fullchain/nginx/cert.pem \
--reloadcmd     "service nginx force-reload"
```

(一个小提醒, 这里用的是 `service nginx force-reload`, 不是 `service nginx reload`, 据测试, `reload` 并不会重新加载证书, 所以用的 `force-reload`)

Nginx 的配置 `ssl_certificate` 使用 `/etc/nginx/ssl/fullchain.cer` ，而非 `/etc/nginx/ssl/<domain>.cer` ，否则 [SSL Labs](https://link.zhihu.com/?target=https%3A//www.ssllabs.com/ssltest/) 的测试会报 `Chain issues Incomplete` 错误。

`--install-cert`命令可以携带很多参数, 来指定目标文件. 并且可以指定 reloadcmd, 当证书更新以后, reloadcmd 会被自动调用, 让服务器生效.

详细参数请参考: [https(超文本传输协议安全（Hyper Te...)://github.com/Neilpang/acme.sh#3-install-the-issued-cert(证书)-to-apachenginx-etc](https://link.zhihu.com/?target=https%3A//github.com/Neilpang/acme.sh%233-install-the-issued-cert-to-apachenginx-etc)

值得注意的是, 这里指定的所有参数都会被自动记录下来, 并在将来证书自动更新以后, 被再次自动调用.

4. 查看已安装证书信息
------------

```
acme.sh --info -d example.com
# 会输出如下内容：
DOMAIN_CONF=/root/.acme.sh/example.com/example.com.conf
Le_Domain=example.com
Le_Alt=no
Le_Webroot=dns_ali
Le_PreHook=
Le_PostHook=
Le_RenewHook=
Le_API=https://acme-v02.api.letsencrypt.org/directory
Le_Keylength=
Le_OrderFinalize=https://acme-v02.api.letsencrypt.org/acme/finalize/23xxxx150/781xxxx4310
Le_LinkOrder=https://acme-v02.api.letsencrypt.org/acme/order/233xxx150/781xxxx4310
Le_LinkCert=https://acme-v02.api.letsencrypt.org/acme/cert/04cbd28xxxxxx349ecaea8d07
Le_CertCreateTime=1649358725
Le_CertCreateTimeStr=Thu Apr  7 19:12:05 UTC 2022
Le_NextRenewTimeStr=Mon Jun  6 19:12:05 UTC 2022
Le_NextRenewTime=1654456325
Le_RealCertPath=
Le_RealCACertPath=
Le_RealKeyPath=/etc/acme/example.com/privkey.pem
Le_ReloadCmd=service nginx force-reload
Le_RealFullChainPath=/etc/acme/example.com/chain.pem
```

5. nginx([网络] 服务器) 配置
---------------------

执行完成第 3 步后，不出意外会你的目录下面 `/path/to/keyfile/in/nginx/` 生成两个文件 `key.pem` 和 `cert.pem` 一个是证书文件，一个密钥文件.

打开 nginx.conf 或者是你单独的配置文件，添加一下内容：

```
server {
     #HTTPS的默认访问端口443。
     #如果未在此处配置HTTPS的默认访问端口，可能会造成Nginx无法启动。
     listen 443 ssl;

     #填写证书绑定的域名
     server_name example.com; # 需要根据自己实际的域名配置

     #填写证书文件绝对路径
     ssl_certificate "/path/to/keyfile/in/nginx/cert.pem";
     #填写证书私钥文件绝对路径
     ssl_certificate_key "/path/to/keyfile/in/nginx/key.pem";

     ssl_session_cache shared:SSL:1m;
     ssl_session_timeout 5m;

     #自定义设置使用的TLS协议的类型以及加密套件（以下为配置示例，请您自行评估是否需要配置）
     #TLS协议版本越高，HTTPS通信的安全性越高，但是相较于低版本TLS协议，高版本TLS协议对浏览器的兼容性较差。
     #ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
     #ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3;
     #表示优先使用服务端加密套件。默认开启
     ssl_prefer_server_ciphers on;

    root /wwwroot/example.com;
    access_log /var/log/nginx/example_com_access.log;
    error_log /var/log/nginx/example_com_error.log;
    location / {
           #root html;
           index index.html index.htm;
    }
}
```

配置完成, 执行命令 `nginx -s reload` 重启 nginx 即可.

6. 更新证书
-------

目前证书在 60 天以后会自动更新, 你无需任何操作. 今后有可能会缩短这个时间, 不过都是自动的, 你不用关心.

请确保 cronjob 正确安装, 看起来是类似这样的:

```
crontab  -l

56 * * * * "/root/.acme.sh"/acme.sh --cron --home "/root/.acme.sh" > /dev/null
```

7. 关于修改 ReloadCmd
-----------------

目前修改`ReloadCmd`没有专门的命令，可以通过重新安装证书来实现修改`reloadCmd`的目的。 此外，安装证书后，相关信息是保存在`~/.acme.sh/example.com/example.conf`文件下的，内容就是`acme.sh --info -d example.com`输出的信息，不过`ReloadCmd`在文件中使用了 Base64 编码。理论上可以通过直接修改该文件来修改`ReloadCmd`，且修改时，无需 Base64 编码，直接写命令原文`acme.sh`也可以识别。 不过，`example.conf`文件的位置和内容格式以后可能会改变！`example.conf`一直都是内部使用, 后面有可能会改为用 sqlite 或者 mysql 格式存储. 所以一般不建议自己修改。

8. 更新 acme(极致).sh
-----------------

目前由于 acme 协议和 letsencrypt CA 都在频繁的更新, 因此 acme.sh 也经常更新以保持同步.

升级 acme(极致).sh 到最新版 :

```
acme.sh --upgrade
```

如果你不想手动升级, 可以开启自动升级:

```
acme.sh --upgrade --auto-upgrade
```

之后, acme.sh 就会自动保持更新了.

你也可以随时关闭自动更新:

```
acme.sh --upgrade --auto-upgrade  0
```

9. 出错怎么办：
---------

如果出错, 请添加 debug log：

```
acme.sh --issue  .....  --debug
```

或者：

```
acme.sh --issue  .....  --debug  2
```

请参考： [https(超文本传输协议安全（Hyper Te...)://github.com/Neilpang/acme.sh/wiki(维基)/How-to-debug-acme(极致).sh](https://link.zhihu.com/?target=https%3A//github.com/Neilpang/acme.sh/wiki/How-to-debug-acme.sh)

在 DNS 验证模式下如果 debug 中出现诸如 "timed out" 等字样可能是因为 GFW 拦截了相应请求，需要添加 http(s) proxy 环境变量。（请按照自己实际设定修改）

```
export http_proxy="socks5h://localhost:1081" && export https_proxy="socks5h://localhost:1081"
```

如果是使用 docker 则完整示例配置如下：

```
docker run --rm  -it  \
  -v "/etc/acme":/acme.sh  \
  -e "CF_Token=[填入自己的信息]" \
  -e "CF_Account_ID=[填入自己的信息]" \
  -e "CF_Zone_ID=[填入自己的信息]" \
  -e http_proxy="socks5h://[代理A]:1234" \
  -e https_proxy="socks5h://[代理A]:1234" \
  --network container:[代理A]\
  neilpang/acme.sh \
  --issue -d example.com --dns dns_cf --debug
```

上述例子中使用 cloudflare 的 DNS 来签发证书，并通过把 acme.sh 链接到容器 [代理 A]，来转发 curl 请求（请按照自己实际设定修改）

最后, 本文并非完全的使用说明, 还有很多高级的功能, 更高级的用法请参看其他 wiki 页面.

[https(超文本传输协议安全（Hyper Te...)://github.com/Neilpang/acme.sh/wiki(维基)](https://link.zhihu.com/?target=https%3A//github.com/Neilpang/acme.sh/wiki)

10. 最后需要的注意点
------------

部署成功后你可以在浏览器中使用 https 去浏览你的网址，如果网页没有任何输出，或者是报错找不到服务的时候，如果你是新买的云服务的话，那么你需要在你的服务器开发对应的 ssl 端口 443 。比如阿里云：

![](/img/user/Z-attach/v2-8156bd0091a170024942abc6a76a0e84_r.jpg.png)

如此你就可以愉快的上云玩了。