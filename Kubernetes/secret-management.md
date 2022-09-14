# Secret管理

## TOC

- [Secret管理](#secret管理)
  - [TOC](#toc)
  - [目的](#目的)
  - [Terraform](#terraform)
  - [SecretsProviderClass](#secretsproviderclass)

## 目的

- K8s Secret管理方法をまとめること。

## Terraform

- `kubernetes_secret` リソースで作成する。
- `.tfstate` に情報が残る。
- `helm_release` などでArgoCDをTerraformで管理している場合に有効。
- Terraformへの受け渡し方法
  - variablesに指定する。
  - AWS SecretsManagerやAWS SSM PSから取得する。

## SecretsProviderClass

- 外部から値を取得する。詳細は公式参照[^2]。
- K8s Secretsを作成する機能がある。
  - Podのmount時にSecretが作成されるため、Podが立ち上がっていないとSecretが作成されない。
