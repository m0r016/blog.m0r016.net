---
title: shareボタンを導入する
date: 2021-03-06 23:17:11
updated: 2021-09-11 02:35:00
categories: [blog, hexo, share-botton]
tags:
  - share-botton
description: "hexoにshareボタンを導入する"
#thumbnail: "/2021/03/06/install-share-botton/addtoany-logo.png"
#cover: "/2021/03/06/install-share-botton/addtoany-title.png"
---

### 実行したこと

hexo に share ボタンを導入する

<!-- more -->
<!-- toc -->

### 1.icarus を fork する

どうせなら変更も github 上で管理できるようにしておきたいので[icarus](https://github.com/ppoffice/hexo-theme-icarus)を fork する。fork したものを submodule として追加。
{% codeblock terminal lang:bash line_number:false %}
git submodule add https://github.com/m0r016/hexo-theme-icarus.git
{% endcodeblock %}
これで submodule として追加することができた。

### 2.\_config.icarus.yml を編集する

share を探す。
{% codeblock _config.icarus.yml lang:diff first_line:187 %}
share:
type: sharethis
install_url: ''
{% endcodeblock %}
とある。
[ここ](https://ppoffice.github.io/hexo-theme-icarus/categories/Plugins/Share/)に設定の変更方法が書いてあるらしいので見てみる。

addtoany がシンプルに済みそうだ。addtoany に変更
{% codeblock _config.icarus.yml lang:diff first_line:187 %}
share:

- type: addtoany

* type: sharethis
* install_url: ''
  {% endcodeblock %}

### 3.share botton を取得する

[AddToAny](https://www.addtoany.com/)で取得できるらしい。
"Get the Share Botton"とあるのでクリック、"Any Website"を選択。
"Choose Services"で適当に選択。
"Get Button Code"で必要なコードを入手することができる。

### 4.share ボタンの設定

AddToAny の[レイアウトファイル](https://github.com/ppoffice/hexo-component-inferno/blob/0.2.2/src/view/share/addtoany.jsx)をリポジトリから引っ張ってくる必要があるらしい。これを`/themes/icarus/layout/search/addtoany.jsx`に書き込み、AddToAny で入手したコードを貼り付ける。
{% codeblock /themes/icarus/layout/share/addtoany.jsx lang:diff first_line:5 %}
const { Component, Fragment } = require('inferno');

- const { cacheComponent } = require('../../util/cache');

* const { cacheComponent } = require('hexo-component-inferno/lib/util/cache');

...Some code is skipped here...

class AddToAny extends Component {
render() {
return <Fragment>

-           <div class="a2a_kit a2a_kit_size_32 a2a_default_style">
-               <a class="a2a_dd" href="https://www.addtoany.com/share"></a>
-               <a class="a2a_button_facebook"></a>
-               <a class="a2a_button_twitter"></a>
-               <a class="a2a_button_telegram"></a>
-               <a class="a2a_button_whatsapp"></a>
-               <a class="a2a_button_reddit"></a>
-           </div>
-           <script src="https://static.addtoany.com/menu/page.js" defer={true}></script>

*           <!-- AddToAny HTML code you just got... -->
*           <div class="a2a_kit a2a_kit_size_32 a2a_default_style">
*               <a class="a2a_dd" href="https://www.addtoany.com/share"></a>
*               <a class="a2a_button_facebook"></a>
*               <a class="a2a_button_twitter"></a>
*               <a class="a2a_button_email"></a>
*           </div>
*           <script async src="https://static.addtoany.com/menu/page.js"></script>
          </Fragment>;
      }
  }
  {% endcodeblock %}
  保存して終了。

### 5.動作確認

`hexo g`をして問題ないことを確認、git に push する。
{% asset_img 00111.png browser %}
きちんと反映されている。

### P.S.

下の著作権表記が少し邪魔に感じるのでどこかにまとめておきたいところ

### 参考

[Icarus User Guide - Share Buttons](https://ppoffice.github.io/hexo-theme-icarus/categories/Plugins/Share/)
