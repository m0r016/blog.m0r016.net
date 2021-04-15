---
title: Raspberry Piをセットアップする - 後編 -
date: 2021-04-07 14:27:03
updated: 2021-04-13 10:48:00
categories: [RaspberryPi]
tags: 
- Raspberry Pi SetUp
description: "Raspberry Piをセットアップする - 後編 -"
---
### はじめに
[前編](https://blog.m0r016.net/2021/04/07/setup-raspi/)では、Raspberry Pi 3BにUbuntu 20.04 LTSをインストールした。
ここではRaspberry Piにいろいろ設定を加えていく。

<!-- toc -->
<!-- more -->
### 1.ユーザーを作成する。
ubuntuでもいいのだが、せっかくだしHNのm0r016を使う。
`sudo adduser m0r016`
パスワードを聞かれるので適当にタイプ。名前を聞かれるが未記入で。
`Is the information correct`はEnter
sudoも与えておく。
`sudo gpasswd -a m0r016 sudo`
`su - m0r016`でm0r016にログイン。

### 2.タイムゾーンの変更。
`date`とタイプ。
UTCになっているので日本時間に変更する。
`sudo timedatectl set-timezone Asia/Tokyo`。
`date`で確認。JSTになっている。

### 3.hostnameの変更。
今の状態では`ubuntu`と割り当てられているが、わかりやすいようにhostnameを変更する。
このマシンはリレーサーバにする予定なので
`sudo hostnamectl set-hostname magi-system-relay`
とタイプ。

再ログインする。
`m0r016@magi-system-relay`。反映された。

### 4.ipアドレスを固定する。
このままでは勝手にipアドレスが変わってしまうので固定する。
`ip l`
とタイプ、`2: eth0:`と書かれている。
NICの名前はeth0らしい。これを覚えておく。
設定ファイルを変更する
```
sudo nano /etc/netplan/50-cloud-init.yaml
# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        eth0:
            + addresses: [192.168.1.32/24]
            +    gateway4: 192.168.1.1
            +    nameservers:
            +        addresses: [192.168.1.1]
            +        search: []
            + #dhcp4: true
            - dhcp4: true
            optional: true
    version: 2
```
`addresses`は割り当てたいアドレスを。`gateway`はゲートウェイ、`nameservers`内の`addresses`はネームサーバーだ。
`sudo netplan apply`
と打つとipアドレスが即座に変更されて、ssh接続が確立されなくなるので注意。

### 5.swapの作成。
RAM1GBでは足らないので2GBのswapを作成する。
```
sudo apt install dphys-swapfile
Reading package lists... Done
Building dependency tree
Reading state information... Done
sudo apt install dphys-swapfileThe following additional packages will be installed:
  dc
The following NEW packages will be installed:
  dc dphys-swapfile
0 upgraded, 2 newly installed, 0 to remove and 45 not upgraded.
Need to get 62.4 kB of archives.
After this operation, 204 kB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Get:1 http://ports.ubuntu.com/ubuntu-ports focal/main arm64 dc arm64 1.07.1-2build1 [47.4 kB]
Get:2 http://ports.ubuntu.com/ubuntu-ports focal/universe arm64 dphys-swapfile all 20100506-6 [15.0 kB]
Fetched 62.4 kB in 2s (27.7 kB/s)
Selecting previously unselected package dc.
(Reading database ... 99337 files and directories currently installed.)
Preparing to unpack .../dc_1.07.1-2build1_arm64.deb ...
Unpacking dc (1.07.1-2build1) ...
Selecting previously unselected package dphys-swapfile.
Preparing to unpack .../dphys-swapfile_20100506-6_all.deb ...
Unpacking dphys-swapfile (20100506-6) ...
Setting up dc (1.07.1-2build1) ...
Setting up dphys-swapfile (20100506-6) ...
Created symlink /etc/systemd/system/multi-user.target.wants/dphys-swapfile.service → /lib/systemd/system/dphys-swapfile.service.
Processing triggers for systemd (245.4-4ubuntu3.4) ...
Processing triggers for man-db (2.9.1-1) ...
Processing triggers for install-info (6.7.0.dfsg.2-5) ...
```
結構待たされる。

設定を変更する
```
sudo nano /etc/dphys-swapfile
# /etc/dphys-swapfile - user settings for dphys-swapfile package
# author Neil Franklin, last modification 2010.05.05
# copyright ETH Zuerich Physics Departement
#   use under either modified/non-advertising BSD or GPL license

# this file is sourced with . so full normal sh syntax applies

# the default settings are added as commented out CONF_*=* lines


# where we want the swapfile to be, this is the default
#CONF_SWAPFILE=/var/swap

# set size to absolute value, leaving empty (default) then uses computed value
#   you most likely don't want this, unless you have an special disk situation
+ CONF_SWAPSIZE=2048
- #CONF_SWAPSIZE=

# set size to computed value, this times RAM size, dynamically adapts,
#   guarantees that there is enough swap without wasting disk space on excess
#CONF_SWAPFACTOR=2

# restrict size (computed and absolute!) to maximally this limit
#   can be set to empty for no limit, but beware of filled partitions!
#   this is/was a (outdated?) 32bit kernel limit (in MBytes), do not overrun it
#   but is also sensible on 64bit to prevent filling /var or even / partition
#CONF_MAXSWAP=2048
```

`sudo service dphys-swapfile restart`
で反映だ。
`htop`で確認する。
`Swp`が`2G`であれば問題ない。
ざっくり設定するのはこの程度かな。お疲れさまでした。