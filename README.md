### AWSCloudFormationを利用してMincecraftサーバーを構築

## 概要

CloudFormation をより深掘りして学んでいく為、AWS 無料枠内で 実際に遊べる Minecraft のサーバー構築を利用したアウトプットをしていきます。
- EC2 インスタンスを起動時に ElasticIPでIPアドレスの固定をしないと、毎起動時に Public IPアドレスが変わってしまいます。
  ですが、ElasticIPは1日16円ほどの料金が発生する為、今回は検証、アウトプットを含めた構築なので利用しません。
- AWSの無料枠では、t2.microインスタンス1か月あたり 750時間まで無料です。
- **Minecraft サーバーはデフォルトで2GBのRAM**を必要とするため、**無料枠 t2.micro インスタンスには1GBのRAM**しかありません。起動できない可能性があります。
- Amazon Linux AMI を利用し、起動時に Minecraft サーバーをインストールして実行するようスクリプトを組みます。

## 前提条件

- 統合版(BEDROCK)Minecraft を持っている
- AWS IAM にて適切なユーザーを作成済み

## 使用技術

- EC2
- CloudFormation

---

## ファイルを作成

CloudFormation は JSON 形式と YAML 形式で記述が可能です。
コメントが書けたり、括弧のネストに悩まされることがない、圧倒的に便利な YAML 形式を今回は利用します。

- AWS CloudFormation デザイナーを利用したら YAML と JSON、どちらにも変換可能です

## 宣言文

```
AWSTemplateFormatVersion: "2010-09-09"
```

[公式リファレンス](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/format-version-structure.html)にも掲載されている通り、こちらのバージョン以外は無いようです。

人間はミスをする生き物だと存じています・・・ですので、なるべく手打ちは避けて、公式ドキュメントのテンプレ、先人様が作成したテンプレをコピペするようにします。

## テンプレの説明

```
Description: This template is for building and maintaining a BEDROCK Minecraft server.
```

テンプレの説明文です。
リソースをデプロイした後、マネジメントコンソールからテンプレートの説明文を確認できます。

こちらは**あっても、無くてもどちらでも問題ない**です。
分かり易いよう記述していきます。

## Resource とは

**テンプレで必須**です。少なくとも 1 つのリソースをここで定義します。
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
```
# Parameteresで設定した定義
MinecraftVersion: # Minecraftのバージョンを指定
    Type: String
    Default: 1.20.41.02
```
上記で、定義した``MinecraftVersion``を
```
{
  minecraft_version: !Ref MinecraftVersion,
}
```
で参照して使いまわせます。

## Parameters と MetaData とは
Parametersを使うことで、スタック作成時に好きなパラメータを受け付けるようにすることが出来ます。
テンプレを使い回したい、1 つのテンプレートで複数の環境を管理したい、テンプレにベタ書きしたくない時に便利です。

ざっくり言うと、Parameters はテンプレートの再利用性、Metadata はテンプレートの可読性を分かり易くするための追加情報です。  


```
# ============== Minecraftパラメーター ==============
Parameters: # スタック作成者に入力を求める項目を作成。
  VPCId:
    Type: AWS::EC2::VPC::Id
  MinecraftVersion: # Minecraftのバージョンを指定
    Type: String
    Default: 1.20.41.02 # Mincecraft公式サイト https://www.minecraft.net/ja-jp/download/server/bedrock から最新のサーバー情報を確認できます
  ServerName: # Minecraftサーバー名を指定
    Type: String
    Default: My-Minecraft-Server
  MaxPlayers: # 最大プレイヤー数
    Type: String
    Default: 3
    AllowedValues:
      - 1
      - 2
      - 3
      - 4
```
上記のパラメーターを定義すると、  
マネージコンソールにてyamlファイルをアップロードした際、こんな感じでParametersが出てきます。
![スクリーンショット 2023-11-14 21 32 16](https://github.com/Kana-Karin/aws-free-minecraft-cloudformation/assets/84316229/deff4fb8-c37e-475a-93be-1c404595a9f5)

詳細なParametersのリファレンスはこちらになります。[AWS公式 - CloudFormation - パラメータ](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html  )
プロパティやAWS固有のパラメータタイプも公式リファレンスに詳細に掲載されています。

  
おまけ
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

``Mappings``は文字列(string)を定義することができます。

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
UserDataは、AWS CloudFormationのテンプレートでEC2インスタンス起動時に、  
実行されるスクリプトやコマンドを指定するためのプロパティです。

スクリプトを使うことにより、  
起動時にホスト名、時刻同期、文字コード、タイムゾーンの設定をスクリプト化することでEC2の構築作業を省力化できます。


``Fn:Base64``は、CloudFormationテンプレート内で文字列をBase64エンコードするための関数です。

``!Sub``  
!Sub は、文字列内の変数を置換することができます。

```
Fn::Base64: !Sub
          - |
            #!/usr/bin/env bash # bashシェルを使用

            mkdir /var/minecraft && cd /var/minecraft/ # /var/minecraftディレクトリを作成して/var/minecraftに移動
            wget -O bedrock-server.zip https://minecraft.azureedge.net/bin-linux/bedrock-server-${minecraft_version}.zip # ここで!Sub関数が使われ、Parametersの値を参照しています
            sudo apt install -y unzip && unzip bedrock-server.zip
            sed -i -e 's/^\(server-name=\)Dedicated Server/\1${server_name}/g' server.properties
```
指定された文字列内の ${VariableName} 形式の変数を、その変数の値に置換します。
リソースの識別子や他のパラメータを動的に文字列に組み込むのに使われます。

``#!/usr/bin/env bash`` スクリプトを実行する際に使用するシェルを指定します。ここではbashシェルを使用しています。

``set -eu``
``set -e``　スクリプト内でエラーが発生した場合、それ以降のコマンドは実行せずにスクリプトを終了します。  
``set -u``　未定義の変数を参照した場合、エラーとして扱いスクリプトを終了します。  

```mkdir /var/minecraft && cd /var/minecraft/```  
**/var/minecraft**というディレクトリを作成し、そのディレクトリに移動します。

``wget -O bedrock-server.zip https://minecraft.azureedge.net/bin-linux/bedrock-server-${minecraft_version}.zip``  
MinecraftのBedrockサーバーのZIPファイルをダウンロードします。${minecraft_version}はMinecraftのバージョンを指定する変数です。


``sudo apt install -y unzip && unzip bedrock-server.zip``  
**unzipパッケージをインストール**し、ダウンロードしたvedrock-server.zipファイルを解凍します。

``sed -i -e 's/^\(server-name=\)Dedicated Server/\1${server_name}/g' server.properties``  
sed コマンドを使ってserver.propertiesファイルの内容を編集しています。  
各行は特定の設定項目（サーバー名、プレイヤー数、ゲームモードなど）を変更します。${}で囲まれた部分は変数で、スクリプトの実行時に適切な値で置換されます。


``screen -UAmdS minecraft ./bedrock_server``  
``screen``コマンドを使用して、bedrock_serverをバックグラウンドで実行します。

実際の詳細なLinuxコマンドについてはこちらを参照しています。  
[Linux コマンド 一覧表（manページ一覧）](https://kazmax.zpp.jp/cmd/)

## 実際にCloudFormationにデプロイ
~~このデプロイが一番緊張します~~
![スクリーンショット 2023-11-16 21 49 36](https://github.com/Kana-Karin/aws-free-minecraft-cloudformation/assets/84316229/d040c365-c76a-4863-9ce3-af036dbd8ba1)    

EC2インスタンスが構築されているのが確認できます。 
![スクリーンショット 2023-11-16 21 52 46](https://github.com/Kana-Karin/aws-free-minecraft-cloudformation/assets/84316229/f69fe1bc-3eb3-422d-b8f8-1bfdb2d9506c)    
  
    

EC2のマネージコンソールへ行き、パブリックアドレスをメモします。
![スクリーンショット 2023-11-16 21 55 48](https://github.com/Kana-Karin/aws-free-minecraft-cloudformation/assets/84316229/b3ca0dcc-e97f-4299-b9fa-c1ea4cc7a879)


サーバー名、接続先アドレス、ポート番号を入力。
![IMG_3977](https://github.com/Kana-Karin/aws-free-minecraft-cloudformation/assets/84316229/8b40be97-a8eb-463b-ba46-1f261a8f8896)  
無料枠だとスペック不足でワールド生成すらできませんでした・・・


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
