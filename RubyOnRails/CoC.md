# CoC(Convention over Configuration)

設定よりも規約

### コントローラーは複数系

```ruby
controllers
```

### パーシャルのファイル名

```ruby
_user.html.erb
```

## ActiveRecord

### モデルのクラスは単数形

```ruby
User
BookAuthor #複数の単語の場合はキャメルケース
```

### テーブル名はモデル名の複数系

```ruby
users
book_authors #複数の単語の場合はスネークケース
```

### 主キーはid

Userモデルを参照する外部キーは、 user_idとなる