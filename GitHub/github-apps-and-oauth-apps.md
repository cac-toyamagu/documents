# GitHub Apps and OAuth Apps

## 目的

GitHub Apps、OAuth Appsに関する基本的な事項をまとめること。

## GitHub Apps・OAuth Appsとは

名前は似ているが、用途は全く異なる[differences-between-github-apps-and-oauth-apps]。  
Personal Access Token・OAuth Apps・GitHub Appsのうちどの認証を選べばよいかは、公式ドキュメントにわかりやすい図がある。
![intro-to-apps-flow](https://docs.github.com/assets/cb-1777535/images/intro-to-apps-flow.png)

- GitHub Apps
  - 独自のidentityを持つ。
  - GitHubユーザーアカウントを作成せずに、マシンユーザーとして使える。
  - GitHub Actionsなどのワークフローを実行する実態として有用。
- OAuth Apps
  - GitHubユーザーのように振る舞う。
  - 外部アプリケーションのSSO認証に利用できる。
  - マシンユーザーとして使う際はGitHubユーザーアカウント作成が必要。

## GitHub Apps

### Actionsでの利用例

Secretsに `APP_ID` と `PEM_{APP_ID}` を登録しておく。

```yaml
- name: Generate token
  id: generate-token
  uses: tibdex/github-app-token@v1
  with:
    app_id: ${{ secrets.APP_ID }}
    private_key: ${{ secrets[format('PEM_{0}', secrets.APP_ID)] }}
```

[differences-between-github-apps-and-oauth-apps]: https://docs.github.com/en/developers/apps/getting-started-with-apps/differences-between-github-apps-and-oauth-apps
