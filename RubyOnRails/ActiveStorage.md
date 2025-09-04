# ActiveStorage

Active Storageは、Amazon S3、Google Cloud Storage、Microsoft Azure Storageなどのクラウドストレージサービスへのファイルのアップロードや、ファイルをActive Recordオブジェクトにアタッチする機能を提供

## セットアップ

```bash
bin/rails active_storage:install
bin/rails db:migrate
```

## 内部的には以下のテーブルが使用される

```ruby
# ファイルのメタデータ
active_storage_blobs

# モデルとblobの関連付け
active_storage_attachments

# 画像のバリエーション情報
active_storage_variant_records
```

## 設定

```ruby
# Active Storageのサービスはconfig/storage.ymlで宣言
local:
  service: Disk
  root: <%= Rails.root.join("storage") %>

test:
  service: Disk
  root: <%= Rails.root.join("tmp/storage") %>

# `bin/rails credentials:edit`でAWS secretsを設定すること（aws:access_key_id|secret_access_keyとして）
amazon:
  service: S3
  access_key_id: <%= Rails.application.credentials.dig(:aws, :access_key_id) %>
  secret_access_key: <%= Rails.application.credentials.dig(:aws, :secret_access_key) %>
  bucket: your_own_bucket-<%= Rails.env %>
  region: "" # 例: "us-east-1"
```

```ruby
#必要に応じて設定が必要
# config/environments/development.rb
# ファイルをローカルに保存する
config.active_storage.service = :local

# ファイルをAmazon S3に保存する
config.active_storage.service = :amazon

# ローカルファイルシステム上のアップロード済みファイルを一時ディレクトリに保存する
config.active_storage.service = :test
```

## has_one_attached

ファイルをレコードに添付する
レコードとファイルの間に1対1のマッピングを設定します。
レコード1件ごとに1個のファイルを添付できます。

```ruby
class User < ApplicationRecord
  has_one_attached :avatar
end
```

## 既存のuserにアバター画像を添付する

```ruby
user.avatar.attach(params[:avatar])
```

## ファイルの削除

```
user.avatar.purge

#バックグラウンド削除
user.avatar.purge_later
```

