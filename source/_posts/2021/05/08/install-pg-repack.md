---
title: pg_repackをインストールする。
date: 2021-05-08 14:36:31
updated: 2021-05-08 14:36:31
categories: [Ubuntu, zfs]
tags:
- zfsutils-linux
- Mastodon
description: "サービスを提供しつつdbのサイズを削減する。"
---
### はじめに
sudo nano /etc/fstab
mnt/md0 削除
sudo reboot

sudo mdadm --fail /dev/md0 /dev/sdd
sudo mdadm --remove /dev/md0 /dev/sdd
sudo zpool create -f raid1 /dev/sdd
sudo zfs set mountpoint=/mnt/md0 raid1
sudo zfs set compress=on raid1
sudo chown m0r016:m0r016 /mnt/md0/
sudo mkdir /mnt/sdc
sudo mount /dev/md0 /mnt/sdc
rsync -av /mnt/sdc/ /mnt/md0/
sudo reboot
sudo zpool attach raid1 /dev/sdc /dev/sdd