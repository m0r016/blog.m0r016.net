---
title: cronのログを出力する。
date: 2021-04-30 14:54:55
updated: 2021-05-02 00:06:00
categories: [Ubuntu, cron]
tags:
- cron
description: "cronのログを出力する。"
---
### はじめに
{% post_link bulding-rss-bot MastodonにAtomの情報を投稿する。 %}がきちんと動いているか、不安になったため、cronで実行したログを出させることにする。

### 1.コメントアウトを外す。
{% codeblock /etc/rsyslog.d/50-default.conf lang:diff %}
//  Default rules for rsyslog.
//
//                       For more information see rsyslog.conf(5) and /etc/rsysl>

//
// First some standard log files.  Log by facility.
//
auth,authpriv.*                 /var/log/auth.log
*.*;auth,authpriv.none          -/var/log/syslog
- //cron.*                          /var/log/cron.log
+ cron.*                          /var/log/cron.log 
daemon.*                        -/var/log/daemon.log
kern.*                          -/var/log/kern.log
//lpr.*                          -/var/log/lpr.log
mail.*                          -/var/log/mail.log
//user.*                         -/var/log/user.log
{% endcodeblock %}

### 2.サービスを再起動する。
{% codeblock terminal lang:bash %}
sudo service rsyslog restart
{% endcodeblock %}