# SockShopへのService Meshの導入

## 事前準備（管理者向け）
Service Meshのコントロールプレーンをインストールするには、いくつかのOperatorの事前のインストールが必要です。
下記のOperatorのインストールを済ませておきましょう。

- Jaeger operator
- ElasticSearch operator
- ServiceMesh operator
- Kiali operator

## Service Meshのインストール

```
$ oc new-project sockshop-istio-system
$ oc apply -f manifests/istio-control-plane.yaml -n sockshop-istio-system
$ oc get pod -n sockshop-istio-system
NAME                                      READY   STATUS    RESTARTS   AGE
grafana-7df65c57d5-8tzqn                  2/2     Running   12         5d19h
istio-citadel-54fc4655b4-hb28n            1/1     Running   6          5d19h
istio-egressgateway-6bb97d479d-smwck      1/1     Running   2          29h
istio-galley-797746b78d-b5vxv             1/1     Running   6          5d19h
istio-ingressgateway-66fcccb49d-c4s9d     1/1     Running   0          140m
istio-pilot-567c947974-xjp8l              2/2     Running   4          29h
istio-policy-5fcfd5f79c-7rk44             2/2     Running   5          29h
istio-sidecar-injector-75577f4fdb-gz9vk   1/1     Running   6          5d19h
istio-telemetry-dbc897cbb-v7sv9           2/2     Running   5          29h
jaeger-778bcb6bf4-t8n6t                   2/2     Running   12         5d19h
kiali-78b6f44646-pqzq8                    1/1     Running   1          17h
prometheus-884ff6bf9-5mbh7                2/2     Running   12         5d19h
```

## SockShopへの適応
### SockShopをmember rollに追加

```
$ oc apply -f servicemesh/member-roll.yaml
servicemeshmemberroll.maistra.io/default created

$ oc get ServiceMeshMemberRoll default -o yaml | grep -A 1 configuredMembers
  configuredMembers:
  - user1-sockshop
```

この時点で、今まで通りRoute経由でSockShopへはアクセスできなくなります。
理由は、NetworkPolicyの設定でRouterから直接の通信は遮断されているからです。IstioのIngress Gatewayを利用してアクセスするように設定変更をする必要があります。

この時点でアクセスしたい場合は、ポートフォワードを活用すると良いです。

```
$ oc port-forward service/front-end 8080:80
Forwarding from 127.0.0.1:8080 -> 8079
Forwarding from [::1]:8080 -> 8079
```

### 各サービスへEnvoy Proxyを導入
Service Meshの要である、Envoy Proxyを各アプリケーションに配置していきます。
deploymentの `"sidecar.istio.io/inject": "true"` を入れることでAdmissionControllerが自動的にサイドカーコンテナとしてEnvoy Proxyを注入します。

```
for deployment_name in $(oc get deploy | sed '1d' | awk '{print $1}')
do
    oc patch deploy $deployment_name --type='json' -p "[{\"op\": \"add\", \"path\": \"/spec/template/metadata\", \"value\": {\"annotations\":{\"sidecar.istio.io/inject\": \"true\"}, \"labels\":{\"name\":\"$deployment_name\"}}}]"
done

for deployment_name in $(oc get dc | sed '1d' | awk '{print $1}')
do
        oc patch dc $deployment_name --type='json' -p "[{\"op\": \"add\", \"path\": \"/spec/template/metadata\", \"value\": {\"annotations\":{\"sidecar.istio.io/inject\": \"true\"}, \"labels\":{\"name\":\"$deployment_name\"}}}]"
done
```

### dbデータの再登録

### Ingressトラフィックの切り替え
![ingress-gateway](../image/servicemesh-ingressgateway.png)

```
$ oc apply -f virtualservice.yaml
$ oc apply -f gateway.yaml
$ oc apply -f route.yaml -n userX-istio-system
$ oc delete route front-end
```

この時点で一度Kialiにログインをし、Envoy Proxyを導入したことによる恩恵を確認してみよう。



## レジリエンス
cartsサービスのタイムアウトを待っていたのをなくしたい

## トレーシング
特に何も設定することなしに分散トレーシングが実現

## トラフィック制御