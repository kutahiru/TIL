# railsのバージョンアップ

## 参考

https://zenn.dev/wed_engineering/articles/4a0194badf6ee8

## Gemfileの修正

```
# 今は8.0.2が8.0系の最新
gem "rails", "~> 8.0.2
```

```bash
bundle update rails
```

## railsの設定更新

```bash
rails app:update
```

## 修正箇所

```ruby
# app/models/user.rb
#enum description_read: { not_read: false, read: true }, _prefix: :description
# Rails 8対応：位置引数 + 型明示
attribute :description_read, :boolean, default: false
enum :description_read, { not_read: false, read: true }, prefix: :description
```

```ruby
# config/routes.rb
#resources :medication_group_invitations, shallow: true, only: %i[create show accept]
resources :medication_group_invitations, shallow: true, only: %i[create show] do
  member do
    patch :accept
  end
end
```

