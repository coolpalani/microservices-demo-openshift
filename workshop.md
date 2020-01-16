# Sock Shop on OpenShiftでマイクロサービス体験
本資料は、Weaveworksが公開しているマイクロサービスのデモアプリケーション[Sock Shop](https://microservices-demo.github.io/)を利用(on OpenShift)して、マイクロサービスを体験するハンズオンです。

## Sock Shopについて
https://github.com/microservices-demo/microservices-demo

### Application Design
Sock Shopのデザインは下記の通りで、Java, NodeJS, Goなどとマイクロサービスの特徴であるポリグロットを体現しています。詳細は[こちら](front-end-rh-mori-sock-shop.3d6a.kepcodevops.openshiftapps.com)。

![application-design](https://github.com/microservices-demo/microservices-demo.github.io/blob/HEAD/assets/Architecture.png?raw=true)

## 事前準備
- openshiftクラスタ
- Google Chrome
- ocコマンドのインストール
- curlのインストール
- jqのインストール (recommended)
- postmanのインストール (optional)

## Sock ShopのOpenShiftへのデプロイ
本レポジトリのマニフェストファイルは、SCCの変更やcluster-adminを持っていなくてもOpenShift上で動くように修正しています。

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

1. ユーザ登録をする
    - デフォルトでユーザは登録されていない
1. カタログから靴下を検索、閲覧する
1. カートに入れてみる
1. カートの中身を確認する
1. ペイメントと住所を登録せずに、何かをオーダーする
1. ペイメントの登録する
1. 住所の登録する
1. もういちどオーダーする
1. オーダーの詳細をする

### データ項目の予想(Optional)
Sock Shopで利用されているデータを書き出してみましょう。
後ほど実際にデータベースの構造を見ていきますが、現時点でどんなデータがどこにあるか意識しておくといいでしょう。

## Sock Shopのデータ構造をみる

### データ構造を確認する
どのサービスがどのようなデータを保持しているかは、非常に重要です。
どのサービスがデータベースをもっていて、どのような情報をもっているか確かめてみましょう。

#### データベースを持っているサービス
- catalogue(mysql)
- user(monogo)
- cart(mongo)
- order(mongo)

#### 確認方法例
例として、データ構造の確認方法を記載します。
`registry.gitlab.com/mosuke5/debug-container`は、mysqlとmongodbのクライアントをインストールしてあるCentOSベースのイメージです。

```
$ oc run --generator=run-pod/v1 debug --image registry.gitlab.com/mosuke5/debug-container -it /bin/bash 
# mysql -uroot -pfake_password socksdb -h catalogue-db
mysql>
mysql> show tables;
+-------------------+
| Tables_in_socksdb |
+-------------------+
| sock              |
| sock_tag          |
| tag               |
+-------------------+
3 rows in set (0.01 sec)
mysql>
mysql> desc sock;
???
```

```
# mongo -u sock-user -p password user-db/users
> db.customers.find()
{ "_id" : ObjectId("5e1311e9019de10001d6d2db"), "firstName" : "taro", "lastName" : "ebisu", "email" : "taro.ebisu@xxxxxx.com", "username" : "mosuke5", "password" : "146d934cd057fa9dd4024df3c2c8dce86e03aade", "links" : {  }, "salt" : "a54efc2f95e33d8ad759c38153b808a57178e08d", "addresses" : [ ], "cards" : [ ] }
```

#### データ構造

## Sock ShopのAPIを実行してみる
### OpenAPI, Swagger
APIの仕様はどのように定義すればいいでしょうか。また、どのようにその定義を確認したら良いでしょうか。  
OpenAPIやSwaggerぜひ学んでみましょう。

Sock ShopのAPIドキュメント  
https://microservices-demo.github.io/api/index

### APIの検証方法
APIの開発や検証では、欠かせないツールがあります。
Curlや[Postman](https://www.getpostman.com/)の使い方はよく覚えておくことをおすすめします。
詳細な使い方は、ここでは割愛しますが、本ページに出てくるいくつかの基本的な項目について記載します。

#### メソッドの指定
curlでは`curl https://google.com`と指定すると
デフォルトではGETメソッドを利用します。
システムのAPIではGETのほかにPOSTやDELETEなどのメソッドを使うことも多くあります。
`-X`オプションで指定することができます。

```
curl -X POST https://xxxxx/user
```

#### データの送信
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
ユーザエージェントやコンテンツタイプなどの指定でHTTPヘッダーを追加することがあります。

```
$ curl -H 'User-Agent: hoge' https://xxxxx/
```

#### Cookie
APIによっては、認証が必要なことがあります。
例えば、ユーザAの個人情報がユーザBから取得できてしまってはいけません。
認証の方法もAPI仕様によるので、必ず確認しましょう。
本ワークショップでは、クッキーを利用するのでその方法を紹介します。
詳しくは、「ログイン」の項目で実践してみましょう。

```
curl -XGET -c cookie.txt https://xxxxx/login
```

### フロントエンドアドレス
フロントエンドアドレスを環境変数に指定しておきます。
```
$ export FRONTEND_ADDRESS=<your-frontend-address>
```

### カタログ情報API
カタログ情報はログインなしでも実行できる簡単なAPI。
商品ID `3395a43e-2d88-40de-b95f-e00e1502085b`の検索。

```
$ curl -X GET $FRONTEND_ADDRESS/catalogue/3395a43e-2d88-40de-b95f-e00e1502085b
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
$ curl -XGET $FRONTEND_ADDRESS/orders
{"message":"User not logged in.","error":{}}
```

ログインには"username:password"をbase64でエンコードしたものが必要です。コマンドラインで生成するか、こちらのような[Base64エンコードをしてくれるWebサービス](https://uic.jp/base64encode/)で生成しましょう。

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

ログインができているか確認します。オーダー情報が返ってきていれば成功です。

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
$ curl -XGET -b cookie.txt $FRONTEND_ADDRESS/cart
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
$ curl -XPOST -b cookie.txt $FRONTEND_ADDRESS/orders
{"id":"5e008861dc2f2e0006f35ee6","customerId":"57a98d98e4b00679b4a830b2","customer":{"id":null,"firstName":"User","lastName":"Name","username":"user","addresses":[],"cards":[]},"address":{"id":null,"number":"246","street":"Whitelees Road","city":"Glasgow","postcode":"G67 3DL","country":"United Kingdom"},"card":{"id":null,"longNum":"5544154011345918","expires":"08/19","ccv":"958"},"items":[{"id":"5e006ff5ea192a0006ead37d","itemId":"3395a43e-2d88-40de-b95f-e00e1502085b","quantity":3,"unitPrice":18}],"shipment":{"id":"33c17bcf-2b1e-4a1b-83a2-c02a79b032cc","name":"57a98d98e4b00679b4a830b2"},"date":"2019-12-23T09:26:57.477+0000","total":58.989998
```

上でオーダーしたものがオーダーリストに入っているか確認してみましょう。
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
また、モノリスなサービスの場合と比べてどうか考えてみましょう。

### 非同期通信
アーキテクチャデザインの図を見ると、shippingサービスはRabbitMQに対してデータを送っています。これまで、見てきたAPIはすべて同期的なデータ連携でしたが、

## Sock Shopの回復性、スケール、デプロイ
### サービスを落としてみよう
マイクロサービスの回復性について考えてみます。  
cartsサービスを落としてみて、どんな影響があるか確認してみよう。

```
$ oc scale --replicas=0 deployment/carts
```

- オーダーはできるか？
- 他のコンポーネントへの影響は？
- ユーザ体験はどう変わったか？

#### デグレードの実装
cartsサービスを落としたあと、UI上からカートボタンが消えたのに気づきましたか？
これは、マイクロサービスでよくあるデグレードの実装が行われているからです。
できる人は、実際にデグレードの実装が行われているコードを探してみましょう。

https://github.com/microservices-demo/front-end/blob/5d9a4272fec3983250364917d8ea7a210cdbf58c/public/navbar.html#L227

#### サーキットブレーカについて
ブラウザのデバッグツールを開いて、cartsサービスへの通信状況を見てみましょう。  
アクセスの度にcartsサービスのタイムアウトを待っているのがわかるのではないでしょうか。
サーキットブレーカを実装した場合にはどのような動きになるか考えてみよう。

### サービスをスケールさせてみよう
マイクロサービスのスケーリングについて考えてみます。  
マイクロサービスでは特定のサービスのみをスケールすることが容易であることが特徴の１つです。
ためしにフロントエンドサービスをスケールさせてみてみましょう。

```
$ oc scale --replicas=3 deployment/front-end 
```

### サービスをデプロイしてみよう
フロントエンドのコンテナイメージを変更してデプロイしてみよう。  
サンプルで、背景色を変更したフロントエンドのイメージを用意しました。利用したい人は下記のように入れ替えてデプロイしてみましょう。

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: front-end
  namespace: sock-shop
spec:
  replicas: 1
  template:
    metadata:
      labels:
        name: front-end
    spec:
      containers:
      - name: front-end
        image: mosuke5/front-end:master-bd5039f4
        #image: weaveworksdemos/front-end:0.3.12
```

```
$ oc apply -f complete-demo.yml
```

## トレーシング
続いて、トレーシングを体験するためにjaegerなどのコンポーネントをデプロイします。

```
$ oc apply -f tracing/jaeger.yml
$ oc expose service/jaeger-query
$ oc get route
NAME           HOST/PORT                                                           PATH      SERVICES       PORT         TERMINATION   WILDCARD
jaeger-query   jaeger-query-sock-shop.xxxxx             jaeger-query   query-http                 None
```

`jaeger-query`のURLに接続し下記のような画面にアクセスできれば成功です。

![jaeger-query](/images/jaeger-query.png)

トレーシングコンポーネント起動後に、もう一度Sock Shopで購入などの様々なアクションを実行しましょう。検索から`orders`サービスで検索をし、`orders: http:/orders`を探してみましょう。

![jaeger-query-order](/images/jaeger-query-order.png)

paymentサービスに障害が起きたと仮定し、paymentサービスを落としたあとに、もう一度購入操作をしてみましょう。
この複雑な処理のなかでもどこでエラーが起きたか一目瞭然に確認することができます。