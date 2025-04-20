# DB

## コマンド

### DB削除

```bash
rails db:drop
```

### DB作成

```bash
rails db:create
```

### リセット

drop、 create、 migrate の順に操作される

```bash
rails db:migrate:reset
```

### 直前のファイルをロールバック

```bash
rails db:rollback
```

### 適用状況確認

```bash
rails db:migrate:status
```

### テスト環境にコマンドを適用

```bash
db:migrate:status RAILS_ENV=test
```

### マイグレーションファイル作成

```bash
rails g migration Xxxxx 
```

### 列追加のマイグレーションファイル作成

```bash
rails g migration AddTelToUsers tel:string
```

### マイグレーションファイル実行

```bash
rails db:migrate
```

## マイグレーションファイル

### create_table

```ruby
class CreateArticles < ActiveRecord::Migration[7.2]
  def change
    create_table :articles do |t|
      t.string :title, null: false, default: '記事のタイトル'
      t.text :body
      t.timestamps
    end
  end
end
```

### add_column

```ruby
class AddPriceToArticles < ActiveRecord::Migration[7.2]
  def change
    add_column :articles, :price, :integer
  end
end
```

