# gem

## インストール

```
docker compose run web bundle install
```

## gemを探す方法
RubyGems.org
https://rubygems.org/

## gemの実装を見る方法
```bash
bundle open <gem名>
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

#なんか権限でエラーになる
docker compose exec --user=root web mkdir -p /.cache
docker compose exec --user=root web chmod 777 /.cache
docker compose exec web bundle exec rubocop
```

## sorcery

Railsアプリケーションにおける認証システムの実装を支援するライブラリ
current_userが使用可能になる

```
gem 'sorcery', '0.16.3'
```

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
```
gem 'rails-i18n', '~> 7.0.0'
```

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

## draper（ドレッパー）

デコレーターを作成できる
ユーザーに表示するデータを整形・加工するための処理（ビューに関するロジック）をモデルから分離し、コードの保守性と読みやすさを向上させることができます。

```
gem 'draper', '4.0.2'
```

```bash
#Userモデルのdecoratorを作成する
docker compose exec web rails g decorator User
```

```ruby
class UserDecorator < Draper::Decorator
  delegate_all #元のオブジェクトのすべてのメソッドを使えるようにするためのもの

  #object は、decoratorがラップしている元のオブジェクト
  def full_name
    "#{object.last_name} #{object.first_name}"
  end
end
```

```ruby
#実行
user.decorate.full_name
```

## faker

ランダムなデータを生成するためのライブラリ

```ruby
10.times do
  User.create!(last_name: Faker::Name.last_name,
              first_name: Faker::Name.first_name,
              email: Faker::Internet.unique.email,
              password: "password",
              password_confirmation: "password")
end
```

## carrierwave

ファイルアップロードを簡単に行うためのライブラリ
ファイルのリサイズやフォーマット変換、バリデーションなどの機能も提供

### BoardImageUploader生成する

```bash
docker compose exec web rails generate uploader BoardImage
```

### Boardsテーブルにboard_imageカラムを追加・データベースに反映させる

```bash
docker compose exec web rails g migration add_board_image_to_boards board_image:string
```

```ruby
class AddBoardImageToBoards < ActiveRecord::Migration[7.0]
  def change
    add_column :boards, :board_image, :string
  end
end
```

### Boardモデルにboard_imageカラムにBoardImageUploaderをマウントすることを記述する

Boardモデルに対して CarrierWave の アップローダークラス（BoardImageUploader）をマウント
Boardモデルのインスタンスで board_image という属性を持つことができ、画像のアップロードや取得が簡単に行える
CarrierWave が提供するアップロードされた画像のURLを取得することができるようになる`board_image` - アップローダーインスタンスを返す

board_image - ファイルをセットする 
board_image_url - ファイルのURLを取得 
remove_board_image - アップロードされたファイルを削除 
remove_board_image - ファイル削除のフラグをセット 
board_image_cache - キャッシュされたファイルの識別子を取得 
board_image_cache - キャッシュを設定

```ruby
class Board < ApplicationRecord
  ... 省略 ...
  mount_uploader :board_image, BoardImageUploader #DBのboard_imageにcarrierwave機能のマッピング
end
```

### アップロードできるファイルを jpg, jpeg, png, gif のみにする

このメソッドを定義することで、アップロードできるファイルの種類を制限できる

```ruby
class BoardImageUploader < CarrierWave::Uploader::Base
  ... 省略 ...

  def extension_allowlist
    %w[jpg jpeg gif png]
  end

  ... 省略 ...
end
```

### フォーム

hidden_fieldは、フォーム内に非表示のフィールドを生成するメソッド
今回の場合ファイルアップロードのキャッシュを保持するための隠しフィールド
フォームのバリデーションエラーが発生した場合でも、ユーザーが再度ファイルを選択する必要がない

```erb
<div class="mb-3">
  <%= f.label :board_image %>
  <%= f.file_field :board_image, class: "form-control", accept: 'image/*' %>
  <%= f.hidden_field :board_image_cache %>
</div>
```

### View側での参照

```erb
  <div id="board-id-<%= board.id %>">
    <div class="card">
      <%= image_tag board.board_image_url, class: "card-img-top", width: "300", height:"200" %>
```

