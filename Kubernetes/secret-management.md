# Secret管理

## TOC

- [Secret管理](#secret管理)
  - [TOC](#toc)
  - [目的](#目的)
  - [Terraform](#terraform)
  - [SecretsProviderClass](#secretsproviderclass)
  - [External Secrets Operator](#external-secrets-operator)

## 目的

- K8s Secret管理方法をまとめること。

## Terraform

- `kubernetes_secret` リソースで作成する。
- `.tfstate` に情報が残る。
- `helm_release` などでArgoCDをTerraformで管理している場合に有効。
- Terraformへの受け渡し方法
  - variablesに指定する。
    - `.tfvars` の管理が面倒。
    - CI/CD 中の環境変数で指定するならありかも。 `TF_VARS_HOGEHOGE` とか。
  - AWS SecretsManagerやAWS SSM PSから取得する。
    - スクリプトを用いた初期セットアップ方法は検討した [^1]。
      1. 適当な初期値を代入したパラメータを外部シークレットストレージに保管する。
      1. variablesに値が格納されていれば、 `null` リソースを用いた `local_exec` を用いてスクリプトを実行し、外部クレデンシャルストレージに保管する。
      1. `data` を用いてシークレットを外部ストレージから取得する。
      1. 変更する値は `ignore_changes` で指定する。

## SecretsProviderClass

- 外部から値を取得する。詳細は公式参照[^2]。
- K8s Secretsを作成する機能がある。
  - Podのmount時にSecretが作成されるため、Podが立ち上がっていないとSecretが作成されない。

## External Secrets Operator

- 外部シークレットストアからK8s Secretを作成する[^2]。
- Podにマウントする必要がないので便利そう。

[^1]: https://github.com/toyamagu-2021/terraform-kubernetes-bootstrap-argocd/tree/main/tests/eks
[^2]: https://external-secrets.io/v0.6.0-rc1/
