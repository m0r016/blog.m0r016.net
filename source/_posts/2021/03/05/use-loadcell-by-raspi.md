---
title: ラズパイを用いて重量を図ってみる。
date: 2021-03-05 15:04:38
categories: [RaspberryPi, loadcell, hx711]
tags: 
- raspi-loadcell
thumbnail: "/2021/03/05/use-loadcell-by-raspi/raspi.jpg"
cover: "/2021/03/05/use-loadcell-by-raspi/raspi-title.png"
---

### 目次
<!-- toc -->

### はじめに
ロードセルという重量を図るものをRaspberry Piに接続し、それを見ることはできないか、と言われ調べてやってみたことをここに記す。
Googleで検索してみてもこの話題は出てこないため、少しでも参考になればいいなと思う。

### 環境

・SSH接続が可能な[Raspberry Pi 3b+](https://www.raspberrypi.org/products/raspberry-pi-3-model-b-plus/) (Python 2.7.16 Raspbian)
・ロードセル([VLS-50K](https://www.valcom.co.jp/product/lc/vls/))
・ADコンバータ([hx711](https://akizukidenshi.com/catalog/g/gK-12370/))
・[ジャンパ線](https://akizukidenshi.com/catalog/g/gC-08934/)
・ネット環境

<!-- more -->
### 1.ADコンバータを組み立てる

入手できた[hx711](https://akizukidenshi.com/catalog/g/gK-12370/)は端子台の取り付けが必要だったためはんだ付けを施した。
出来栄えはこんな感じ↓

表面
{% asset_img 0.jpg 表面 %}

裏面
{% asset_img 1.jpg 裏面 %}

裏面のJ3, J4はBチャンネルを使用しない場合にGNDに接続するはんだジャンパーらしい。今回はBチャンネルを使わないのではんだジャンパをした。

はんだジャンパとははんだで二点(ここではJ3とJ4)を繋げる方法だ。

### 1.ロードセル(VLS-50K)の配線確認
[製品サイト](https://www.valcom.co.jp/product/lc/vls/)の配線接続図を見た。+入力の赤, +出力の緑, -入力の白, -出力の黒, シールドとある。シールドは透明なようだ。

hx711の[取説](https://akizukidenshi.com/download/ds/akizuki/ae-hx711-sip_20190607.pdf)を見るにCN2, CN3が該当すると考えられる。

CN2-1がロードセル用電源とあるので、赤
CN2-2がGND(グラウンド)、白
CN3-1がAch-入力とあるので、緑
CN3-2がAch-入力とあるので、黒

|線の色|hx711側|
|---:|---:|
|赤|CN2-1|
|白|CN2-2| 
|黒|CN3-1|
|緑|CN3-2|

↑の通りにA/Dコンバータとロードセルを接続した。

### 3.ADコンバータとRaspberry Piを接続する

ADコンバータのCN1に接続する。

CN1-1が電源入力(VDD)とあるので、2
CN1-2がデータ出力(DAT)とあるので、39
CN1-3がクロック入力(CLK)とあるので、31
CN1-6がGNDとあるので6
CN1-4, 5はBチャンネル用なので使用しない。

|線の色|ラズパイ側|
|---:|---:|
|VDD|2(5v)|
|DAT|39(GPIO5)|
|CLK|31(GPIO6)|
|GND|6(GND)|
|INNB|none|
|INPB|none|

↑の通りにA/DコンバータとRaspberry Piを接続した

GPIOのポートについてはここを参考にしてほしい→[GPIO](https://www.raspberrypi.org/documentation/usage/gpio/)

{% asset_img GPIO.png GPIO %}

### 4.動作確認

接続ができたら動作確認をする。(SSH,Python実行環境は既に構築できているものとする)

リポジトリはtatobari氏のものを使った→[tatobari](https://github.com/tatobari/hx711py)

```console
ssh [id]@[ipaddress]
git clone https://github.com/tatobari/hx711py
cd hx711py
python setup.py install
python example.py
```

コンソール上に数字列が数秒おきに出力された。成功だ。

### 最後に

簡易的ではあるがロードセルの値を拾うことができた、次はcsvに落とし込んでみたい。