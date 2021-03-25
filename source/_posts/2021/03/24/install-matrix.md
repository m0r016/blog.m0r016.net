---
title: matrixを構築する。
date: 2021-03-24 20:27:16
categories: [Ubuntu, matrix]
tags:
- matrix
description: "matrixを構築する。"
---
### はじめに
最近、Discordの買収が[危惧](https://www.famitsu.com/news/202103/23216397.html)されている。
Fediverseを眺めていたら、[matrix](https://www.matrix.org/)というチャットツールがあるらしい。
これを構築して見ようと思う。

<!-- toc -->
<!-- more -->
### matrixとは
matrixは、SlackやDiscordのようなコミュニケーションツールであり、まずメッセージのやり取り、通話機能、E2Eでの通話暗号化、ブリッジ機能がある。

### 環境
OS: Ubuntu 20.04LTS
実行環境: Python3.9

### 1.実行環境の準備
公式の[ドキュメント](https://github.com/matrix-org/synapse/blob/develop/INSTALL.md)に従い、実行環境を準備する。
```
sudo apt update && sudo apt upgrade
sudo apt install build-essential python3-dev libffi-dev python3-pip python3-setuptools sqlite3 libssl-dev virtualenv libjpeg-dev libxslt1-dev
```

私は、バックグラウンドアプリを構築する際は大体専用ユーザーを作るのが好きだ。
```
sudo adduser matrix
sudo gpasswd -a matrix sudo
su - matrix
```

### 2.matrixをインストールする。
matrixをインストールする。gitから引っ張ってくるわけではないみたい。
```
mkdir -p ~/synapse
virtualenv -p python3 ~/synapse/env
source ~/synapse/env/bin/activate
pip install --upgrade pip
pip install --upgrade setuptools
pip install matrix-synapse
```

updateする際は`source ~/synapse/env/bin/activate && pip install -U matrix-synapse`を実行するといいらしい・・・？

### 3.configの生成
matrixを設定していく。
```
cd ~/synapse
python -m synapse.app.homeserver \
    --server-name my.domain.name \
    --config-path homeserver.yaml \
    --generate-config \
    --report-stats=[yes|no]
```
my.domain.nameはmatrixを置きたいdomainに変更する、
`--report-stats=[yes|no]`はyesかnoを指定してあげないとだめだ。

### 4.matrixの設定
デフォルトではSQLiteを使用するらしいが、確かにSQLiteは簡単ではあるが重いのでPostresを使うようにする。
```
~/synapse/env/bin/pip install "matrix-synapse[postgres]"
sudo -u postgres bash
createuser --pwprompt synapse_user
psql
```

psql↓
```
CREATE DATABASE synapse
 ENCODING 'UTF8'
 LC_COLLATE='C'
 LC_CTYPE='C'
 template=template0
 OWNER synapse_user;
quit;
```

synapse_userで指定したパスワードを`homeserver.yaml`の`database:`内に追加する。
```
nano homeserver.yaml
database:
  name: psycopg2
  args:
    user: synapse_user
    password: <pass>
    database: synapse
    host: localhost
    cp_min: 5
    cp_max: 10
```

pb_hba.confも変えておく。私の場合`/etc/postgres/10/main/`内にあった。
```
sudo nano /etc/postgres/10/main/pg_hba.conf
host    synapse     synapse_user    ::1/128     md5
```

synapse_userにパスワードを指定していない場合、`pg_hba.conf`内に
`host    all         all             ::1/128     ident`
と書き足しておく。

`homeserver.yaml`を変更しておく。
```
nano homeserver.yaml
server_name: matrix.slum.cloud
web_client_location: https://matrix.slum.cloud
```

### 5.nginxの設定
[ドキュメント](https://github.com/matrix-org/synapse/blob/develop/docs/reverse_proxy.md)を参考に設定する。
```
sudo nano /etc/nginx/sites-enabled/matrix.slum.cloud
server {
    listen 443 http2;
    listen [::]:443 http2;

    # For the federation port
    listen 8448 http2 ;
    listen [::]:8448 http2 ;

    server_name matrix.slum.cloud;

    location / {
        proxy_pass http://localhost:8008;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $host;

        # Nginx by default only allows file uploads up to 1M in size
        # Increase client_max_body_size to match max_upload_size defined in homeserver.yaml
        client_max_body_size 50M;
    }
}
```

certbotで証明書を取得したら、`listen hoge http2`にsslを書き足し`listen hoge ssl http2`のようにする。
```
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    # For the federation port
    listen 8448 ssl http2 ;
    listen [::]:8448 ssl http2 ;

    server_name matrix.slum.cloud;

    location / {
        proxy_pass http://localhost:8008;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $host;

        # Nginx by default only allows file uploads up to 1M in size
        # Increase client_max_body_size to match max_upload_size defined in hom>
        client_max_body_size 50M;
    }

    ssl_certificate /etc/letsencrypt/live/matrix.slum.cloud/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/matrix.slum.cloud/privkey.pem;
}
```

再びmatrixのconfigを開く。
```
nano homeserver.yaml
# TLSセクション内のtls_certificate_pathとtls_private_key_pathを先ほど取得した証明書のパスを書き込む
tls_certificate_path: "/etc/letsencrypt/live/matrix.slum.cloud/fullchain.pem"
tls_private_key_path: "/etc/letsencrypt/live/matrix.slum.cloud/privkey.pem"
```

### 6.実行する
```
cd ~/synapse
source env/bin/activate
synctl start
```

無事動いたらしい。[matrix](https://matrix.slum.cloud/)
{% asset_img comp.png %}

ユーザーを作成しておく。
```
register_new_matrix_user -c homeserver.yaml http://localhost:8008
```

### 7.ログインする。
クライアントが必要っぽいので[element](https://element.io/get-started)から引っ張ってくる。

サインインを選択
{% asset_img login.png %}

Home Serverを変更し認証情報を入力
{% asset_img login1.png %}

ログインすることができあ
{% asset_img login2.png %}

これで使うようにできた。

### 8.デーモン化する。
```
sudo nano /etc/systemd/system/matrix.slum.cloud.service
[Unit]
Description=Synapse Matrix homeserver
# If you are using postgresql to persist data, uncomment this line to make sure
# synapse starts after the postgresql service.
After=postgresql.service

[Service]
Type=notify
NotifyAccess=main
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-abort

User=matrix
Group=matrix

WorkingDirectory=/home/matrix/synapse
ExecStart=/home/matrix/synapse/env/bin/python -m synapse.app.homeserver --config-path=/home/matrix/synapse/homeserver.yaml
SyslogIdentifier=matrix-synapse

# adjust the cache factor if necessary
# Environment=SYNAPSE_CACHE_FACTOR=2.0

[Install]
WantedBy=multi-user.target
```

設定したら
```
sudo systemctl start matrix.slum.cloud
sudo systemctl enable matrix.slum.cloud
```
これでデーモン化することができる。
[参考](https://github.com/matrix-org/synapse/blob/master/contrib/systemd/matrix-synapse.service#L17)