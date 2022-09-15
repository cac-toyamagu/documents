# RBAC

- [RBAC](#rbac)
  - [目的](#目的)
  - [前提となる知識](#前提となる知識)
    - [設定例](#設定例)
  - [RBAC設定構造](#rbac設定構造)
  - [RBACリソースとアクション](#rbacリソースとアクション)
  - [アクション](#アクション)
  - [サンプル１：開発環境/開発者用設定例](#サンプル１開発環境開発者用設定例)
    - [方針](#方針)
    - [具体的な設定](#具体的な設定)
    - [具体的設定ファイル](#具体的設定ファイル)
  - [サンプル２：本番環境/開発者用設定例](#サンプル２本番環境開発者用設定例)
    - [方針](#方針-1)
    - [具体的な設定](#具体的な設定-1)
    - [具体的な設定ファイル](#具体的な設定ファイル)
  - [サンプル３：開発環境/運用者用設定例](#サンプル３開発環境運用者用設定例)
    - [方針](#方針-2)
    - [具体的な設定](#具体的な設定-2)
    - [具体的な設定ファイル](#具体的な設定ファイル-1)
  - [サンプル４：本番環境/運用者用設定例](#サンプル４本番環境運用者用設定例)
    - [方針](#方針-3)
    - [具体的な設定ファイル](#具体的な設定ファイル-2)

## 目的

- RBACを具体的にどう設定すればよいか、設定例を元に説明すること。
- 設定例を示す際は、結果の設定ファイルが導けるように、設定の方針を具体的に記述すること。

## 前提となる知識

以下のサンプルファイルを読む上で、前提となる知識をまとめる。

### 設定例

まず、簡単な設定ファイルを示す。現時点で完全に理解できなくとも良いが、以下での説明を読む際に適宜比較するための具体例として示す。

- `my-org:team-x` はSSO my-org のユーザーグループ team-x と読み替えるとわかりやすい。
- `my-org:team-alpha` ユーザーグループにArgoCDプロジェクト my-project 以下の sync 権限を allow する。
- `my-org:team-beta` ユーザーグループに role:admin 権限を与える。

```csv
# Grant all members of the group 'my-org:team-alpha; the ability to sync apps in 'my-project'
p, my-org:team-alpha, applications, sync, my-project/*, allow
# Grant all members of 'my-org:team-beta' admins
g, my-org:team-beta, role:admin
```

## RBAC設定構造

- Projectに属するリソース
  `p, <role/user/group>, <resource>, <action>, <object>`
- Projectに属するリソース( Application, `log`, `exec`)
  `p, <role/user/group>, <resource>, <action>, <project>/<object>`

## RBACリソースとアクション

| リソース  | Projectに属するか  |  説明|
|---|---|---|
| clusters  | x  | デプロイ対象のクラスター |
| projects | x | Project |
| applications | o | Application |
| repositories | either | Repository |
| certificates | x | 証明書 |
| accounts | x | アカウント |
| gpgkeys | x | gpgキー |
| logs | o | Podなどのログ |
| exec | o | createを許可するとUIからPodに `kubectl exec` に似たことを実行できる |

## アクション

| action名  |   説明|
|---|---|
|get, create, update, delete | CRUD |
|sync | Application同期権限 |
|override | parameter override権限[^1] |
|action/<group/kind/action-name> | 定義済みのスクリプトか、カスタムスクリプトを実行することができる。`restart`等。<br> |

## サンプル１：開発環境/開発者用設定例

開発環境の開発者用の設定例を記述する。

### 方針

- 可能な限りApplicationのデプロイとデバッグを自由に行ってもらい、開発を阻害しないこと。
- GitOpsの理念に従い、宣言的な作成を行うこと。
- ガバナンスのため、Application以外のインフラに近いリソースへのアクセスは許可しないこと。
  - K8sクラスターの追加やVCSリポジトリの追加など

### 具体的な設定

- `project/*` の Application リソースの `create`, `override`, `update` を許可しない。
- `project/*` のapplications リソースの sync, get を許可する。
- `project/*` の logs リソースのget を許可する。
  - 開発環境のlogに機密情報が含まれることは想定しないため、セキュリティ上の懸念事項はない想定。
- `project/*` の `exec` リソースの `create` を許可する。
  - 開発環境でのデバッグを容易にするため。
  - 開発者は元々開発環境の Pod に `kubectl exec` や `kubectl debug` できる想定のため、セキュリティ上の懸念事項は少ない想定。

### 具体的設定ファイル

```csv
g, org:dev-team, role:dev

p, role:dev, applications, sync, project/*, allow
p, role:dev, applications, get, project/*, allow
p, role:dev, logs, get, project/*, allow
p, role:dev, exec, create, project/*, allow
```

## サンプル２：本番環境/開発者用設定例

### 方針

- 最小権限を付与しつつ、本番環境障害時に、迅速にデバッグを行うことができる方針を定めること。

### 具体的な設定

- `project/*` の `applications` リソースの `get` を許可する。
  - デプロイされているアプリケーションの確認を許可するため。
  - セキュリティリスクなし想定。
- project/* の logs リソースのget を許可する。
  - 障害時にログからデバッグを行うため。
  - ログに機密情報が含まれる場合、別途検討。

### 具体的な設定ファイル

```csv
g, org:dev-team, role:dev

p, role:dev, applications, get, project/*, allow
p, role:dev, logs, get, project/*, allow
```

## サンプル３：開発環境/運用者用設定例

開発環境の運用者用の設定例を記述する。  
開発環境では、開発効率を上げるため、開発者のリーダーにこのポリシーをアタッチすることも検討する。  

### 方針

- 運用者のため、基本的に全ての権限を付与して良い。
- ただし、 そもそも利用しない機能は許可しないものとする。

- application/create・applications/override・applications/update
  - 宣言的な作成を強制するため、applicationの作成・変更は不許可とする。
- exec ・ action ・gpgkey ・clusters
  - 該当機能を利用しない場合。
  - 外部K8sクラスタを利用しない場合、clustersも不許可。
- account
  - アカウント管理にはSSOを利用するため、該当機能を利用しない場合。
- project毎に運用者を設定する場合、applicationsのprojectも制限することにする。

### 具体的な設定

action ・gpgkey ・account 関連は利用しないと仮定する。

### 具体的な設定ファイル

```csv
g, org:opr-team, role:opr

p, role:opr    , applications, get   , */*, allow
p, role:opr    , applications, delete, */*, allow
p, role:opr    , applications, sync  , */*, allow

p, role:opr    , logs        , get   , */*, allow

p, role:opr    , exec        , create, */*, allow

p, role:opr    , clusters    , create, *  , allow
p, role:opr    , clusters    , update, *  , allow
p, role:opr    , clusters    , get   , *  , allow
p, role:opr    , clusters    , delete, *  , allow

p, role:opr    , projects    , create, *  , allow
p, role:opr    , projects    , update, *  , allow
p, role:opr    , projects    , get   , *  , allow
p, role:opr    , projects    , delete, *  , allow

p, role:opr    , repositories, create, *  , allow
p, role:opr    , repositories, update, *  , allow
p, role:opr    , repositories, get   , *  , allow
p, role:opr    , repositories, delete, *  , allow

p, role:opr    , certificates, create, *  , allow
p, role:opr    , certificates, update, *  , allow
p, role:opr    , certificates, get   , *  , allow
p, role:opr    , certificates, delete, *  , allow
```

## サンプル４：本番環境/運用者用設定例

### 方針

- 本番環境へのデプロイや、迅速な障害対応を行う権限を付与すること。
- 基本的には開発環境と同一で良いが、必要に応じて以下観点を検討すること。
  - 限られた `project` のみに許可するか。
  - `exec` 権限を付与してよいか。
  - 通常運用時の読み取り権限のみのロールと緊急時やリリース時の強い権限を持ったロールで分離する。
  - IdPのグループなどの属性割当で対応する。

### 具体的な設定ファイル

```csv
g, org:opr-team, role:opr

p, role:opr    , applications, get   , */*, allow
p, role:opr    , applications, delete, */*, allow
p, role:opr    , applications, sync  , */*, allow

p, role:opr    , logs        , get   , */*, allow

p, role:opr    , exec        , create, */*, allow

p, role:opr    , clusters    , create, *  , allow
p, role:opr    , clusters    , update, *  , allow
p, role:opr    , clusters    , get   , *  , allow
p, role:opr    , clusters    , delete, *  , allow

p, role:opr    , projects    , create, *  , allow
p, role:opr    , projects    , update, *  , allow
p, role:opr    , projects    , get   , *  , allow
p, role:opr    , projects    , delete, *  , allow

p, role:opr    , repositories, create, *  , allow
p, role:opr    , repositories, update, *  , allow
p, role:opr    , repositories, get   , *  , allow
p, role:opr    , repositories, delete, *  , allow

p, role:opr    , certificates, create, *  , allow
p, role:opr    , certificates, update, *  , allow
p, role:opr    , certificates, get   , *  , allow
p, role:opr    , certificates, delete, *  , allow
```

[^1]: https://argo-cd.readthedocs.io/en/stable/user-guide/parameters/
