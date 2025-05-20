# view

### ファイル名

top.html.erb


### パーシャル(部分テンプレート)

ビュー画面を共通化するために使用
呼び出しは「render ファイル名（ディレクトリを指定）」
引数はパーシャル内でローカル変数として参照可能
ファイル名は_で始める。

```erb
#index.html.erb
<div id="users">
  <% @users.each do |user| %>
    <%= render user %>
    <p>
      <%= link_to "Show this user", user %>
    </p>
  <% end %>
</div>

#「render user」は以下の省略形
<%= render partial: "users/user", locals: { user: user } %>

#「render "form", user: @user」は以下の省略形
<%= render partial: "form", locals: { user: @user } %>
```

```erb
#_user.html.erb
<div id="<%= dom_id user %>">
  <p>
    <strong>名前:</strong>
    <%= user.name %>
  </p>
</div>
```

「render user」ではRailsは自動的に以下のことを行っている。

1. ユーザーオブジェクトの種類（モデル名）を判断する
2. そのモデル名に基づいて適切なパーシャルを探す（通常は`_user.html.erb`というファイル）
3. そのパーシャルにユーザーオブジェクトを渡してレンダリングする

### collectionオプションを使ったrenderメソッドの記述

- collectionの省略記法が使える条件
  - 部分テンプレートが呼び出し元のテンプレートと同じディレクトリ内にある
  - 部分テンプレートのファイル名が指定した変数の単数形である
  - 部分テンプレート内で使用する変数名が、オプションで指定した変数の単数形である

```erb
<% if @boards.present? %>
  <%= render @boards %>
<% else %>
  <div class="mb-3">掲示板がありません</div>
<% end %>

#↑の省略前
<%= render partial: 'board', collection: @boards %>
```

### link_to

aタグを生成するヘルパーメソッド

```erb
link_to("profile link", edit_profile_path)
```

### link_toでHTMLを表示したい場合

第一引数ではなく、以下のようにブロックで記載する必要がある。

```erb
<%= link_to("/likes/#{@post.id}/destroy", {method: "post"}) do %>
  <span class="fa fa-heart liked-btn"></span>
<% end %>   
```

### 削除ボタン_アラート付き

```erb
<%= link_to board_path(board), id: "button-delete-#{board.id}", data: { turbo_method: :delete, turbo_confirm: t('defaults.delete_confirm') } do %>
  <i class="bi bi-trash-fill"></i>
<% end %>
```

### link_to_unless

第一引数の条件に一致しなかったら
link_to_ifは一致したら

```erb
<%=
   link_to_unless(@current_user.nil?, "Reply", { action: "reply" }) do |name|
     link_to(name, { controller: "accounts", action: "signup" })
   end
%>
```

### render

現在のリクエスト内でビューをレンダリング（描画）する処理
相対パスを設定する。
処理の失敗時はこっち

### redirect_to

別のURLにリダイレクト（転送）する」処理
絶対パスを設定する。
処理の成功時はこっち

### form_with

フォームを構築するためのヘルパーメソッド
モデルの情報を元に送信URLやフィールドの内容を埋めたりすることができる。
https://railsguides.jp/form_helpers.html

#### シンプルなフォーム

```erb
<%= form_with do |form| %>
  <!-- Form contents ->
<% end %>
```

#### フォームに設定可能なヘルパーメソッド

「テキストフィールド」「チェックボックス」「ラジオボタン」などのこと。
これらのメソッドの第1引数は、常に入力の名前。
この名前がフォームデータとともに`params`ハッシュでコントローラに渡される

```erb
<%= form_with url: search_path do |form| %>
  <%= form.text_field :query %>  # paramsでは params[:query] でアクセス
  <%= form.submit "検索" %>
<% end %>

#コントローラでは以下でアクセス可能となる
params[:query]
```

#### モデルオブジェクトを指定したフォーム

:modelオプションを使うと、フォームビルダーオブジェクトをモデルオブジェクトに紐付けできるようになる
モデルオブジェクトの値がフォームのフィールドに自動入力される

Railsはそのモデルの状態（新規または既存）に基づいて適切なリクエスト先（アクション）とHTTPメソッド（POSTまたはPATCH/PUT）を自動で選択

@userが新規オブジェクトの場合、フォームは users#create へのPOSTリクエストを行いますが、既存データの編集の場合は users#update へのPATCH/PUTリクエスト

```ruby
@book = Book.find(42)
# => #<Book id: 42, title: "Walden", author: "Henry David Thoreau">
```

```erb
<%= form_with model: @book do |form| %>
  <div>
    <%= form.label :title %>
    <%= form.text_field :title %>
    <%# 上は以下のようなHTMLが作成される%>
    <%# <input type="text" name="book[title]" id="book_title" value="本のタイトルの現在の値" /> %>
  </div>
  <div>
    <%= form.label :author %>
    <%= form.text_field :author %>
  </div>
  <%= form.submit %>
<% end %>

#コントローラでアクセスする際はStrongParametersを使用する
```

```erb
#sample
<div class="container">
  <div class="row">
    <div class="col-md-10 col-lg-8 mx-auto">
      <h1>ユーザー登録</h1>
      <%= form_with model: @user do |f| %>
        <div class="mb-3">
          <%= f.label :last_name, class: "form-label" %>
          <%= f.text_field :last_name, class: "form-control" %>
        </div>
        <div class="mb-3">
          <%= f.label :first_name, class: "form-label" %>
          <%= f.text_field :first_name, class: "form-control" %>
        </div>
        <div class="mb-3">
          <%= f.label :email, class: "form-label" %>
          <%= f.email_field :email, class: "form-control" %>
        </div>
        <div class="mb-3">
          <%= f.label :password, class: "form-label" %>
          <%= f.password_field :password, class: "form-control" %>
        </div>
        <div class="mb-3">
          <%= f.label :password_confirmation, class: "form-label" %>
          <%= f.password_field :password_confirmation, class: "form-control" %>
        </div>
        <%= f.submit "登録", class: "btn btn-primary" %>
      <% end %>
      <div class='text-center'>
        <%= link_to 'ログインページへ', login_path %>
      </div>
    </div>
  </div>
</div>
```

#### モデルを指定しないフォーム

URLを直接指定する方法は、特定のアクションにフォームデータを送信する場合
f.email_field :emailについて、
form_withヘルパーメソッドの中で使われるemail_fieldメソッド
「<input type="email">」を生成する
`:email` という名前（name属性）でフォームデータを送信する

```erb
<div class="container">
  <div class="row">
    <div class=" col-md-10 col-lg-8 mx-auto">
      <h1>ログイン</h1>
      <%= form_with url: login_path do |f| %>
        <div class="mb-3">
          <%= f.label :email %>
          <%= f.email_field :email, class: "form-control" %>
        </div>
        <div class="mb-3">
          <%= f.label :password %>
          <%= f.password_field :password, class: "form-control" %>
        </div>
        <%= f.submit "ログイン", class: "btn btn-primary" %>
      <% end %>
      <div class='text-center'>
        <%= link_to '登録ページへ', new_user_path %>
        <%= link_to 'パスワードをお忘れの方はこちら', '#' %>
      </div>
    </div>
  </div>
</div>
```

### helperとは

ビューで使用するメソッドを定義するための場所
app/helpers/application_helper.rb

### content_forメソッド

Railsのビューヘルパーの一つで、ビューテンプレート内で特定の名前付きコンテンツブロックを定義し、そのコンテンツを別の場所（通常はレイアウトファイル）で表示するために使用する。
以下はページのタイトルとして利用

```ruby
module ApplicationHelper
  def page_title(title = '')
    base_title = 'RUNTEQ BOARD APP'
    title.present? ? "#{title} | #{base_title}" : base_title
  end
end
```

```erb
#application.html.erb
<title><%= page_title(yield(:title)) %></title>
```

```erb
#各viewファイル
<% content_for(:title, t('.title')) %>
```

### メッセージ カスタムflash

カスタムフラッシュタイプの追加

```erb
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  add_flash_types :success, :error, :info, :warning
end
```

```erb
#app/views/shared/_flash_message.html.erb
<div class="container mx-auto px-4">
  <% flash.each do |message_type, message| %>
    <% if message_type == "success" %>
      <div class="flex w-full max-w-sm overflow-hidden bg-white rounded-lg shadow-md">
        <div class="flex items-center justify-center w-12 bg-emerald-500">
          <svg class="w-6 h-6 text-white fill-current" viewBox="0 0 40 40" xmlns="http://www.w3.org/2000/svg">
            <path d="M20 3.33331C10.8 3.33331 3.33337 10.8 3.33337 20C3.33337 29.2 10.8 36.6666 20 36.6666C29.2 36.6666 36.6667 29.2 36.6667 20C36.6667 10.8 29.2 3.33331 20 3.33331ZM16.6667 28.3333L8.33337 20L10.6834 17.65L16.6667 23.6166L29.3167 10.9666L31.6667 13.3333L16.6667 28.3333Z" />
          </svg>
        </div>

        <div class="px-4 py-2 -mx-3">
          <div class="mx-3">
            <span class="font-semibold text-emerald-500">Success</span>
            <p class="text-sm text-gray-600"><%= message %></p>
          </div>
        </div>
      </div>
    <% elsif message_type == "error" %>
      <div class="flex w-full max-w-sm overflow-hidden bg-white rounded-lg shadow-md">
        <div class="flex items-center justify-center w-12 bg-red-500">
          <svg class="w-6 h-6 text-white fill-current" viewBox="0 0 40 40" xmlns="http://www.w3.org/2000/svg">
            <path d="M20 3.36667C10.8167 3.36667 3.3667 10.8167 3.3667 20C3.3667 29.1833 10.8167 36.6333 20 36.6333C29.1834 36.6333 36.6334 29.1833 36.6334 20C36.6334 10.8167 29.1834 3.36667 20 3.36667ZM19.1334 33.3333V22.9H13.3334L21.6667 6.66667V17.1H27.25L19.1334 33.3333Z" />
          </svg>
        </div>

        <div class="px-4 py-2 -mx-3">
          <div class="mx-3">
            <span class="font-semibold text-red-500">Error</span>
            <p class="text-sm text-gray-600"><%= message %></p>
          </div>
        </div>
      </div>
    <% elsif message_type == "info" %>
      <div class="flex w-full max-w-sm overflow-hidden bg-white rounded-lg shadow-md">
        <div class="flex items-center justify-center w-12 bg-blue-500">
          <svg class="w-6 h-6 text-white fill-current" viewBox="0 0 40 40" xmlns="http://www.w3.org/2000/svg">
            <path d="M20 3.33331C10.8 3.33331 3.33337 10.8 3.33337 20C3.33337 29.2 10.8 36.6666 20 36.6666C29.2 36.6666 36.6667 29.2 36.6667 20C36.6667 10.8 29.2 3.33331 20 3.33331ZM21.6667 28.3333H18.3334V25H21.6667V28.3333ZM21.6667 21.6666H18.3334V11.6666H21.6667V21.6666Z" />
          </svg>
        </div>

        <div class="px-4 py-2 -mx-3">
          <div class="mx-3">
            <span class="font-semibold text-blue-500">Info</span>
            <p class="text-sm text-gray-600"><%= message %></p>
          </div>
        </div>
      </div>
    <% elsif message_type == "warning" %>
      <div class="flex w-full max-w-sm overflow-hidden bg-white rounded-lg shadow-md">
        <div class="flex items-center justify-center w-12 bg-yellow-400">
          <svg class="w-6 h-6 text-white fill-current" viewBox="0 0 40 40" xmlns="http://www.w3.org/2000/svg">
            <path d="M20 3.33331C10.8 3.33331 3.33337 10.8 3.33337 20C3.33337 29.2 10.8 36.6666 20 36.6666C29.2 36.6666 36.6667 29.2 36.6667 20C36.6667 10.8 29.2 3.33331 20 3.33331ZM21.6667 28.3333H18.3334V25H21.6667V28.3333ZM21.6667 21.6666H18.3334V11.6666H21.6667V21.6666Z" />
          </svg>
        </div>

        <div class="px-4 py-2 -mx-3">
          <div class="mx-3">
            <span class="font-semibold text-yellow-400">Warning</span>
            <p class="text-sm text-gray-600"><%= message %></p>
          </div>
        </div>
      </div>
    <% end %>
  <% end %>
</div>
```

利用方法

```erb
#app/views/layouts/application.html.erb
    <div class="container">
      <%= render "shared/flash_message" %>
      <%= yield %>
    </div>
```

```ruby
  def create
    @user = User.new(user_params)
    if @user.save
      redirect_to root_path, success: "ユーザー登録が完了しました"　#成功時
    else
      flash.now[:danger] = 'ユーザー登録に失敗しました' #失敗時
      render :new, status: :unprocessable_entity #HTTPステータスコード422
    end
  end
```

●status: :unprocessable_entity を指定する必要がある理由
render メソッドは、ビューをレンダリングし、クライアントにレスポンスを提供するための主要なメソッドですが、デフォルトでは 200 OK ステータスコードが返されます。そのため、バリデーションエラーが発生していたとしても、クライアント側は正常なリクエストと認識してしまうためエラーメッセージが表示されません。status: :unprocessable_entity と指定することで、HTTPステータスコード422が返され、クライアントにリクエストの処理ができなかったことを適切に伝えています。

### 
