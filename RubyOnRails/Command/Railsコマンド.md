# Ralisコマンド

### Railsコンソールの起動

```bash
bin/rails console
```

### コントローラーの作成

```bash
rails g controller posts #コントローラーだけ作成する場合
rails g controller posts index #対応するルーティング、Viewも作成される
```

### ルーティング確認

```bash
rails routes
rails routes　-c comments #コントローラの指定
```

### テーブル定義の一覧確認

```bash
#rails console
ActiveRecord::Base.connection.tables
ActiveRecord::Base.connection.execute("SELECT * FROM admin_users WHERE id = 2").to_a
```



### DB

DB.mdのほうに記載



