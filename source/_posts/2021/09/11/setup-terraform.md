---
title: TerraformでOCIのインスタンスを確保する
date: 2021-09-11 02:05:30
updated: 2021-09-21 02:04:30
categories: [Terraform]
tags:
  - Terraform
description: "クラウド上のコンピュータやネットワークの構築を自動化するTerraformというツールを用いて、Oracle上のインスタンスを確保する"
---

> Ubuntu(20.04)より操作。

### なぜ Oracle なのか

VPS を使うときに GCP や AWS、さくらクラウドなど様々なものがあるが、なるべくコストをかけずに済ませたいと考えたときに Free tier という存在がある。
GCP はサービスにもよるが一年間、AWS もサービスによるが一年間というイメージだ。
その中で Oracle が運営している Oracle Cloud Infrastructure は、
・1/8 OCPU と 1GB メモリをそれぞれ備えた 2 つの AMD ベースのコンピュート VM
・1 つの VM または最大 4 つの VM として使用可能な 4 つの Arm ベースの Ampere A1 コアと 24GB のメモリ
を Free tier として提供している。

あまりにも豊富な Free tier のため、使ってみたいと思うのはいうまでもない。
私も[pleroma](https://wut.m0r016.net)や身内で使っている Minecraft サーバーを下の`1つのVMまたは最大4つのVMとして使用可能な4つのArmベースのAmpere A1コアと24GBのメモリ`で運用している。
今のところ使用する上でのストレスはほぼ感じていない。がこの OCI やはり他が提供している Free tier より提供しているマシンが強いため、人気がとてもある。
そのため Tokyo リージョンはほぼほぼ取れない。（Osaka リージョンは楽に取れたと夏頃風の噂で聞いたが）
管理画面から確保できるまで粘るのもいいが、Terraform という自動でインスタンスを確保してくれる便利なツールがある。今回はこれを使う。

<!-- more -->
<!-- toc -->

### 1.Terraform のインストール

Terraform をインストールする
{% codeblock terminal lang:bash line_number:false %}
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=$(dpkg --print-architecture)] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt update
sudo apt install terraform
{% endcodeblock %}

`terraform -v`で terraform が入ってることが確認できたら ok

### 2.RSA キーを作成する

`<your-rsa-key-name>`は自分の好きな名前に変更してほしい

.oci ディレクトリを作成する。
{% codeblock terminal lang:bash line_number:false %}
mkdir $HOME/.oci
{% endcodeblock %}

次に秘密鍵を生成する
{% codeblock terminal lang:bach line_number:false %}
openssl genrsa -out $HOME/.oci/<your-rsa-key-name>.pem 2048
{% endcodeblock %}

権限を変更する、自分だけが読み書きできるようにする
{% codeblock terminal lang:bach line_number:false %}
chmod 600 $HOME/.oci/<your-rsa-key-name>.pem
{% endcodeblock %}

公開鍵を生成する
{% codeblock terminal lang:bash line_number:false %}
openssl rsa -pubout -in $HOME/.oci/<your-rsa-key-name>.pem -out $HOME/.oci/<your-rsa-key-name>\_public.pem
{% endcodeblock %}

公開鍵をコピーする
{% codeblock terminal lang:bash line_number:false %}
cat $HOME/.oci/<your-rsa-key-name>\_public.pem
{% endcodeblock %}

### 3.公開鍵をユーザーアカウントに追加する

{% asset_img userAvatar.png [ユーザーアバター] %}
ユーザーアバターをクリックし、

{% asset_img userSettings.png [ユーザー設定] %}
ユーザー設定をクリックする。

{% asset_img resource.png [リソース] %}
リソースから API キーをクリックする

{% asset_img apiKey.png [APIキーの追加] %}
API キーの追加をクリックし

{% asset_img apiKeyModal.png [APIキーの追加モーダル] %}
公開キーの貼り付けを選択し、`BEGIN PUBLIC KEY`から`END PUBLIC KEY`までを入力欄に貼り付け、追加をクリック。

### 4.必要な情報を収集する

Terraform に必要な情報を収集する。

- tenancy-ocid: <tenancy-ocid>
  - ユーザーアバターから、Tenancy:your-tenancy をクリック
  - OCID の横にあるコピーをクリックし、メモ帳にコピーする
- user-ocid: <user-ocid>
  - ユーザーアバターから、ユーザー設定を選択
  - OCID の横にあるコピーをクリックし、メモ帳にコピーする
- finger: <fingerprint>
  - ユーザーアバターから、ユーザー設定に移動
  - リソース内にある API キーをクリックする
  - 登録した公開鍵のフィンガープリントをメモ帳にコピーする
- region-identifier: <region-identifier>
  - 上部のリージョンが書いてある部分をクリック
  - リージョンの管理をクリックする
  - 一番上にあるはずの利用しているリージョンのリージョン識別子をメモ帳にコピーする
- rsa-private-key-path: <rsa-private-key-path>
  - 2.RSA キーを生成するで生成したパスをコピー
  - (私の環境ではうまくいかなかったため`/home/your-user/.oci/<your-rsa-key-name>.pem とした)

### 5.スクリプトを作成する

ディレクトリを作成する
{% codeblock terminal lang:bash line_number:false %}
mkdir tf-provider
{% endcodeblock %}

`provider.tf`というファイルを作成する
{% codeblock terminal lang:bash line_number:false %}
touch provider.tf
{% endcodeblock %}

`provider.tf`を編集する。
4の必要な情報を収集するで収集した<hoge>を当てはめる、文字列は""で囲むこと
{% codeblock provider.tf lang:bash line_number:false %}
provider "oci"{
  tenancy_ocid = "<tenancy-ocid>"
  user_ocid = "<user-ocid>"
  private_key_path = "<rsa-private-key-path>"
  fingerprint = "<fingerprint>"
  region = "<region-identifier>"
}
{% endcodeblock %}

`availability-domeins.tf`というファイルを作成する。
{% codeblock terminal lang:bash line_number:false %}
touch availability-domains.tf
{% endcodeblock %}

`availability-domains.tf`を編集する。
4の必要な情報を収集するで収集した<tenancy-ocid>を置き換える、文字列は""で囲むこと
{% codeblock availability-domains.tf lang:bash line_numner:false %}
// Source from https://registry.terraform.io/providers/hashicorp/oci/latest/docs/data-sources/identity_availability_domains

// <tenancy-ocid> is the compartment OCID for the root compartment.
// Use <tenancy-ocid> for the compartment OCID.

data "oci_identity_availability_domains" "ads" {
  compartment_id = "<tenancy-ocid>"
}
{% endcodeblock %}

`output.tf`を作成する
{% codeblock terminal lang:bash line_number:false %}
touch output.rf
{% endcodeblock %}

`output.tf`を以下のように編集する
{% codeblock output.tf lang:bash line_number:false %}
// Output the "list" of a;; availability domains
output "all-availability-domains-in-your-tenancy" {
  value = data.oci_identity_availability_domains.ads.availability_domains
}
{% endcodeblock %}

### 3.スクリプトを実行スクリプトを実行
`terraform init`を実行する。
`ls -al`で`tf-provider`内に`.terraform`というフォルダが生成されているのを確認する。

`terraform plan`を実行し、`Plan: 0 to add, 0 to change, 0 to destroy`を返していることを確認する。

`terraform apply`を実行し`Apply complete! Resources: 0 added, 0 changed, 0 destroyed.`と返していることを確認する。

これで問題なければ、terraformでociを叩けることができた。

### 4.tfファイルを生成する。
インスタンスを作るときに、`作成`ではなく`スタックとして保存`を使う。
インスタンス名やコンパーメントを選んだら作成をクリック。
{% asset_img stack-create.png [stack-create] %}

管理画面に遷移したら、tfファイルをダウンロードする
{% asset_img tf-download.png [tf-download.png] %}

ダウンロードしたzipファイルを解答し、中身のmain.tfを適当な名前に変え、編集する
{% codeblock create-instance lang:diff line_number:false %}
- provider "oci" {}

resource "oci_core_instance" "generated_oci_core_instance" {
･･･
{% endcodeblock %}
provider "oci" {}は既に定義済みなので削除する。

保存できたら`terraform plan`で`1 added`となっていることを確認し、`terraform apply`で実際に生成される。

### 5.cronに設定する
always free枠を狙う場合、毎回毎回コマンド実行するのも面倒なのでcronに設定する。今回は一分おきにした
{% codeblock cron line:diff line_number:false %}
  */1  *   *   *   * cd /terraform-dir/ && /usr/bin/terraform apply -auto-approve >> /var/log/cron.log
{% endcodeblock %}
terraformの構成ファイルが入っているディレクトリにアクセス、そのディレクトリでサイレントで構築するような形態になっている。