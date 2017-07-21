---
title: 使用Nginx代理ws为wss协议
date: 2017-07-21 16:34:44
tags: websocket
categories: Web知识
description:
---


https站点中只能使用wss协议(WebsocketSecure)，需要通过nginx进行一个协议代理，实现ws到wss的转换。

<!-- more -->

## 一、安装nginx

有两种方法。
源码安装和yum安装，都可以。
推荐yum安装，更方便。
源码安装见之前的笔记。


## 二、创建自签名

### 引言

使用HTTP（超文本传输）协议访问互联网上的数据是没有经过加密的。也就是说，任何人都可以通过适当的工具拦截或者监听到在网络上传输的数据流。但是有时候，我们需要在网络上传输一些安全性或者私秘性的数据，譬如：包含信用卡及商品信息的电子订单。这个时候，如果仍然使用HTTP协议，势必会面临非常大的风险。

HTTPS（超文本传输安全）协议无疑可以有效的解决这一问题。所谓HTTPS，**其实就是HTTP和SSL/TLS的组合**，用以提供加密通讯及对网络服务器的身份鉴定。HTTPS的主要思想是在不安全的网络上创建一安全信道，防止黑客的窃听和攻击。

SSL（安全套接层）可以用来对Web服务器和客户端之间的数据流进行加密。

SSL利用非对称密码技术进行数据加密。加密过程中使用到两个秘钥：**一个公钥和一个与之对应的私钥**。使用公钥加密的数据，只能用与之对应的私钥解密；而使用私钥加密的数据，也只能用与之对应的公钥解密。因此，如果在网络上传输的消息或数据流是被服务器的私钥加密的，则只能使用与其对应的公钥解密，从而可以保证客户端与与服务器之间的数据安全。

### 数字证书

在HTTPS的传输过程中，有一个非常关键的角色——数字证书，那什么是数字证书？又有什么作用呢？

所谓数字证书，是一种用于电脑的身份识别机制。由数字证书颁发机构（CA）对使用私钥创建的签名请求文件做的签名（盖章），表示CA结构对证书持有者的认可。

但这种证书需要钱，用于测试的话只需要创建一个自签名证书即可。

### 创建自签名的步骤

*注意：以下步骤仅用于配置内部使用或测试需要的SSL证书。*


#### 第一步：生成私钥

使用openssl工具生成一个RSA私钥
```powershell
$ openssl genrsa -des3 -out yourname.key 2048
```
说明：生成rsa私钥，des3算法，2048位强度，yourname.key是秘钥文件名。

注意：生成私钥，需要提供一个至少4位的密码。

#### 第二步：生成CSR（证书签名请求）

生成私钥之后，便可以创建csr文件了。

此时可以有两种选择。理想情况下，可以将证书发送给证书颁发机构（CA），CA验证过请求者的身份之后，会出具签名证书（很贵）。另外，如果只是内部或者测试需求，也可以使用OpenSSL实现自签名，具体操作如下：
```powershell
$ openssl req -new -key yourname.key -out yourname.csr
```

说明：需要依次输入国家，地区，城市，组织，组织单位，Common Name和Email。其中Common Name，可以写自己的名字或者域名，如果要支持https，Common Name应该与域名保持一致，否则会引起浏览器警告。
```powershell
Country Name (2 letter code) [AU]: CN
State or Province Name (full name) [Some-State]: Shanghai
Locality Name (eg, city) []: Shanghai
Organization Name (eg, company) [Internet Widgits Pty Ltd]: condata
Organizational Unit Name (eg, section) []: info technology
Common Name (e.g. server FQDN or YOUR name) []: demo.condata.cn
Email Address []: caojy@condata.cn
```

#### 第三步：删除私钥中的密码
在第1步创建私钥的过程中，由于必须要指定一个密码。而这个密码会带来一个副作用，那就是在每次Apache启动Web服务器时，都会要求输入密码，这显然非常不方便。要删除私钥中的密码，操作如下：
```powershell
cp server.key server.key.org
openssl rsa -in server.key.org -out server.key
```


#### 第四步：生成自签名证书

如果你不想花钱让CA签名，或者只是测试SSL的具体实现。那么，现在便可以着手生成一个自签名的证书了。

需要注意的是，在使用自签名的临时证书时，浏览器会提示证书的颁发机构是未知的。
```powershell
$ openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```

说明：crt上有证书持有人的信息，持有人的公钥，以及签署者的签名等信息。当用户安装了证书之后，便意味着信任了这份证书，同时拥有了其中的公钥。证书上会说明用途，例如服务器认证，客户端认证，或者签署其他证书。当系统收到一份新的证书的时候，证书会说明，是由谁签署的。如果这个签署者确实可以签署其他证书，并且收到证书上的签名和签署者的公钥可以对上的时候，系统就自动信任新的证书。

#### 第五步：安装私钥和证书

将私钥和证书文件复制到某个目录下(如`/etc/pki`)，然后添加到nginx的配置文件中。


## 配置Nginx

编辑`/etc/nginx/nginx.conf`：

```
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
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
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

    upstream cjywss {
	    server 192.10.10.133:5438;   # 原ws的端口
    }

    server {
	    listen 5555;
	    server_name 192.10.10.133:5555;
	    ssl on;
	    ssl_certificate "/etc/pki/cjywss.crt";      # 刚才生成的crt和key文件写在这里
        ssl_certificate_key "/etc/pki/cjywss.key";
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 SSLv2 SSLv3;

        access_log /var/log/cjy_access_log main;

        location / {
    	    proxy_pass http://cjywss;
    	    proxy_set_header Host $host;
    	    proxy_set_header X-Real-IP $remote_addr;
    	    proxy_set_header X-Forwarded-F $remote_addr;
    	    proxy_http_version 1.1;
    	    # proxy_set_header Upgrade $http_upgrade;
    	    proxy_set_header Upgrade "websocket";
    	    proxy_set_header Connection "Upgrade";
	        proxy_set_header Sec-Websocket-Version 13;
        }

    }
}

```


## 启动Nginx

```
$ nginx -t  	# 查看配置是否正确
$ nginx		# 启动Nginx
```
连接`wss://192.10.10.133:5555`，成功，可以进行通信。

<!-- more -->