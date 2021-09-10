---
title: Raspberry Piをセットアップする - 後編 -
date: 2021-04-07 14:27:03
updated: 2021-09-11 02:31:00
categories: [RaspberryPi]
tags:
  - Raspberry Pi SetUp
description: "Raspberry Piをセットアップする - 後編 -"
---

### はじめに

[前編](https://blog.m0r016.net/2021/04/07/setup-raspi/)では、Raspberry Pi 3B に Ubuntu 20.04 LTS をインストールした。
ここでは Raspberry Pi にいろいろ設定を加えていく。

<!-- more -->
<!-- toc -->

### 1.ユーザーを作成する。

ubuntu でもいいのだが、せっかくだし HN の m0r016 を使う。
`sudo adduser m0r016`
パスワードを聞かれるので適当にタイプ。名前を聞かれるが未記入で。
`Is the information correct`は Enter
sudo も与えておく。
`sudo gpasswd -a m0r016 sudo`
`su - m0r016`で m0r016 にログイン。

### 2.タイムゾーンの変更。

`date`とタイプ。
UTC になっているので日本時間に変更する。
`sudo timedatectl set-timezone Asia/Tokyo`。
`date`で確認。JST になっている。

### 3.hostname の変更。

今の状態では`ubuntu`と割り当てられているが、わかりやすいように hostname を変更する。
このマシンはリレーサーバにする予定なので
`sudo hostnamectl set-hostname magi-system-relay`
とタイプ。

再ログインする。
`m0r016@magi-system-relay`。反映された。

### 4.ip アドレスを固定する。

このままでは勝手に ip アドレスが変わってしまうので固定する。
`ip l`
とタイプ、`2: eth0:`と書かれている。
NIC の名前は eth0 らしい。これを覚えておく。
設定ファイルを変更する
{% codeblock /etc/netplan/99-ip.yaml lang:diff %}
network:
ethernets:
eth0:

-            addresses: [192.168.1.32/24]
-               gateway4: 192.168.1.1
-               nameservers:
-                   addresses: [192.168.1.1]
-                   search: []
-            #dhcp4: true

*            dhcp4: true
              optional: true
      version: 2
  {% endcodeblock %}
  `addresses`は割り当てたいアドレスを。`gateway`はゲートウェイ、`nameservers`内の`addresses`はネームサーバーだ。
  `sudo netplan apply`
  と打つと ip アドレスが即座に変更されて、ssh 接続が確立されなくなるので注意。

### 5.swap の作成。

RAM1GB では足らないので 2GB の swap を作成する。
`sudo apt install dphys-swapfile`
結構待たされる。

設定を変更する
{% codeblock /etc/dphys-swapfile lang:diff %}
// /etc/dphys-swapfile - user settings for dphys-swapfile package
// author Neil Franklin, last modification 2010.05.05
// copyright ETH Zuerich Physics Departement
// use under either modified/non-advertising BSD or GPL license

// this file is sourced with . so full normal sh syntax applies

// the default settings are added as commented out CONF\__=_ lines

// where we want the swapfile to be, this is the default
//CONF_SWAPFILE=/var/swap

// set size to absolute value, leaving empty (default) then uses computed value
// you most likely don't want this, unless you have an special disk situation

- CONF_SWAPSIZE=2048

* #CONF_SWAPSIZE=

// set size to computed value, this times RAM size, dynamically adapts,
// guarantees that there is enough swap without wasting disk space on excess
//CONF_SWAPFACTOR=2

// restrict size (computed and absolute!) to maximally this limit
// can be set to empty for no limit, but beware of filled partitions!
// this is/was a (outdated?) 32bit kernel limit (in MBytes), do not overrun it
// but is also sensible on 64bit to prevent filling /var or even / partition
//CONF_MAXSWAP=2048
{% endcodeblock %}

`sudo service dphys-swapfile restart`
で反映だ。
`htop`で確認する。
`Swp`が`2G`であれば問題ない。
ざっくり設定するのはこの程度かな。お疲れさまでした。
