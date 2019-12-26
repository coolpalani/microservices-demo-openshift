# Sock Shop on OpenShiftでマイクロサービス体験
本資料は、Weaveworksが公開しているマイクロサービスのデモアプリケーション[Sock Shop](https://microservices-demo.github.io/)を利用(on OpenShift)して、マイクロサービスを体験するハンズオンです。

## Sock Shopについて
https://github.com/microservices-demo/microservices-demo

### Application Design
![application-design](https://github.com/microservices-demo/microservices-demo.github.io/blob/HEAD/assets/Architecture.png?raw=true)

## 事前準備
- openshiftクラスタ
- Google Chrome
- ocコマンドのインストール
- curlのインストール
- jqのインストール(optional)
- postmanのインストール(optional)

## Sock ShopのOpenShiftへのデプロイ
本レポジトリのマニフェストファイルは、

```
$ oc apply -f complete-demo.yaml
deployment.extensions/carts-db created
service/carts-db created
deployment.extensions/carts created
service/carts created
deployment.extensions/catalogue-db created
service/catalogue-db created
deployment.extensions/catalogue created
service/catalogue created
deployment.extensions/front-end created
service/front-end created
deployment.extensions/orders-db created
service/orders-db created
deployment.extensions/orders created
service/orders created
deployment.extensions/payment created
service/payment created
deployment.extensions/queue-master created
service/queue-master created
deployment.extensions/rabbitmq created
service/rabbitmq created
deployment.extensions/shipping created
service/shipping created
deployment.extensions/user-db created
service/user-db created
deployment.extensions/user created
service/user created

$ oc get pod   
NAME                            READY     STATUS    RESTARTS   AGE
carts-db-86b89bcb4c-m68xh       1/1       Running   0          57s
carts-ffc5d77cd-9vf84           1/1       Running   0          56s
catalogue-6d8895f8b9-tjxk8      1/1       Running   0          52s
catalogue-db-746b88dc5c-mfg8p   1/1       Running   0          54s
front-end-f975554b6-f9qrc       1/1       Running   0          51s
orders-65df94c9f5-786hf         1/1       Running   0          48s
orders-db-5ff5cdd64c-cv4cb      1/1       Running   0          49s
payment-f5dcc89c5-6k2jl         1/1       Running   0          46s
queue-master-787b68b7fd-sjsxb   1/1       Running   0          45s
rabbitmq-7db7568778-89t2h       1/1       Running   0          43s
shipping-5c6867f94f-tzwdg       1/1       Running   0          42s
user-6866b4bcf4-rbr7p           1/1       Running   0          38s
user-db-78c5b86bb8-njsz8        1/1       Running   0          40s
```

### Sock Shopの公開
現在のままでは、サービスは公開されていないので、公開していきます。
必要に応じて、ここでOpenShiftのRouteやServiceについての解説をします。

```
$ oc get route
No resources found.

$ oc get service
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
carts          ClusterIP   172.30.233.183   <none>        80/TCP      2m
carts-db       ClusterIP   172.30.171.202   <none>        27017/TCP   2m
catalogue      ClusterIP   172.30.223.222   <none>        80/TCP      2m
catalogue-db   ClusterIP   172.30.148.67    <none>        3306/TCP    2m
front-end      ClusterIP   172.30.184.134   <none>        80/TCP      2m
orders         ClusterIP   172.30.195.188   <none>        80/TCP      2m
orders-db      ClusterIP   172.30.0.235     <none>        27017/TCP   2m
payment        ClusterIP   172.30.75.209    <none>        80/TCP      2m
queue-master   ClusterIP   172.30.188.159   <none>        80/TCP      2m
rabbitmq       ClusterIP   172.30.106.129   <none>        5672/TCP    2m
shipping       ClusterIP   172.30.193.109   <none>        80/TCP      2m
user           ClusterIP   172.30.24.155    <none>        80/TCP      2m
user-db        ClusterIP   172.30.48.244    <none>        27017/TCP   2m

$ oc expose service/front-end
route.route.openshift.io/front-end exposed

$ oc get route
NAME        HOST/PORT                                           PATH      SERVICES    PORT      TERMINATION   WILDCARD
front-end   front-end-sock-shop.apps.8aad.example.opentlc.com             front-end   web                     None
```

ブラウザを起動して、作成したrouteのURLに接続して以下のようなSock Shopの画面が返ってくるか確認しましょう。

![sockshop-index](/images/sockshop-index.png)

## Sock Shopを触ってみよう
まずは、Sock Shopのサービスをブラウザから触って概要を理解しよう。

1. ログインする
    - デフォルトユーザでuser/passwordが用意されている。
1. カタログから靴下を検索、閲覧する
1. カートに入れてみる
1. カートの中身を確認してみる
1. オーダーしてみる
1. 一度ログアウトして、新規ユーザを作ってみる
1. ペイメントと住所を登録せずに、何かをオーダーしてみる
1. ペイメントの登録をしてみる
1. 住所の登録をしてみる
1. もういちどオーダーしてみる
1. オーダーの詳細をみてみる

## Sock Shopを調べてみよう
### 各サービス
https://github.com/microservices-demo/microservices-demo/blob/master/internal-docs/design.md

### kubernetesにおける概念

### データ構造を確認する
- catalogue(mysql)
- user(monogo)
- cart(mongo)
- order(mongo)

### HTTPの基礎
methodとか参考になる実装

### Swagger
APIの仕様をみるためにはどうしたらいいのか？
Swaggerについてまなんでみよう。
https://microservices-demo.github.io/api/index

### 他社のAPIドキュメントを参考にする
- Github
- Twitter
- Qiita

### API検証のいい方法
APIの開発や検証では、欠かせないツールがあります。
CurlやPostmanの使い方はよく覚えておくことをおすすめします。
詳細な使い方は、ここでは割愛しますが、本ページに出てくるいくつかの基本的な項目について記載します。

#### メソッドの指定
curlでは`curl https://google.com`と指定すると
デフォルトではGETメソッドを利用します。
システムのAPIではGETのほかにPOSTやDELETEなどのメソッドを使うことも多くあります。
`-X`オプションで指定することができます。

```
curl -X POST https://xxxxx/user
```

#### データ
POSTなどのメソッドを利用する場合は、リクエストともにデータを送信することがよくあります。
例えばユーザ情報の登録APIにおいては、登録するデータを送信する必要があります。
データの形式はAPI仕様によるので、必ず確認してください。
下記では、jsonで`name`という項目が必要な場合の例です。

```
$ curl -XPOST \
-d '{"name":"mosuke5"}' \
-H 'Content-Type:application/json; charset=UTF-8' \
https://xxxxx/user
```

#### HTTPヘッダーの追加
```
$ curl -H 'User-Agent: hoge' https://xxxxx/
```

#### クッキー
APIによっては、認証が必要なことがあります。
例えば、ユーザAの個人情報がユーザBから取得できてしまってはいけません。
認証の方法もAPI仕様によるので、必ず確認しましょう。
本ワークショップでは、クッキーを利用するのでその方法を紹介します。
詳しくは、「ログイン」の項目で実践してみましょう。

```
curl -XGET -c cookie.txt https://xxxxx/login
```

### カタログ情報API
カタログ情報はログインなしでも実行できる簡単なAPI。
商品ID `3395a43e-2d88-40de-b95f-e00e1502085b`の検索。

```
curl -X GET $FRONTEND_ADDRESS/catalogue/3395a43e-2d88-40de-b95f-e00e1502085b
{
  "id": "3395a43e-2d88-40de-b95f-e00e1502085b",
  "name": "Colourful",
  "description": "proident occaecat irure et excepteur labore minim nisi amet irure",
  "imageUrl": [
    "/catalogue/images/colourful_socks.jpg",
    "/catalogue/images/colourful_socks.jpg"
  ],
  "price": 18,
  "count": 438,
  "tag": [
    "brown",
    "blue"
  ]
}
```

### ログイン
商品のオーダーなどは当然ながらログインした状態でないと実行できない。試しに、ログインセッションを持たないまま、オーダーAPIを実行してみるとログインしてくれとエラーが返ってきます。

```
curl -XGET $FRONTEND_ADDRESS/orders                                    
{"message":"User not logged in.","error":{}}
```

```
$ echo -n "user:password" | base64
dXNlcjpwYXNzd29yZA==

$ curl -XGET -c cookie.txt -H "Authorization: Basic dXNlcjpwYXNzd29yZA==" -v $FRONTEND_ADDRESS/login

$ cat cookie.txt
# Netscape HTTP Cookie File
# https://curl.haxx.se/docs/http-cookies.html
# This file was generated by libcurl! Edit at your own risk.

xxxxxxx
```

chromeからログインセッションをとってきます。

ログインの確認
```
$ curl -XGET -b cookie.txt $FRONTEND_ADDRESS/orders                                    
[
    {
        "customerId": "57a98d98e4b00679b4a830b2",
        "customer": {
            "firstName": "User",
            "lastName": "Name",
            "username": "user",
            "addresses": [],
            "cards": []
        },
        "address": {
            "number": "246",
            "street": "Whitelees Road",
            "city": "Glasgow",
            "postcode": "G67 3DL",
            "country": "United Kingdom"
        },
        "card": {
            "longNum": "5544154011345918",
            "expires": "08/19",
            "ccv": "958"
        },
        "items": [
            {
                "itemId": "808a2de1-1aaa-4c25-a9b9-6612e8f29a38",
                "quantity": 1,
                "unitPrice": 17.32
            }
        ],
        "shipment": {
            "name": "57a98d98e4b00679b4a830b2"
        },
        "date": "2019-12-23T06:35:49.925+0000",
        "total": 22.31,
        "_links": {
            "self": {
                "href": "http://orders/orders/5e006045dc2f2e0006f35ee4"
            },
            "order": {
                "href": "http://orders/orders/5e006045dc2f2e0006f35ee4"
            }
        }
    }
]
```

### カートAPI
それではAPI経由でカートに商品を追加してみます。

```
$ curl -XPOST -b cookie.txt \
-d '{"id":"3395a43e-2d88-40de-b95f-e00e1502085b"}' \
-H 'Content-Type:application/json; charset=UTF-8' \
$FRONTEND_ADDRESS/cart
```

```
$ curl -XGET -b cookie.txt $FRONTEND_ADDRESS/cart | jq .
[
  {
    "id": "5e006ff5ea192a0006ead37d",
    "itemId": "3395a43e-2d88-40de-b95f-e00e1502085b",
    "quantity": 2,
    "unitPrice": 18
  }
]
```

### オーダー
同じ要領でAPI経由でオーダーしてみよう

```
$ curl -XPOST -b cookie.txt http://front-end-sock-sho
p.apps.8aad.example.opentlc.com/orders
{"id":"5e008861dc2f2e0006f35ee6","customerId":"57a98d98e4b00679b4a830b2","customer":{"id":null,"firstName":"User","lastName":"Name","username":"user","addresses":[],"cards":[]},"address":{"id":null,"number":"246","street":"Whitelees Road","city":"Glasgow","postcode":"G67 3DL","country":"United Kingdom"},"card":{"id":null,"longNum":"5544154011345918","expires":"08/19","ccv":"958"},"items":[{"id":"5e006ff5ea192a0006ead37d","itemId":"3395a43e-2d88-40de-b95f-e00e1502085b","quantity":3,"unitPrice":18}],"shipment":{"id":"33c17bcf-2b1e-4a1b-83a2-c02a79b032cc","name":"57a98d98e4b00679b4a830b2"},"date":"2019-12-23T09:26:57.477+0000","total":58.989998
```

```
curl -XGET -b cookie.txt $FRONTEND_ADDRESS/orders | jq .
[
  {
    "customerId": "57a98d98e4b00679b4a830b2",
    "customer": {
      "firstName": "User",
      "lastName": "Name",
      "username": "user",
      "addresses": [],
      "cards": []
    },
    "address": {
      "number": "246",
      "street": "Whitelees Road",
      "city": "Glasgow",
      "postcode": "G67 3DL",
      "country": "United Kingdom"
    },
    "card": {
      "longNum": "5544154011345918",
      "expires": "08/19",
      "ccv": "958"
    },
    "items": [
      {
        "itemId": "808a2de1-1aaa-4c25-a9b9-6612e8f29a38",
        "quantity": 1,
        "unitPrice": 17.32
      }
    ],
    "shipment": {
      "name": "57a98d98e4b00679b4a830b2"
    },
    "date": "2019-12-23T06:35:49.925+0000",
    "total": 22.31,
    "_links": {
      "self": {
        "href": "http://orders/orders/5e006045dc2f2e0006f35ee4"
      },
      "order": {
        "href": "http://orders/orders/5e006045dc2f2e0006f35ee4"
      }
    }
  }
]
```

### フロントエンド
フロントエンドサービスの役割を考えてみよう。  
ひとつひとつのAPIがどんなものかわかってきたところで、フロントエンドサービスのコードをのぞいてみよう。

#### Gatewayサービスの役割
- メリット
    - アプリケーションの内部仕様を隠蔽できる
    - クライアントからみて、各サービスの接続先を気にしなくて良い
    - クライアントサイドを簡素化
    - クライアントからのリクエスト数の削減
    - 認証認可などのオフロード
- デメリット
    - レスポンスタイムの増加
    - コンポーネントが増えることによる複雑化
    - ボトルネックの可能性
    - Gatewayのモノリシック化

[API ゲートウェイ パターンと、クライアントからマイクロサービスへの直接通信との比較](https://docs.microsoft.com/ja-jp/dotnet/architecture/microservices/architect-microservice-container-applications/direct-client-to-microservice-communication-versus-the-api-gateway-pattern)

### フロントエンドでのデータの扱い
例えば、オーダー詳細をブラウザからみてみよう。オーダーの情報はもちろん商品の情報も表示されている。しかし、オーダーAPIではitemのidしか返していません。
どのようにしてitemの情報を取得しているでしょうか？
また、モノリスなサービスの場合と比べてどうか考えてみましょう。。

### 非同期通信
アーキテクチャデザインの図を見ると、shippingサービスはRabbitMQに対してデータを送っています。これまで、見てきたAPIはすべて同期的なデータ連携でしたが、

## あるコンポーネントを落としてみよう
orderサービスを落としてみて、どんな影響があるか確認してみよう。

```
$ oc scale --replicas=0 deployment/orders
```

- オーダーはできる？
- 他のコンポーネントへの影響は？
- ユーザ体験はどう変わったか？

## あるコンポーネントをスケールさせてみよう
フロントエンドサービスをスケールさせてみる。

```
$ oc scale --replicas=3 deployment/front-end 
```

## あるコンポーネントを更新してみよう
フロントエンドのコンテナイメージを変更してデプロイしてみよう。

### 自分でイメージを作る

### サンプルイメージを利用する

## トレーシング