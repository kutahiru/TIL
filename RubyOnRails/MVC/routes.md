# routes

```
root "homes#top"
```

### ルーティングを一覧で確認

http://localhost:3000/rails/info/routes

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
- \_pathと同様に_urlも作成される。

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
- \_pathと同様に_urlも作成される。

| Helper          | HTTP verb    | Path        | コントローラー#アクション |
| --------------- | ------------ | ----------- | ------------------------- |
| new_login_path  | GET          | /login/new  | logins#new                |
| login_path      | POST         | /login      | logins#create             |
| login_path      | GET          | /login      | logins#show               |
| edit_login_path | GET          | /login/edit | logins#edit               |
| login_path      | PATCH or PUT | /login/     | logins#update             |
| login_path      | DELETE       | /login/     | logins#destroy            |

### ネストしたルーティング

```ruby
Rails.application.routes.draw do
  ... 省略 ...
  resources :boards, only: %i[index new create show] do
    resources :comments, only: %i[create edit destroy], shallow: true
  end
  ... 省略 ...
end
```

### shallow オプションについて

shallowオプションは、ネストしたリソースの一部のアクションに対して、親リソースのIDを含まないURLを生成するために使用する。

- 親リソース（掲示板）のIDが必要なアクション

  collection アクション（index, new, create）には親リソースのID
  ネストされたURLが生成されます
  POST /boards/:board_id/comments → コメント作成

- 親リソースのIDが不要なアクション

  member アクション（show, edit, update, destroy）では親リソースのIDを含めない
  ネストの浅いシンプルなURLが生成されます
  GET /comments/:id/edit → コメント編集フォーム
  DELETE /comments/:id → コメント削除

### ネストされているケース

| ヘルパー                 | HTTP verb | パス                     | コントローラー#アクション |
| ------------------------ | --------- | ------------------------ | ------------------------- |
| board_comments_path()    | GET       | /boards/:id/comments     | comments#index            |
| new_board_comment_path() | GET       | /boards/:id/comments/new | comments#new              |
| board_comments_path()    | POST      | /boards/:id/comments     | comments#create           |
| comment_path()           | GET       | /comments/:id            | comments#show             |
| edit_comment_path()      | GET       | /comments/:id/edit       | comments#edit             |
| comment_path()           | PATCH/PUT | /comments/:id            | comments#update           |
| comment_path()           | DELETE    | /comments/:id            | comments#destroy          |

### RESTfulなルーティングにアクションを追加する

個々の掲示板（board）に対してプレビューを行いたいとかではなく、
掲示板全体（boards）の中からブックマークされている掲示板の一覧を表示したいということで、collectionを使って get :bookmarks を記述

```ruby
resources :boards, only: %i[index new create show edit update destroy] do
    resources :comments, only: %i[create edit destroy], shallow: true
    collection do
      get :bookmarks
    end

#以下のルーティングが定義される
#ルーティングヘルパーの命名規則では、コレクションアクションの場合、「アクション名_リソース名_path」という形式
prefix: bookmarks_boards
HTTPメソッド: GET
URLパターン: /boards/bookmarks
コントローラー#アクション: boards#bookmarks
```

