---
title: install-soapbox
date: 2021-04-23 14:00:25
categories: [Fediverse, pleroma, soapbox]
tags:
- soapbox
description: "SoapboxFEをサブドメインにインストールする。"
---
### はじめに
[SoapboxFE](https://soapbox.pub/)を導入する。

### 1.configを書き換える。
pleromaユーザーにログインし、configを書き換える。場所は`/opt/pleroma/config/*`
config.exsをもとにprod.secret.exsに設定を書き足す
{% codeblock prod.secret.exs lang:elixir fist_line:78 %}
config :pleroma, :frontends,
  available: %{
    "kenoma" => %{
      "name" => "kenoma",
      "git" => "https://git.pleroma.social/lambadalambda/kenoma",
      "build_url" =>
        "https://git.pleroma.social/lambadalambda/kenoma/-/jobs/artifacts/${ref}/download?job=build",
      "ref" => "master"
    },
    "pleroma-fe" => %{
      "name" => "pleroma-fe",
      "git" => "https://git.pleroma.social/pleroma/pleroma-fe",
      "build_url" =>
        "https://git.pleroma.social/pleroma/pleroma-fe/-/jobs/artifacts/${ref}/download?job=build",
      "ref" => "develop"
    },
    "fedi-fe" => %{
      "name" => "fedi-fe",
      "git" => "https://git.pleroma.social/pleroma/fedi-fe",
      "build_url" =>
        "https://git.pleroma.social/pleroma/fedi-fe/-/jobs/artifacts/${ref}/download?job=build",
              "ref" => "master",
      "custom-http-headers" => [
        {"service-worker-allowed", "/"}
      ]
    },
    "admin-fe" => %{
      "name" => "admin-fe",
      "git" => "https://git.pleroma.social/pleroma/admin-fe",
      "build_url" =>
        "https://git.pleroma.social/pleroma/admin-fe/-/jobs/artifacts/${ref}/download?job=build",
      "ref" => "develop"
    },
    "soapbox-fe" => %{
      "name" => "soapbox-fe",
      "git" => "https://gitlab.com/soapbox-pub/soapbox-fe",
      "build_url" =>
        "https://gitlab.com/soapbox-pub/soapbox-fe/-/jobs/artifacts/${ref}/download?job=build-production",
      "ref" => "v1.2.3",
      "build_dir" => "static"
    }
  }

  config :pleroma, :frontends,
  primary: %{
    "name" => "soapbox-fe",
    "ref" => "v1.2.3"
  },
  admin: %{
    "name" => "admin",
    "ref" => "develop"
  }
{% endcodeblock %}

### 2.コマンドを実行
{% codeblock terminal lang:bash %}
su - pleroma
cd /opt/pleroma
MIX_ENV=prod mix pleroma.frontend install soapbox-fe
{% endcodeblock %}