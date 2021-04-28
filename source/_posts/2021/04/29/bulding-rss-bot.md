---
title: MastodonにAtomの情報を投稿する。
date: 2021-04-29 00:47:20
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
### 1.botアカウントを作成する
{% codeblock terminal lang:bash line_number:false %}
su - mastodon 
cd live/
RAILS_ENV=production bundle exec bin/tootctl accounts create USERNAME --email=EMAIL --confirmed
{% endcodeblock %}

### 2.インストールする
`README.md`に従い
{% codeblock terminal lang:bash line_number:false %}
pip install feediverse
{% endcodeblock %}

### 3.設定する
{% codeblock terminal lang:bash line_number:false %}
$ feediverse
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
これで投稿されるようになった。試しに`feediverse`とタイプするとどばーっと投稿される。

細かく設定したいときは`.feediverse`を編集する。
恐らく`template: '{title} {url}'`を`template: '更新: {title} {url}'`などするとよいだろう。
{% codeblock .feediverse lang:yaml %}
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

### crontabに設定する。
`*/15 * * * * /usr/local/bin/feediverse`と最後に表示されたが、実際は`/home/$user/.local/bin/feediverse`にあった。
もちろん個人個人の環境で変わってくるので`which feediverse`で場所を確認しよう。
`crontab -e`とタイプし、`*/15 * * * * /home/$user/.local/bin/feediverse`とタイプすれば15分おきに設定したフィードをチェックし、投稿される。

