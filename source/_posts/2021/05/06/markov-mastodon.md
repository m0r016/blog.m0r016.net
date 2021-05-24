---
title: Mastodonに流れてくる投稿からマルコフ連鎖をさせ、投稿する。
date: 2021-05-06 10:55:10
updated: 2021-05-23 15:55:10
categories: [Fediverce, Mastodon]
tags:
  - Mastodon-markov
description: "Mastodonに流れてくる投稿からマルコフ連鎖をさせ、投稿する。"
---

### はじめに

マルコフ連鎖というものを使って、Mastodon 上に流れる投稿から要素を見つけ、それを再構築して投稿する。

マルコフ連鎖については、ニコニコ大百科にわかりやすく書いてある。→[ニコニコ大百科 - マルコフ連鎖](https://dic.nicovideo.jp/a/%E3%83%9E%E3%83%AB%E3%82%B3%E3%83%95%E9%80%A3%E9%8E%96)

<!-- toc -->
<!-- more -->

### Git から落としてくる

{% codeblock terminal lang:bash %}
https://github.com/m0r016/mastodon-markov-bot.git
{% endcodeblock %}

### Mastodon のアクセストークンを取得する

[新規アプリ](https://slum.cloud/settings/applications/new)にアクセス。
{% asset_img accesstoken-create.png %}
で適当に名前を決めて変更を保存。
{% asset_img app.png %}
アプリページに飛ばされるので、作成したアプリのページを開き、アクセストークンをコピーしてくる

### config をいじる

{% codeblock terminal lang:bash %}
cp src/config.ini.sample src/config.ini
{% endcodeblock %}

{% codeblock src/config.ini lang:diff %}
[read]
// 学習元アカウントの情報をここに入力
// domain にはインスタンスのドメイン(https://抜き)・access_token にはアクセストークンを入力

- domain =

* domain = slum.cloud

- access_token =

* access_token = get_account_token
  [write]
  // 投稿用アカウントの情報をここに入力
  // access_token にはアクセストークンを入力
  // 学習用アカウントと同じアカウントを利用したい場合は同じアクセストークンを入力(非推奨)

- access_token =

* access_token = post_account_token
  {% endcodeblock %}
  設定がすんだら docker を立ち上げる。

{% codeblock terminal lang:bash %}
$ pip install attrs
$ docker-compose up -d
WARNING: apt does not have a stable CLI interface. Use with caution in scripts.
{% endcodeblock %}
というエラーが出てくるので、
{% codeblock Dockerfile lang:diff %}
[read]
FROM python:3.8.5

- RUN apt update

* RUN apt-get update

- RUN apt install -y mecab libmecab-dev mecab-ipadic-utf8 swig

* RUN apt-get install -y mecab libmecab-dev mecab-ipadic-utf8 swig
  RUN git clone --depth=1 https://github.com/neologd/mecab-ipadic-neologd
  RUN cd ./mecab-ipadic-neologd && ./bin/install-mecab-ipadic-neologd -y -p /var/lib/mecab/dic/mecab-ipadic-neologd
  RUN rm -rf ./mecab-ipadic-neologd
  RUN ln -s /var/lib/mecab/dic /usr/lib/mecab/dic
  RUN mkdir /var/app
  WORKDIR /var/app
  COPY src/Pipfile ./
  COPY src/Pipfile.lock ./
  RUN pip install pipenv
  RUN pipenv install --system
  {% endcodeblock %}
  `apt`を`apt-get`に変更。

{% codeblock terminal lang:bash %}
docker-compose up -d
{% endcodeblock %}
時間はかかるがきちんと実行される。

{% mastodon https://slum.cloud/@project2501/106185996878493380 %}
はじめて投稿されたツイートがこれ・・・？おもしろい。

#bot がついているのが気に入らないので、app.py にアクセスし、#bot を消す。
{% codeblock src/app.py lang:diff first_line:33 %}
with open("./chainfiles/{}@{}.json".format(account_info["username"].lower(), domain)) as f:
textModel = markovify.Text.from_json(f.read())
sentence = textModel.make_sentence(tries=300)

-        sentence = "".join(sentence.split()) + ' #bot'

*        sentence = "".join(sentence.split())
          sentence = re.sub(r'(:.*?:)', r' \1 ', sentence)
          print(sentence)
  {% endcodeblock %}

投稿する頻度を 20 分から 1 時間にする。
{% codeblock src/app.py lang:diff first_line:45 %}

- def schedule(f, interval=3600, wait=True):

* def schedule(f, interval=1200, wait=True):
  base_time = time.time()
  next_time = 0
  while True:
  t = threading.Thread(target=f)
  t.start()
  if wait:
  t.join()
  next_time = ((base_time - time.time()) % interval) or interval
  time.sleep(next_time)

{% endcodeblock %}
設定がすんだら docker を立ち上げる。
