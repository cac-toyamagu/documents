# GitHub Actions

- [GitHub Actions](#github-actions)
  - [目的](#目的)
  - [GitHub Actionsとは](#github-actionsとは)
  - [各クラウドとの連携](#各クラウドとの連携)

## 目的

GitHub Actionsに関する基本的な事項をまとめること。

## GitHub Actionsとは

GitHubで動くCICDツール。  
GitHubイベントを元にスクリプトの実行ができる。

## 各クラウドとの連携

OIDC機能を用いて、AWSやAzure、GCPなどのクラウドとの連携ができる[oidc]。  
例えばAWSでは、IAM OIDCを用いて、資格情報なしでAWSコマンドの実行ができる[zenn-github-actions-support-openid-connect]。

[oidc]: https://docs.github.com/en/actions/deployment/security-hardening-your-deployments
[zenn-github-actions-support-openid-connect]: https://zenn.dev/miyajan/articles/github-actions-support-openid-connect
