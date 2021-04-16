---
title: サイトの読み込み速度を高速化する。
date: 2021-04-05 11:39:35
updated: 2021-04-16 21:00:00
categories: [blog, hexo, plugin]
tags:
- hexo-all-minifier
description: "サイトの読み込み速度を高速化する。"
---
### はじめに
サイトを運営することによって気にすることは何が。
一つにページそのものの"重さ"がある。
転送量が増えると外部回線で見ているものはより料金がかかるしストレスを生む。
そうしないためにいくらか圧縮をかけてあげる必要がある。

<!-- toc -->
<!-- more -->
### 1.サイトの読み込み時間を確認する。
Googleが読み込み速度を確認するサイトを用意してくれているのでそこで確認する→[Google PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/)

{% asset_img pc.png %}
pcはそこそこではあるが、

{% asset_img mobile.png %}
スマホはまぁまぁ重い

そのため、[hexo-all-minifier](https://github.com/chenzhutian/hexo-all-minifier)というプラグインを導入する。

### 2.hexo-all-minifierを導入する。
ターミナル上で`npm install hexo-all-minifier --save`と打つ。
そしたら`_config.yml`に`all_minifier: true`と追加する。

### 3.反映しどれくらいよくなったか確かめる。
deployし、どれくらい早くなったかを確かめる。
buildに170秒もかかっている。Github pagesでは使えないな。
vercelだからできたこと。
ではPageSpeed Insightsにかけてみる。

{% asset_img pc-comp.png %}
{% asset_img mobile-comp.png %}
...
そんな変わっていない。
多分そういうことを見ているのではないのだろうな・・・
もうちょいましにならないか調べてみる。

### 4.画像を遅延読み込みにする。
調べていたところ、このようなものを発見。
[Hexoで画像にloading="lazy"を自動で追加して画像を遅延読み込みする](https://pixelog.net/post/vo9d9z/)
`themes/icarus/scripts/`に`lazyload.js`というファイルを作成し、
中身をこのようにする
{% codeblock themes/icarus/scripts/lazyload.js lang:javascript %}
hexo.extend.filter.register('after_post_render', function(data){
  data.content = data.content.replace(/<img src=/g, '<img loading="lazy" src=');
  return data;
});
{% endcodeblock %}

これでどうだろうか
{% asset_img pc-lazy.png %}
{% asset_img mobile-lazy.png %}

ぐぬぬ・・・もう少し調べる必要がある。

