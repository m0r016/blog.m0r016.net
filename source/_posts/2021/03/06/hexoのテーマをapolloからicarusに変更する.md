---
title: hexoのテーマをapolloからicarusに変更する
date: 2021-03-05 19:34:40
categories: [blog, hexo, theme]
tags:
- hexo-theme-apollo
- hexo-theme-icarus
description: "hexoのテーマをapolloからicarusに変更する"
---

### 実行したこと
hexoのテーマをapolloからicarusに変更する

### 目次
<!-- toc -->

### 概要
[apollo](https://github.com/AthenaYin/hexo-theme-apollo.git)を採用していたが、ネットサーフィンをしていたところ、[icarus](https://github.com/ppoffice/hexo-theme-icarus)というものを発見、これに変更する
<!-- more -->

### 1.前提パッケージを準備する
icarusには前提パッケージが必要らしい。知らずにgithubに上げてしまい、netlify側でエラーを吐かれた。

```
npm install --save bulma-stylus@0.8.0 hexo-renderer-inferno@^0.1.3 hexo-component-inferno@^0.10.5 inferno@^7.3.3 inferno-create-element@^7.3.3
npm audit fix
```

### 2.icarusを追加する
`git clone`でもいいのだが管理がめんどくさいので
```
npm install hexo-theme-icarus
```

### 3._config.ymlを変更する
hexoに読み込ませるためthemeを変更する
```
hexo config theme icarus
```
`./_config.yml`内のテーマを書き換えてもいいが、私の場合`_config.icarus.yml`が生成されなかったのでお勧めしない（一度ローカルで`hexo g`する必要があったのだろう）

### 4.Githubにアップロードする
```
git add .
git commit -m "change themes/icarus"
git push origin master
```

### 5.動作確認
netlifyでdeployできているか確認できたので設定に進む

### 6._config.icarus.ymlを変更
icarusの設定がしたいため`./_config.icarus.yml`を変更


### 7.動作確認して終了
割とたくさんの設定項目があり、細かく指定できるのだなと感じた。