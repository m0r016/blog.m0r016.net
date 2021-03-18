---
title: hexoのテーマをapolloからicarusに変更する
date: 2021-03-05 19:34:40
categories: [blog, hexo, theme]
tags:
- hexo-theme-changes
description: "hexoのテーマをapolloからicarusに変更する"
thumbnail: "/2021/03/05/change-theme/icarus.png"
cover: "/2021/03/05/change-theme/icarus-title.png"
---

### はじめに
hexoのテーマ変更が何気ややこしかったので備忘録としてここに記す。参考になれば幸いだ。

### 実行したこと
hexoのテーマをapolloからicarusに変更する

### 目次
<!-- toc -->

### 概要
[apollo](https://github.com/AthenaYin/hexo-theme-apollo.git)を採用していたが、ネットサーフィンをしていたところ、[icarus](https://github.com/ppoffice/hexo-theme-icarus)というものを発見。これに変更する
<!-- more -->

### 1.前提パッケージを準備する
icarusには前提パッケージが必要らしい。知らずにgithubに上げてしまい、netlify側でエラーを吐かれた。

```
cd hexo-directory
npm install --save bulma-stylus@0.8.0 hexo-renderer-inferno@^0.1.3 hexo-component-inferno@^0.10.5 inferno@^7.3.3 inferno-create-element@^7.3.3
npm audit fix
```

### 2.icarusを追加する
`git clone`でもいいのだが、変更を加える場合があるので`git submodule`する
```
git submodule add https://github.com/ppoffice/hexo-theme-icarus
```

### 3._config.ymlを変更しicarus側の変更も加える
hexoに読み込ませるためthemeを変更する
```
hexo config theme icarus
hexo s
```
この時点で一度icarusに変わっている確認するといいだろう。
`hexo-ditectory`下に`_config.icarus.yml`に生成されているはず、これに変更を加えていく。
適所自分に変更してほしい。

```
# Version of the configuration file
(省略)
head:
  manifest:
    name: your-blog-name
    shot_name: your-blog-short-name
    start_url: your-blog-url
  open_graph:
    twitter_card: summary
    twitter_id: your-id
    twitter_site: your-twitter-url
```
open_graph内のtwitter_cardとは、TwitterにURLを張り付けたときにURLのみではなく一部の中身の情報が見れる機能のことである。ここではsummaryを選択した。[ツイートをカードで最適化する](https://developer.twitter.com/ja/docs/tweets/optimize-with-cards/guides/getting-started)

```
navbar:
  menu: 
    Home: /
    Archives: /archives
    Tags: /tags
    About: /About
    ReadMe: /readme
```
AboutやReadMeは初めから生成されておらず、ブラウザからアクセスすると404エラーが出る。そのため
```hexo new page "about"```
と打つと`source/_posts`下に生成されるのではなく`source/`下に生成され、参照できるようになる。

```
  links:
    your-sns:
      icon: your-sns-icon
      url: 'your-sns-url'
```
`your-sns-icon`は[ここのサイト](https://fontawesome.com/icons?d=gallery&p=2)にアクセスし
{% asset_img fontawesome.png fontawesome %} 
検索窓からsnsの名前を入力、使いたいアイコンをクリックし
{% asset_img fontawesome1.png fontawesome %}
のfab~の部分をyour-sns-iconに入力するとアイコンが反映される。

```
widgets:
  author: your-name
  author_title: your-title
  location: your-location
  avator: your-avator.png
  avator_rounded: true #アイコンの角を丸くするかどうか
  follow_ling: your-follow-url
  social_links:
    Github:
      icon: fab fa-github
      url: your-github-url
    Twitter:
      icon: fab fa-twitter
      url: your-twitter-url
```
変更したのは大体この程度だ。`hexo s`で確認しておくとよいだろう。ポート指定は`hexo s -p port`

### 4.Githubにアップロードする
```
git add .
git commit -m "change themes/icarus"
git push origin master
```

### 5.動作確認して終了
netlifyにdeployさせて終了、反映されていた。ここまで細かくいじれるとなると楽しい。
ほかにもいじれる箇所は多いので、調べてみると面白いだろう。

### 参考
[icarusの公式ドキュメント](https://blog.zhangruipeng.me/hexo-theme-icarus/)