# 概要
GKEへのIstio導入及びCloudRun on GKEの設定検証、分散トレーシングの検証結果をまとめます。

# 検証結果
## GKEへのIstio導入方法及びCloudRun on GKEの有効化
Istio及びCloudRun On GKEを有効化したクラスタ作成の手順は以下の通り(どちらの手順でもIstio及びCloudRunが有効になっていることを確認済み)。  
なお、以下の点に留意が必要。
* CloudRun on GKEを有効化するためにはstackdriverによるロギング及びモニタリングの有効化、及びvCPU数が2以上のインスタンスでノード数3以上のクラスタ作成が必須になる。
* IstioのバージョンはGKEのバージョンに依存し、Istio単体でバージョンアップできない。もし標準のIstioより新しいバージョンのIstioを使用したい場合、GKEのIstioではなく、istioctlコマンドなどからistioをインストールする必要がある。[参考](https://cloud.google.com/istio/docs/istio-on-gke/overview?hl=ja#should_i_use_istio_on_gke)
### クラスタの作成(Webコンソール)
GKEクラスタ作成画面で、ノードプール作成枠下の「CloudRun for Anthosを有効にする」にチェックを入れる。同じ画面で、一番下の「可用性、...、その他の機能」から「Istio(ベータ版)を有効にする」にチェックを入れる。
### クラスタの作成(CLI)
最低限必要なコマンドは以下の通り。
```
$ gcloud beta container clusters create <cluster name> --addons=Istio,HorizontalPodAutoscaling,HttpLoadBalancing,CloudRun --istio-config=auth=(MTLS_PERMISSIVE|MTLS_STRICT) --machine-type=<machine type(over 2vcpu instance type required)> --num-nodes=<Num of nodes(3 or more node required)> --cluster-version=<cluster version>  --enable-stackdriver-kubernetes
```
## 動作確認
Istioが実際に有効になっているか、CloudRunが有効になっているかそれぞれ確認を行なった。確認はWebコンソール、CLIそれぞれからデプロイしたクラスタに対し行なった。
### CloudRun
Webコンソールからgcr.io/cloudrun/helloをデプロイし、接続を行い結果を確認した。
```
$ curl -H "Host: hello.default.example.com" http://104.197.37.123
<!doctype html>
<html lang=en>
<head>
<meta charset=utf-8>
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Congratulations | Cloud Run</title>
<link href="https://fonts.googleapis.com/css?family=Roboto" rel="stylesheet">
(以下略。-Iヘッダでも付ければよかったです...)
```
また、接続オプションを内部に設定した場合、同じKubernetesクラスタ上からのアクセス以外は拒否した。
### Istio
Istioのサンプルアプリ(bookinfo)をデプロイし、Web画面が正常に表示されることを確認した。また、`kubectl describe`コマンドを使用し、sidecarのEnvoyが動作していることを確認した。

# 今後の調査(1/13)
分散トレーシングシステムについて調査中。以下現段階で分かったことのまとめ。
* 分散トレーシングシステムは以下の四要素から構成される。[^1]
    * アプリケーションに組み込むトレーサ(ライブラリ)
    * トレーサが送るフォーマット
    * 送られてきたデータを集計して表示してくれるバックエンド
    * マイクロサービス間でスパンを紐付けるPropagation
* 1マイクロサービスでの処理に関する情報が**スパン**。マイクロサービスは一つのリクエストに対し、複数のマイクロサービスで処理を行ってレスポンスを返すので、リクエストからレスポンスまでの一連のマイクロサービスのスパンをまとめたものが**トレース**。[^2] [^3]
* Stackdriverにはzipkinコレクタを使うと情報を送れる。らしい。[^4]

[^1]: https://qiita.com/mumoshu/items/d4065a96a9d7e319eceb#%E5%88%86%E6%95%A3%E3%83%88%E3%83%AC%E3%83%BC%E3%82%B7%E3%83%B3%E3%82%B0%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0%E3%81%AE4%E8%A6%81%E7%B4%A0
[^2]: https://dev.classmethod.jp/server-side/microservices-cncf-opentracing-jaeger/
[^3]: http://nakawatch.hatenablog.com/entry/envoy-tracing
[^4]: https://cloud.google.com/trace/