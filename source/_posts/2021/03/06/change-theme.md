---
title: hexoのテーマをapolloからicarusに変更する
date: 2021-03-05 19:34:40
updated: 2022-03-30 21:02:00
categories: [blog, hexo, theme]
tags:
  - hexo-theme-changes
description: "hexoのテーマをapolloからicarusに変更する"
thumbnail: "/2021/03/05/change-theme/icarus.png"
cover: "/2021/03/05/change-theme/icarus-title.png"
---

### はじめに

hexo のテーマ変更が何気ややこしかったので備忘録としてここに記す。参考になれば幸いだ。

### 実行したこと

hexo のテーマを apollo から icarus に変更する

<!-- more -->
<!-- toc -->

### 概要

[apollo](https://github.com/AthenaYin/hexo-theme-apollo.git)を採用していたが、ネットサーフィンをしていたところ、[icarus](https://github.com/ppoffice/hexo-theme-icarus)というものを発見。これに変更する

### 1.前提パッケージを準備する

icarus には前提パッケージが必要らしい。知らずに github に上げてしまい、netlify 側でエラーを吐かれた。
{% codeblock terminal line_number:false %}
cd hexo-directory
npm install --save bulma-stylus@0.8.0 hexo-renderer-inferno@^0.1.3 hexo-component-inferno@^0.10.5 inferno@^7.3.3 inferno-create-element@^7.3.3
npm audit fix
{% endcodeblock %}

### 2.icarus を追加する

`git clone`でもいいのだが、変更を加える場合があるので`git submodule`する
{% codeblock terminal line_number:false %}
git submodule add https://github.com/ppoffice/hexo-theme-icarus themes/icarus
{% endcodeblock %}

また`git pull`したときに、`submodule`は自動で引っ張ってこないため、`git submodule update --init`を実行する
### 3.\_config.yml を変更し icarus 側の変更も加える

hexo に読み込ませるため theme を変更する
{% codeblock terminal line_number:false %}
hexo config theme icarus
hexo s
{% endcodeblock %}

この時点で一度 icarus に変わっている確認するといいだろう。
`hexo-ditectory`下に`_config.icarus.yml`に生成されているはず、これに変更を加えていく。
適所自分に変更してほしい。

{% codeblock _config.icarus.yml lang:diff first_line:6 mark:6 %}

- logo: /img/logo.svg

* logo:
* text: m'Tech
  head:
  favicon: /img/favicon.svg
  manifest:

-         name:

*         name: m'Tech

-         short_name:

*         short_name: m'Tech

-         start_url:

*         start_url: https://blog.m0r016.net/
        theme_color:
        background_color:
        display: standalone
        icons:
            -
                src: ''
                sizes: ''
                type:
  open_graph:
  title:
  type: blog
  url:
  image:
  site_name:
  author:
  description:

-         twitter_card:

*         twitter_card: summary

-         twitter_id:

*         twitter_id: m0r016

-         twitter_site:

*         twitter_site: 'https://twitter.com/m0r016'
          google_plus:
          fb_admins:
          fb_app_id:
  {% endcodeblock %}
  open_graph 内の twitter_card とは、Twitter に URL を張り付けたときに URL のみではなく一部の中身の情報が見れる機能のことである。ここでは summary を選択した。
  参考: [ツイートをカードで最適化する](https://developer.twitter.com/ja/docs/tweets/optimize-with-cards/guides/getting-started)

{% codeblock _config.icarus.yml lang:diff first_line:99 %}
navbar:
menu:
Home: /
Archives: /archives

- Categories: /categories
  Tags: /tags
  About: /about

* ReadMe: /readme
  {% endcodeblock %}
  About や ReadMe は初めから生成されておらず、ブラウザからアクセスすると 404 エラーが出る。そのため `hexo new page "about"` と打つと`source/_posts`下に生成されるのではなく`source/`下に生成され、参照できるようになる。

{% codeblock _config.icarus.yml lang:diff first_line:108 %}
links:

-     Download on GitHub:

*     Mastodon:

-       icon: fab fa-github

*       icon: fab fa-mastodon

-       url: 'https://github.com/ppoffice/hexo-theme-icarus'

*       url: 'https://slum.cloud/@m0r016'
  {% endcodeblock %}
  `your-sns-icon`は[ここのサイト](https://fontawesome.com/icons?d=gallery&p=2)にアクセスし
  {% asset_img fontawesome.png fontawesome %}
  検索窓から sns の名前を入力、使いたいアイコンをクリックし
  {% asset_img fontawesome1.png fontawesome %}
  の fab~の部分を your-sns-icon に入力するとアイコンが反映される。

{% codeblock _config.icarus.yml lang:diff first_line:213 %}
widgets: -
position: left
type: profile

-         author: Your name

*         author: もろ

-         author_title: Your title

*         author_title: m0r016

-         location: Your location

*         location: Ibaraki, Japan

-         avatar:

*         avatar: https://github.com/m0r016/blog.m0r016.net/blob/master/source/_posts/img/m0r016pm.png?raw=true

-         avatar_rounded: false

*         avatar_rounded: true
        gravatar:

-         follow_link: 'https://github.com/ppoffice'

*         follow_link: 'https://twitter.com/intent/follow?screen_name=m0r016'
        social_links:
            Github:
                icon: fab fa-github

-                 url: 'https://github.com/ppoffice'

*                 url: 'https://github.com/m0r016'

-           Facebook:
-             icon: fab fa-facebook
-             url: 'https://facebook.com'

*           Mastodon:
*             icon: fab fa-mastodon
*             url: 'https://slum.cloud/@m0r016'
            Twitter:
                icon: fab fa-twitter

-               url: 'https://twitter.com'

*               url: 'https://twitter.com/m0r016'

-           Dribbble:
-               icon: fab fa-dribbble
-               url: 'https://dribbble.com'
-            RSS:
-               icon: fas fa-rss
-               url: /
  - position: left
    type: toc
    index: true
    collapsed: true
    depth: 3
  - position: left
    type: links
    links:
-           Hexo: 'https://hexo.io'
-           Bulma: 'https://bulma.io'

*           Portfolio: 'https://identity.m0r016.net/'
  - position: left
    type: categories
  - position: left
    type: recent_posts
  - position: left
    type: archives
  - position: left
    type: tags
  - position: left
    type: subscribe_email
    description:
    feedburner_id: ''
  - position: left
    type: adsense

-       client_id: ''

*       client_id: 'your-google-adsense-client-id'

-       slot_id: ''

*       slot_id: 'your-google-adsense-slot-id'
  {% endcodeblock %}
  変更したのは大体この程度だ。`hexo s`で確認しておくとよいだろう。ポート指定は`hexo s -p port`

### 4.Github にアップロードする

{% codeblock terminal line_number:false %}
git add .
git commit -m "change themes/icarus"
git push origin master
{% endcodeblock %}

### 5.動作確認して終了

netlify に deploy させて終了、反映されていた。ここまで細かくいじれるとなると楽しい。
ほかにもいじれる箇所は多いので、調べてみると面白いだろう。

### 参考

[icarus の公式ドキュメント](https://blog.zhangruipeng.me/hexo-theme-icarus/)
[ツイートをカードで最適化する](https://developer.twitter.com/ja/docs/tweets/optimize-with-cards/guides/getting-started)
