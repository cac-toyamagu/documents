# Application

- [Application](#application)
  - [説明](#説明)
  - [方針](#方針)
  - [Application作成](#application作成)
    - [helm](#helm)
  - [App of Appsパターン](#app-of-appsパターン)
    - [背景](#背景)
    - [設定方法概要](#設定方法概要)
    - [概要図](#概要図)
    - [設定例](#設定例)
    - [サンプル](#サンプル)
  - [TIPS](#tips)

## 説明

- K8sリソースのマニフェストが保管されているリポジトリを参照し、対象をK8sクラスタにデプロイする。ArgoCDの中心的なリソース
- 変更を検知し、自動的にデプロイすること（AutoSync）も可能

## 方針

- AppliactionはGitOpsの理念に基づき、宣言的な作成を行うこと。
- Applicationを用いて不必要なリソースをデプロイさせないために、Projectの権限を検討すること。
- AutoSync機能は便利だが、本番環境ではデプロイ事故や不正につながる場合もあるため、十分な検討の上設定すること。

## Application作成

### helm

- `5.0.0` から `argo-cd` の `values.yaml` で `additionalApplications` が deprecatedになった。
- `argocd-apps` を利用する[^3]。
- `values.yaml` は公式リポジトリのものが簡潔かつわかりやすいので、そちら参照[^4]。

## App of Appsパターン

本設では有名なデザインパターンであるApp of Appsパターンを解説する。

### 背景

- GitOpsの理念に基づけば、Applicationは宣言的な作成を行うべきである。
  - その手段としては、ArgoCDのインストール時のチャートに含めてしまうのがわかりやすい。
  - helm chartを用いれば、簡単に実現できる。
- 一方で、開発者がアプリケーションをデプロイしたくなったとき、毎度ArgoCDの設定更新を行うのは運用負荷やインストール時のリスクを考慮すると、明らかに現実的ではない。
- App of Appsパターンは上記の事情をある程度緩和する仕組みである。
- 最近ではApplicationSetsリソースができ、多くのクラスター・NSにデプロイする場合そちらのほうが便利かもしれない[^1] [^2]。

### 設定方法概要

App of Appsを利用する実際の手順は以下の通りである。

1. Applicationを統括するApplication（以降親アプリケーションとする）を作成する。
    - 親Applicationは基盤に近いリソースであり、かつ大量に生成することは考えにくいため、ArgoCDのインストール時に含めてしまうことができる。
    - 後述の子Applicationを配置するためのリポジトリをターゲットとして登録しておく。
1. 親Applicationのターゲットリポジトリに実際にDeploymentなどのK8sリソースを配置するApplication（以降子アプリケーション）を配置する。
    - この作業にArgoCD関連の権限は必要ないため、ほとんどVCSのブランチ運用権限のみを考慮すれば良い。
    - 子Applicationのターゲットには、実際にデプロイするK8sマニフェストが配置されているリポジトリを指定する。
1. 親ApplicationをSyncすることにより、子Applicationがデプロイされる。
1. 子ApplicationをSyncすることにより、実際のK8sマニフェストがデプロイされる。

上記の手順により、Applicationの宣言的な作成を強制しつつ、Applicationの増加に対してスケールするデプロイの仕組みができる。

### 概要図

以下に概要図を示す。

![app-of-apps](./figs/app-of-apps.drawio.svg)

### 設定例

- <details><summary>親Applicationの `values.yaml` 例</summary>

  ```yaml
  applications:
  - name: apps
    namespace: argocd # Must match the namespace of your ArgoCD instance
    additionalLabels: {}
    additionalAnnotations: {}
    finalizers:
    - resources-finalizer.argocd.argoproj.io
    project: apps
    source:
      repoURL: ${repo_url}
      targetRevision: ${target_revision}
      path: ${path}
      directory:
        recurse: true
    destination:
      server: https://kubernetes.default.svc
      namespace: argocd # Must match the namespace of your ArgoCD instance
    syncPolicy:
      automated:
        prune: true
        selfHeal: true
  ```

  </details>

- <details><summary>子Applicationの例</summary>

  ```yaml
  apiVersion: argoproj.io/v1alpha1
  kind: Application
  metadata:
    name: guestbook
    namespace: argocd
    finalizers:
    - resources-finalizer.argocd.argoproj.io
  spec:
    project: argocd-apps
    source:
      repoURL: https://github.com/argoproj/argocd-example-apps.git
      targetRevision: HEAD
      path: guestbook
    destination:
      server: https://kubernetes.default.svc
      namespace: argocd-apps
    syncPolicy:
      automated:
        prune: true
        selfHeal: true
  ```

  </details>

### サンプル

- Terraformを用いたKindのbootstrapサンプル[^6]
  - App of Appsを用いてGuestbookをデプロイする例。
- Terraformを用いたEKSのbootstrapサンプル[^7]
  - App of Appsを用いてinfra appsを作成し、Nginx ingress controllerを自動的にデプロイしている。
  - インターネットからArgoCDへのSSOログインまで自動化されている。

## TIPS

- ApplicationのDelete[^5]
  - Cascading delete（Applicationによって生成されたリソースのデリート）を行う場合、 `finalizer` を指定しておくこと。
  - コンソール上では `forground` `background` どちらかを選択すれば良い。

[^1]: https://argo-cd.readthedocs.io/en/stable/user-guide/application-set/
[^2]: https://codefresh.io/blog/argo-cd-best-practices/
[^3]: https://github.com/argoproj/argo-helm/tree/argo-cd-5.4.3/charts/argocd-apps
[^4]: https://github.com/argoproj/argo-helm/blob/argo-cd-5.4.3/charts/argocd-apps/values.yaml
[^5]: https://argo-cd.readthedocs.io/en/stable/user-guide/app_deletion/
[^6]: https://github.com/toyamagu-2021/terraform-kubernetes-bootstrap-argocd/tree/main/tests/kind
[^7]: https://github.com/toyamagu-2021/terraform-kubernetes-bootstrap-argocd/tree/main/tests/eks
