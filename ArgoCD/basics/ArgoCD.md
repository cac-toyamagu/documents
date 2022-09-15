# ArgoCD

- [ArgoCD](#argocd)
  - [目的](#目的)
  - [ArgoCDとは](#argocdとは)
  - [ArgoCD基本方針](#argocd基本方針)
  - [用語一覧](#用語一覧)
  - [インストール](#インストール)
    - [helm chart](#helm-chart)

## 目的

ArgoCDに関する基本的な事項をまとめること。

## ArgoCDとは

K8s用のGitOpsに基づくCDツール[^1]。  
ソースリポジトリを指定し、ソースリポジトリの変更に応じて、実環境のK8sクラスタにリソースをデプロイすることができる。

## ArgoCD基本方針

- IaCを実現するためにArgoCDのリソースは全て宣言的に記述されていること。
  - コンソール上での手作業でのリソース変更などは明確な理由がない限り認めない。
- 実環境へのデプロイを行うため、環境に応じてガバナンスを明確化すること。
- 上述のガバナンスのために、開発環境での開発効率を下げることがないようにすること。

## 用語一覧

| 用語  | 説明  |
| --- | --- |
| [Repository](repository.md)  | Single source of truth となるリポジトリを登録するためのリソース  |

## インストール

- インストール用マニフェストの各環境に応じた書き換えには、 helm[^2] か kustomizeが利用できる。
  - helmのほうがパラメータ設定は楽だが、community maintainedであることに注意。
    - 公式マニフェストの方はサポート期限[^4]が切れてもパッチを当ててくれている。
      - helm chartの方はそこまでしてくれないと思うので、最新マイナーバージョンへのアップデートが前提になるかも。
    - CRDのアップデートは別途考慮が必要[^5]。
    - コンテナイメージをプライベートECRに登録しなければならない場合など細かい部分は helmで対応されていない部分がある。
    - 細かい調整が必要になった場合、kustomized helm利用も検討する。
  - kustomizeのほうがパラメータ調整のときに記述量が多いが、ソースとして公式チャートを利用できる。
    - 任意の調整も可能。
  - helmfile で大本をhelm chartで管理し、必要に応じてkustomizeするとわかりやすい。

### helm chart

- defaultの `values.yaml` は公式参照[^3]。

[^1]: https://github.com/argoproj/argo-cd
[^2]: https://github.com/argoproj/argo-helm/tree/main/charts/argo-cd
[^3]: https://github.com/argoproj/argo-helm/blob/main/charts/argo-cd/values.yaml
[^4]: https://github.com/argoproj/argo-cd/blob/master/SECURITY.md#supported-versions
[^5]: https://github.com/argoproj/argo-helm/tree/main/charts/argo-cd#custom-resource-definitions
