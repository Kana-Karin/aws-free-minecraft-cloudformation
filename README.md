### AWS CloudFormation を利用して Mincecraft サーバーを構築

## 概要

CloudFormation をより深掘りして学んでいく為、AWS 無料枠内で 実際に遊べる Minecraft のサーバー構築を利用したアウトプットをしていきます。

- EC2 インスタンスを起動時に ElasticIP で IP アドレスの固定をしないと、毎起動時に Public IP アドレスが変わってしまいます。
  ですが、ElasticIP は 1 日 16 円ほどの料金が発生する為、今回は検証、アウトプットを含めた構築なので利用しません。
- AWS の無料枠では、t2.micro インスタンスが 1 か月あたり 750 時間まで無料です。
- **Minecraft サーバーはデフォルトで 2GB の RAM**を必要とするため、**無料枠 t2.micro インスタンスには 1GB の RAM**しかありません。
  したがって、サーバーのパフォーマンスに影響が出る可能性があります。
- Amazon Linux AMI を利用し、起動時に Minecraft サーバーをインストールして実行するようスクリプトを組みます。

## 前提条件

- 統合版(BEDROCK)Minecraft を持っている
- AWS IAM にて適切なユーザーを作成済み
- AWS CLI がインストールされている
- AWS CLI 認証情報を設定済み

## 使用技術

- EC2
- CloudFormation

---

## ファイルを作成

CloudFormation は JSON 形式と YAML 形式で記述が可能です。
コメントが書けたり、括弧のネストに悩まされることがない、圧倒的に便利な YAML 形式を今回は利用します。

- AWS CloudFormation デザイナーを利用したら YAML と JSON、どちらにも変換可能です

### 宣言文

```
AWSTemplateFormatVersion: "2010-09-09"
```

[公式リファレンス](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/format-version-structure.html)にも掲載されている通り、こちらのバージョン以外は無いようです。

人間はミスをする生き物だと存じています・・・ですので、なるべく手打ちは避けて、公式ドキュメントのテンプレ、先人様が作成したテンプレをコピペするようにします。

### テンプレの説明

```
Description: This template is for building and maintaining a BEDROCK Minecraft server.
```

テンプレの説明文です。
リソースをデプロイした後、マネジメントコンソールからテンプレートの説明文を確認できます。

こちらは**あっても、無くてもどちらでも問題ない**です。
分かり易いよう記述していきます。

### Resource とは

テンプレで必須です。少なくとも 1 つのリソースをここで定義します。

簡単に言ってしまえば、AWS のサービス・設定をここで定義します。

```
# EC2インスタンス構築を例に挙げると
Resources:
 EC2Instance:
    Type: AWS::EC2::Instance # Typeで何を作りたいか指定
    Properties: # 作ったもので何をしたいのか指定
      InstanceType: t2.micro # インスタンスタイプを指定する
      ImageId: !FindInMap [EC2, !Ref AWS::Region, AmazonLinux2023]
      IamInstanceProfile: !Ref EC2InstanceProfile
      Tags:
        - Key: Name
          Value: minecraft
```

Fn::Ref/(短縮系 !Ref)
Paramaters で指定した値や Resources で定義したリソース名を参照するために使います。
AMI ID のように頻繁に変わるかつ様々な値や、パスワードや認証情報などパラメータの中に記述すべきで名はない値を動的に取得する際に利用します。

### Parameters と MetaData とは

スタック作成時に、マネジメントコンソールから値を受け取るために使用します。
テンプレを使い回したい、1 つのテンプレートで複数の環境を管理したい、テンプレにベタ書きしたくない時に便利です。

ざっくり言うと、Parameters はテンプレートの再利用性、Metadata はテンプレートの可読性を分かり易くするための追加情報です。

```
# MinecraftのParametersを基にMetaDataを記述してみます
Metadata:
  TemplateInformation:
    Author: "Name" # テンプレートを作成した人の名前
    Date: "2023-11-14" # テンプレートの作成日または最終更新日
    Description: >
      Minecraftのバージョン、サーバー名、最大プレイヤー数、ゲームモード、難易度、チートの許可、ワールドのシード値、デフォルトの権限レベルをカスタマイズ可能
    Version: "1.0" # テンプレのバージョン
    MinecraftServerDetails:
      MinecraftVersionInfo: "Minecraftのバージョン選択"
      MaxPlayersInfo: "サーバーで許可される最大プレイヤー数。1から4の範囲で選択可能"
      GameModeInfo: "ゲームモードの選択。'survival', 'creative', 'adventure'から選択"
      DifficultyInfo: "ゲームの難易度設定。'peaceful', 'easy', 'normal', 'hard'から選択"
      AllowCheatsInfo: "チートの許可設定。'true'または'false'を選択。"
      SeedInfo: "ワールド生成のためのシード値の設定"
      DefaultPermissionInfo: "プレイヤーの権限レベル。'visitor', 'member', 'operator'から選択"

```

### Mappings とは

``Mappings`は文字列(string)を定義することができます。

- スタックの作成後は`Mappings`の定義を変更することはできません。

`Mappings`で定義した関数は、`Fn::FindInMap(短縮系 !FindInMap)`関数を使って、`Mappings`で定義した値を参照できます。

簡単な例を挙げると、
異なるリージョンで、異なる AMI ID を使用する場合

```
# Mappingsの定義
Mappings:
  RegionMap:
    us-east-1:
      "AMI": "ami-1234567"
    eu-west-1:
      "AMI": "ami-abcdef"
```

```
# !FindInMap 関数の使用
Resources:
  MyEC2Instance:
    Type: "AWS::EC2::Instance"
    Properties:
      ImageId: !FindInMap
        - RegionMap
        - !Ref "AWS::Region"
        - AMI
```

このようになり、値を参照することができます。

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
