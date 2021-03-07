---
title: 出力されたロードセルの値をCSVに落とし込んでみる
date: 2021-03-06 19:42:14
categories: [RaspberryPi, loadcell, Python, csv]
tags: 
- Python
- loadcell
- hx711
description: "ラズパイでロードセルの値を拾うことができたのでcsvに落とし、ログとして残せるようにする"
---

### 目次
<!-- toc -->

### 実行したこと
[ラズパイでロードセルの値を拾うことができた](https://blog.m0r016.net/2021/03/05/%E3%83%A9%E3%82%BA%E3%83%91%E3%82%A4%E3%82%92%E7%94%A8%E3%81%84%E3%81%A6%E3%83%AD%E3%83%BC%E3%83%89%E3%82%BB%E3%83%AB%E3%81%AE%E5%80%A4%E3%82%92%E6%8B%BE%E3%81%A3%E3%81%A6%E3%81%BF%E3%82%8B/)のでcsvに落とし、ログとして残せるようにする

### 環境

・Raspberry Pi 3b+ (Python 2.7.16 Raspbian)
・ロードセル([VLS-50K](https://www.valcom.co.jp/product/lc/vls/))
・A/Dコンバータ([hx711](https://akizukidenshi.com/catalog/g/gK-12370/))
<!-- more -->
### 1.ソースを見る

[github](https://github.com/tatobari/hx711py)のexample.pyを見る。

```python
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

# I've found out that, for some reason, the order of the bytes is not always the same between versions of python, numpy and the hx711 itself.
# Still need to figure out why does it change.
# If you're experiencing super random values, change these values to MSB or LSB until to get more stable values.
# There is some code below to debug and log the order of the bits and the bytes.
# The first parameter is the order in which the bytes are used to build the "long" value.
# The second paramter is the order of the bits inside each byte.
# According to the HX711 Datasheet, the second parameter is MSB so you shouldn't need to modify it.
hx.set_reading_format("MSB", "MSB")

# HOW TO CALCULATE THE REFFERENCE UNIT
# To set the reference unit to 1. Put 1kg on your sensor or anything you have and know exactly how much it weights.
# In this case, 92 is 1 gram because, with 1 as a reference unit I got numbers near 0 without any weight
# and I got numbers around 184000 when I added 2kg. So, according to the rule of thirds:
# If 2000 grams is 184000 then 1000 grams is 184000 / 2000 = 92.
#hx.set_reference_unit(113)
hx.set_reference_unit(referenceUnit)

hx.reset()

hx.tare()

print("Tare done! Add weight now...")

# to use both channels, you'll need to tare them both
#hx.tare_A()
#hx.tare_B()

while True:
    try:
        # These three lines are usefull to debug wether to use MSB or LSB in the reading formats
        # for the first parameter of "hx.set_reading_format("LSB", "MSB")".
        # Comment the two lines "val = hx.get_weight(5)" and "print val" and uncomment these three lines to see what it prints.
        
        # np_arr8_string = hx.get_np_arr8_string()
        # binary_string = hx.get_binary_string()
        # print binary_string + " " + np_arr8_string
        
        # Prints the weight. Comment if you're debbuging the MSB and LSB issue.
        val = hx.get_weight(5)
        print(val)

        # To get weight from both channels (if you have load cells hooked up 
        # to both channel A and B), do something like this
        #val_A = hx.get_weight_A(5)
        #val_B = hx.get_weight_B(5)
        #print "A: %s  B: %s" % ( val_A, val_B )

        hx.power_down()
        hx.power_up()
        time.sleep(0.1)

    except (KeyboardInterrupt, SystemExit):
        cleanAndExit()
```

雰囲気、54行目のwhile文のtryの中にある
```python
        val = hx.get_weight(5)
        print(val)
```
ここを書き換えればcsvに落とせるのではないかと考えた。

### 2.実装する
まずライブラリを増やさなければいけないためcsvのアクセスを可能とする`csv`と時間を取得する`datetime`を4行目以降に書き込んだ
```python
import time
import sys
import csv
import datetime
```

そして54行目付近の
```python
        val = hx.get_weight(5)
        print(val)
```
を書き換える。

今回は配列を使うことにした。
```python
        val = [0,0] #配列を用意する
        val[0] = datetime.datetime.now() #現在時間の取得(Raspberry Pi側で設定されている時間に依存する)
        val[1] = hx.get_weight(5) #ロードセルからの値を拾う
        print('now time ' + str(val[0]) + ',' + ' now wight ' + str(val[1])) #時間,値となるようにターミナルに出力する
```
配列の準備は完了した。

次はcsvにアクセスする
```python
        with open('./weight.csv', 'a') as f: #./にweight.csvを生成する、そこに追記するように書き込む('a')
            writer = csv.writer(f)
            writer.writerow(val) #配列の中身を書き込む
```
以上だ。

このようになった
```Python
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

# I've found out that, for some reason, the order of the bytes is not always the same between versions of python, numpy and the hx711 itself.
# Still need to figure out why does it change.
# If you're experiencing super random values, change these values to MSB or LSB until to get more stable values.
# There is some code below to debug and log the order of the bits and the bytes.
# The first parameter is the order in which the bytes are used to build the "long" value.
# The second paramter is the order of the bits inside each byte.
# According to the HX711 Datasheet, the second parameter is MSB so you shouldn't need to modify it.
hx.set_reading_format("MSB", "MSB")

# HOW TO CALCULATE THE REFFERENCE UNIT
# To set the reference unit to 1. Put 1kg on your sensor or anything you have and know exactly how much it weights.
# In this case, 92 is 1 gram because, with 1 as a reference unit I got numbers near 0 without any weight
# and I got numbers around 184000 when I added 2kg. So, according to the rule of thirds:
# If 2000 grams is 184000 then 1000 grams is 184000 / 2000 = 92.
#hx.set_reference_unit(113)
hx.set_reference_unit(referenceUnit)

hx.reset()

hx.tare()

print("Tare done! Add weight now...")

# to use both channels, you'll need to tare them both
#hx.tare_A()
#hx.tare_B()

while True:
    try:
        # These three lines are usefull to debug wether to use MSB or LSB in the reading formats
        # for the first parameter of "hx.set_reading_format("LSB", "MSB")".
        # Comment the two lines "val = hx.get_weight(5)" and "print val" and uncomment these three lines to see what it prints.
        
        # np_arr8_string = hx.get_np_arr8_string()
        # binary_string = hx.get_binary_string()
        # print binary_string + " " + np_arr8_string
        
        # Prints the weight. Comment if you're debbuging the MSB and LSB issue.
        
        val = [0,0]
        val[0] = datetime.datetime.now()
        val[1] = hx.get_weight(5)
        print('now time ' + str(val[0]) + ',' + ' now weight ' + str(val[1]))

        with open('./weight.csv', 'a') as f:
            writer = csv.writer(f)
            writer.writerow(val)


        # To get weight from both channels (if you have load cells hooked up 
        # to both channel A and B), do something like this
        #val_A = hx.get_weight_A(5)
        #val_B = hx.get_weight_B(5)
        #print "A: %s  B: %s" % ( val_A, val_B )

        hx.power_down()
        hx.power_up()

    except (KeyboardInterrupt, SystemExit):
        cleanAndExit()
```

### 4.動作確認
`python example.py`、ターミナルからは`now time 2021-03-06 19:59:06.036324, now weight -52`と出力され、csvには`2021-03-06 19:59:26.693445,562`と保存されるようになった。これでGoogle Sheetsに落とし込んでグラフ化したり、JavaScriptでグラフを作り、Web上で公開することができるようになった。

### 5.最後に

Pythonは扱いやすい言語と言われているが本当にそう感じた。JavaScriptを用いてグラフ出力をやってみたいと思っている。

### 参考

・[PythonでCSVファイルを読み込み・書き込み（入力・出力）](https://note.nkmk.me/python-csv-reader-writer/)
・[prog-8.com](https://prog-8.com/languages/python)