---
marp: true
header: "カスタムドメインの設定方法"
paginate: true
---
<!--
headingDivider: 1
-->
<style>
  section.title {
    justify-content: center;
    text-align: left;
  }

  section {
    justify-content: start;
  }
</style>

# 目次
1. 目標
2. 前提
3. カスタムドメイン設定
4. APIマッピング
5. CloudFront（Route53）設定

# 目標
## 独自ドメインを利用して、複数APIGatewayへのルーティングを設定したい

# 前提
* APIGatewayが2つ構築済
  * APIGateway1 (execute-api.***APIGateway1***.amazonaws.com/)
    * aaaステージ (execute-api.***APIGateway1***.amazonaws.com/***aaa***)
    * bbbパス (execute-api.***APIGateway1***.amazonaws.com/***aaa/bbb***)

  * APIGateway2 (execute-api.***APIGateway2***.amazonaws.com/)
    * AAAステージ (execute-api.***APIGateway2***.amazonaws.com/***AAA***)
    * BBBパス (execute-api.***APIGateway2***.amazonaws.com/***AAA/BBB***)


# 前提
* ACMで証明書を発行済
* ルートドメインは**customdomain.com**
  * サブドメインは**APItest.customdomain.com**とする

# カスタムドメイン設定
登録済のドメインをAPIGatewayで利用できるようにする
1. APIGatewayコンソールから「カスタムドメイン名」を選択
2. 「作成」をクリック
3. 使用したいドメインを入力（サブドメインでもOK）
4. CloudFrontを利用する場合「エッジ最適化」を選択
5. 使用する証明書を選択
6. 「ドメイン名を作成」をクリック


これでカスタムドメインの設定は完了。
**randam.cloudfront.net**のようなAPIGatewayドメイン名が作成されている。

# APIマッピング
作成したカスタムドメインに各APIのパスをマッピングする。つまり、
1. どのAPIの
2. どのステージに（の）
3. どのパス（オプション）

アクセスさせるかの設定を行う
<br>
1. カスタムドメインの「APIマッピングを設定」をクリック
2. API、ステージ、パス（オプション）の各設定値を埋める

# APIマッピング
各設定値について
* API：関連付ける対象のAPI
  * （ex:APIGateway1）
* ステージ：そのAPIのどのステージにマッピングするのか
  * （ex:aaa、AAA）
* パス（オプション）：どのパスにアクセスした場合、設定したステージにルーティングするか
  * （ex:APItest.customdomain.com/aaaをaaaにルーティングしたい場合、ステージにaaa、パスに/aaaを設定）


# CloudFront（Route53）設定
ここまでで、APIGatewayに対してのカスタムドメイン名の設定は完了。
**しかし**、カスタムドメインとエンドポイントの紐付けが出来ていないため、まだカスタムドメイン（**APItest.customdomain.com**）でアクセスする事は出来ない。
**randam.cloudfront.net**と**APItest.customdomain.com**を紐付ける必要がある。

Route53のサービス画面で、「ホストゾーン」から**customdomain.com**を選択して、レコードの作成を行う。


# CloudFront（Route53）設定
1. レコードタイプ：Ａ
   ただし、IP指定ではなく、AWSサービスとの統合
2. レコード名：設定したカスタムドメインになるように埋める。
   APItestを入力し、枠外と合わせて**APItest.customdomain.com**となるようにする
3. トラフィックのルーティング先で「エイリアス」に切り替える
4. 「API Gateway APIへのエイリアス」を選択し、先ほどのCloudFrontエンドポイントを選択
5. 「レコードを作成」をクリック


正常に作成されれば完了