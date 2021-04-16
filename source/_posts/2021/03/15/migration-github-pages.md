---
title: netlifyからgithub pagesに乗り換える
date: 2021-03-15 14:40:34
updated: 2021-04-16 19:47:00
categories: [blog, hexo, github-pages]
tags:
- github-pages
description: "netlifyからgithub pagesに乗り換える"

---

### はじめに
netlifyはサイトのロード時間が長いと感じたため、github pagesに乗り換えようと思う。

### 目次
<!-- toc -->
<!-- more -->
### github pagesとは
[github pages](https://docs.github.com/ja/github/working-with-github-pages/about-github-pages)とは、githubが運営しているウェブホスティングサービスだ。
netlify同様、githubに上げたリポジトリをもとに、自動的に生成。もちろん送信すると自動的に再生成してくれる。

### 1.ブランチを作成する
github pagesではリポジトリではなく、ブランチ単位で生成できるため、ブランチを生成する
{% codeblock terminal lang:bash line_number:false %}
git branch public
{% endcodeblock %}

### 2.hexo-deployer-gitをインストールする
hexoにデプロイさせるため拡張機能をインストールする
{% codeblock terminal lang:bash line_number:false %}
npm install hexo-deployer-git --save
{% endcodeblock %}

インストールが完了したら、[hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git)をもとに設定を行う。`_config.yml`を開き、追加していく。
{% codeblock _config.yml lang:diff %}
+ deploy:
+   type: git
+   repo: your_repo
+   branch: public
{% endcodeblock %}

_config.ymlを保存する。

### 3.deployする
設定が完了したため、deployしていく。
{% codeblock terminal lang:bash line_number:false %}
hexo clean && hexo deploy
{% endcodeblock %}
デプロイが完了したら自分のリポジトリにアクセスし、Settingをクリック
{% asset_img github-pages.png %}

branchをpublicに指定、rootはそのままにsaveをする。
{% asset_img github-pages1.png %}

Custom domainが指定できるので指定したいドメインを追加。
{% asset_img github-pages2.png %}

CNAME、Aレコードどちらでも構わないが、私の場合CNAMEにした。
{% asset_img cname.png %}
コンテンツはyourgithubid.github.io
DNSのみにしておかないと、いろいろとめんどくさいので注意が必要。

### 4.pingを打ってみる。
本当にnetlifyからgithub pagesで配信されているのか調べるために、pingを打ってみた。
{% codeblock terminal lang:bash line_number:false %}
ping blog.m0r016.net
PING m0r016.github.io (185.199.109.153) 56(84) bytes of data.
{% endcodeblock %}
きちんと配信されていることがわかる。

### おわりに
netlifyからgithub pagesに乗り換えて、自環境では読み込みが早くなったように感じるが、[PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/)を見てみたが、割かしnetlifyのほうが早かった。何故だろうか・・・いろいろと調べてみたい。