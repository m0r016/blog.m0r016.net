---
title: matrixを構築する。
date: 2021-03-24 20:27:16
updated: 2021-09-11 02:32:00
categories: [Ubuntu, matrix]
tags:
  - matrix
description: "matrixを構築する。"
---

### はじめに

最近、Discord の買収が[危惧](https://www.famitsu.com/news/202103/23216397.html)されている。
Fediverse を眺めていたら、[matrix](https://www.matrix.org/)というチャットツールがあるらしい。
これを構築して見ようと思う。

<!-- more -->
<!-- toc -->

### matrix とは

matrix は、Slack や Discord のようなコミュニケーションツールであり、メッセージのやり取り、通話機能、E2E での通話暗号化、ブリッジ機能がある。

### 環境

OS: Ubuntu 20.04LTS
実行環境: Python3.9

### 1.実行環境の準備

公式の[ドキュメント](https://github.com/matrix-org/synapse/blob/develop/INSTALL.md)に従い、実行環境を準備する。
{% codeblock terminal lang:bash line_number:false %}
sudo apt update && sudo apt upgrade
sudo apt install build-essential python3-dev libffi-dev python3-pip python3-setuptools sqlite3 libssl-dev virtualenv libjpeg-dev libxslt1-dev
{% endcodeblock %}

私は、バックグラウンドアプリを構築する際は大体専用ユーザーを作るのが好きだ。
{% codeblock terminal lang:bash line_number:false %}
sudo adduser matrix
sudo gpasswd -a matrix sudo
su - matrix
{% endcodeblock %}

### 2.matrix をインストールする。

matrix をインストールする。git から引っ張ってくるわけではないみたい。
{% codeblock terminal lang:bash line_number:false %}
mkdir -p ~/synapse
virtualenv -p python3 ~/synapse/env
source ~/synapse/env/bin/activate
pip install --upgrade pip
pip install --upgrade setuptools
pip install matrix-synapse
{% endcodeblock %}

update する際は`source ~/synapse/env/bin/activate && pip install -U matrix-synapse`を実行するといいらしい。

### 3.config の生成

matrix を設定していく。
{% codeblock terminal lang:bash line_number:false %}
cd ~/synapse
python -m synapse.app.homeserver \
 --server-name my.domain.name \
 --config-path homeserver.yaml \
 --generate-config \
 --report-stats=[yes|no]
{% endcodeblock %}
my.domain.name は matrix を置きたい domain に変更する、
`--report-stats=[yes|no]`は yes か no を指定してあげないとだめだ。

### 4.matrix の設定

デフォルトでは SQLite を使用するらしいが、確かに SQLite は簡単ではあるが重いので Postres を使うようにする。
{% codeblock terminal lang:bash line_number:false %}
~/synapse/env/bin/pip install "matrix-synapse[postgres]"
sudo -u postgres bash
createuser --pwprompt synapse_user
psql
{% endcodeblock %}

{% codeblock psql lang:psql line_number:false %}
CREATE DATABASE synapse
ENCODING 'UTF8'
LC_COLLATE='C'
LC_CTYPE='C'
template=template0
OWNER synapse_user;
quit;
{% endcodeblock %}

synapse_user で指定したパスワードを`homeserver.yaml`の`database:`内に追加する。
{% codeblock homeserver.yaml lang:diff first_line:806 %}
database:
name: psycopg2
args:
user: synapse_user

- password: secretpassword

* password: <pass>
  database: synapse
  host: localhost
  cp_min: 5
  cp_max: 10
  {% endcodeblock %}

pb_hba.conf も変えておく。私の場合`/etc/postgres/10/main/`内にあった。
{% codeblock /etc/postgres/10/main/pg_hba.conf lang:diff line_number:false %}

- host synapse synapse_user ::1/128 md5
  {% endcodeblock %}

synapse_user にパスワードを指定していない場合、`pg_hba.conf`内に
{% codeblock /etc/postgres/10/main/pg_hba.conf lang:diff line_number:false %}
host all all ::1/128 ident
{% endcodeblock %}
と書き足しておく。

`homeserver.yaml`を変更しておく。
{% codeblock homeserver.yaml lang:diff first_line:54 %}

- server_name: "SERVERNAME"

* server_name: matrix.slum.cloud
  {% endcodeblock %}
  {% codeblock homeserver.yaml lang:diff first_line:68 %}

- web_client_location: https://riot.example.com/

* web_client_location: https://matrix.slum.cloud
  {% endcodeblock %}

### 5.nginx の設定

[ドキュメント](https://github.com/matrix-org/synapse/blob/develop/docs/reverse_proxy.md)を参考に設定する。
{% codeblock /etc/nginx/sites-enabled/matrix.slum.cloud lang:diff %}
server {
listen 443 http2;
listen [::]:443 http2;

    // For the federation port
    listen 8448 http2 ;
    listen [::]:8448 http2 ;

    server_name matrix.slum.cloud;

    location / {
        proxy_pass http://localhost:8008;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $host;

        // Nginx by default only allows file uploads up to 1M in size
        // Increase client_max_body_size to match max_upload_size defined in homeserver.yaml
        client_max_body_size 50M;
    }

}
{% endcodeblock %}

certbot で証明書を取得したら、`listen hoge http2`に ssl を書き足し`listen hoge ssl http2`のようにする。
{% codeblock /etc/nginx/sites-enabled/matrix.slum.cloud lang:diff %}
server {

- listen 443 http2;
- listen [::]:443 http2;

* listen 443 ssl http2;
* listen [::]:443 ssl http2;


    # For the federation port

- listen 8448 http2 ;
- listen [::]:8448 http2 ;

* listen 8448 ssl http2 ;
* listen [::]:8448 ssl http2 ;


    server_name matrix.slum.cloud;

    location / {
        proxy_pass http://localhost:8008;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $host;

        // Nginx by default only allows file uploads up to 1M in size
        // Increase client_max_body_size to match max_upload_size defined in hom>
        client_max_body_size 50M;
    }

- ssl_certificate /etc/letsencrypt/live/matrix.slum.cloud/fullchain.pem;
- ssl_certificate_key /etc/letsencrypt/live/matrix.slum.cloud/privkey.pem;
  }
  {% endcodeblock %}

再び matrix の config を開く。
{% codeblock homeserver.yaml lang:diff first_line:552 %}
// TLS セクション内の tls_certificate_path と tls_private_key_path を先ほど取得した証明書のパスを書き込む

- #tls_certificate_path: "CONFDIR/SERVERNAME.tls.crt"

* tls_certificate_path: "/etc/letsencrypt/live/matrix.slum.cloud/fullchain.pem"

// PEM-encoded private key for TLS
//

- #tls_private_key_path: "CONFDIR/SERVERNAME.tls.key"

* tls_private_key_path: "/etc/letsencrypt/live/matrix.slum.cloud/privkey.pem"
  {% endcodeblock %}

### 6.実行する

{% codeblock terminal lang:bash line_number:false %}
cd ~/synapse
source env/bin/activate
synctl start
{% endcodeblock %}

無事動いたらしい。[matrix](https://matrix.slum.cloud/)
{% asset_img comp.png %}

ユーザーを作成しておく。
{% codeblock terminal lang:bash line_number:false %}
register_new_matrix_user -c homeserver.yaml http://localhost:8008
{% endcodeblock %}

### 7.ログインする。

クライアントが必要っぽいので[element](https://element.io/get-started)から引っ張ってくる。

サインインを選択
{% asset_img login.png %}

Home Server を変更し認証情報を入力
{% asset_img login1.png %}

ログインすることができあ
{% asset_img login2.png %}

これで使うようにできた。

### 8.デーモン化する。

{% codeblock /etc/systemd/system/matrix.slum.cloud.service lang:ini %}
[Unit]
Description=Synapse Matrix homeserver
// If you are using postgresql to persist data, uncomment this line to make sure
// synapse starts after the postgresql service.
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

// adjust the cache factor if necessary
// Environment=SYNAPSE_CACHE_FACTOR=2.0

[Install]
WantedBy=multi-user.target
{% endcodeblock %}

設定したら
{% codeblock terminal lang:bash line_number:false %}
sudo systemctl start matrix.slum.cloud
sudo systemctl enable matrix.slum.cloud
{% endcodeblock %}
これでデーモン化することができる。

### 参考

・[Matrix の Homeserver、Synapse を Debian 10 にインストールする](https://qiita.com/eniehack/items/268f5903d50fcd8dc309)
・[Slack 系 分散 SNS「Matrix」を構築する](https://qiita.com/guskma/items/20fa88c25bfdbfc99c2a)
・[systemd](https://github.com/matrix-org/synapse/blob/master/contrib/systemd/matrix-synapse.service#L17)
