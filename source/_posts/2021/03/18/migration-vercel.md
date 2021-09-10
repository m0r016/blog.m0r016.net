---
title: vercelに乗り換える
date: 2021-03-18 11:33:26
updated: 2021-09-11 02:33:00
categories: [blog, hexo, vercel]
tags:
  - vercel
description: "github pagesからvercelに乗り換える"
---

### はじめに

github pages ですら少し重いなと感じたので vercel に乗り換える

### 目次

<!-- more -->
<!-- toc -->

### vercel とは

[vercel](https://vercel.com/)とは、ウェブホスティングサービスであり、
netlify 同様 github に上げたリポジトリをもとに、自動的に生成。

### 1.deploy する

deproy する。
[vercel](https://vercel.com/)にログインし、Start Deploying を指定。
{% asset_img start-deploy.png %}

このような画面になるので deploy したいリポジトリを指定。
{% asset_img start-deploy1.png %}

personal account を指定。
{% asset_img start-deploy2.png %}

deploy が完了するので open dashboard をクリック
{% asset_img start-deploy3.png %}

settings をクリック →domain をクリック
{% asset_img setting-domain.png %}

mywebsite.com に公開したいアドレスを入力
{% asset_img setting-domain1.png %}

DNS に追加してよと言われるので、言われたとおりにする
{% asset_img setting-domain2.png %}

### 2.ping を打ってみる。

github pages から vercel で配信されているのか調べるために、ping を打ってみた。
{% codeblock terminal lang:bash line_number:false %}
ping blog.m0r016.net
PING cname.vercel-dns.com (76.76.21.21) 56(84) bytes of data.
{% endcodeblock %}
きちんと配信されていることがわかる。

### おわりに

vercel になって、爆速に感じた。このサービスはサイト運営するくらいでは無料分で十分なのだ。素晴らしい。
