---
title: cloudflare-pages
date: 2021-04-16 22:01:32
categories: [blog, hexo, cloudflare-pages]
tags:
- cloudflare-pages
description: "cloudflare pagesを使ってみる。"
---
### はじめに
最近、CloudflareからVercelやGithub Pagesみたいなホスティングサービスがリリースされたらしい。
早速使ってみる。

<!-- toc -->
<!-- more -->

### 1.プロジェクトを作成
自分の[ダッシュボード](https://dash.cloudflare.com/)にアクセスし、プロジェクトを作成する。
{% asset_img cloudflare-pages.png %}

GitHubアカウントを接続しろといわれるので接続する。
{% asset_img cp1.png %}

All repositoriesを選択
{% asset_img cp2.png %}

パスワードをタイプする
{% asset_img cp3.png %}

そうするとリダイレクトされ、リポジトリが選択できるのでリポジトリを選択する。選択できたらセットアップの開始を押す。
{% asset_img cp4.png %}

フレームワーク プリセットからhexoを選ぶ。大体はこの時に読み込まれた設定で問題ない。選択できたら保存してデプロイを押す。
{% asset_img cp5.png %}

deployが始まる。しばらく待つ。
{% asset_img cp6.png %}

完了したらプロジェクトに進む
{% asset_img cp7.png %}

コントロール画面はこんな感じ。
{% asset_img cp8.png %}

カスタムドメインに行き、カスタムドメインを設定を押す
{% asset_img cp9.png %}

ドメインをタイプし続行。
{% asset_img cp10.png %}

ドメインをアクティブにするを選択。
{% asset_img cp11.png %}

しばらく待つと、設定がすみアクセスできるようになる。
{% asset_img cp12.png %}

有効化された。これでcloudflareから配信されることになる。
{% asset_img cp13.png %}