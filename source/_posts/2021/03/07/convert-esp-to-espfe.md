---
title: Fallout4のespファイルが増えてきたのでespFE化する
date: 2021-03-07 15:57:30
updated: 2021-09-11 02:34:30
categories: [PC, Game, Fallout4]
tags: Fallout4
description: "Modを入れていくうちにespファイルが多くなってしまったのでespFE化をする。"
---

### 実行したこと

Mod を入れていくうちに esp ファイルが多くなってしまったので espFE 化をする。

### 目次

<!-- more -->
<!-- toc -->

### 1.espFE？

Fallout4 のデータファイルにはさまざまな形式があるが、mod を構成する際によく使う esm, esp, esl ファイルとある。
esm, esp ファイルはロードオーダーを 00-FD まで使ってしまうが、esl ファイルとはロードオーダーの FE の部分を利用する。
そうすることで FD までという制限を突破し、より多くの mod を導入することができるようになる。

### 2.ロードオーダーの話

Fallout4 のオブジェクトには Form ID というものがある。
いくら mod を入れようが、ある値を超えなければ同じ Form ID を使うことはない。
つまり A より B のほうが大きい場合、A の機能が B に上書きされてしまい、A の機能が使えなくなる、といったことが避けられるのだ。
Form ID は 8 ケタで表記されるが、最上位 2 桁はロードオーダー順になる。01 ,02 ,03 といった感じだ。
esp と esl の違いはここにある。esp は下 6 桁をすべて使えるのだが、esl は下 2 桁を使えない。
esp は多くの Form ID が扱え、esl は少ない Form ID しか使えない。
esl の最上位 2 桁は FE に固定され、次の 3 桁が esl 内のロードオーダーの順番に使われる。以降の 3 桁(800-FFF)までが esl として利用できる Form ID となる。
esl の最上位 2 ケタは FE で固定される。
次に続く 3 ケタは esl 間のロードを表し、000 ～ FFF までの 4096 個が esl を導入できる理論上の上限となる。

### 3.espFE ファイルとは

esp ファイルに ESL フラグをつけ、拡張子はそのままで esl として使えるようにしたファイルのことである。外見は変わらないが中身が変わっているといったイメージだ。
esl ファイルに対し espFE のロード順は好きに変更することができる。

### 4.Fo4edit の準備

[Fo4edit](https://www.nexusmods.com/fallout4/mods/2737)をダウンロードし、任意の場所に展開、`FO4Edit.exe`を右クリックしショートカットを作成、プロパティを開きリンク先に` -PseudoESL`と追加。
これで espFE 化できるファイルを選別し表示してくれる。(MO2 のユーザーは MO2 から起動するように設定する)

### 5.変換作業

Fo4edit を起動したらダイアログが出てくる。
{% asset_img fo4edit-load.png %}
プラグインにチェックマークを付け、OK を押す。

右に`Background Loader: finished`と表示されるまで待つ。
{% asset_img fo4edit-load-complete.png %}

左側に Form ID とありその下に[FE XXX]と表示されているものがあるだろう。それが espFE 化が可能な esp 達だ。
{% asset_img fo4edit.png %}

選択したらダブルクリックする。そうすると右側の画面に mod の中身が表示される。
Record Header 内の Record Flags をダブルクリック。
{% asset_img fo4edit1.png %}

警告画面が出てくるので、Yes I'm absolutely sure をクリック
{% asset_img fo4edit-warn.png %}

真ん中あたりにある esl の右側にあるチェックボックスをクリック
{% asset_img fo4edit-esl-check.png %}
すると espFE 化が完了する。`Ctrl + S`で保存することができる

### P.S.

だいぶ減った。素晴らしい・・・mod 沼がこうすることでもっと深くなってしまうんだな
{% asset_img mo2.png mo2 %}
