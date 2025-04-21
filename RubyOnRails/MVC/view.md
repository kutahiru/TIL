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

### link_to

aタグを生成するヘルパーメソッド

```erb
link_to("profile link", edit_profile_path)
```

### link_toでHTMLを表示したい場合

第一引数ではなく、以下のように記載する必要がある。

```erb
<%= link_to("/likes/#{@post.id}/destroy", {method: "post"}) do %>
  <span class="fa fa-heart liked-btn"></span>
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
モデルオブジェクトの値がフォームのフィールドに自動入力されるv

- フォームの`action`属性には適切な値`action="/books"`が自動的に入力されます。書籍を更新する場合は`action="/books/42"`のようになります。
- フォームのフィールド名は`book[...]`でスコープ化されます。つまり、`params[:book]`はこれらのフィールドの値をすべて含むハッシュになります。
- 送信ボタンには、適切なテキスト値（この場合は「Create Book」）が自動的に入力されます。

```ruby
@book = Book.find(42)
# => #<Book id: 42, title: "Walden", author: "Henry David Thoreau">
```

```erb
<%= form_with model: @book do |form| %>
  <div>
    <%= form.label :title %>
    <%= form.text_field :title %>
  </div>
  <div>
    <%= form.label :author %>
    <%= form.text_field :author %>
  </div>
  <%= form.submit %>
<% end %>

#コントローラでアクセスする際はStrongParametersを使用する
```





