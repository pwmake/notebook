---
title: SSL 购买与设定
date: 2017-12-29
layout: post
feature-img: "https://images.pexels.com/photos/60504/security-protection-anti-virus-software-60504.jpeg?cs=srgb&dl=cyber-security-cybersecurity-device-60504.jpg&fm=jpg"
---

最近公司产品需要上 *阿婆店* ，说是要弄个 SSL，否则无法通过审核。好吧，挊起袖子，干啊……对比一圈，捡了便宜点的 GeoTrust，妈蛋，它家的文档真是屎到没谁了！！！

注意区分 **域名证书** 、**中间CA证书** 以及 **根证书** 。

<!--more-->

## 某些概念

SSL 分几种认证方式：

**域名认证**（Domain Validation）：最低级别认证，可以确认申请人拥有这个域名。对于这种证书，浏览器会在地址栏显示一把锁。

**公司认证**（Company Validation）：确认域名所有人是哪一家公司，证书里面会包含公司信息。

**扩展认证**（Extended Validation）：最高级别的认证，浏览器地址栏会显示公司名

根据覆盖范围分：

> - **单域名证书**：只能用于单一域名，`foo.com`的证书不能用于`www.foo.com`
> - **通配符证书**：可以用于某个域名及其所有一级子域名，比如`*.foo.com`的证书可以用于`foo.com`，也可以用于`www.foo.com`
> - **多域名证书**：可以用于多个域名，比如`foo.com`和`bar.com`

认证级别越高、覆盖范围越广的证书，价格越贵。

之前一直在用 Let's Encrypt 免费的域名认证证书，每3个月手动更新一次，全部自动化，不是太了解其中的原理。

之前的SSL到期，手动更新了一波，发现不生效，死活不能使用 https 访问，以为是 VPS 出问题了，还把 nginx 的 default 配置也删除之，问题依旧……后来放弃折腾了，不管了。偶然机会下，把网站添加到墙规则，居然可以访问了，我艹。切换到手机网络「联通」，不翻墙正常访问。从国外服务器访问也正常。问题确定了，

`天朝电信443端口被封了，无法正常访问！！！`

## 实操

以下一步步来申请 SSL 证书。

- 创建 CSR

linux 下执行

```shell
openssl req -nodes -sha256 -newkey rsa:2048 -keyout zshmobi.key -out zshmobi.csr -subj "/C=CN/ST=Shanghai/L=Shanghai/O=zshmobi/OU=IT/CN=zshmobi.com"
```

或者去除 `-suj "xxx"` 内容，进行问答式设置生成。

获得两个文件： zshmobi.csr「CSR文件」与 zshmobi.key 「私钥文件」

- 获取域名证书

按要求提交上述两个文件，等待获取 `域名证书` ， 假设是 `zshmobi.crt`

```shell
-----BEGIN CERTIFICATE-----
xxxxxxxxx
-----END CERTIFICATE-----
```

此时的域名证书还不能使用，需要添加 `Intermediate CA.crt` ，即 中间CA 证书，通常文档或者邮件会附上，通常会有两段

```shell
-----BEGIN CERTIFICATE-----
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
-----END CERTIFICATE-----
```

接着是 `Root CA.crt` ，根证书，同样会在邮件或者官方文档上提供下载。

将 三者 组合在一起，形成新的 总证书，方能正常使用。

```shell
cat Root Ca.crt >> zshmobi.crt
cat Intermediate CA.crt >> zshmobi.crt
```

- 在nginx中使用

以下是某个虚拟主机的配置片段

```shell
server 
{
    listen       80;
    server_name  zshmobi.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen  443;
    server_name  zshmobi.com;
    ssl     on;
    ssl_certificate /etc/ssl/zshmobi.crt;
    ssl_certificate_key /etc/ssl/zshmobi.key;
    include /usr/local/nginx/conf/options-ssl-nginx.conf;

    index index.html index.php;
    root  /data/web;

    charset utf-8;
    access_log /data/logs/987app.com_access.log;
    error_log /data/logs/987app.com_error.log debug;

    access_log off;

    location ~ .*\.(swf|css|xml|js|jpg|gif|png|mp3|xx|mpt|xmlback)$
    {
        expires      30d;
    }
}
```

具体的 SSL 算法配置是 `options-ssl-nginx.conf`

```shell
# /usr/local/nginx/conf/options-ssl-nginx.conf
# 参考 let's encrypt 脚本自动生成的配置

ssl_session_cache shared:le_nginx_SSL:1m;
ssl_session_timeout 1440m;

ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
ssl_prefer_server_ciphers on;

# 以下注释掉的是默认的配置，存在 DH 弱算法 风险
#ssl_ciphers "ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS:ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4:!DH:!DHE";
ssl_ciphers "ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4:!DH:!DHE";
```

重启 nginx 测试即可。



## 参考链接

> [GeoTrust CA Download](https://knowledge.geotrust.com/support/knowledge-base/index?page=content&id=INFO1421#links)
>
> [SSL安全评估之：myssl ](https://myssl.com/)
>
> [SSL安全评估之：ssllabs](https://www.ssllabs.com/ssltest/analyze.html)
>
> [SSL安全评估之：GeoTrust SSL Report](https://cryptoreport.geotrust.com/checker/)
>
> [nginx 自动跳转 https 设置](https://www.getssl.cn/support/nginx-%E8%87%AA%E5%8A%A8%E8%B7%B3%E8%BD%AC%E5%88%B0https/)



