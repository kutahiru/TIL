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

```ruby
#app\uploaders\board_image_uploader.rb
def default_url
  'board_placeholder'
end
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

### StrongParameter

board_image_cacheは、
フォームで画像をアップロードした後、他の項目でバリデーションエラーがあっても、再度画像を選択させないため
フォームのhidden_fieldと紐づいている

```ruby
  def board_params
    params.require(:board).permit(:title, :body, :board_image, :board_image_cache)
  end
```

## turbo-rails

非同期処理

```
gem 'turbo-rails', '1.1.1'
```

```erb
①
#app\views\boards\_bookmark.html.erb
<%= link_to bookmarks_path(board_id: board.id), id: "bookmark-button-for-board-#{board.id}", data: { turbo_method: :post } do %>
  <i class="bi bi-star"></i>
<% end %>
```

```erb
②
#app\views\bookmarks\create.turbo_stream.erb
<%= turbo_stream.replace "bookmark-button-for-board-#{@board.id}" do %>
  <%= render 'boards/unbookmark', board: @board %>
<% end %>
```

①では、link_toのオプションに data: { turbo_method: :xxxx } が記述されていることで
リダイレクトしない際に xxxxx.turbo_stream.erbファイルを探してレスポンスする

②では、turbo_stream.replace の引数に渡されているID属性、unbookmark-button-for-board-xxx
を探して、対象のDOMをブロック内のものと置き換える

### Append

comments という ID を持つ要素の**内部**に、@comment の部分テンプレートを追加する。
これにより、新しいコメントがリストの最後に追加される
Appendアクションは、新しい要素を既存のコンテンツの後に追加する際に使用する

```erb
<%= turbo_stream.append "comments" do %>
  <%= render @comment %>
<% end %>
```

### Prepend

comments という ID を持つ要素の**内部**の最初に、@comment の部分テンプレートを追加する。
これにより、新しいコメントがリストの先頭に追加される

```erb
<%= turbo_stream.prepend "comments" do %>
  <%= render @comment %>
<% end %>
```

### Replace

comment_#{@comment.id} という ID を持つ要素を、@comment の部分テンプレートで置き換える。
これにより、編集後のコメントが元のコメントと入れ替わる
Replaceアクションは、既存の要素を新しい内容で置き換える際に使用する

```erb
<%= turbo_stream.replace "comment-#{@comment.id}" do %>
  <%= render @comment %>
<% end %>
```

### Update

comment_likes_#{@comment.id} という ID を持つ要素の内容を、@comment.likes_count で更新する
これにより、いいね数がリアルタイムで更新される。
Updateアクションは、既存の要素の内容を部分的に更新する際に使用する

```erb
<%= turbo_stream.update "comment-likes_#{@comment.id}" do %>
  <%= @comment.likes_count %>
<% end %>
```

### Remove

comment_#{@comment.id} という ID を持つ要素を削除する。
これにより、指定したコメントが DOM から削除される。
Remove アクションは、不要になった要素を DOM から取り除く際に使用する。

```erb
<%= turbo_stream.remove "comment-#{@comment.id}" %>
```

### Before

comment_#{@comment.id} という ID を持つ要素の**直前**に、
@new_comment の部分テンプレートを追加する。
これにより、新しいコメントが指定した位置の前に追加される
Before アクションは、特定の要素の前に新しい要素を追加する際に使用する

```erb
<%= turbo_stream.before "comment-#{@comment.id}" do %>
  <%= render @new_comment %>
<% end %>
```

### After

comment_#{@comment.id} というIDを持つ要素の直後に、@new_comment の部分テンプレートを追加する。
これにより、新しいコメントが指定した位置の後に追加される
After アクションは、特定の要素の後に新しい要素を追加する際に使用する

```erb
<%= turbo_stream.after "comment-#{@comment.id}" do %>
  <%= render @new_comment %>
<% end %>
```

## kaminari

Ruby on Rails アプリケーションでページネーション機能を簡単に実装するためのライブラリ
https://github.com/kaminari/kaminari

```
gem 'kaminari', '1.2.2'
```

```bash
#configファイル作成
rails g kaminari:config
```

```ruby
app\controllers\boards_controller.rb
  def index
    @boards = Board.includes(:user).page(params[:page]) #必要に応じて.per(20)とかを追加
  end
```

```erb
#app\views\boards\index.html.erb
<%= paginate @boards %> #ページ数
```

## bootstrap5-kaminari-views

gem 'kaminari' と Bootstrap 5 を組み合わせてページネーションのスタイルを簡単に適用するためのgem
https://github.com/felipecalvo/bootstrap5-kaminari-views

```
gem 'bootstrap5-kaminari-views'
```

```erb
<%= paginate @boards, theme: 'bootstrap-5' %> #ページ数
```

## ransack

検索機能の強化
https://activerecord-hackery.github.io/ransack/

```
gem 'ransack', '3.2.1'
```

```ruby
def index
  @q = Board.ransack(params[:q])
  @boards = if @q.present?
              @q.result.includes(:user).page(params[:page])
            else
              Board.includes(:user).page(params[:page])
            end
end

def bookmarks
  @q = current_user.bookmark_boards.ransack(params[:q])
  @bookmark_boards = if @q.present?
                       @q.result.page(params[:page])
                     else
                       current_user.bookmark_boards.page(params[:page])
                     end
end
```

```erb
#app\views\boards\_search_form.html.erb
<%= search_form_for q, url: url do |f| %>
  <div class="input-group mb-3">
    <%= f.search_field :title_or_body_cont, class: 'form-control', placeholder: t('defaults.search_word') %>
    <div class="input-group-append">
      <%= f.submit class: 'btn btn-primary' %>
    </div>
  </div>
<% end %>
```

title_or_body_contは、「title列、もしくはbody列に検索ワードが含まれる」という条件指定

### カスタム述語

```ruby
#config\initializers\ransack.rb
#vはcreate_dateになるから、create_date.end_of_dayを比較対象にしようする述語を作成している
Ransack.configure do |config|
  config.add_predicate 'lteq_end_of_day',
    arel_predicate: 'lteq',
    formatter: proc { |v| v.end_of_day }
end

#使用時
<%= search_form_for @q, url: admin_boards_path do |f| %>
  <div class="row">
    <div class="d-flex align-items-center justify-content-center">
      <div class="d-inline me-3">
        <%= f.search_field :title_or_body_cont, class: 'form-control', placeholder: t('defaults.search_word') %>
      </div>
      <div class="d-inline me-3">
        <%= f.date_field :created_at_gteq, class: 'form-control' %>
      </div>
      <div class="d-inline me-3">
        <%= f.date_field :created_at_lteq_end_of_day, class: 'form-control' %>
      </div>
      <%= f.submit class: 'btn btn-primary' %>
    </div>
  </div>
<% end %>
```

## letter_opener_web

開発時にテストメールの確認が可能になる。
https://github.com/fgrehm/letter_opener_web?tab=readme-ov-file

```bash
#development内
gem 'letter_opener_web', '2.0.0'
```

## config

環境ごとに異なる設定を一元管理することができ、
開発環境、テスト環境、本番環境で異なる設定値を使いたい場合に非常に便利
config/settings.ymlファイル に共通の設定を書き、以下に環境ごとの設定を書き分ける

- config/settings/development.yml
- config/settings/production.yml
- config/settings/test.yml 

https://github.com/rubyconfig/config

```bash
gem 'config', '4.0.0'
```

```bash
rails g config:install

#以下のファイルが作成される
config/settings.yml #すべての環境で利用する定数を定義
config/settings.local.yml
config/settings/development.yml
config/settings/production.yml
config/settings/test.yml
```

## enum_help

gem 'enum_help'とは、enumで定義した値を簡単にi18n化するためのgem

```
gem 'enum_help', '0.0.19'
```

