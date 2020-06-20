# SockShopへのService Meshの導入

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

<details><summary>クリックで展開します</summary>
<p>
 - リスト1
 - リスト2
 - リスト3
</p>
</details>

### 各サービスへEnvoy Proxyを導入

### Ingressトラフィックの切り替え

## レジリエンス
cartsサービスのタイムアウトを待っていたのをなくしたい

## トレーシング
特に何も設定することなしに分散トレーシングが実現

## トラフィック制御