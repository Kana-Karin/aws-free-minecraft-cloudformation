### AWS CloudFormation を利用して Mincecraft サーバーを構築

## 概要

CloudFormation をより深掘りして学んでいく為、AWS 無料枠内で 実際に遊べる Minecraft のサーバー構築を利用したアウトプットをしていきます。

- EC2 インスタンスを起動時に ElasticIP で IP アドレスの固定をしないと、毎起動時に Public IP アドレスが変わってしまいます。
  ですが、ElasticIP は 1 日 16 円ほどの料金が発生する為、今回は検証、アウトプットを含めた構築なので利用しません。
- AWS の無料枠では、t2.micro インスタンスが 1 か月あたり 750 時間まで無料です。
- **Minecraft サーバーはデフォルトで 2GB の RAM**を必要とするため、**無料枠 t2.micro インスタンスには 1GB の RAM**しかありません。
  したがって、サーバーのパフォーマンスに影響が出る可能性があります。
- Amazon Linux AMI を利用し、起動時に Minecraft サーバーをインストールして実行するようスクリプトを組みます。

## 使用技術

- EC2
- CloudFormation

---

## ファイルを作成

CloudFormation は JSON 形式と YAML 形式で記述が可能です。
コメントが書けたり、括弧のネストに悩まされることがない、圧倒的に便利な YAML 形式を今回は利用します。

- AWS CloudFormation デザイナーを利用したら YAML と JSON、どちらにも変換可能です

### 宣言文

`AWSTemplateFormatVersion: "2010-09-09"`
[公式リファレンス](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/format-version-structure.html)にも掲載されている通り、こちらのバージョン以外は無いようです。

人間はミスをする生き物だと思いますので、なるべく手打ちは避けて、公式ドキュメントのテンプレ、先人様が作成したテンプレをコピペしていきます。

### テンプレの説明

```
Description: This template is for building and maintaining a BEDROCK Minecraft server.
```

テンプレの説明文です。
リソースをデプロイした後、マネジメントコンソールからテンプレートの説明文を設定できます。

こちらは**あっても、無くてもどちらでも問題ない**です。
分かり易いよう記述していきます。

### リソースの作成

### Parameters とは

### Mapping とは

### UserData でシェル変数を使用する

---

## 参照させていただいたサイト

[Zenn - CloudFomation ドキュメントとの向き合い方](https://zenn.dev/mjxo/articles/9d68410e2c624f)
[Developers.IO - CloudFormation の中の EC2 のユーザーデータでシェル変数を使用する](https://dev.classmethod.jp/articles/using-variables-in-ec2-user-data-in-cloudformation/)
[AWS 公式 - CloudFormation AWS リソースおよびプロパティタイプのリファレンス](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)
[AWS 公式 CloudFormation ユーザーガイド - Fn::Base64](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-base64.html)
[AWS 公式 - AWS::EC2::Instance](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-instance.html)
[AWS 公式 - 組み込み関数リファレンス](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html)
[AWS 公式 - 擬似パラメータ参照](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/pseudo-parameter-reference.html)
[Minecraft 公式 - BEDROCK 版 MINECRAFT 専用サーバーをダウンロード](https://www.minecraft.net/ja-jp/download/server/bedrock)
