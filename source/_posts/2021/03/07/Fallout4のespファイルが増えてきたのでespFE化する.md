---
title: Fallout4のespファイルが増えてきたのでespFE化する
date: 2021-03-07 15:57:30
categories: [Game, PC]
tags: Fallout4
description: "Modを入れていくうちにespファイルが多くなってしまったのでespFE化をする。"
thumbnail: "/2021/03/06/Fallout4のespファイルが増えてきたのでespFE化する/mo2.png"
---

### 実行したこと
Modを入れていくうちにespファイルが多くなってしまったのでespFE化をする。

### 目次
<!-- toc -->

<!-- more -->

### 1.なぜ？
Fallout4はesm, esp, eslファイルとある。esm, espファイルはロードオーダーを00-FDまで使ってしまうのだが、eslファイルはFEのみしか使わない。
ここを利用しespファイルをFE領域にあるように認識させてあげると、導入できるmodの数がかなり向上する。これを利用する。

### 2.eslファイルとは
eslファイルというものはForm IDの無駄を切り詰めることにより従来のespファイルの一枠分に収まるようにする。
Fallout4のForm IDは8ケタで表記されるが、eslの最上位2ケタはFEで固定される。
次に続く3ケタはesl間のロードを表し、000～FFFまでの4096個がeslを導入できる理論上の上限となる。
次に続く3ケタが一つのeslファイルで扱える上限となる。

つまりこのようになる
FEXXXYYY
Xがesl内のロードオーダ、Yがesl内のForm ID。

一つ癖がありFE000000～FE000FFFまでが使えるように思うかもしれないが実際はFE000800～FE000FFFまでであるため、FE000800以下を使用するForm IDは使えない。そのためFE000800～FE000FFFまでの2048個がesl内で使えるForm IDになる。
eslファイルは好きにロード順を変えることができない。

### 3.espFEファイルとは
espファイルにESLフラグをつけ、拡張子はそのままでeslとして使えるようにしたファイルのことである。外見は変わらないが中身が変わっているといったイメージだ。
eslファイルに対しespFEのロード順は好きに変更することができる。

### 4.Fo4editの準備
[Fo4edit](https://www.nexusmods.com/fallout4/mods/2737)をダウンロードし、任意の場所に展開、`FO4Edit.exe`を右クリックしショートカットを作成、プロパティを開きリンク先に` -PseudoESL`と追加。これでespFE化できるファイルを選別し表示してくれる。(MO2のユーザーはMO2から起動するように設定する)

### 5.変換作業
Fo4editを起動したらプラグインにチェックマークを付け、右に`Background Loader: finished`と表示されるまで待つ。左側にForm IDとありその下に[FE XXX]と表示されているものがあるだろう。それがespFE化が可能なesp達だ。
選択したらRecord Header内のRecord Flagsをダブルクリック。警告画面が出てくるので、Yes I'm absolutely sureをクリック、真ん中あたりにあるeslを選別するとespFE化が完了する。`Ctrl + S`で保存することができる

### P.S.
だいぶ減った。素晴らしい・・・mod沼がこうすることでもっと深くなってしまうんだな
{% asset_img mo2.png mo2 %}