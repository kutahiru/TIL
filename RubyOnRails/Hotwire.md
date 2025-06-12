# Hotwire

https://zenn.dev/shita1112/books/cat-hotwire-turbo/viewer/intro
https://qiita.com/jinta_02/items/7a4e14ba5973823551ad
Hotwireではfetchに対してJSONではなくHTMLをレスポンスする。
これによってレンダリングはサーバーサイドでのみ行うようにできる。
状態管理するのはサーバーサイドだけでいいし、モデルやバリデーションはサーバーサイドにだけ用意すればいい
ReactはJSONをレスポンスするためそこが異なる。

## Turbo

JavaScriptを書かずにSPAのようなインタラクティブなアプリケーションを作れる

### Turbo Drive

画面遷移を高速にしてくれるTurboの機能
リンク、フォームのリクエストをTurbo Driveがインターセプトして、fetchによる非同期リクエストに差し替える。
そしてレスポンスされたHTMLの`<body>`要素だけを抜き出して、現在のページの`<body>`要素を置換する。

結果、画面遷移しても今のページのCSS・JavaScriptをそのまま利用できるため、
CSS・JavaScriptを初期化してページに適用する処理をスキップできる。
これによって画面遷移が高速になる
コードの修正自体は不要

### Turbo Frames

Turbo Driveの部分置換版
`<turbo-frame>...</turbo-frame>`というHTMLタグのようなもので囲った箇所だけを置換する

```erb
<%= turbo_frame_tag "cats-list" do %>
  <div>置換したい箇所</div>
<% end %>
```

- 例として以下はページネーションを囲んだ場合の処理の流れ
  id="cats-list"を設定することで新旧の該当部分を入れ替えている

1. ページネーションのリンクをクリックした時に、Turboが通常のリクエストをインターセプトして、fetchを行う
2. サーバー側はリクエストヘッダーにTurbo-Frameが存在するとTurbo Frameリクエストだと判断
   Turbo Frameリクエストの場合、高速化のためにレイアウトテンプレート（application.html.erb）のレンダリングはせずに、メインテンプレート（index.html.erb）だけをレンダリングしてレスポンスする
3. クライアント側はレスポンス（index.html.erbのレンダリング結果）を受け取り<turbo-frame>のidが一致する
   <turbo-frame id="cats-list">部分（一覧部分）だけを置換

- URLが変更されない理由
  サーバでは新しいURLでの処理がリクエストされて処理されているが、
  クライアントへの返却では、一部のみが使用されるため。

#### 検索機能に対して使用する場合

```erb
# data-turbo-frame属性を指定する
<%= search_form_for @search, html: { data: { turbo_frame: "cats-list" } } do |f| %>
```

#### 編集リンククリック時の処理

一覧内で1行を選択してそのまま編集できる画面

turbo_frame_tagの引数にはcatオブジェクトを渡している。
turbo_frame_tagが内部でdom_idを利用してcatを"cat_1"のようなidに変換してくれる
_cat.html.erbはCatの数だけレンダリングされるので、<turbo-frame>のidがかぶらないようにオブジェクトを引数に渡す必要がある

```erb
# 1行毎のパーシャル
# app/views/cats/_cat.html.erb
# 全体をturbo_frame_tagで囲う
<%= turbo_frame_tag cat do %>
  <div class="row py-2 border-top">
    <div class="col-4 my-auto">
      <%= cat.name %>
    </div>
    <div class="col-4 my-auto">
      <%= cat.age %>
    </div>
    <div class="col-4">
      <div class="d-flex justify-content-end">
        <%= link_to "編集", edit_cat_path(cat), class: "btn btn-sm btn-outline-primary me-2" %>
        <%= button_to "削除", cat, method: :delete, class: "btn btn-sm btn-outline-danger" %>
      </div>
    </div>
  </div>
<% end %>
```

以下はedit.html.erb内からrenderしている_form.html.erb
注目ポイントはturbo_frame_tagの引数にcatを利用してる点
これでindex.html.erbの<turbo-frame id="cat_1">と
edit.html.erb（内でrenderしている_form.html.erb）の<turbo-frame id="cat_1">がマッチするようになる

つまり編集リンクをクリックすると、_cat.html.erbの<turbo-frame id="cat_1">部分が、_
form.html.erbの<turbo-frame id="cat_1">部分に置換される

編集ボタンを押すと、
edit.html.erb内の_form.html.erb内のturbo_frame_tag部分が返却される。
　※_form.html.erbにturbo_frame_tagが設定されているから、
これと、一覧画面側のturbo_frame_tagが入れ替わる

```erb
# app/views/cats/_form.html.erb
# 全体をturbo_frame_tagで囲う
# 編集リンクをクリックすると、_cat.html.erbの<turbo-frame>部分がこの部分に置換される
<%= turbo_frame_tag cat do %>
  <%= bootstrap_form_with(model: cat) do |form| %>
    <div class="row py-2 border-top">
      <div class="col-4">

        <%# _cat.html.erbの見た目に合うように、フォームの見た目を修正する %>
        <%# オプションはbootstrap_formのもので、詳細は以下の通り %>
        <%# skip_label: ラベルは不要 %>
        <%# label_as_placeholder: ラベルをプレースホルダーとして使う %>
        <%# wrapper: <div>ラッパーは不要 %>
        <%# control_class: コントロールのclass属性を指定 %>
        <%= form.text_field :name,
                            skip_label: true,
                            label_as_placeholder: true,
                            wrapper: false,
                            control_class: "form-control form-control-sm"
        %>
      </div>
      <div class="col-4">
        <%= form.number_field :age,
                              skip_label: true,
                              label_as_placeholder: true,
                              wrapper: false,
                              control_class: "form-control form-control-sm"
        %>
      </div>

      <div class="col-4">
        <div class="d-flex justify-content-end">
          <%= form.primary class: "btn btn-primary btn-sm me-2" %>
        </div>
      </div>
    </div>
  <% end %>
<% end %>
```

#### 更新失敗時の処理

```ruby
app/controllers/cats_controller.rb
  def update
    if @cat.update(cat_params)
      redirect_to cats_path, notice: "ねこを更新しました。"
    else
      # バリデーションエラー時にはedit.html.erbをrenderする
      render :edit, status: :unprocessable_entity
    end
  end
```

#### 更新成功時の処理

```ruby
# app/controllers/cats_controller.rb
  def update
    if @cat.update(cat_params)
      # 成功時は一覧ページへのリダイレクト
      redirect_to cats_path, notice: "ねこを更新しました。"
    else
      render :edit, status: :unprocessable_entity
    end
  end
```

#### キャンセル時の動作

```erb
     <div class="col-4">
       <div class="d-flex justify-content-end">
         <%= form.primary class: "btn btn-primary btn-sm me-2" %>
         <%= link_to "キャンセル", cat, class: "btn btn-sm btn-outline-secondary" %>
       </div>
     </div>
   </div>
```

### Turbo Streams

複数箇所のHTML要素を同時に更新できる

更新と同時のフラッシュメッセージの表示

以下は成功時のリダイレクトを削除しているが、この場合実際にrenderされるのは、
「app/views/cats/update.turbo_stream.erb」になる。
　※Turbo Frame(turbo-frame内)からのリクエストだから。

```ruby
# app/controllers/cats_controller.rb
# PATCH/PUT /cats/1
 def update
   if @cat.update(cat_params)
     # リダイレクトを削除（リダイレクトがないと暗黙的に`render`が実行される）
     #redirect_to @cat, notice: "ねこを更新しました。"
   else
     render :edit, status: :unprocessable_entity
   end
 end
```

```erb
# app/views/cats/update.turbo_stream.erb
<%= turbo_stream.replace @cat %>
```

#### turbo_stream.replaceメソッド

turbo-railsが用意したビューヘルパー
内部的にはパーシャルをrenderする

```erb
<%# renderを省略する場合 %>
<%= turbo_stream.replace @cat %>

<%# renderを省略しない場合 %>
<%# partialとlocalsは@catから推測できるため、通常は省略される %>
<%= turbo_stream.replace @cat do %>
  <%= render partial: "cats/cat",
             locals: { cat: @cat } %>
<% end %>
```

```erb
# ↑のようにビューヘルパーを使わない場合は以下。普通はビューヘルパー使うけど。
<turbo-stream action="replace" target="cat_1">
  <template>
    コンテンツ
  </template>
</turbo-stream>

# targetで操作する対象のHTML要素のIDを指定
# actionでtargetの操作方法を指定

# 追加
- append: targetの末尾に追加
- prepend: targetの先頭に追加
- before: targetの前に追加
- after: targetの後に追加

# 更新
- replace: targetを更新（target要素も含めて更新）
- update: targetを更新（target要素のコンテンツだけ更新）

# 削除
- remove: targetを削除
```

結果的に`#cat_1`（id属性が`cat_1`の要素）を`<template>...<template>`のコンテンツで`replace`（置換する）する
という処理

#### 更新処理の流れ

1. 「更新する」ボタンをクリックした時に、Turboがフォームからのリクエストをインターセプトする
   **フォーム**からのリクエストの場合、Acceptリクエストヘッダーに`turbo_stream`フォーマットを追加してfetchする。
   具体的には`Accept: text/vnd.turbo-stream.html, text/html, application/xhtml+xml`となる
2. Acceptヘッダーの`text/vnd.turbo-stream.html`により、フォーマットが`turbo_stream`になる。
   そのため`render`で`update.turbo_stream.erb`ビューを利用する
3. レスポンスされた`<turbo-stream action="replace" target="cat_1">...</turbo-stream>`をTurboが処理して、
   `#cat_1`を`<template>`要素のコンテンツで`replace`

フォーマットがTurbo Streamになるのは、フォームからGET以外のHTTPメソッドが利用された時だけ
リンクやフォームからGETする場合にはTurbo Streamにはならない
update/create/destroyでTurbo Streamが使えるということ。

#### Flash部分

```erb
# app/views/layouts/application.html.erb
# flashにIDを設定
     <div class="col my-4">
       <div id="flash">
         <%= render "flash" %>
       </div>
       <%= yield %>
     </div>
```

```ruby
# app/controllers/cats_controller.rb
# Flashは通常だとリダイレクト時に使うのでflash.noticeを使って設定する
# Turbo Streamsでは今回のリクエストに対してFlashを設定したいため、flash.now.noticeを使用
# flash.noticeだと、次のページ遷移(redirect)でFlashが表示されてしまう。
def update
   if @cat.update(cat_params)
     flash.now.notice = "ねこを更新しました。"
   else
     render :edit, status: :unprocessable_entity
 end
```

```erb
# app/views/cats/update.turbo_stream.erb
 <%= turbo_stream.replace @cat %>
 <%= turbo_stream.update "flash", partial: "flash" %> #flashを置き換える処理
```

```erb
<%# renderを省略する場合 %>
<%= turbo_stream.update "flash", partial: "flash" %>

<%# renderを省略しない場合 %>
<%= turbo_stream.update "flash" %>
  <%= render partial: "flash" %>
<% end%>
```

```erb
# ↑のようにビューヘルパーを使わない場合は以下。普通はビューヘルパー使うけど。
<turbo-stream action="update" target="flash">
  <template>

    <!-- `render partial: "flash"`の結果 -->
    <div class="alert alert-primary alert-dismissible">
      <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
      ねこを更新しました。
    </div>

  </template>
</turbo-stream>
```

```ruby
# 実際はflashメッセージを複数個所で使用されるため、ビューヘルパーとして切り出しておく
# app/helpers/application_helper.rb
  def turbo_stream_flash
    turbo_stream.update "flash", partial: "flash"
  end
```

```erb
# app/views/cats/update.turbo_stream.erb
 <%= turbo_stream.replace @cat %>
 <%= turbo_stream_flash %>　#ビューヘルパーを利用
```

#### 登録のTurbo Streams化

登録ボタンをクリックした際に、
Turbo Frameリクエストで`new.html.erb`（内でrenderされる`_form.html.erb`）を取得して、
Cat一覧の一番上に表示させる。
登録リンクに`data-turbo-frame`を設定して、
一覧の上に取得したTurbo Frameを置換するための`<turbo-frame>`を置く

```erb
# app/views/cats/index.html.erb
       <div class="col-4 d-flex justify-content-end">
         <%= link_to icon_with_text("plus-circle", "登録"),
                     new_cat_path,
                     class: "btn btn-outline-primary",
                     data: { turbo_frame: "new_cat" }
         %>
       </div>
     </div>

     <%# 登録リンククリック時に、cats#newの新規登録フォーム部分をここに置換する %>
     <%= turbo_frame_tag "new_cat" %>
     <div id="cats">
       <%= render @cats %>
     </div>

     <div class="d-flex justify-content-end mt-3">
       <%= paginate @cats %>
```

```ruby
# app/controllers/cats_controller.rb
# flash.now.noticeを使う
# 暗黙的にcreate.turbo_stream.erbのrenderが実行される
   @cat = Cat.new(cat_params)

   if @cat.save
     flash.now.notice = "ねこを登録しました。"
   else
     render :new, status: :unprocessable_entity
   end
```

```erb
# create.turbo_stream.erb
<%# `#cats`の先頭に登録されたcatを追加する %>
<%= turbo_stream.prepend "cats", @cat %>

<%# 登録の入力フォームの中身を空にする %>
<%= turbo_stream.update Cat.new, "" %>

<%# Flashを表示 %>
<%= turbo_stream_flash %>
```

#### 削除のTurbo Streams化

```ruby
# app/controllers/cats_controller.rb
# DELETE /cats/1
# 暗黙的にdestroy.turbo_stream.erbのrenderが実行される
 def destroy
   @cat.destroy
   flash.now.notice = "ねこを削除しました。"
 end
```

```erb
# app/views/cats/destroy.turbo_stream.erb
<%# Cat詳細（`#cat_1`となる要素）を削除する %>
<%= turbo_stream.remove @cat %>

<%# Flashを表示 %>
<%= turbo_stream_flash %>
```

```erb
   <div class="col-4">
     <div class="d-flex justify-content-end">
       <%= link_to "編集", edit_cat_path(cat), class: "btn btn-sm btn-outline-primary me-2" %>
       <%= button_to "削除", cat, method: :delete, class: "btn btn-sm btn-outline-danger", form: { data: { turbo_confirm: "本当に削除しますか？" } } %>
     </div>
   </div>
```

## Stimulus

Turboと相性が良いJavaScriptのライブラリ
カオスになりがちなJavaScriptにレールを敷く役割
JavaScriptを書かずにサーバーサイドレンダリング + fetchでHTML要素を更新できる
Turboでできないことができる。

- コントローラー: `data-controller`属性の値とJavaScriptのファイル名が紐づく
- ターゲット: `data-<コントローラー>-target`属性の値と`static targets = []`で定義された値が紐づく
- アクション: `data-action`属性の値とメソッドが紐づく

```js
# src/controllers/hello_controller.js
import { Controller } from "@hotwired/stimulus"

// コントローラークラスを定義する
export default class extends Controller {
  // ターゲット（操作対象のDOM）のプロパティを作成
  static targets = [ "name" ]

  // アクション（イベントに紐づく処理）を定義する
  greet() {
    // xxxxTargetでターゲットとなるDOMにアクセスできる
    // ターゲット（今回であれば<input>）のvalueをログ吐き
    console.log(this.nameTarget.value)
  }
}
```

```erb
<!-- コントローラーをdiv要素にアタッチする -->
<div data-controller="hello">
  <!-- <input>をターゲット（操作対象のDOM）をとして指定する -->
  <input data-hello-target="name" type="text">

  <!-- アクションとイベントを紐付ける -->
  <!-- click時にhelloコントローラーのgreetアクションを実行する -->
  <button data-action="click->hello#greet">Greet</button>
</div>
```

### Stimulusのdebugモードを有効化

```js
# app/javascript/controllers/application.js
 application.debug = true
```

### インスタントサーチの実装

Rails7から`stimulus-rails`というgemがデフォルトになった
RailsからStimulusを使えるようにしてくれる機能と、
Stimulusコントローラーファイルを作成するためのジェネレーター機能の2つの機能

```bash
# Stimulusコントローラーの作成
rails g stimulus form
```

```js
# app/javascript/controllers/form_controller.js
import { Controller } from "@hotwired/stimulus"

// Connects to data-controller="form"
export default class extends Controller {
  connect() {
  }
}
```

```js
# app/javascript/controllers/index.js
# Stimulusコントローラーをアプリケーションに登録するためのコード
 import FormController from "./form_controller.js"
 application.register("form", FormController)
```

```js
# app/javascript/controllers/form_controller.js

import { Controller } from "@hotwired/stimulus"

// Connects to data-controller="form"
export default class extends Controller {
  // コントローラーに紐づく要素（=フォーム）をsubmitするアクション
  submit() {
    this.element.requestSubmit()
  }
}
```

```erb
# app/views/cats/index.html.erb
   <%= search_form_for(
     @search,
     html: {
       data: {
         turbo_frame: "cats-list",
         controller: "form",
         action: "input->form#submit"
       }
     }) do |f| %>
```

`form`コントローラーが`<form>`要素にアタッチされて、
`input`時（テキスト入力時）に`form`コントローラーの`submit`アクションを実行されるようになる

#### Debounceの実装

複数のキー入力をまとめて1つの入力とみなすため

```js
# app/javascript/controllers/form_controller.js

import { Controller } from "@hotwired/stimulus"

// Connects to data-controller="form"
export default class extends Controller {
 submit() {
   // セットされているTimeoutをクリアする
   clearTimeout(this.timeout)

   // Timeoutをセットする
   // 200ms後にリクエストを実行する
   // 連続で実行されるとTimeoutはクリアされるため、最後の処理だけしか実行されない
   this.timeout = setTimeout(() => {
     this.element.requestSubmit()
   }, 200)
 }
}
```

### 編集・登録のモーダル化

```bash
# Stimulusコントローラーの作成
rails g stimulus modal
```

```js
# app/javascript/controllers/modal_controller.js

import { Controller } from "@hotwired/stimulus"
// BootstrapのModalをimport
import { Modal } from "bootstrap"

export default class extends Controller {
  // `connect()`はStimulusのライフサイクルコールバックの1つ
  // コントローラーがHTML要素にアタッチされた時（=HTML要素が画面に表示された時）に実行される
  connect() {
    // モーダル生成
    this.modal = new Modal(this.element)

    // モーダルを表示する
    this.modal.show()
  }

  // アクション定義
  // 保存成功時にモーダルを閉じる
  close(event) {
    // event.detail.successは、レスポンスが成功ならtrueを返す
    // バリデーションエラー時はモーダルを閉じたくないので、成功時のみ閉じる
    if (event.detail.success) {
      // モーダルを閉じる
      this.modal.hide()
    }
  }
}
```

```erb
# app/views/application/_modal.html.erb
<%# `"modal"`という<turbo-frame>で囲う %>
<%= turbo_frame_tag "modal" do %>
  <%# modalコントローラーをアタッチする %>
  <div class="modal fade" data-controller="modal">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title"><%= title%></h5>
          <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
        </div>
        <div class="modal-body">
          <%= yield %>
        </div>
      </div>
    </div>
  </div>
<% end %>
```

```erb
# app/views/cats/_form.html.erb
<%= turbo_frame_tag cat do %>
  <%# レイアウトをモーダル用に整える %>
  <%# turbo:submit-end（Turboのsubmit後）イベント発火時に、modal#close（モーダルを閉じる）を実行する %>
  <%= bootstrap_form_with(model: cat, data: { action: "turbo:submit-end->modal#close" }) do |form| %>
    <%= form.text_field :name %>
    <%= form.text_field :age %>
    <%= form.primary %>
  <% end%>
<% end %>
```

```erb
# app/views/cats/edit.html.erb
<%= render "modal", title: "編集" do %>
  <%= render "form", cat: @cat %>
<% end %>
```

```erb
# app/views/cats/new.html.erb
<%= render "modal", title: "登録" do %>
  <%= render "form", cat: @cat %>
<% end %>
```

```erb
# app/views/layouts/application.html.erb
       </div>
     </div>
   </div>

   <%# 編集・登録リンクのTurbo Framesのターゲット %>
   <%# 編集・登録リンクをクリックすると、ここにサーバーから取得したモーダルが置換される %>
   <%# モーダルの置き場所はどこでもいいので、とりあえず<body>の一番下に置いておく %>
   <%= turbo_frame_tag "modal" %>
 </body>
/html>
```

```erb
# app/views/cats/index.html.erb
         <%= link_to icon_with_text("plus-circle", "登録"),
                     new_cat_path,
                     class: "btn btn-outline-primary",
                     data: { turbo_frame: "modal" }
         %>
       </div>
     </div>

     <div id="cats">
       <%= render @cats %>
     </div>
```

