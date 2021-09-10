---
title: netlifyからgithub pagesに乗り換える
date: 2021-03-15 14:40:34
updated: 2021-09-11 02:33:00
categories: [blog, hexo, github-pages]
tags:
  - github-pages
description: "netlifyからgithub pagesに乗り換える"
---

### はじめに

netlify はサイトのロード時間が長いと感じたため、github pages に乗り換えようと思う。

### 目次

<!-- more -->
<!-- toc -->

### github pages とは

[github pages](https://docs.github.com/ja/github/working-with-github-pages/about-github-pages)とは、github が運営しているウェブホスティングサービスだ。
netlify 同様、github に上げたリポジトリをもとに、自動的に生成。もちろん送信すると自動的に再生成してくれる。

### 1.ブランチを作成する

github pages ではリポジトリではなく、ブランチ単位で生成できるため、ブランチを生成する
{% codeblock terminal lang:bash line_number:false %}
git branch public
{% endcodeblock %}

### 2.hexo-deployer-git をインストールする

hexo にデプロイさせるため拡張機能をインストールする
{% codeblock terminal lang:bash line_number:false %}
npm install hexo-deployer-git --save
{% endcodeblock %}

インストールが完了したら、[hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git)をもとに設定を行う。`_config.yml`を開き、追加していく。
{% codeblock _config.yml lang:diff %}

- deploy:
- type: git
- repo: your_repo
- branch: public
  {% endcodeblock %}

\_config.yml を保存する。

### 3.deploy する

設定が完了したため、deploy していく。
{% codeblock terminal lang:bash line_number:false %}
hexo clean && hexo deploy
{% endcodeblock %}
デプロイが完了したら自分のリポジトリにアクセスし、Setting をクリック
{% asset_img github-pages.png %}

branch を public に指定、root はそのままに save をする。
{% asset_img github-pages1.png %}

Custom domain が指定できるので指定したいドメインを追加。
{% asset_img github-pages2.png %}

CNAME、A レコードどちらでも構わないが、私の場合 CNAME にした。
{% asset_img cname.png %}
コンテンツは yourgithubid.github.io
DNS のみにしておかないと、いろいろとめんどくさいので注意が必要。

### 4.ping を打ってみる。

本当に netlify から github pages で配信されているのか調べるために、ping を打ってみた。
{% codeblock terminal lang:bash line_number:false %}
ping blog.m0r016.net
PING m0r016.github.io (185.199.109.153) 56(84) bytes of data.
{% endcodeblock %}
きちんと配信されていることがわかる。

### おわりに

netlify から github pages に乗り換えて、自環境では読み込みが早くなったように感じるが、[PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/)を見てみたが、割かし netlify のほうが早かった。何故だろうか・・・いろいろと調べてみたい。
