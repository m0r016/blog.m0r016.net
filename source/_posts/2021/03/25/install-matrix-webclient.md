---
title: matrixのwebclientを構築する
date: 2021-03-25 20:14:43
categories: [Ubuntu, matrix]
tags:
- element
description: "matrixを構築する。"
---
### はじめに
[前回](https://blog.slum.cloud/2021/03/24/install-matrix/)はmatrixのインストールした。
matrixのwebclientが存在するらしいのでビルドする。
ソースは[ここから](https://github.com/vector-im/element-web)。

### 1.nginxをいじる
nginxの設定ファイルにアクセスし、設定を書き足す。
```
nano /etc/nginx/sites-enabled/matrix.slum.cloud
server {
        listen 80 ;
        listen [::]:80 ;
        location / { return 301 https://$host$request_uri; }
        server_name matrix.slum.cloud;
}
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    # For the federation port
    listen 8448 ssl http2 ;
    listen [::]:8448 ssl http2 ;

    server_name matrix.slum.cloud;

    add_header X-Frame-Options SAMEORIGIN;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header Content-Security-Policy "frame-ancestors 'none'";

    location / {
        proxy_pass http://localhost:8008;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $host;
        client_max_body_size 50M;
    }

    ssl_certificate /etc/letsencrypt/live/matrix.slum.cloud/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/matrix.slum.cloud/privkey.pem;
}
```

### 2.buildする
```
sudo apt install nodejs
sudo npm install --global yarn
cd ~/
git clone https://github.com/vector-im/element-web.git
cd element-web/
git checkout v1.7.23
yarn install
```

### 3.設定ファイルの変更
```
cp config.sample.json config.json
nano config.json
{
    "default_server_config": {
        "m.homeserver": {
            "base_url": "https://matrix.slum.cloud",
            "server_name": "matrix.org"
        },
        "m.identity_server": {
            "base_url": "https://vector.im"
        }
    },
    "disable_custom_urls": false,
    "disable_guests": false,
    "disable_login_language_selector": false,
    "disable_3pid_login": false,
    "brand": "Element",
    "integrations_ui_url": "https://matrix.slum.cloud/",
    "integrations_rest_url": "https://matrix.slum.cloud/api",
    "integrations_widgets_urls": [
        "https://matrix.slum.cloud/_matrix/integrations/v1",
        "https://matrix.slum.cloud/api",
        "https://matrix.slum.cloud/_matrix/integrations/v1",
        "https://matrix.slum.cloud/api",
        "https://matrix.slum.cloud/scalar/api"
    ],
    "bug_report_endpoint_url": "https://element.io/bugreports/submit",
    "defaultCountryCode": "GB",
    "showLabsSettings": false,
    "features": {
        "feature_new_spinner": false
    },
    "default_federate": true,
    "default_theme": "light",
    "roomDirectory": {
        "servers": [
            "matrix.slum.cloud"
        ]
    },
    "piwik": {
        "url": "https://piwik.riot.im/",
        "whitelistedHSUrls": ["https://matrix.org"],
        "whitelistedISUrls": ["https://vector.im", "https://matrix.org"],
        "siteId": 1
    },
    "enable_presence_by_hs_url": {
        "https://matrix.org": false,
        "https://matrix-client.matrix.org": false
    },
    "settingDefaults": {
        "breadcrumbs": true
    },
    "jitsi": {
        "preferredDomain": "jitsi.riot.im"
    }
}
```

### 4.デプロイする。
```
yarn dist
```

何も問題が出ていなければいけているはず。

### 5.matrix側の設定をする。
```
cd ~/synapse
nano homeserver.yaml
web_client_location: /home/matrix/element-web/webapp/
  - port: 8008
    tls: false
    type: http
    x_forwarded: true
    bind_addresses: ['::1', '127.0.0.1']

    resources:
      - names: [client, federation, webclient]
        compress: false

```

### 6.確認
[matrix.slum.cloud](https://matrix.slum.cloud)にアクセスする。
{% asset_img login.png %}
構築成功、ブラウザからでもアクセスできるようになった。
