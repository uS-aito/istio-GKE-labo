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