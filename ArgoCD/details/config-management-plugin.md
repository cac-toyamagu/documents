## 概要

* ArgoCD v2.8からConfigManagementPlugin (CMP) ConfigMapが廃止されるため、CMP SideCarに移行する必要がある
* 何もわからんのでとりあえずどのように動いているかまとめたい

## ConfigManagementPlugin とは

ArgoCDでは Kustomize や Helm などのツールを使ってマニフェストを生成し、GitOpsを実現している。
KustomizeやHelm以外のツールを使いたい場合は、ConfigManagementPluginを利用して自分の好きなコマンドを使うことができる。
helmfileを実行するためにCMPを用いている。

### CMP ConfigMap

ArgoCD repo-server (K8s manifestを生成するマイクロサービス) コンテナ内でPluginを実行する。
Pluginの設定はConfigMapに記述し、それをそのままrepo-serverでparseしてコマンドを実行することからこの名前がついている。
長らく廃止するよ廃止すると言われていたが、v2.8でようやく廃止された [PR](https://github.com/argoproj/argo-cd/pull/13755)。

理由としてはSideCar型と2つ方法があるとメンテ難しいし、repo-serverとコンテナレベルで分離した方が分離性的に高いよねということらしい。
確かにメンテナンスは大変だし、repo-serverはclientから通信を受け取ってマニフェストを返却したりキャッシュするのが責務なので、必ずしもrepo-serverと同じコンテナで動かなくても良い気がする。

### CMP SideCar

repo-serverのサイドカーコンテナとしてpluginコンテナを動かす形式。
実態としてはUnixドメインソケットを利用したgRPCサーバーである。repo-serverをクライアントとして、Pluginとして定義されたコマンドを実行し、マニフェストを返却する。
図としては以下の感じ。

![CMP SideCar](./cmp-sidecar.drawio.svg)

[このPR](https://github.com/argoproj/argo-cd/pull/6585)で実装された。

## CMP SideCarの設定

[公式ドキュメント参照](https://argo-cd.readthedocs.io/en/stable/operator-manual/config-management-plugins/)
要約すると、適当にConfigMapを作ってSideCarの所定の場所にマウントしてねという感じ。

長時間動作させる場合の重要事項としては以下のとおり、

1. repositoryから不要ファイルを排除する (`.git` など)
1. argocd-server <-> repo-server 間のtimeoutを長くする (default: 60s)
1. argocd-application-controller <-> repo-server 間のtimeoutを長くする (default: 60s)
1. CMP実行時間を長くする (default: 90s)
1. gRPCのkeepaliveを長くする (default: 10s, 内部で2倍されている)
    * https://github.com/argoproj/argo-cd/issues/15656
    * https://github.com/argoproj/argo-cd/pull/15806

## CMP SideCarの動作

↑のPRを読むか、↓にコードがあるので頑張って読めば良い。割と小さめの普通のコードだった。
https://github.com/argoproj/argo-cd/blob/master/cmpserver

要約すると大体↓の流れ。

1. CMP serverを起動
    * [socket listen](https://github.com/argoproj/argo-cd/blob/v2.9.0-rc2/cmpserver/server.go#L87)
    * [socketをserveしてgRPCサーバ起動](https://github.com/argoproj/argo-cd/blob/v2.9.0-rc2/cmpserver/server.go#L87)
1. repo-serverからのgRPC通信を受け取る。 Client-side streaming method `GenerateManifest`
    * [protoc](https://github.com/argoproj/argo-cd/blob/v2.9.0-rc2/cmpserver/plugin/plugin.proto#L64-L65)
      * repo-serverはtgz化したマニフェスト生成のためのファイルを送信する
    * [GenerateManifest](https://github.com/argoproj/argo-cd/blob/v2.9.0-rc2/cmpserver/plugin/plugin.go#L212)
1. 受け取ったファイルを元にテンプレート化する
    * [generateManifestGeneric](https://github.com/argoproj/argo-cd/blob/72f7b145944d0b753180379429c0dd1df0f618bf/cmpserver/plugin/plugin.go#L216)
    * [receive manifest streaming from repo-server](https://github.com/argoproj/argo-cd/blob/v2.9.0-rc2/cmpserver/plugin/plugin.go#L225)
      * この部分で↑で書いたファイルの除外が効いてくる
    * [generate manifest](https://github.com/argoproj/argo-cd/blob/72f7b145944d0b753180379429c0dd1df0f618bf/cmpserver/plugin/plugin.go#L234)
    * [response manifest](https://github.com/argoproj/argo-cd/blob/72f7b145944d0b753180379429c0dd1df0f618bf/cmpserver/plugin/plugin.go#L238-L239)

ちなみに、repo-server側で呼び出すのは以下の部分。Kustomize, Helm, Pluginで分岐している。わかりやすくて良い。
[Call CMP SideCar Plugin](https://github.com/argoproj/argo-cd/blob/v2.9.0-rc2/reposerver/repository/repository.go#L1378-L1379)

大体以上のように動作する。Unixドメインソケットを使っているので通信は割と高速なはずだが、それでもtgz化してstreamingしているのでボトルネックは結構ありそう。

## まとめ

CMP SideCarの動作をざっくりまとめた
Unixドメインソケットを使ってgRPCサーバを立て、repo-serverとプロセス間通信していることがわかった。
オーバーヘッドはCMP ConfigMapと比べて大きい気がするので、きちんとtimeoutやファイルの排除など設定してあげないといけない。
