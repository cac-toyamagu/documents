# ArgoCD

## TOC

- [ArgoCD](#argocd)
  - [TOC](#toc)
  - [目的](#目的)
  - [ArgoCDとは](#argocdとは)
  - [ArgoCD基本方針](#argocd基本方針)
  - [用語一覧](#用語一覧)
  - [Repository](#repository)
    - [説明](#説明)
    - [方針](#方針)
    - [Repository Secret作成方法](#repository-secret作成方法)
    - [Repository Secrets自動作成方法検討](#repository-secrets自動作成方法検討)
  - [Project](#project)
  - [Application](#application)
  - [ユーザー管理](#ユーザー管理)
  - [RBAC](#rbac)
  - [References](#references)

## 目的

ArgoCDに関する基本的な事項をまとめること。

## ArgoCDとは

K8s用のGitOpsに基づくCDツール。  
ソースリポジトリを指定し、ソースリポジトリの変更に応じて、実環境のK8sクラスタにリソースをデプロイすることができる。

## ArgoCD基本方針

- 実環境へのデプロイを行うため、環境に応じてガバナンスを明確化すること。
- 上述のガバナンスのために、開発環境での開発効率を下げることがないようにすること。
- IaCを実現するためにArgoCDのリソースは全て宣言的に記述されていること。
  - コンソール上での手作業でのリソース変更などは明確な理由がない限り認めない。
- ArgoCDインストール用マニフェストの各環境に応じた書き換えには helm か kustomizeが利用できる。
  - helmのほうがパラメータ設定は楽だが、community maintaineddであることに注意。
    - 公式マニフェストの方はサポート期限が切れてもパッチを当ててくれているが、あまり当ててくれていないみたいなので、最新マイナーバージョンへのアップデートが前提になるかも。
    - defaultの `values.yaml` は公式参照[^3]。
    - コンテナイメージをプライベートECRに登録しなければならない場合など細かい部分は helmで対応されていない部分がある。
    - 細かい調整が必要になった場合、kustomized helm利用も検討する。
  - kustomizeのほうがパラメータ調整のときに記述量が多いが、ソースとして公式チャートを利用できる。

## 用語一覧

| 用語  | 説明  |
| --- | --- |
| Repository  | Single source of truth となるリポジトリを登録するためのリソース  |

## Repository

### Repository説明

- ArgoCDがSigle source of truth (SSoT) として参照するリポジトリの資格情報などを登録するためのリソース.
  - SSoTがパブリックリポジトリなら良いが、プライベートリポジトリの場合資格情報が必要になる。
  - HTTPS・SSH・GitHub Appの手法が利用できる。
- K8s Secretを作成することで、自動的に作成することが可能[^1]。

### Repository方針

- 宣言的な記述を行うため、ArgoCDのbootstrap時にRepository Secretsを作成し、自動的なRepository登録を行う。

### Repository Secret作成方法

- 以下のようにArgoCDをインストールした名前空間にK8s Secretを作成する。
- `metadata.labels` に `argocd.argoproj.io/secret-type: repository` を指定する。

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: private-repo
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repository
stringData:
  type: git
  url: https://github.com/argoproj/private-repo
  password: my-password
  username: my-username
```

### Repository Secret自動作成方法検討

- Terraform
  - 悪くない。
- SecretProviderClass
  - ArgoCDにはRepository管理用の `repo-server` があるので、そこにマウントする形とする。
  - <details>
    <summary>SecretProviderClass</summary>

    ```yaml
    apiVersion: secrets-store.csi.x-k8s.io/v1
    kind: SecretProviderClass
    metadata:
      name: argocd-repo-secret
    spec:
      provider: aws
      parameters:
        objects: |
          - objectName: "argocd"
            objectType: "secretsmanager"
            jmesPath:
                - path: url
                  objectAlias: url
                - path: username
                  objectAlias: username
                - path: password
                  objectAlias: password
      secretObjects:
      - data:
        - key: url
          objectName: url
        - key: username
          objectName: username
        - key: password
          objectName: password
        secretName: argocd-repo-secret
        type: Opaque
        labels:
          app.kubernetes.io/part-of: argocd
          argocd.argoproj.io/secret-type: repository  # Must
    ```

    </details>
    
  - <details>
    <summary> ArgoCD helm Chartの`values.yaml` </summary>

    ```yaml
    repoServer:
      volumeMounts:
      - name: repo-secret
        mountPath: /mnt/repo-secret
        readOnly: true

      volumes:
        - name: repo-secret
          csi:
            driver: secrets-store.csi.k8s.io
            readOnly: true
            volumeAttributes:
              secretProviderClass: argocd-repo-secret
    ```

    </details>

## Project

## Application

## ユーザー管理

## RBAC


[^1]: https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/
[^2]: https://secrets-store-csi-driver.sigs.k8s.io/introduction.html
[^3]: https://github.com/argoproj/argo-helm/blob/main/charts/argo-cd/values.yaml
