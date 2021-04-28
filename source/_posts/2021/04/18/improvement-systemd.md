---
title: timerを使わず、serviceのみで再起動させる。
date: 2021-04-18 02:32:32
updated: 2021-04-18 02:32:32
categories: [Ubuntu, systemd]
tags:
- systemd
description: "timerを使わず、serviceのみで再起動させる。"
---
### はじめに
systemdを使ってサービスを指定した時間に再起動するときに、私の場合timerファイルを作って再起動するように仕込んでいたのだが、timerファイルを使わず、`RuntimeMaxSec=`を使うことにより、自動的に再起動することができるようになるみたいだ。（実際のところ違うみたいだが）

<!-- toc -->
<!-- more -->

### manを読む
manとはコマンドのマニュアルを表示するものだ。systemd.service項目を見る。`man systemd.service`
{% codeblock "man systemd.service" lang:man first_line:580 %}
       RuntimeMaxSec=
           Configures a maximum time for the service to run. If this is used
           and the service has been active for longer than the specified time
           it is terminated and put into a failure state. Note that this
           setting does not have any effect on Type=oneshot services, as they
           terminate immediately after activation completed. Pass "infinity"
           (the default) to configure no runtime limit.

           If a service of Type=notify sends "EXTEND_TIMEOUT_USEC=...", this
           may cause the runtime to be extended beyond RuntimeMaxSec=. The
           first receipt of this message must occur before RuntimeMaxSec= is
           exceeded, and once the runtime has exended beyond RuntimeMaxSec=,
           the service manager will allow the service to continue to run,
           provided the service repeats "EXTEND_TIMEOUT_USEC=..."  within the
           interval specified until the service shutdown is achieved by
           "STOPPING=1" (or termination). (see sd_notify(3)).
{% endcodeblock %}
なるほどわからん。こういう時に役立つのは[DeepL](https://www.deepl.com)

{% codeblock "man systemd.service -> deepl" lang:man %}
サービスを実行する最大時間を設定します。この設定を行った場合、指定した時間を超えてサービスが稼動していると、サービスは終了し、障害状態になります。この設定は、Type=oneshotのサービスには影響を与えません。infinity」（デフォルト）を指定すると、実行時間の制限を設けません。

Type=notify のサービスが「EXTEND_TIMEOUT_USEC=...」を送信した場合、ランタイムが RuntimeMaxSec=を超えて延長される可能性がある。このメッセージの最初の受信は、RuntimeMaxSec=を超える前に行われなければならない。ランタイムがRuntimeMaxSec=を超えて延長されると、サービスマネージャは、サービスが "STOPPING=1 "でサービスを停止する(または終了する)まで、指定された間隔で "EXTEND_TIMEOUT_USEC=... "を繰り返すことを条件に、サービスの実行を継続することを許可する。(sd_notify(3)参照)。

www.DeepL.com/Translator（無料版）で翻訳しました。
{% endcodeblock %}

なるほど、指定時間を超えると意図的に終了し障害状態になるから`Type`が`oneshot`以外で`Restart`が`Always`である場合、再起動するという感じか。
mastodonは定期的にsidekiqの再起動が必要になるといううわさを聞くのでsidekiq辺りは再起動するようにしておこうかな。

### systemdをいじる
私の場合、sidekiqを`default`, `mailers`, `pull`, `push`と全体を含む`sidekiq`に分けている。
万が一問題が起きてもいいため、`sidekiq`以外を再起動させる。
{% codeblock /etc/systemd/system/mastodon-sidekiq-default.service lang:diff %}
[Unit]
Description=mastodon-sidekiq
After=network.target

[Service]
Type=simple
User=mastodon
WorkingDirectory=/home/mastodon/live
Environment="RAILS_ENV=production"
Environment="DB_POOL=400"
Environment="LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libjemalloc.so"
ExecStart=/home/mastodon/.rbenv/shims/bundle exec sidekiq -c 50 -q default -q push -q mailers -q pull -q scheduler
TimeoutSec=15
Restart=always
+ RuntimeMaxSec=86400

[Install]
WantedBy=multi-user.target
{% endcodeblock %}
秒単位を要求されてるので`24h * 60m * 60s`で`86400`なので`86400`を指定

サービスを再読み込み、再起動すれば反映される。
{% codeblock terminal lang:bash %}
sudo systemctl daemon-reload
sudo systemctl restart mastodon-sidekiq*.service
{% endcodeblock %}

### 参考
{% mastodon https://fedibird.com/@noellabo/106073678076908162 %}