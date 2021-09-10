---
title: よく使うコマンド集
date: 2021-06-06 23:17:47
updated: 2021-09-11 02:27:31
categories: 雑記
---

### はじめに

私がよく使うコマンドで忘れてしまって毎回毎回調べてしまうものをここにまとめる。

<!-- more -->
<!-- toc -->

### Oracle cloud 用

Oracle Cloud では`ufw`が使えないらしく、`iptables`を使う必要があるらしい。
{% codeblock terminal lang:bash line_number:false %}
sudo echo -A INPUT -p tcp -m state --state NEW -m tcp --dport [port] -j ACCEPT >> /etc/iptables/rules.v4
{% endcodeblock %}
tcp は状況に応じて udp に、`>>`はかならず`>>`にすること。**`>`にしたら追記ではなく上書きになってしまう。** [参考](https://dacelo.space/linux/entry-979.html)
心配なのであれば nano 等のエディタを使うこと。

### Raspberry Pi の IP アドレスを特定する

{% codeblock terminal lang:bash line_number:false %}
sudo arp-scan -l --interface wlan0
{% endcodeblock %}
`wlan0`は NIC の名前
