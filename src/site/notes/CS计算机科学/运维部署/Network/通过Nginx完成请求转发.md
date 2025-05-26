---
{"dg-publish":true,"permalink":"/CS计算机科学/运维部署/Network/通过Nginx完成请求转发/","noteIcon":"","created":"2024-06-22T22:30:29.056+08:00","updated":"2024-04-24T00:31:17.000+08:00"}
---


作者：游鱼思

---
## 问题描述

建立一个代理服务器，进行特定接口的转发。例如，客户端请求 https://example.com/priapi/v5/ecotrade/public/trade-data 时，我的服务器会把请求转发到 https://www.okx.com/priapi/v5/ecotrade/public/trade-data

通过Nginx配置完成请求的转发是一个非常常见且有效的方法。Nginx作为一个高性能的HTTP和反向代理服务器，能够处理这种代理和请求转发的需求。以下是一个基本的Nginx配置示例，演示如何将客户端对`https://example.com/priapi/v5/ecotrade/public/trade-data`的请求转发到`https://www.okx.com/priapi/v5/ecotrade/public/trade-data`。

### 基本的Nginx配置步骤：

1. **打开你的Nginx配置文件**：这通常是`nginx.conf`，或者在`/etc/nginx/sites-available/`下的一个专用配置文件。
2. **配置一个新的server块**（如果`example.com`还未配置）或编辑已有的对应`server`块，在其中加入`location`指令来定义请求转发规则。

```nginx
server {
    listen 80; # 或者443如果是SSL
    server_name example.com;
    
	location /priapi/v5/ecotrade/public/trade-data {
	    proxy_pass https://www.okx.com/priapi/v5/ecotrade/public/trade-data;
	    proxy_set_header Host www.okx.com;
	    proxy_set_header X-Real-IP $remote_addr;
	    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	    proxy_set_header X-Forwarded-Proto $scheme;
	
	    # 根据需要添加的SSL配置
	    proxy_ssl_server_name on; # 开启SNI支持
	    proxy_ssl_verify on; # 如果你完全信任上游服务器，可以暂时关闭SSL证书验证进行测试
	}

}
```

-  当客户端请求包含查询参数（如`latestNum=7&uniqueName=C343256953163322&t=1711209292866`）时，这些参数会原封不动地被转发到目标服务器。
- `proxy_pass` 指令指定了所有匹配该location的请求应该被转发到的地址。Nginx会自动将原始请求的查询参数附加到`proxy_pass`指令指定的URL后面。
- `proxy_set_header` 指令用于设置或修改传递给代理服务器的请求头。这对于确保目标服务器接收到所有必要的信息很有帮助，比如原始的主机名和客户端IP地址。
- **`proxy_ssl_server_name on;`**：这一行启用了SNI（Server Name Indication），允许Nginx在SSL握手过程中发送目标域名。这对于目标服务器托管多个SSL证书的情况非常重要。 —— 非常重要，不开启容易找不到目标资源。
- **`proxy_ssl_verify off;`**：这将禁用代理SSL证书的验证。这对于调试有用，但在生产环境中，你应该启用它来确保安全。如果启用，你可能还需要设置`proxy_ssl_trusted_certificate`指令，指向包含了所有上游服务器证书的可信证书文件。

3. **保存配置并重启Nginx**：更改后，你需要重新加载或重启Nginx服务以应用新的配置。你可以使用以下命令之一

```bash
sudo nginx -s reload
# 或
sudo systemctl restart nginx
```

### 注意事项：

- **HTTPS支持**：如果你的服务需要通过HTTPS提供，确保你已经正确设置了SSL证书，并且`listen`指令设置为`443 ssl`。此外，你可能需要添加`ssl_certificate`和`ssl_certificate_key`指令来指定你的证书文件。
- **头部转发**：上面的配置中包括了一些`proxy_set_header`指令，确保请求在转发过程中保持正确的头信息。这对于保持客户端信息和支持加密协议很重要。
- **域名解析**：确保`example.com`已经正确解析到托管Nginx的服务器上。

通过上述配置，Nginx将能够接收到对`example.com`的特定请求，并将这些请求转发到`www.okx.com`，同时保留原始请求路径和查询参数。这是一个高效且强大的方法来实现请求的代理和转发。