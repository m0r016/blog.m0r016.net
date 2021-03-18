---
title: vercelに乗り換える
date: 2021-03-18 11:33:26
categories: [blog, hexo, vercel]
tags:
- vercel
description: "github pagesからvercelに乗り換える"

---

### はじめに
github pagesですら少し重いなと感じたのでvercelに乗り換える

### 目次
<!-- toc -->
<!-- more -->
### vercelとは
[vercel](https://vercel.com/)とは、ウェブホスティングサービスであり、
netlify同様githubに上げたリポジトリをもとに、自動的に生成。

### 3.deployする
deproyする。
[vercel](https://vercel.com/)にログインし、Start Deployingを指定。
{% asset_img start-deploy.png %}

このような画面になるのでdeployしたいリポジトリを指定。
{% asset_img start-deploy1.png %}

personal accountを指定。
{% asset_img start-deploy2.png %}

deployが完了するのでopen dashboardをクリック
{% asset_img start-deploy3.png %}

settingsをクリック→domainをクリック
{% asset_img setting-domain.png %}

mywebsite.comに公開したいアドレスを入力
{% asset_img setting-domain1.png %}

DNSに追加してよと言われるので、言われたとおりにする
{% asset_img setting-domain2.png %}

### 4.pingを打ってみる。
github pagesからvercelで配信されているのか調べるために、pingを打ってみた。
```
ping blog.m0r016.net
PING cname.vercel-dns.com (76.76.21.21) 56(84) bytes of data.
```
きちんと配信されていることがわかる。

### おわりに
vercelになって、爆速に感じた。このサービスはサイト運営するくらいでは無料分で十分なのだ。素晴らしい。