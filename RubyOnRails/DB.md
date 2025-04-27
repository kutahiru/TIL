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

### 適用前に戻す場合

```bash
rails db:rollback STEP=1 #1つ前のマイグレーションに戻す
```

### テスト環境にコマンドを適用

```bash
db:migrate:status RAILS_ENV=test
```

### マイグレーションファイル作成

```bash
rails g migration Xxxxx 
#db/migrateディレクトリ配下に、データベースのテーブル構造を定義するマイグレーションファイルも生成
#モデルは追加しないので、主に定義変更時に使用する
```

### 列追加のマイグレーションファイル作成

```bash
rails g migration AddTelToUsers tel:string
```

### マイグレーションファイル実行

```bash
rails db:migrate
```

### 新しいモデルとマイグレーションファイルを作成

```bash
rails g model モデル名
#app/modelsディレクトリに〇〇.rbというファイルが生成
#db/migrateディレクトリ配下に、データベースのテーブル構造を定義するマイグレーションファイルも生成
```

## マイグレーションファイル

### create_table

```ruby
class CreateBoards < ActiveRecord::Migration[7.0]
  def change
    create_table :boards do |t|
      t.string :title, null: false
      t.text :body, null: false
      #referencesは外部キー設定用のメソッド、列名はモデル名_id
      t.references :user, foreign_key: true

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

## MySQLに接続する

### コンテナ内部に入る場合

```bash
#サービス名はdocker-compose.ymlに記載がある。
docker compose exec サービス名 bash
```

### コマンド

```bash
#パスワードはdocker-compose.ymlに記載がある。
mysql -u root -p #ログイン
show databases; #DB一覧
create database DB名; #DB作成
use runteq_study; #DBに移動する
show tables; #テーブル確認
SHOW CREATE TABLE users; #テーブルの詳細確認

#テーブル作成
CREATE TABLE user(
    id INT(11) AUTO_INCREMENT NOT NULL,
    first_name VARCHAR(30) NOT NULL,
    last_name VARCHAR(30) NOT NULL,
    PRIMARY KEY (id));
```

## 開発環境の初期データ

db/seeds.rb

```ruby
10.times do
  User.create!(last_name: Faker::Name.last_name,
              first_name: Faker::Name.first_name,
              email: Faker::Internet.unique.email,
              password: "password",
              password_confirmation: "password")
end

user_ids = User.ids

20.times do |index|
  user = User.find(user_ids.sample)
  user.boards.create!(title: "タイトル#{index}", body: "本文#{index}")
end
```

### 適用コマンド

```bash
rails db:seed
```

