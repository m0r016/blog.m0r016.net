---
title: dockerでリレーサーバを構築する。
date: 2021-04-07 15:02:55
categories: [pub-relay]
tags: 
- Mastodon pub-relay
description: "dockerでリレーサーバを構築する。"
---
### はじめに
そろそろDockerでもろもろ構築できるようになっておきたいと思ったため、リレーサーバを構築することにした。
> リレーサーバとは
> Mastodonのインスタンス同士を繋げる中継役のようなものである。

<!-- toc -->
<!-- more -->
### Dockerとは
仮想マシンに比べてコンテナ単位で実行できる代物らしい。
仮想マシンはホストOS→ゲストOS→アプリケーションとなっているが、
DockerはホストOS→アプリケーションとなっているので、仮想マシンに比べ高速で軽量なのだ。

### 0.前提環境のインストール
`$ sudo apt install docker-compose && sudo service docker start`

### 1.リポジトリをクローンする。
ラズパイで構築する。
せっかくなのでユーザーを作ってしまおう。
```
$ sudo adduser relay
$ sudo gpasswd -a relay sudo
$ su - relay
```

ユーザーを作ったらリポジトリをクローンする。
今回使うリポジトリは[yukimochi](https://toot.yukimochi.jp/@YUKIMOCHI)さんの[Activity-Relay](https://github.com/yukimochi/Activity-Relay/wiki/Installing)だ。
[ここ](https://github.com/yukimochi/Activity-Relay/wiki/Installing)を参考に進める。
```
$ git clone https://github.com/yukimochi/Activity-Relay.git 
$ cd Activity-Relay
$ git checkout v0.2.8
```

### 2.秘密鍵の生成
```
$ openssl genrsa > actor.pem
```

### 3.docker-compose.ymlに設定を入力

```
$ nano docker-compose.yml
version: "2.3"
services:
  redis:
    restart: always
    image: redis:alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
    volumes:
      - "./redisdata:/data"

  worker:
    build: .
    image: yukimochi/activity-relay
    restart: always
    init: true
    command: worker
    environment:
      - "ACTOR_PEM=/actor.pem"
      - "RELAY_DOMAIN=relay.slum.cloud"
      - "RELAY_SERVICENAME=relay slum.cloud"
      - "RELAY_BIND=0.0.0.0:8080"
      - "REDIS_URL=redis://redis:6379"
    volumes:
      - "./actor.pem:/actor.pem"
      # - "./config.yaml:/Activity-Relay/config.yaml"
    depends_on:
      - redis

  server:
    build: .
    image: yukimochi/activity-relay
    restart: always
    init: true
    command: server
    ports:
      - 127.0.0.1:8080:8080
    environment:
      - "ACTOR_PEM=/actor.pem"
      - "RELAY_DOMAIN=relay.slum.cloud"
      - "RELAY_SERVICENAME=relay slum.cloud"
      - "RELAY_BIND=0.0.0.0:8080"
      - "REDIS_URL=redis://redis:6379"
    volumes:
      - "./actor.pem:/actor.pem"
      # - "./config.yaml:/Activity-Relay/config.yaml"
    depends_on:
      - redis
```

### 4.dockerの起動
いよいよdockerを起動する。
`$ sudo docker-compose up -d`

と思ったらエラーが発生。
```
go: github.com/RichardKnop/machinery@v1.7.8: missing go.sum entry; to add it:
	go mod download github.com/RichardKnop/machinery
ERROR: Service 'worker' failed to build: The command '/bin/sh -c mkdir -p /rootfs/usr/bin &&      apk add -U --no-cache git &&      go build -o /rootfs/usr/bin/server -ldflags "-X main.version=$(git describe --tags HEAD)" . &&      go build -o /rootfs/usr/bin/worker -ldflags "-X main.version=$(git describe --tags HEAD)"  ./worker &&      go build -o /rootfs/usr/bin/ar-cli -ldflags "-X main.version=$(git describe --tags HEAD)"  ./cli' returned a non-zero code: 1
```

調べていたところ、[issue](https://github.com/yukimochi/Activity-Relay/issues/42)を発見。
[このように](https://github.com/yukimochi/Activity-Relay/issues/42#issuecomment-809392759)してあげるといいらしい。

```
$ nano Dockerfile
- FROM golang:alpine AS build
+ FROM golang:1.13-alpine AS build
```

再度実行する。
起動した。

### 5.nginxを設定する
```
$ sudo nano /etc/nginx/sites-enabled/relay.slum.cloud
server {
        listen 80 ;
        listen [::]:80 ;
#        location / { return 301 https://$host$request_uri; }
        server_name relay.slum.cloud;
        root /var/www/html;
}
```
まず、Lets encrypt用に80ポートで受け付ける。

### 6.証明書の取得
```
$ sudo certbot
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator nginx, Installer nginx

Please choose an account
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: magi-system@2019-08-27T03:57:49Z (aeee)
2: MAGI-SYSTEM@2019-03-09T17:12:48Z (37c0)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 1
```
複数アカウントがあるときに聞かれる。私の場合1

```
Which names would you like to activate HTTPS for?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: kyt-giken.com
2: m0r016.net
3: file.m0r016.net
4: identity.m0r016.net
5: msbtn.m0r016.net
6: pl.m0r016.net
7: www.m0r016.net
8: slum.cloud
9: dev.slum.cloud
10: game.slum.cloud
11: matrix.slum.cloud
12: msb.slum.cloud
13: note.slum.cloud
14: relay.slum.cloud
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel): 14
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for relay.slum.cloud
Waiting for verification...
Cleaning up challenges
Deploying Certificate to VirtualHost /etc/nginx/sites-enabled/relay.slum.cloud
```
今回取得したいドメインを選択。`relay.slum.cloud`が欲しいので14を選択。

```
Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 1
```
httpsにリダイレクトするかと聞かれるので1を選択。

```
Future versions of Certbot will automatically configure the webserver so that all requests redirect to secure HTTPS access. You can control this behavior and disable this warning with the --redirect and --no-redirect flags.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Your existing certificate has been successfully renewed, and the new certificate
has been installed.

The new certificate covers the following domains: https://relay.slum.cloud

You should test your configuration at:
https://www.ssllabs.com/ssltest/analyze.html?d=relay.slum.cloud
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/relay.slum.cloud/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/relay.slum.cloud/privkey.pem
   Your cert will expire on 2021-07-11. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

```
完了だ。

### 7.nginxの設定
設定を進める
```
$ sudo nano /etc/nginx/sites-enabled/relay.slum.cloud
server {
        listen 80 ;
        listen [::]:80 ;
        location / { return 301 https://$host$request_uri; }
        server_name relay.slum.cloud;
#        root /var/www/html;
}
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name relay.slum.cloud;

    location / {
        proxy_pass http://relay_ip;
    }
    ssl_certificate /etc/letsencrypt/live/relay.slum.cloud/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/relay.slum.cloud/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}
```

次にリレーサーバのnginxをいじる。
```
$ ssh m0r016@relay_ip
$ sudo apt install nginx
$ sudo /etc/nginx/sites-enabled/relay.slum.cloud
server {
        listen 80 default_server;
        listen [::]:80 default_server;
        location / {
        proxy_pass http://localhost:8080;
        }
        server_name relay.slum.cloud;
}
```
できたら`$ sudo service nginx restart`を実行し適用する。

そしたら母機に戻り、`curl`を実行する。
`curl raspi_ip/actor`

`{"@context":...`のようなものが表示されたら成功だ。