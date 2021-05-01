---
title: MastodonにAtomの情報を投稿する。
date: 2021-04-29 00:47:20
updated: 2021-04-30 14:44:00
categories: [Fediverse, Mastodon]
tags:
- Mastodon
description: "MastodonにAtomの情報を投稿する。"
---
### はじめに
このサイトでは[Atom](https://blog.m0r016.net/atom.xml)を設置している。
Mastodon上にbotを作り、Atomフィードを投稿させるいい手段がないかなと思って探していたら、次のようなリポジトリを発見[feediverse](https://github.com/edsu/feediverse)。設定していく。

<!-- toc -->
<!-- more -->

### 注意事項
設定する前に記事のヘッダーの`updated`を設定しておこう。
じゃないとデプロイ時に`updated`が自動更新してしまい、ちょっと面倒なことになる。

### 1.botアカウントを作成する
{% codeblock terminal lang:bash line_number:false %}
su - mastodon 
cd live/
RAILS_ENV=production bundle exec bin/tootctl accounts create USERNAME --email=EMAIL --confirmed
{% endcodeblock %}

### 2.インストールする
`README.md`に従い
{% codeblock terminal lang:bash line_number:false %}
sudo pip install feediverse
{% endcodeblock %}

### 3.設定する
{% codeblock terminal line_number:false %}
$ sudo feediverse
What is your Mastodon Instance URL? your_instance
Do you have your app credentials already? [y/n] n
Ok, I'll need a few things in order to get your access token
app name (e.g. feediverse): feediverse
mastodon username (email): your_bot@email
mastodon password (not stored): your_bot_password
RSS/Atom feed URL to watch: yout_feed
Shall already existing entries be tooted, too? [y/n] y
Shall images be included in the toot? [y/n] y

Your feediverse configuration has been saved to /home/user/.feediverse
Add a line line this to your crontab to check every 15 minutes:
*/15 * * * * /usr/local/bin/feediverse
{% endcodeblock %}
これで投稿されるようになった。試しに`sudo feediverse`とタイプするとどばーっと投稿される。

細かく設定したいときは`/root/.feediverse`を編集する。
恐らく`template: '{title} {url}'`を`template: '更新: {title} {url}'`などするとよいだろう。
{% codeblock /root/.feediverse lang:yaml %}
access_token: 
client_id: 
client_secret: 
feeds:
- template: '{title} {url}'
  url: https://blog.m0r016.net/atom.xml
include_images: true
name: feediverse
updated: '2021-04-28T15:37:15.281000+00:00'
url: slum.cloud
{% endcodeblock %}

設定が終わったら
{% codeblock terminal line_number:false %}
sudo feediverse -n -v
{% endcodeblock %}
でテストしておくとよいだろう。
### 4.crontabに設定する。
`sudo which feediverse`で場所を確認する。`/usr/local/bin/feediverse`とでた。
`sudo crontab -e`とタイプし、`*/15 * * * * /usr/local/bin/feediverse`とタイプすれば15分おきに設定したフィードをチェックし、投稿される。

### 参考
・[勝手 Mastodon tootctl リファレンス](https://qiita.com/neustrashimy/items/870769d7db4d95cde238)