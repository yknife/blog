---
title: nginx上部署hexo同时配置https
tags: 
 - nginx
 - hexo
 - https
comments: true
date: 2018-05-02 00:34:14
---
### 阅读这篇文章的场景
你已经有了自己hexo博客，然后想把你的hexo博客变成https，找了网上其他的方式要收费或者太复杂，兴许你可以看看这篇文章，可能会对你有帮助。
### 环境
* 服务器：阿里云ecs 1核1G
* os：centos_7

### 前序准备
* 安装nginx
* 安装hexo

### 阿里云申请免费https证书
1. 进入阿里云管理控制台->安全（云盾）->CA证书服务->购买证书。
2. 选择免费的证书，并购买,并等待审核通过
![图片加载中](https://wx2.sinaimg.cn/mw690/7d1cdd3aly1fqzjd9cpclj21kw0xyn66.jpg "选择免费证书")。
3. 补全信息（会填写一些必要信息）。
4. 审核过程中可能会遇到一直在“审核中”，需要在dns解析中增加一条记录。
![图片加载中](https://wx3.sinaimg.cn/mw690/7d1cdd3agy1fqzjqbb1rfj21kw0ycahv.jpg "CA证书审核过程中，增加一条dns记录")
5. 证书状态 (全部) 变成“已签发”，表示申请成功。
6. 下载该证书，选择“Nginx/Tengine”,下载下来的zip中会有两个文件*.pem,*.key,后面nginx配置中会用到。
7. 当然也可以用openSSL等其他的方式获得证书，只是阿里云提供了现成的方式。

### 配置nginx
找到你服务器上对应的nginx.conf文件
```
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user root;#设置成你的服务器用户名
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;


server {
    listen 443;
    server_name localhost;#设置成本地
    ssl on;
    root /root/blog/public;#hexo generate会生成的静态文件夹
    index index.html index.htm;
    ssl_certificate   /root/yknife.pem;#申请证书步骤解压得到
    ssl_certificate_key  /root/yknife.key;#申请证书步骤解压得到
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    location / {
        root /root/blog/public;
        index index.html index.htm;
    }  	
}

}

```
 ### 配置hexo
 1. 找到你的hexo博客中对应的_config.yml文件,找到URL部分
 ```
 # URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: https://www.yoursite.com
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:
 ```
2. 执行
 ```
 hexo generate
 ```
 命令重新生成public文件夹。
3. 执行
```
nginx -s reload
```
加载nginx配置文件。


