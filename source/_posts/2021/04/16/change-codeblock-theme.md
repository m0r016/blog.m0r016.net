---
title: 従来のcodeblockの配色が気に入らないので変更する
date: 2021-04-16 21:11:17
updated: 2021-04-16 21:29:00
categories: [blog, hexo, codeblock]
tags:
- hexo-icarus-customize
description: "従来のcodeblockの配色が気に入らないので変更する"

---

### はじめに
{% asset_img codeblock.png %}
従来のcodeblockの配色が気に入らないので変更する

<!-- toc -->
<!-- more-->
### _config.icarus.ymlを変更する
色々調べていたところ、`_config.icarus.yml`の`highlight`を変更すればいいという。実際にやってみる。
{% codeblock _config.icarus.yml lang:diff first_line:125 %}
    highlight:
        // Code highlight themes
        // https://github.com/highlightjs/highlight.js/tree/master/src/styles
-        theme: atom-one-light
+        theme: monokai
        // Show copy code button
        clipboard: true
        // Default folding status of the code blocks. Can be "", "folded", "unfolded"
        fold: unfolded
{% endcodeblock %}

`monokai`に変更した。
[highlight.js demo](https://highlightjs.org/static/demo/)にDemoがある。
欲しいテーマを選び、同じ文字列をこの[リポジトリ](https://github.com/highlightjs/highlight.js/tree/9.18.1/src/styles)から探し、themeの中に書き込む。

{% asset_img monokai.png %}
変更できた。