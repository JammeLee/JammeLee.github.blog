---
layout: post
title: V2ray+ws+tls
date: 2020-03-20 23:04:06
author: JammeLee
subtitle: V2ray
categories:
	- Other
tags: 
	- V2ray
cdn: 'header-off'
header-img: /2018/03/31/hello/wallpaper-2572384.jpg
---
### 安装V2Ray

装机系统版本：CentOs7

首先，安装V2ray：
``` bash
bash <(curl -L -s https://install.direct/go.sh)
```

然后，设置开机启动：
``` bash
systemctl enable v2ray
```

### 安装SSL证书

安装EPEL：
``` bash
yum -y install epel-release
```

安装certbot用于签发SSL证书：
``` bash
yum -y install certbot
```
我这里在安装certbot时，出现了错误：
```bash
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: repos-lax.psychz.net
 * elrepo-kernel: repos.lax-noc.com
 * extras: repos-lax.psychz.net
 * updates: repos-lax.psychz.net
No package certbot available.
Error: Nothing to do
```

需要执行以下几行命令：
```bash
yum remove epel-release
yum clean all
yum -y install yum-utils
yum -y install epel-release
```

申请SSL证书：
```bash
certbot certonly --standalone -d example.com
```
这里的*example.com*替换成自己的域名

如果申请成功，证书和私钥路径如下：
> /etc/letsencrypt/live/example.com/fullchain.pem
> /etc/letsencrypt/live/example.com/privkey.pem

### 配置Nginx

添加一个Nginx安装源：
```bash
vi /etc/yum.repos.d/nginx.repo
```

写入：
```bash
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
```

写入完成按ESC，然后输入:wq，回车保存并退出

安装Nginx：
```bash
yum -y install nginx
```

设置开机启动：
```bash
systemctl enable nginx
```

新建一个Nginx站点配置文件：
```bash
vi /etc/nginx/conf.d/v2ray.conf
```

写入如下内容：（注意example.com请更换为自己的域名）
```bash
server {
    listen       443 ssl;
    server_name  example.com;

    ssl_certificate    /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key    /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    error_page 497  https://$host$request_uri;

location /ray {
    proxy_pass       http://127.0.0.1:10000;
    proxy_redirect             off;
    proxy_http_version         1.1;
    proxy_set_header Upgrade   $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host      $http_host;
    }
}
```

其中443是网站端口同时也是V2Ray传输端口，127.0.0.1:10000其中的10000是监听端口，可以自行更改，然后防火墙放行所需端口，或者直接关闭防火墙

### 配置V2Ray服务端

备份一下v2ray的默认配置文件：
```bash
cp /etc/v2ray/config.json /etc/v2ray/config.json.bak
```

清空配置文件的内容：
```bash
echo "" > /etc/v2ray/config.json
```

编辑配置文件：
```bash
vi /etc/v2ray/config.json
```

写入如下内容：
```bash
{
  "inbounds": [
    {
      "port": 10000,
      "listen":"127.0.0.1",
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "你的UUID",
            "alterId": 64
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "wsSettings": {
        "path": "/ray"
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    }
  ]
}
```

注：`port`字段要用上方nginx配置文件中`proxy_pass`字段里的端口号，客户端中的端口号要使用nginx中`listen`字段填写的端口号。
UUID可以用这个网站生成：https://www.uuidgenerator.net
全部完成之后，关闭系统防火墙或者自行更改配置：

```bash
systemctl stop firewalld.service
```

同时把SELinux也关了：
```bash
vi /etc/selinux/config
SELINUX=disabled
setenforce 0
```

启动v2ray和nginx：
```bash
systemctl start v2ray
systemctl start nginx
```

检查是否运行正常：
```bash
systemctl status v2ray
systemctl status nginx
```

两个都显示为绿色的active(running)则说明运行成功
到此，服务端的配置完成

### CloudFlare

HTTP ports supported by Cloudflare:

- 80
- 8080
- 8880
- 2052
- 2082
- 2086
- 2095
HTTPS ports supported by Cloudflare:

- 443
- 2053
- 2083
- 2087
- 2096
- 8443

参考文章：
https://www.ecsoe.com/archives/38.html











