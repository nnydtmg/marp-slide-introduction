---
marp: true
header: "カスタムドメインの設定方法"
paginate: true
---
<!--
headingDivider: 1
-->
<style>
  h2, h3, h4, h5, header, footer {
    color: black;
  }

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
4. CloudFront（Route53）設定
5. APIマッピング
6. 具体例


# 目標
## 独自ドメインを利用して、複数APIGatewayへのルーティングを設定したい

# 前提
* APIGatewayが2つ構築済
  * APIGateway1 (***APIGateway1***.execute-api.ap-northeast-1.amazonaws.com/)
    * aaaステージ
      <span style="font-size:24px">(***APIGateway1***.execute-api.ap-northeast-1.amazonaws.com/***aaa***)</span>
    * bbbパス
      <span style="font-size:24px">(***APIGateway1***.execute-api.ap-northeast-1.amazonaws.com/***aaa/bbb***)</span>

  * APIGateway2 (***APIGateway2***.execute-api.ap-northeast-1.amazonaws.com/)
    * AAAステージ
      <span style="font-size:24px">(***APIGateway2***.execute-api.ap-northeast-1.amazonaws.com/***AAA***)</span>
    * BBBパス
      <span style="font-size:24px">(***APIGateway2***.execute-api.ap-northeast-1.amazonaws.com/***AAA/BBB***)</span>


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


# CloudFront（Route53）設定
ここまでで、APIGatewayに対してのカスタムドメイン名の設定は完了。

**しかし**、カスタムドメインとエンドポイントの紐付けが出来ていないため、
まだカスタムドメイン（**APItest.customdomain.com**）でアクセスする事は出来ない。

**randam.cloudfront.net**と**APItest.customdomain.com**を紐付ける必要がある。

Route53のサービス画面で、「ホストゾーン」から **customdomain.com** を選択して、レコードの作成を行う。


# CloudFront（Route53）設定
1. レコードタイプ：Ａ
   ただし、IP指定ではなく、AWSサービスとの統合
2. レコード名：設定したカスタムドメインになるように埋める。
   APItestを入力し、枠外と合わせて**APItest.customdomain.com**となるようにする
3. トラフィックのルーティング先で「エイリアス」に切り替える
4. 「API Gateway APIへのエイリアス」を選択し、先ほどのCloudFrontエンドポイントを選択
5. 「レコードを作成」をクリック


正常に作成されれば完了


# APIマッピング
作成したカスタムドメインに各APIのパスをマッピングする。つまり、
1. どのAPIの
2. どのステージ（の）
3. どのパス（オプション）

に、アクセスさせるかの設定を行う
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
  * （ex:APItest.customdomain.com/testをaaaにルーティングしたい場合、ステージにaaa、パスに/testを設定）


# APIマッピング
マッピング設定の例

<span style="font-size:24px">

|対象API|パス|API|ステージ|アクセスURL|
|:--|:--|:--|:--|:--|
|aaa|/dev|APIGateway1|aaa|APItest.customdomain.com/dev|
|bbb|/dev|APIGateway1|aaa|APItest.customdomain.com/dev/bbb|
|AAA|/prd|APIGateway2|AAA|APItest.customdomain.com/prd|
|BBB|/prd|APIGateway2|AAA|APItest.customdomain.com/prd/BBB|


APIGateway1にxxxステージを追加してbbbのAPIをテストしたい場合、
|対象API|パス|API|ステージ|アクセスURL|
|:--|:--|:--|:--|:--|
|bbb|/test|APIGateway1|xxx|APItest.customdomain.com/test/bbb|

で別APIとして検証（バージョンアップ）することも可能。
</span>


# 具体例1
## 1API Gatewayに複数ステージがある場合
A社が提供するAPIのバージョンアップを行う。
つまり、同一API Gatewayでステージ切り替えによって、商用・検証用を分ける使い方。

<span style="font-size:24px">

|対象API|パス|API|ステージ|アクセスURL|
|:--|:--|:--|:--|:--|
|提供中API（aaa）|/api|APIGateway1|prd|APItest.customdomain.com/api/aaa|
|検証中API（aaa）|/dev|APIGateway1|dev|APItest.customdomain.com/dev/aaa|

</span>


# 具体例2
## 複数API Gatewayで同じドメインを利用したい場合
A社がECサイトを運営していて、注文用APIと検索用APIを別チームで運用しているため、API Gatewayが2つある。
マイクロサービスを意識したアーキテクチャを採用している使い方。

<span style="font-size:24px">

|対象API|パス|API|ステージ|アクセスURL|
|:--|:--|:--|:--|:--|
|注文用API（order）|/order|APIGateway1|prd|API.customdomain.com/order|
|検索用API（search）|/search|APIGateway2|prd|API.customdomain.com/search|

</span>

この場合、本番用と検証用のカスタムドメインを切り替える。
（TestAPI.customdomain.com/searchなど）
