---
title: Raspberry Piをセットアップする - 前編 -
date: 2021-04-07 11:00:15
categories: [RaspberryPi]
tags: 
- Raspberry Pi SetUp
---
### はじめに
一台余っている[Raspberry Pi](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/)があるのでセットアップする。

<!-- toc -->
<!-- more -->
### Raspberry Pi 3Bと3B+の違い。
ラズパイには3Bと3B+がある。
CPUは同じであるが、3B+のほうが0.2GHzUPの1.4GHz。
SoCもBCM2837からBGM2837B0。
LANは1000BASE-T対応など、3B+のほうがアップグレードされている。PoEにも対応しているらしい。
あと電源回りも少し変わっているらしい。

### 1.セットアップ
まず、SDカードにOSをインストールする。
[ここ](https://www.raspberrypi.org/software/)からSDカードにOSを書き込むソフトを入手する。
`Download for Windows`をクリック
{% asset_img download-imager.png %}

`imager_X.X.X.exe`を開く。恐らくUACが出てくるので「はい」を押す。
インストールを選択。
{% asset_img install-imager.png %}

`Run Raspberry Pi Imager`にチェックを入れFinish
{% asset_img comp-imager.png %}

`Choose OS`をクリックする。
{% asset_img chooseos.png %}

私の場合、Ubuntuを使いたいので`Other general purpose OS`を選択。
{% asset_img chooseos1.png %}

`Ubuntu`を選択。
{% asset_img chooseos2.png %}

`Ubuntu 20.04 LTS`を選択。
{% asset_img chooseos3.png %}

`Storage`の`Choose Sto...`を選択。
{% asset_img storage.png %}

使うSDカードを選択。
{% asset_img storage1.png %}

選択できたら`Write`を選択。
{% asset_img write.png %}

フォーマットしていいかと聞かれるので`Yes`を選択。
{% asset_img yes.png %}

書き込みが終わったらSDカードにアクセスする。
私の場合`system-boot`だった。拡張子のないファイル[ssh]を作る。
Wifiを使う場合、`network-config`を編集する。
``` network-config
# This file contains a netplan-compatible configuration which cloud-init
# will apply on first-boot. Please refer to the cloud-init documentation and
# the netplan reference for full details:
#
# https://cloudinit.readthedocs.io/
# https://netplan.io/reference
#
# Some additional examples are commented out below

version: 2
ethernets:
  eth0:
    dhcp4: true
    optional: true
+wifis:
+  wlan0:
+    dhcp4: true
+    optional: true
+    access-points:
+      "wifi-ssid":
+        password: "wifi-password"
#wifis:
#  wlan0:
#    dhcp4: true
#    optional: true
#    access-points:
#      myhomewifi:
#        password: "S3kr1t"
#      myworkwifi:
#        password: "correct battery horse staple"
#      workssid:
#        auth:
#          key-management: eap
#          method: peap
#          identity: "me@example.com"
#          password: "passw0rd"
#          ca-certificate: /etc/my_ca.pem
```
`wifi-ssid`はwi-fiのssid、`wifi-password`はwi-fiのpasswordをタイプ。

編集できたら取り出し、ラズパイに入れる。
しばらく待つと接続できるのでその間にTeraTermのセットアップ

### 2.TeraTermのインストール
[TeraTerm](http://ttssh2.osdn.jp/)とは国産SSHクライアントである。SSHのクライアントといえばこれである。[ダウンロード](https://ja.osdn.net/projects/ttssh2/releases/)してくる。

exeを選択。
{% asset_img teraterm-download.png %}

またUACが出てくるので「はい」
日本語を選択する。
{% asset_img teraterm-install.png %}

同意を選択。
{% asset_img teraterm-install1.png %}

デフォルトで問題ないと思う。TeraTerm Menuはチェックしておくと便利だ。
{% asset_img teraterm-install2.png %} 

日本語を選択。
{% asset_img teraterm-install3.png %}

次へ
{% asset_img teraterm-install4.png %}

インストールをクリック。
{% asset_img teraterm-install5.png %}

今すぐTeraTermを実行するを選択し完了。
{% asset_img teraterm-install6.png }

### 3.Raspberry PiのIPアドレス特定。
Windowsの場合、Windowsキーを押して`cmd`とタイプ
{% asset_img cmd.png %}

クリックし実行
{% asset_img cmd1.png %}

`for /l %i in (0,1,255) do ping -w 1 -n 1 192.168.1.%i`とタイプしエンター
{% asset_img cmd2.png %}
文字が流れ出すので終了するまで待つ。
`192.168.1.%i`はLANに合わせて変更する。

終了したら`arp -a`とタイプしエンター
{% asset_img cmd3.png %}
`b8-27`から始まるものがラズパイだ。
私の場合、`192.168.1.18`だった。

先ほどインストールしたTeraTermに戻り、ホストを`192.168.1.18`にする。
{% asset_img tt.png %}

続行を選択。
{% asset_img tt2.png %}

ユーザーを`ubuntu`、パスフレーズも`ubuntu`にしてok
{% asset_img tt3.png %}

接続成功だ。
{% asset_img tt4.png %}

次にパスワードを変更する。`Current password:`と聞かれているので、`ubuntu`とタイプ。
`New password`と出てくるので自分で決めたパスワードをタイプ。
`Retype new password`と出てくるのでもう一度先ほどのパスワードをタイプ。
TeraTermが閉じてしまうので、先ほど通り、ホストは`192.168.1.18`、ユーザーは`ubuntu`、パスフレーズは先ほど決めたパスワードを入力してokするとターミナルに戻ってこれる。

ひとまずはこの辺で終了だ。