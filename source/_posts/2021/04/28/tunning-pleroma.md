---
title: PosgtreSQLを調整し、Pleromaのレスポンスを向上する。
date: 2021-04-28 21:27:56
updated: 2021-09-21 22:23:17
categories: [Fediverse, Pleroma, Postgresql]
tags:
  - Postgresql
  - Pleroma
description: "PosgtreSQLを調整し、Pleromaのレスポンスを向上する。"
---

### はじめに

`(DBConnection.ConnectionError) connection not available and request was dropped from queue after xxxxms`というエラーが出てきてしまったため、PosgtreSQL を調整し、Pleroma のレスポンスを向上する。
ユーザーは pleroma で進めていく。

<!-- more -->
<!-- toc -->

### 1.Pleroma 側の調整

{% codeblock /opt/pleroma/config/prod.secret.exs lang:diff %}
config :pleroma, Pleroma.Repo,
adapter: Ecto.Adapters.Postgres,
username: "pleroma",
password: "db_password",
database: "pleroma_db",
hostname: "localhost",

- pool_size: 20,
- timeout: 120000,
- queue_target: 200,
- queue_interval: 4000

- config :pleroma, :dangerzone, override_repo_pool_size: true
  {% endcodeblock %}

### 2.PostgreSQL 側の設定

postgresql の設定ファイルは大体`/etc/postgresql/$VER/main/`下に存在する。
この`postgresql.conf`は`pg_createcluster`をしたときにしか生成されないのであらかじめバックアップを取っておく。$VERはpostgresqlのバージョン
{% codeblock terminal lang:bash line_number:false %}
sudo cp /etc/postgresql/$VER/main/postgresql.conf /etc/postgresql/$VER/main/postgresql.back
{% endcodeblock %}

バックアップから戻す場合は
{% codeblock terminal lang:bash line_number:false %}
sudo cp /etc/postgresql/$VER/main/postgresql.back /etc/postgresql/$VER/main/postgresql.conf
{% endcodeblock %}
とする。

このサイトの`Example configurations`の`1GB RAM, 1CPU`を参考にする。
`nano`の場合`Ctrl + W`で検索し、値を変更する。
{% codeblock /etc/postgresql/$VER/main/postgresql.conf lang:diff %}

- max_connections = 20

+ max_connections = 100

- shared_buffers = 256MB

+ shared_buffers = 128MB

- effective_cache_size = 768MB

+ // effective_cache_size = 4GB

- maintenance_work_mem = 64MB

+ // maitenance_work_mem = 64MB

- work_mem = 13107kB

+ // work_mem = 4MB
  {% endcodeblock %}
  変更した。いったん様子を見る。
  `sudo service pleroma restart`

[PGTune](https://pgtune.leopard.in.ua/)で最適な値を手に入れる。
使い方は簡単で、フォームを埋めるだけ。
DB Version は 12, OS Type は Linux, DB Type は Web application, Total Memory(RAM)は 1GB, Number of CPUs は 4, Number of Connections は 20, Data Storage は SD カードがなかったため SSD Strage とした。これをもとに`postgresql.conf`を書き換える。
{% asset_img pgtune.png pgtune_result %}
{% codeblock /etc/postgresql/$VER/main/postgresql.conf lang:diff %}

- checkpoint_completion_target = 0.9

+ // checkpoint_completion_target = 0.5

- wal_buffers = 7834kB

+ wal_buffers = -1

- default_statistics_targer = 100

+ // default_statistics_targer = 100

- random_page_cost = 1.1

+ // random_page_cost = 4.0

- effective_io_concurrency = 200

+ // effective_io_concurryency = 1

- work_mem = 6553kB

+ work_mem = 13107kB

- min_wal_size = 1GB

+ min_wal_size = 80MB

- max_wal_size = 4GB

+ max_wal_size = 1GB

- max_worker_processes = 4

+ max_worker_processes = 8

- max_parallel_workers_per_gather = 2

+ // max_parallel_workers_par_gather = 2

- max_parallel_workers = 4

+ max_parallel_workers = 8

- max_parallel_workers = 2

+ // max_parallel_workers = 2
  {% endcodeblock %}
  書き換えたら`sudo service pleroma restart`をして完了

### 3.再起動を組み込む。

{% post_link improvement-systemd 'timerを使わず、serviceのみで再起動させる。' %}を参考に再起動するようにしておく。

{% codeblock /etc/systemd/system/pleroma.service lang:diff %}
[Unit]
Description=Pleroma social network
After=network.target postgresql.service

[Service]
ExecReload=/bin/kill $MAINPID
KillMode=process
Restart=always
User=pleroma
Environment="MIX_ENV=prod"
WorkingDirectory=/opt/pleroma
ExecStart=/usr/bin/mix phx.server

- RuntimeMaxSec=86400

[Install]
WantedBy=multi-user.target
{% endcodeblock %}

### 参考

・[db_connection](https://hexdocs.pm/db_connection/DBConnection.html#start_link/2-options)
・[Ecto - Troubleshooting](http://blog.tap349.com/elixir/ecto/2018/12/28/ecto-troubleshooting/)
