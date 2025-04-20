# routes

### 7つのアクション

```
- index：一覧表示
- new：新規画面表示
- create：新しいレコードを作成する
- edit：編集画面表示
- show：詳細画面表示
- update：レコード内容を更新する
- destroy：レコードを削除する
```

## ルーティングの設定

### 複数のリソースを扱う場合(idが必要な場合)

```ruby
resources :users #modelは単数形のuser
```

このルーティング定義を行うと、RESTfulな7つのアクションが自動的に生成される。

- 複数形で指定します（:posts）
- index（一覧）アクションが含まれます
- URLに:idパラメータを含むルートが生成されます（/posts/:id）

| **Helper**     | HTTPメソッド | パス            | コントローラー#アクション | 用途                     |
| -------------- | ------------ | --------------- | ------------------------- | ------------------------ |
| users_path     | GET          | /users          | users#index               | ユーザー一覧表示         |
| new_user_path  | GET          | /users/new      | users#new                 | 新規ユーザー作成フォーム |
| users_path     | POST         | /users          | users#create              | ユーザー作成処理         |
| user_path      | GET          | /users/:id      | users#show                | 特定ユーザー表示         |
| edit_user_path | GET          | /users/:id/edit | users#edit                | ユーザー編集フォーム     |
| user_path      | PATCH/PUT    | /users/:id      | users#update              | ユーザー更新処理         |
| user_path      | DELETE       | /users/:id      | users#destroy             | ユーザー削除処理         |

### 単一のリソースを扱う場合(idが不要な場合)

```ruby
resource :login
resource :login, only: %i[ new create] #特定のアクションのみ作成する場合
```

- 単数形で指定します（:login）
- indexアクションが含まれません（一覧表示する必要がないため）
- URLに:idパラメータを含むルートが生成されません（/login）

| Helper          | HTTP verb    | Path        | コントローラー#アクション |
| --------------- | ------------ | ----------- | ------------------------- |
| new_login_path  | GET          | /login/new  | logins#new                |
| login_path      | POST         | /login      | logins#create             |
| login_path      | GET          | /login      | logins#show               |
| edit_login_path | GET          | /login/edit | logins#edit               |
| login_path      | PATCH or PUT | /login/     | logins#update             |
| login_path      | DELETE       | /login/     | logins#destroy            |

