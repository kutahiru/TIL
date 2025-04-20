# view

### パーシャル(部分テンプレート)

ビュー画面を共通化するために使用
呼び出しは「render ファイル名（ディレクトリを指定）」
引数はパーシャル内でローカル変数として参照可能
ファイル名は_で始める。

```erb
<%= render user %> #引数なし
<%= render partial: "product", locals: { my_product: @product } %> #引数あり
```

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

## form_with

フォームを構築するためのヘルパーメソッド
モデルの情報を元に送信URLやフィールドの内容を埋めたりすることができる。
https://railsguides.jp/form_helpers.html

### シンプルなフォーム

```erb
<%= form_with do |form| %>
  Form contents
<% end %>
```

### フォームに設定可能なヘルパーメソッド

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

### モデルオブジェクトを指定したフォーム

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

