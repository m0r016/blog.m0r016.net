---
title: cloudflare pagesを使ってみる。
date: 2021-04-16 22:01:32
updated: 2021-09-11 02:29:32
categories: [blog, hexo, cloudflare-pages]
tags:
  - cloudflare-pages
description: "cloudflare pagesを使ってみる。"
---

### はじめに

最近、Cloudflare から Vercel や Github Pages みたいなホスティングサービスがリリースされたらしい。
早速使ってみる。

<!-- more -->
<!-- toc -->

### 1.プロジェクトを作成

自分の[ダッシュボード](https://dash.cloudflare.com/)にアクセスし、プロジェクトを作成する。
{% asset_img cloudflare-pages.png %}

GitHub アカウントを接続しろといわれるので接続する。
{% asset_img cp1.png %}

All repositories を選択
{% asset_img cp2.png %}

パスワードをタイプする
{% asset_img cp3.png %}

そうするとリダイレクトされ、リポジトリが選択できるのでリポジトリを選択する。選択できたらセットアップの開始を押す。
{% asset_img cp4.png %}

フレームワーク プリセットから hexo を選ぶ。大体はこの時に読み込まれた設定で問題ない。選択できたら保存してデプロイを押す。
{% asset_img cp5.png %}

deploy が始まる。しばらく待つ。
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

有効化された。これで cloudflare から配信されることになる。
{% asset_img cp13.png %}
