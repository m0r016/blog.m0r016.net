---
title: Ammo UI Exを導入した
date: 2021-03-06 06:43:33
categories: [Game, PC]
tags: Fallout4
description: "Ammo UI Exを導入、弾薬の名前を英語表記にする"
thumbnail: "/2021/03/05/Ammo-UI-Exを導入した/Photo18.png"
---

### 実行したこと
Ammo UI Exを導入、弾薬の名前を英語表記にする

### 目次
<!-- toc -->

### 概要
私の好きなModder、[Tooun氏](https://www.patreon.com/tooun)の[Ammo UI HUD](https://www.nexusmods.com/fallout4/mods/42915)のEx版を入手したので導入する。
<!-- more -->

### したこと
いつも通り、[MO2](https://www.nexusmods.com/skyrimspecialedition/mods/6194)からインストールする。MO2は管理が楽でいい

### 導入成功
無事導入に成功した、実際に[動画](https://www.youtube.com/watch?v=Rq9rNsBISHY)のようなHUDになってくれてうれしい。(HP, APバーなど要所要所は書き換えている)
{% asset_img Photo17.png fo4 %}
{% youtube Rq9rNsBISHY %}

### Stringsファイルの変更
modページにも書いてあるが、弾薬名称が英語以外の文字を含んでいると文字化けしてしまうらしい。そのためxTranslatorで弾薬名称を変更した。DLC以外には対応してるはず→[ダウンロード](https://file.m0r016.net/index.php/s/A2r5R6AwoPWEXLr)

英語環境であれば保存した際に`Fallout_ja.*`と生成されてしまうので認識されるように`Fallout_en.*`と書き換える必要がある

### P.S.
UIの位置や自分のアイコンを指定し終了。うちの子はかわいいな
{% asset_img Photo18.png fo4}