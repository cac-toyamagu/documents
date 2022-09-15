# Authentication and Authorization

- [Authentication and Authorization](#authentication-and-authorization)
  - [説明](#説明)
  - [方針](#方針)
  - [Authentication設定](#authentication設定)
    - [helm chart SSO設定例](#helm-chart-sso設定例)
    - [SSO用Secrets作成例](#sso用secrets作成例)
  - [Authorization設定](#authorization設定)
    - [RBAC設定方針](#rbac設定方針)
    - [RBAC設定方法・項目など](#rbac設定方法項目など)

## 説明

- ArgoCDのユーザー管理（認可）は2つ選択肢がある[^1]。
  - ArgoCDデフォルト機能のユーザー管理
    - ログイン監査ができない、グループ管理できないため設定が面倒など機能豊富ではない
  - 外部IdPを利用したユーザー管理
    - GitHub, GitLab, Keycloak, Googleなどが利用でき便利。
- ArgoCDの認証はRBAC機能を用いる[^2]。

## 方針

- Authentication
  - 運用負荷を下げるため、特別な理由がない限り外部IdPを用いたSSO機能を用いる。
    - ArgoCD Dexで設定しておけば、Argo関連の他のプロダクト、例えばArgoWorkflowでもArgoCD Dexを利用できる。
  - セキュリティのため、緊急時やどうしても必要なとき以外は `admin` は `disable` としておく。
- Authorization
  - RBACを用いて最小権限を付与する。
  - アクターを事前に想定し、アクターごとに付与する権限を検討する。
    - まず大きな区分として、開発者・運用者に分割するとよい。

## Authentication設定

### helm chart SSO設定例

以下にGitHubの場合の設定例を示す。  
後述のようにSSO資格情報はK8s secretから取得できる。

```yaml
server:
  admin.enabled: "false"

  config:
    url: "https://argocd.example.com"
    dex.config: |
      connectors:
        # GitHub example
        - type: github
          id: github
          name: GitHub
          config:
            clientID: $argo-secret-sso:client_id # Get from K8s secret argo-secret-sso
            clientSecret: $argo-secret-sso:client_secert # Get from K8s secret argo-secret-sso
          orgs:
          - name: test-org
```

### SSO用Secrets作成例

SSO資格情報はK8s secretとして保管し、参照することができる[^3]。
以下のラベルをつけることを忘れないこと。

```yaml
labels:
  app.kubernetes.io/part-of: argocd
```

- <details><summary>Secretの例</summary>

  ```yaml
  apiVersion: v1
  kind: Secret
  metadata:
    name: argocd-secret-sso
    namespace: argocd
    labels:
      app.kubernetes.io/part-of: argocd
  type: Opaque
  data:
    # Store client secret like below.
    # Ensure the secret is base64 encoded
    client_id: <client-secret-base64-encoded>
    client_secret: <client-secret-base64-encoded>
  ```

  </details>

- <details><summary>SecretProviderClassの例</summary>

  ```yaml
  apiVersion: secrets-store.csi.x-k8s.io/v1
  kind: SecretProviderClass
  metadata:
    name: argocd-secret-sso
  spec:
    provider: aws
    parameters:
      objects: |
        - objectName: argocd-secret-sso
          objectType: "secretsmanager"
          jmesPath:
              - path: "client_id"
                objectAlias: "client_id"
              - path: "client_secret"
                objectAlias: "client_secret"
    secretObjects:
    - data:
      - key: client_id
        objectName: client_id
      - key: client_secret
        objectName: client_secret
      secretName: argocd-secret-sso
      type: Opaque
      labels:
        app.kubernetes.io/part-of: argocd
  ```

  </details>

  - SecretPoviderClassで作成した場合は、 `argocd-server` にmountしておけばよい。

## Authorization設定

ArgoCDの各種リソースに対してRBACを設定できる。  
SSO時に連携される属性（例えばユーザーグループ）などに紐づけてアクターごとに適切なロールを設定する。

### RBAC設定方針

- アクターに対して必要な最小権限を紐づけること。
  - Projectスコープ以外の不必要なリソース（Repostitory, Clusterなど）へのアクセスを許可しない。
- 宣言的な作成を原則とするため、Create権限の付与は十分考慮の上行うこと。
  - 例えば開発者や運用者にApplicationのCreateを許可するのは一見正しいが、宣言的に作成されていないApplicationが存在してしまう可能性がある。
    - VCSが完全なsingle source of truthとして機能しなくなる。
- SSOに用いるIdP及はCIパイプラインと同一であることを推奨する。
  - CI/CDパイプラインはCIのトリガー権限がCDツールの実行権限とほぼ同一になることもあり、全体的な権限管理が重要であるため。
  - 例えばGitHubやGitLabはブランチ運用の権限管理とも合わせて設計できるため、理想的なIdPであると考えられる。

### RBAC設定方法・項目など

TBD

[^1]: https://argo-cd.readthedocs.io/en/latest/operator-manual/user-management/
[^2]: https://argo-cd.readthedocs.io/en/latest/operator-manual/rbac/
[^3]: https://argo-cd.readthedocs.io/en/stable/operator-manual/user-management/#alternative
