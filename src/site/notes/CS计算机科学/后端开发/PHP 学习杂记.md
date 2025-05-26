---
{"dg-publish":true,"permalink":"/CS计算机科学/后端开发/PHP 学习杂记/","tags":["PHP"],"noteIcon":"","created":"2024-06-22T22:30:28.743+08:00","updated":"2024-04-24T00:01:43.000+08:00"}
---


作者: 游鱼思

---
# PHP 学习杂记

## 环境部署

最好安装php7.3，因为目前CentOS yum最高支持就是7.3，各方面配套比较好，不用自己编译 pecl 等乱七八糟的。

但是，如果在php8下已经安装好了drupal，那么再切到7，就会出现：Your Composer dependencies require a PHP version ">= 8.1.0".

下面的安装命令是php7.3版本的，注意：一定要清楚自己php版本是多少，然后安装对应版本，否则后面出问题

yum -y install php73-php-devel php73-php-pear

centos 7 没有提供php8以上版本的 pecl ，得自己编译安装 https://www.w3cschool.cn/phpchinese/download-pecl-extension-library.html

高版本的PHP及配套扩展，在CentOS下可以使用 Remi 源来直接安装，详见： [Linux-使用Remi源安装最新版PHP](https://cloud.tencent.com/developer/article/2285043)

### 宝塔的坑

宝塔的php7.4不带pecl，默认的php7.3却自带。安装完7.3后，pecl命令直接可用。

自己编译，还得处理依赖、路径，可麻烦了。

如果在宝塔安装多个php版本，那么 php-cli 默认版本需要设置：网站-php项目-高级
