---
title: PostgreSQLを再インストールする
date: 2021-05-01 23:58:09
updated: 2021-05-08 14:38:00
categories: [Ubuntu, PostgreSQL]
tags:
- PostgreSQL
description: "PostgreSQLを再インストールする"
---
### はじめに
`sudo apt purge postgresq;l*`をしたときに、設定ファイルが消されずうまく再インストールができないという問題が生ずる。これを解決する。

### 1.PostgreSQLをアンインストールする。
{% codeblock terminal lang:bash %}
sudo apt purge postgresql*
sudo rm -r /etc/postgresql/
sudo rm -r /etc/postgresql-common/
sudo rm -r /var/lib/postgresql/
sudo userdel -r postgres
sudo groupdel postgres
{% endcodeblock %}

### 2.PostgreSQLをインストールする。
{% codeblock terminal lang:bash %}
sudo apt install postgresql
{% endcodeblock %}
