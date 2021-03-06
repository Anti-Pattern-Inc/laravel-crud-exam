## ストーリー

我々は以下の画像のような自社サービスを保有しています。

![sample](./sample.png)

そして、今までこのサービスはAWS以外の環境でデプロイされて保守・運用されてきましたが、AWSに載せ替えたいと考えています。

このサービスは現在成長中で昼にアクセスが集中し始めています。そして、サービスのSLAは99.9%と定義しているため可用性を担保してアプリケーションを構築する必要があります。

さらにプロダクトオーナーからは以下のようなことも言われています。

* __*個人情報を扱っているため、なるべくセキュアな通信を行いたい*__
* __*画像などのアセットはキャッシュを活用してなるべくレスポンス速度を上げてほしい*__
* __*[以前発生したAWSの障害](https://aws.amazon.com/jp/message/56489/)のことを考えると、何らかの対応をしてほしい*__
* __*アクセスが集中する時以外はアプリケーションを実行するサーバの費用は抑えたい*__
* __*メトリクス等は監視しておいて、異常が検知された場合はチャットにアラートを投げてほしい*__
* __*AWSのリソースのログをなるべく残すようにしてほしい、ビジネスサイドとの分析に使いたい*__
* __*XSSやSQLi等の悪意ある攻撃者からアプリケーションを守りたい*__

## 前提条件

* 弊社より試験用に開設したアカウントを使用してください(以下試験用アカウントと呼ぶ)
* 東京リージョン(ap-northeast-1)で構築を行ってください
* 試験用アカウントに共有されている指定のAMIを使用してください: `aws_exam_v1.1(ami-XXXXXXXX)`
    * 東京リージョンでプライベートイメージを選択すると指定のAMIが表示されます
* 指定のAMIでアプリケーションを実行するためには以下のリソースを立ち上げることが最低条件となります
    * Aurora MySQL(エンジンバージョン: 5.6.10a)
    * Elasticache Memcached(エンジンバージョン: 1.5.16)
    * S3
* [リンク](./explain_asset.md)から以下のjsonと画像をダウンロードしてS3にアップロードしてください、詳細は[リンク](./explain_asset.md)に記載されています
    * workers.json
    * workers.jpg
* 試験用アカウントのRoute 53には以下のようなドメインが割り当てられています。DNSレコードを扱う場合は以下をお使いください
    * 割り当てられるドメイン: `{受験番号}.engineed-exam.com` 
    * 例: 受験番号が `00000` の場合に割り当てられるドメイン => `00000.engineed-exam.com`
* アラートを投げる先は弊社で用意するSlackのチャンネルになります
    * 試験開始後にそのSlackチャンネルに招待します
    * Slackチャンネルに投稿するためのWebhook URLも試験開始後に共有されます
* また指定のAMIを実行するときに以下のユーザーデータを指定してください

```bash
#!/bin/bash

######################################################
# このAMIにはaws-cliがインストールされています
# 任意の処理を付け加えたい場合はここに記述してください




######################################################


# 立ち上げたリソースに合わせて以下の環境変数の値を設定してください
export DB_HOST={データベースのホスト名}
export DB_DATABASE={データベース名}
export DB_USERNAME={ユーザーネーム}
export DB_PASSWORD={パスワード}
export AWS_DEFAULT_REGION={デプロイしているリージョン}
export AWS_BUCKET={S3のバケット名}
export MEMCACHED_HOST={Memcachedのホスト名}
export ASSETS_ENDPOINT={アセットを配信するエンドポイント}

sh /home/ec2-user/exam/userdata.sh
```
ユーザーデータが正しく実行された場合、以下のような状態になります。

* アプリケーションが立ち上がり、HTTPで通信が受けられるようになります
* データベースの確認を行って、テーブルやレコードが存在していない場合のみ、テーブルを作成してレコードをインサートします

## アプリケーションの仕様

* ヘルスチェックのパスに `/login` を指定してください
* `/js/`,`/css/` 配下からは静的コンテンツが配信されます
* `/register` からサインアップができます
* サインアップ、ログインをすると `/` に対してアクセスできるようになります
* 指定のAMIでアプリケーションを立ち上げると EC2の `/home/ec2-user/exam/src/storage/logs` 配下に `laravel-yyyy-mm-dd.log` という形式でログが生成されます
* アプリケーションのエラーログは `server.ERROR: ` という文字列を含んでいます

## 提出方法

* 試験用アカウントで上記アプリケーションが動く環境を構築してください
* TerraformやCloudFormationで構築した場合、そのコードも提出してください
* 採点には直接影響しませんが、構成図を書いた場合は提出してください

## リソース作成時のお願い

一部採点を自動化しており、採点対象を明確化するために作成したAWSリソースに以下のタグを必ず付与してください。

`aws-exam-resource: true`

タグが付与できるAWSのリソースにおいて、上記のタグが付与されていない場合、採点の対象外になりますのでご了承ください。

## 期限

一週間を目安にできるところまで構築してください。

一週間を目安にしていますが、オーバーしても構いません。この試験は加点式を採用しているため、思う存分、自分がベストだと思う構成を構築してください。その際は弊社にご連絡ください。

## その他

質問がある場合は、試験開始後に共有されるSlackチャンネルにて、質問内容を投稿してください。

