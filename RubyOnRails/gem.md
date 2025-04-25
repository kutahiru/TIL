# gem

```
docker compose run web bundle install
```

## better_errors

デフォルトのエラー画面をわかりやすく整形してくれる

## pry-byebug

デバッグ実行が可能になる

## Rubocop

RubyのLintチェックをする時によく使用されるツール
```bash
#チェックの実行
docker compose exec web bundle exec rubocop
```

## sorcery

Railsアプリケーションにおける認証システムの実装を支援するライブラリ

```bash
docker compose run web rails g sorcery:install
docker compose restart
docker compose exec web bin/dev
```

```
#以下のファイルが作成される
app/models/user.rb`
`config/initializers/sorcery.rb`
`db/migrate/202◯◯◯◯◯◯◯◯◯◯_sorcery_core.rb`
`spec/factories/users.rb`
`spec/models/user_spec.rb
```

```ruby
#コントローラーに要実装
before_action :require_login

#not_authenticatedもgemで実装されていて、require_loginからログインしていない場合にCALLされる。
#デフォルトではroot_pathにリダイレクトされる。
#変更したい場合は以下のようにオーバーライド
def not_authenticated
  redirect_to login_path
end
```

```ruby
#topのbefore_action
skip_before_action :require_login, only: %i[top]
#userのbefore_action
skip_before_action :require_login, only: %i[new create]
```

```ruby
class UserSessionsController < ApplicationController
  skip_before_action :require_login, only: %i[new create]

  def new; end

  def create
    #sorceryが提供しているメソッド
    #認証が成功するとセッションに値がセットされる。
    @user = login(params[:email], params[:password]) 

    if @user
      redirect_to root_path
    else
      render :new
    end
  end

  def destroy
    logout
    #HTTPステータスコード303 "See Other"はリダイレクトを指示するコード
    #Railsのredirect_toメソッドでstatus: :see_otherを指定すると、POSTリクエスト後の新しいページへのGETリクエスト移動が促され、フォームの再送信を防ぐ
    redirect_to root_path, status: :see_other
  end
end
```

## rails-i18n

ローカライズ
```ruby
#config/application.rb
config.i18n.default_locale = :ja

#Gemfile
gem 'rails-i18n', '~> 7.0.0'

#config/locales/activerecord/ja.yml
#モデルに関する日本語

#config/locales/views/ja.yml
#ビューに関する日本語
```

翻訳対象

```ruby
<%= link_to t('header.profile'), '#', class: 'dropdown-item' %>

#app/views/user_sessions/new.html.erb
#ja/user_sessions/new配下にある日本語を取得
#tメソッドの引数が . から始まる場合、相対キー
<h1><%= t('.title') %></h1>
```

