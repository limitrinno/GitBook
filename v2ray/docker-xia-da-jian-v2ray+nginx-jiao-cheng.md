# docker下搭建v2ray+nginx教程

安装Docker - centos申请SSl-cfdocker下安装nginxdocker下安装v2ray配置V2ray的配置

## 安装Docker - centos

```
docker run -itd --name=v2ray --network=host --restart=always centos:centos7.9.2009
```

```
docker exec -it v2ray bash
```

## 申请SSl-cf

* [https://dash.cloudflare.com/profile/api-tokens](https://dash.cloudflare.com/profile/api-tokens) cf api token

```
wget -N --no-check-certificate https://raw.githubusercontents.com/Misaka-blog/acme-1key/master/acme1key.sh && bash acme1key.sh
```

### 存档

```
cd ~ && yum -y install wget && wget -N --no-check-certificate https://api.2331314.xyz/sh/acme1key.sh && bash acme1key.sh
```

## docker下安装nginx

```
yum -y install epel-release && yum -y install nginx wget vim
```

* vi /etc/nginx/conf.d/v2.conf
* mkdir -p /etc/nginx/ssl && mkdir -p /www/v2
* 申请到的证书放入 /etc/nginx/ssl/ 下

```shell
server {
#       listen       80;
        listen       443 ssl;
        server_name  us.zmlc.club;
        root         /www/v2;
	ssl_certificate  ssl/cert.crt;
	ssl_certificate_key ssl/private.key;
	ssl_session_cache shared:SSL:1m;
	ssl_session_timeout 5m;
	ssl_ciphers HIGH:!aNULL:!MD5;
	ssl_prefer_server_ciphers on;
location /clc51000
{
    proxy_pass http://127.0.0.1:51000;
    proxy_redirect off;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $http_host;
    proxy_read_timeout 300s;
}
}
```

## docker下安装v2ray

```
cd ~
curl -O https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh
curl -O https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-dat-release.sh
bash install-release.sh
bash install-dat-release.sh
systemctl restart v2ray && systemctl enable v2ray
rm -rf install-releases.sh install-dat-release.sh
```

### 配置V2ray的配置

* /usr/local/etc/v2ray/config.json

```
{
  "inbounds": [{
    "port": 51000,
    "listen": "127.0.0.1",
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "a7ffc1e5-5599-4448-a337-bb01c05efa68",
          "level": 1,
          "alterId": 0
        }
      ],
      "disableInsecureEncryption": false
    },
    "streamSettings": {
        "network": "ws",
        "wsSettings": {
            "path": "/clc51000",
            "headers": {
                "Host": "ld.zmlc.club"
            }
        }
    }
  }],
  "outbounds": [{
    "protocol": "freedom",
    "settings": {}
  },{
    "protocol": "blackhole",
    "settings": {},
    "tag": "blocked"
  }],
  "routing": {
    "rules": [
      {
        "type": "field",
        "ip": ["geoip:private"],
        "outboundTag": "blocked"
      }
    ]
  }
}
```

```
nohup /usr/local/bin/v2ray -config /usr/local/etc/v2ray/config.json > /dev/null 2&>1 &
```

\
