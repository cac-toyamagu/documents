# Project

- [Project](#project)
  - [説明](#説明)
  - [方針](#方針)
  - [作成例](#作成例)
    - [helm values](#helm-values)

## 説明

- ArgoDCのリソース（主にApplication）をまとめて管理するためのリソース[^1]。
- 以下の制御ができる。
    1. 何がデプロイされるか
        - ソースリポジトリの管理
    1. どこにデプロイされるか
        - ターゲットクラスタ
        - ターゲットK8s NS
    1. どの種類のK8sオブジェクトがデプロイされ、デプロイされないか
    1. プロジェクトロールの定義
        - プロジェクトに対して実行して良い操作をRBACを用いて制御できる
        - クロスProjectアクセスやプロジェクトに属さないリソースは、ArgoCD自体のRBAC[^2]を用いる。
- その他、以下の機能がある
    1. Sync windowの設定
        - どの間隔でsyncするか
    1. Orphaned resourceの発見
        - ArgoCDで管理されていないTop level resourceを見つけることができる）

## 方針

- default Project利用方針
  - 何でもデプロイできる状態になっているため、利用しない
- Project分割方針
  - Applicationのまとまり毎にProjectを作成し、RBACによる制御を行う。
- デプロイ可能Repository制御方針
  - 本番環境などデプロイ可能なパブリックリポジトリやプライベートリポジトリを制限したい場合は利用する。
  - その他チーム間でK8sマニフェストリポジトリを分けている場合など、ソースリポジトリを制限したい場合は利用する。
- デプロイ可能K8sリソース制御方針
  - 必要がない限り、ClusterRoleなど、Clusterレベルのリソースはブラックリストに登録しておく。
  - その他は必要に応じて設定する。
- RBAC記述方針
  - 小規模なチームの場合はArgoCD RBACでProjectも含むRBACを一元管理する。
    - 小規模チームでは記述量が少なく、分割すると認知コストが増えるため。
  - 大規模なチームの場合はArgoCD RBACでロールのみ定義し、各Projectで権限を割り当てる。
- CI/CDからArgoCDのApplicatioをsyncする必要がある場合はProjectロールトークンを発行して行う。
  - AutoSync機能やhook機能があるため、それで十分な場合も多い。
  - 運用コストが増えることを加味して検討する。
- Projectデプロイ方針
  - 基盤に近い部分なので、ArgoCDのデプロイ時に一緒に含めてしまうと良い。
    - helm chartの場合は後述するように簡単。
  - TerraformでArgoCDをデプロイするなら、 `argocd-apps` helm chartを利用してデプロイする。
    - 参考[^5]

## 作成例

### helm values

- `5.0.0` から `argo-cd` の `values.yaml` で `additionalProjects` が deprecatedになった。
- `argocd-apps` を利用する[^3]。
- `values.yaml` は公式リポジトリのものが簡潔かつわかりやすいので、そちら参照[^4]。

[^1]: https://argo-cd.readthedocs.io/en/stable/user-guide/projects/
[^2]: https://argo-cd.readthedocs.io/en/stable/operator-manual/rbac/
[^3]: https://github.com/argoproj/argo-helm/tree/argo-cd-5.4.3/charts/argocd-apps
[^4]: https://github.com/argoproj/argo-helm/blob/argo-cd-5.4.3/charts/argocd-apps/values.yaml
[^5]: https://github.com/toyamagu-2021/terraform-kubernetes-bootstrap-argocd/tree/main/tests/eks
