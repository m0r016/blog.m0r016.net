---
title: 出力されたロードセルの値をCSVに落とし込んでみる
date: 2021-03-06 19:42:14
updated: 2021-04-16 15:57:00
categories: [RaspberryPi, loadcell, Python, csv]
tags: 
- raspi-loadcell
description: "ラズパイでロードセルの値を拾うことができたのでcsvに落とし、ログとして残せるようにする"
---

### はじめに
ラズパイでロードセルの値を拾うことでできたが、csvにも残せたらグラフに表せたりするため、解析等が楽になると思った。

### 目次
<!-- toc -->

<!-- more -->
### 1.ソースを見る

[github](https://github.com/tatobari/hx711py)のexample.pyを見る。

{% codeblock example.py lang:python %}
#! /usr/bin/python2

import time
import sys

EMULATE_HX711=False

referenceUnit = 1

if not EMULATE_HX711:
    import RPi.GPIO as GPIO
    from hx711 import HX711
else:
    from emulated_hx711 import HX711

def cleanAndExit():
    print("Cleaning...")

    if not EMULATE_HX711:
        GPIO.cleanup()
        
    print("Bye!")
    sys.exit()

hx = HX711(5, 6)
hx.set_reading_format("MSB", "MSB")
hx.set_reference_unit(referenceUnit)
hx.reset()
hx.tare()

print("Tare done! Add weight now...")

while True:
    try:
        val = hx.get_weight(5)
        print(val)
        hx.power_down()
        hx.power_up()
        time.sleep(0.1)

    except (KeyboardInterrupt, SystemExit):
        cleanAndExit()
{% endcodeblock %}

雰囲気、54行目のwhile文のtryの中にある
{% codeblock example.py lang:python first_line:57 %}
        val = hx.get_weight(5)
        print(val)
{% endcodeblock %}
ここを書き換えればcsvに落とせるのではないかと考えた。
While文とは、条件式が真の間だけ、繰り返し実行するものである。

### 2.実装する
まずプログラムに必要なものを取り込まなくてはならない。今回はcsvと時間が欲しいため、csvのアクセスを可能とする`csv`と時間を取得する`datetime`を4行目以降に書き込んだ。
{% codeblock _config.icarus.yml lang:diff %}
import time
import sys
+ import csv
+ import datetime
{% endcodeblock %}

そして54行目付近の
{% codeblock _config.icarus.yml lang:python first_line:65 %}
        val = hx.get_weight(5)
        print(val)
{% endcodeblock %}
を書き換える。

今回は配列を使うことにした。
配列とは変数に比べ、複数の要素を含むことができるもののことだ。
{% codeblock _config.icarus.yml lang:diff first_line:65 %}
-        val = hx.get_weight(5)
-        print(val)
+        val = [0,0] #配列を用意する
+        val[0] = datetime.datetime.now() #現在時間の取得(Raspberry Pi側で設定されている時間に依存する)
+        val[1] = hx.get_weight(5) #ロードセルからの値を拾う
+        print('now time ' + str(val[0]) + ',' + ' now wight ' + str(val[1])) #時間,値となるようにターミナルに出力する

+        with open('./weight.csv', 'a') as f: #./にweight.csvを生成する、そこに追記するように書き込む('a')
+            writer = csv.writer(f)
+            writer.writerow(val) #配列の中身を書き込む
{% endcodeblock %}
以上だ。配列の中に代入したものを取り出し、csvに書き込むことができるようになった。

このようになった
{% codeblock _config.icarus.yml lang:python %}
#! /usr/bin/python2

import time
import sys
import csv
import datetime

EMULATE_HX711=False

referenceUnit = 1

if not EMULATE_HX711:
    import RPi.GPIO as GPIO
    from hx711 import HX711
else:
    from emulated_hx711 import HX711

def cleanAndExit():
    print("Cleaning...")

    if not EMULATE_HX711:
        GPIO.cleanup()
        
    print("Bye!")
    sys.exit()

hx = HX711(5, 6)

hx.set_reading_format("MSB", "MSB")

hx.set_reference_unit(referenceUnit)

hx.reset()

hx.tare()

print("Tare done! Add weight now...")

while True:
    try:
        val = [0,0]
        val[0] = datetime.datetime.now()
        val[1] = hx.get_weight(5)
        print('now time ' + str(val[0]) + ',' + ' now weight ' + str(val[1]))

        with open('./weight.csv', 'a') as f:
            writer = csv.writer(f)
            writer.writerow(val)

        hx.power_down()
        hx.power_up()

    except (KeyboardInterrupt, SystemExit):
        cleanAndExit()
{% endcodeblock %}

### 4.動作確認
{% codeblock terminal lang:bash line_number:false %}
python example.py
now time 2021-03-06 19:59:06.036324, now weight -52
{% endcodeblock %}

ターミナルからは`now time 2021-03-06 19:59:06.036324, now weight -52`と出力され、csvには`2021-03-06 19:59:26.693445,562`と保存されるようになった。
これでGoogle Sheetsに落とし込んでグラフ化したり、JavaScriptでグラフを作り、Web上で公開することができるようになった。

### 5.最後に

Pythonは扱いやすい言語と言われているが本当にそう感じた。JavaScriptを用いてグラフ出力をやってみたいと思っている。

### 参考

・[PythonでCSVファイルを読み込み・書き込み（入力・出力）](https://note.nkmk.me/python-csv-reader-writer/)
・[prog-8.com](https://prog-8.com/languages/python)