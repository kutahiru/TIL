# view

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

