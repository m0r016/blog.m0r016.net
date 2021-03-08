---
title: Fallout4のespファイルが増えてきたのでespFE化する
date: 2021-03-07 15:57:30
categories: [Game, PC]
tags: Fallout4
description: "Modを入れていくうちにespファイルが多くなってしまったのでespFE化をする。"
---

### 実行したこと
Modを入れていくうちにespファイルが多くなってしまったのでespFE化をする。

### 目次
<!-- toc -->

<!-- more -->

### 1.espFE？
Fallout4のデータファイルにはさまざまな形式があるが、modを構成する際によく使うesm, esp, eslファイルとある。
esm, espファイルはロードオーダーを00-FDまで使ってしまうが、eslファイルとはロードオーダーのFEの部分を利用する。
そうすることでFDまでという制限を突破し、より多くのmodを導入することができるようになる。

### 2.ロードオーダーの話
Fallout4のオブジェクトにはForm IDというものがある。
いくらmodを入れようが、ある値を超えなければ同じForm IDを使うことはない。
つまりAよりBのほうが大きい場合、Aの機能がBに上書きされてしまい、Aの機能が使えなくなる、といったことが避けられるのだ。
Form IDは8ケタで表記されるが、最上位2桁はロードオーダー順になる。01 ,02 ,03といった感じだ。
espとeslの違いはここにある。espは下6桁をすべて使えるのだが、eslは下2桁を使えない。
espは多くのForm IDが扱え、eslは少ないForm IDしか使えない。
eslの最上位2桁はFEに固定され、次の3桁がesl内のロードオーダーの順番に使われる。以降の3桁(800-FFF)までがeslとして利用できるForm IDとなる。
eslの最上位2ケタはFEで固定される。
次に続く3ケタはesl間のロードを表し、000～FFFまでの4096個がeslを導入できる理論上の上限となる。

### 3.espFEファイルとは
espファイルにESLフラグをつけ、拡張子はそのままでeslとして使えるようにしたファイルのことである。外見は変わらないが中身が変わっているといったイメージだ。
eslファイルに対しespFEのロード順は好きに変更することができる。

### 4.Fo4editの準備
[Fo4edit](https://www.nexusmods.com/fallout4/mods/2737)をダウンロードし、任意の場所に展開、`FO4Edit.exe`を右クリックしショートカットを作成、プロパティを開きリンク先に` -PseudoESL`と追加。
これでespFE化できるファイルを選別し表示してくれる。(MO2のユーザーはMO2から起動するように設定する)

### 5.変換作業
Fo4editを起動したらダイアログが出てくる。
{% asset_img fo4edit-load.png %}
プラグインにチェックマークを付け、OKを押す。

右に`Background Loader: finished`と表示されるまで待つ。
{% asset_img fo4edit-load-complete.png %}

左側にForm IDとありその下に[FE XXX]と表示されているものがあるだろう。それがespFE化が可能なesp達だ。
{% asset_img fo4edit.png %}

選択したらダブルクリックする。そうすると右側の画面にmodの中身が表示される。
Record Header内のRecord Flagsをダブルクリック。
{% asset_img fo4edit1.png %}

警告画面が出てくるので、Yes I'm absolutely sureをクリック
{% asset_img fo4edit-warn.png %}

真ん中あたりにあるeslの右側にあるチェックボックスをクリック
{% asset_img fo4edit-esl-check.png %}
するとespFE化が完了する。`Ctrl + S`で保存することができる

### P.S.
だいぶ減った。素晴らしい・・・mod沼がこうすることでもっと深くなってしまうんだな
{% asset_img mo2.png mo2 %}