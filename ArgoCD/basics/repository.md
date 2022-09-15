# Repository

- [Repository](#repository)
  - [説明](#説明)
  - [方針](#方針)
  - [Repository Secret作成方法](#repository-secret作成方法)
  - [Repository Secret自動作成方法検討](#repository-secret自動作成方法検討)

## 説明

- ArgoCDがSigle source of truth (SSoT) として参照するリポジトリの資格情報などを登録するためのリソース.
  - SSoTがパブリックリポジトリなら良いが、プライベートリポジトリの場合資格情報が必要になる。
  - HTTPS・SSH・GitHub Appの手法が利用できる。
- K8s Secretを作成することで、自動的に作成することが可能[^1]。

## 方針

- 宣言的な記述を行うため、ArgoCDのbootstrap時にRepository Secretsを作成し、自動的なRepository登録を行う。

## Repository Secret作成方法

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

## Repository Secret自動作成方法検討

- Terraform
  - `kubernetes_secret` リソースを作成する [^4]。
  
- SecretProviderClass[^2]
  - ArgoCDにはRepository管理用の `repo-server` があるので、そこにマウントする形とする。
  - ArgoCD helm chartには `extraObjects` が用意されているため、そこに記述しても良い[^5]。
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

[^1]: https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/
[^2]: https://secrets-store-csi-driver.sigs.k8s.io/introduction.html
[^4]: ../Kubernetes/secret-management.md
[^5]: https://github.com/argoproj/argo-helm/blob/a6a2d1b1db72c0f8805678db8e410324c8afb772/charts/argo-cd/values.yaml#L78
