---
{"dg-publish":true,"permalink":"/CS计算机科学/Drupal/Drupal环境安装/Drupal10无法生成JS、CSS聚合文件/","noteIcon":"","created":"2025-08-26T11:26:34.238+08:00","updated":"2025-09-02T14:42:03.765+08:00"}
---


如果目录权限没问题，那问题就出在Nginx配置文件

[Aggregation not working css and js cache directories not being created [#3381293] | Drupal.org](https://www.drupal.org/project/drupal/issues/3381293#comment-16057123)

参考配置：

```nginx
server {
    listen 80 default_server;

    client_max_body_size 100m;

    root /opt/drupal/web;

    index index.php index.htm index.html;

    sendfile off;
    error_log /var/log/nginx/error.log info;
    access_log /var/log/nginx/access.log;

    location / {
        absolute_redirect off;
        try_files $uri $uri/ /index.php?$query_string;
    }

    location @rewrite {
        rewrite ^ /index.php;
    }

    location ~ ^/sites/.*/files/styles/ {
        try_files $uri @rewrite;
    }

    location ~ '\.php$|^/update.php' {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_buffers 16 16k;
        fastcgi_buffer_size 32k;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param SCRIPT_NAME $fastcgi_script_name;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_intercept_errors off;
        fastcgi_read_timeout 10m;
        fastcgi_param SERVER_NAME $host;
        fastcgi_pass_header "X-Accel-Buffering";
        fastcgi_pass_header "X-Accel-Charset";
        fastcgi_pass_header "X-Accel-Expires";
        fastcgi_pass_header "X-Accel-Limit-Rate";
        fastcgi_pass_header "X-Accel-Redirect";
    }

    location ~* /\.(?!well-known\/) {
        deny all;
    }

    location ~* (?:\.(?:bak|conf|dist|fla|in[ci]|log|psd|sh|sql|sw[op])|~)$ {
        deny all;
    }

    location ^~ /system/files/ {
        log_not_found off;
        access_log off;
        expires 30d;
        try_files $uri @rewrite;
    }

    location ~* \.(jpg|jpeg|gif|png|ico|cur|gz|svg|svgz|mp4|ogg|ogv|webm|webp|htc)$ {
        try_files $uri @rewrite;
        expires max;
        log_not_found off;
    }

    location ~* \.(js|css)$ {
        try_files $uri @rewrite;
        expires -1;
        log_not_found off;
    }
}
```

the most important part:

```nginx
    location ~* \.(js|css)$ {
        try_files $uri @rewrite;
        log_not_found off;
    }
```

That **@rewrite** is important. When the aggregated files are not found on the filesystem, nginx will pass the request to the front controller index.php for Drupal to regenerate the missing files.

Not sure what the actual flow is, but from my observation:

- when clearing cache, or drush cr, the _files/css_ and _files/js_ folder are deleted and not recreated
- on the first request to the aggregated files, Drupal will see that the files are not there and regenerate them. But if nginx doesn't have the @rewrite, the request would never reach Drupal, thus, no aggregated file is recreated

## 1Panel 安装Drupal

1Panel默认提供的drupal伪静态规则不对，会造成重复重定向。

正确的应该是：

```
location / {
    try_files $uri /index.php?$query_string;
}
```