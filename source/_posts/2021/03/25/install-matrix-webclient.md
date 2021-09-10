---
title: matrixのwebclientを構築する
date: 2021-03-25 20:14:43
updated: 2021-09-11 02:32:00
categories: [Ubuntu, matrix]
tags:
  - element
description: "matrixを構築する。"
---

### はじめに

[前回](https://blog.slum.cloud/2021/03/24/install-matrix/)は matrix のインストールした。
matrix の webclient が存在するらしいのでビルドする。
ソースは[ここから](https://github.com/vector-im/element-web)。

<!-- more -->
<!-- toc -->

### 1.nginx をいじる

nginx の設定ファイルにアクセスし、設定を書き足す。
{% codeblock /etc/nginx/sites-enabled/matrix.slum.cloud lang:diff %}

- server {
-        listen 80 ;
-        listen [::]:80 ;
-        location / { return 301 https://$host$request_uri; }
-        server_name matrix.slum.cloud;
- }
  server {
  listen 443 ssl http2;
  listen [::]:443 ssl http2;

      // For the federation port
      listen 8448 ssl http2 ;
      listen [::]:8448 ssl http2 ;

      server_name matrix.slum.cloud;

- add_header X-Frame-Options SAMEORIGIN;
- add_header X-Content-Type-Options nosniff;
- add_header X-XSS-Protection "1; mode=block";
- add_header Content-Security-Policy "frame-ancestors 'none'";


    location / {
        proxy_pass http://localhost:8008;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $host;

-        // Nginx by default only allows file uploads up to 1M in size
-        // Increase client_max_body_size to match max_upload_size defined in homeserver.yaml
          client_max_body_size 50M;
      }

      ssl_certificate /etc/letsencrypt/live/matrix.slum.cloud/fullchain.pem;
      ssl_certificate_key /etc/letsencrypt/live/matrix.slum.cloud/privkey.pem;

  }
  {% endcodeblock %}

### 2.build する

{% codeblock terminal lang:bash line_number:false %}
sudo apt install nodejs
sudo npm install --global yarn
cd ~/
git clone https://github.com/vector-im/element-web.git
cd element-web/
git checkout v1.7.23
yarn install
{% endcodeblock %}

### 3.設定ファイルの変更

{% codeblock config.json lang:diff %}
{
"default_server_config": {
"m.homeserver": {

-            "base_url": "https://matrix-client.matrix.org",
-            "server_name": "matrix.org"

*            "base_url": "https://matrix.slum.cloud",
*            "server_name": "matrix.slum.cloud"
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

- "integrations_ui_url": "https://scalar.vector.im/",
- "integrations_rest_url": "https://scalar.vector.im/api",

* "integrations_ui_url": "https://matrix.slum.cloud/",
* "integrations_rest_url": "https://matrix.slum.cloud/api",

- "integrations_widgets_urls": [
-        "https://scalar.vector.im/_matrix/integrations/v1",
-        "https://scalar.vector.im/api",
-        "https://scalar-staging.vector.im/_matrix/integrations/v1",
-        "https://scalar-staging.vector.im/api",
-        "https://scalar-staging.riot.im/scalar/api"
- ],

* "integrations_widgets_urls": [
*        "https://matrix.slum.cloud/_matrix/integrations/v1",
*        "https://matrix.slum.cloud/api",
*        "https://matrix.slum.cloud/_matrix/integrations/v1",
*        "https://matrix.slum.cloud/api",
*        "https://matrix.slum.cloud/scalar/api"
* ],
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

-            "matrix.org"

*            "matrix.slum.cloud"
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
  {% endcodeblock %}

### 4.デプロイする。

{% codeblock terminal lang:bash line_number:false %}
yarn dist
{% endcodeblock %}

何も問題が出ていなければいけているはず。

### 5.matrix 側の設定をする。

{% codeblock ~/synapse/homeserver.yaml lang:diff first_line:68 %}

- #web_client_location: https://riot.example.com/

* web_client_location: /home/matrix/element-web/webapp/
  {% endcodeblock %}
  {% codeblock ~/synapse/homeserver.yaml lang:yaml first_line:298 %}

  - port: 8008
    tls: false
    type: http
    x_forwarded: true
    bind_addresses: ['::1', '127.0.0.1']

        resources:
          - names: [client, federation, webclient]
            compress: false

    {% endcodeblock %}

### 6.確認

[matrix.slum.cloud](https://matrix.slum.cloud)にアクセスする。
{% asset_img login.png %}
構築成功、ブラウザからでもアクセスできるようになった。
