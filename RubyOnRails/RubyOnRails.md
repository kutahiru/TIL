# Ruby on Rails

- [Ruby on Rails](#ruby-on-rails)
  - [疑問](#疑問)
  - [始め方](#始め方)
  - [Rubyで大事な点](#rubyで大事な点)
  - [ページ表示の仕組み](#ページ表示の仕組み)
      - [まとめ](#まとめ)
  - [コントローラーと対応するViewの作成](#コントローラーと対応するviewの作成)
  - [DBにテーブル作成](#dbにテーブル作成)
    - [既存テーブルの定義変更](#既存テーブルの定義変更)
  - [シンボルについて](#シンボルについて)
  - [フォームに入力されたデータを送信する。](#フォームに入力されたデータを送信する)
  - [フォームに初期値を表示する場合](#フォームに初期値を表示する場合)
  - [特に理由がない場合は絶対パスを使用する。](#特に理由がない場合は絶対パスを使用する)
  - [link\_toでHTMLを表示したい場合](#link_toでhtmlを表示したい場合)
  - [getとpostの使い分け](#getとpostの使い分け)
  - [バリデーションはモデルで設定する](#バリデーションはモデルで設定する)
  - [バリデーションエラー](#バリデーションエラー)
  - [確定後に表示するメッセージなど](#確定後に表示するメッセージなど)
  - [アクションを経由せずにViewを表示したい場合](#アクションを経由せずにviewを表示したい場合)
  - [ログインの保持について](#ログインの保持について)
  - [共通のHTML](#共通のhtml)
  - [全アクションで共通する処理](#全アクションで共通する処理)
  - [ログインしていない場合のアクセス制限](#ログインしていない場合のアクセス制限)
  - [ログインしていない場合のアクション制限](#ログインしていない場合のアクション制限)
  - [コントローラで使用する処理をモデルにメソッドで用意する](#コントローラで使用する処理をモデルにメソッドで用意する)
  - [Gemのインストール](#gemのインストール)
    - [bcrypt ハッシュ化する](#bcrypt-ハッシュ化する)
    - [has\_secure\_passwordのメタプログラミングについて](#has_secure_passwordのメタプログラミングについて)

## 疑問
- C#みたいに定義に移動みたいな機能で呼び出し先や元に移動できたりする機能がないか。

## 始め方
```rb
rails new アプリ名
rails server
```

## Rubyで大事な点
- メソッド呼び出し時の()を省略できる。　しなくて良い派もいるみたい。
- 真偽値のメソッドには名前の最後に?を付けることが多い。
  is_successではなく、success?とする。

## ページ表示の仕組み
ルーティング→コントローラ→ビューの順で処理が行われ、Viewを返す
ブラウザでURLを入力すると、ルーティングがURLを見て、適切なコントローラのアクションを呼び出す
コントローラーを経由してViewをブラウザに返している。
コントローラー内のアクションはブラウザに返すViewをviewsフォルダの中から見つけ出す役割
コントローラーと同じ名前のViewフォルダからアクションと同じ名前のHTMLファイルを探してブラウザに返す

ルーティングは、送信されたURLに対して「どのコントローラの、どのアクション」で
処理するかを決める「対応表」のこと
```rb
#config/routes.rb
Rails.application.routes.draw do
  get "home/top" => "home#top"
end
```

「home/top」というURLにアクセスされたら、
homeコントローラのtopアクションを実行するという意味
 ```rb
 #controlloers/home_controller.rb
 class HomeController < ApplicationController
   def top #topアクションが実行される
   end
 end
 ```
 
コントローラーがアクションと同名のビュー(views/home/top.html.erb)を探す
・views/home/top.html.erb

#### まとめ
ブラウザで http://localhost:3000/home/top にアクセス
ルーティング(routes.rb)がURLを解析し、home#topを呼び出す
HomeControllerのtopアクションが実行される
アクション内では必要に応じてモデルからデータを取得
コントローラーがアクションと同名のビュー(views/home/top.html.erb)を探す
ビューファイル内のERB(埋め込みRuby)が実行され、HTMLが生成される
生成されたHTMLがブラウザに返される


## コントローラーと対応するViewの作成
```cmd
rails g controller posts index
```
- ルーティングが追加される。
```rb
#config/routes.rb
get "posts/index" => "posts#index"
```

- indexアクションを持つpostsコントローラーが作成される。
```rb
#app/controllers/posts_controller.rb
class PostsController < ApplicationController
  def index
  end
end
```

- ビューファイルが作成される。
  app/views/posts/index.html.erb
　　
- CSSが作成される。
  app/assets/stylesheets/posts.scss


## DBにテーブル作成
```cmd
rails g model User name:string email:string
rails db:migrate
```

DB操作に使用するモデルも生成される。
app/models/User.rb
 ※ApplicationRecordを継承している。

INSERT時はモデルのインスタンスを作成後、Save;

- DBからの取得処理
  indexメソッド内に以下のように取得処理を記載できる。
  @posts = Post.all

### 既存テーブルの定義変更
1. マイグレーションファイルを作成する  
db/migrateフォルダに、作成日時_add_image_name_to_usersでファイルが作成される。
```cmd
  rails g migration add_image_name_to_users
```

2. マイグレーションファイルを編集する
```rb
  def change
    add_column(:users, :image_name, :string)
    remove_column(:users, :password, :string)
  end
```

3. 実行する
```cmd
  rails db:migrate
```

## シンボルについて
- シンボルはRubyの特殊なデータ型で、名前の前にコロンを付けて表します。
シンボルはRubyでは不変のオブジェクトで、同じシンボルは常に同じオブジェクトIDを持つ
```rb
:name    # これはシンボル
:desc    # 「降順」を表すシンボル
:created_at  # カラム名を表すシンボル
```

- ハッシュのキーと値を区切る（created_at: ）
```rb
{ key: value }  # 省略形の書き方
{ :key => value }  # 従来の書き方
```

- 両方が使われている例
```rb
order(created_at: :desc)
```
created_at: の部分 - ハッシュのキーを表すコロン（「created_atカラムに対して」という意味）
:desc の部分 - シンボルを表すコロン（「降順」という値を表す）

## フォームに入力されたデータを送信する。
```rb
form_tag(送信先のURL) do
end
```

HTMLで入力された情報を取得するためには、
該当のタグにname属性を設定する。
アクション側でparams[:name属性]で取得可能となる。

- <%= %>で実装する必要がある理由
form_tagメソッドは、
HTML上にフォーム要素（\<form>タグ）を生成するために使われます。
このメソッドは実行されると、
以下のようなHTMLを返します。
form_tagが生成したHTML（開始フォームタグ）を出力するために=が必要なのです。

```html
<form action="/posts/create" method="post">...</form>
```


## フォームに初期値を表示する場合
```rb
<input name="name" value="<%= @user.name %>">
```

## 特に理由がない場合は絶対パスを使用する。
/post/index
理由があれば相対パスを使用する。
post/index

## link_toでHTMLを表示したい場合
第一引数ではなく、以下のように記載する必要がある。

```rb
<%= link_to("/likes/#{@post.id}/destroy", {method: "post"}) do %>
  <span class="fa fa-heart liked-btn"></span>
<% end %>   
```

## getとpostの使い分け
- post
  データベースを変更する場合
  セッションを変更する場合

・link_toの仕様
　link_toの第三引数に「{method: "post"}」を設定しないとgetを参照してしまう。
```rb
　<%= link_to("削除", "/posts/#{@post.id}/edit", {method: "post"}) %>
```

## バリデーションはモデルで設定する
```rb
validates(:content, {presence: true})
validates(:content, {presence: true, length: {maximum: 140}})
```

validates メソッドは、指定された属性（フィールド）に対して、様々な検証ルールを適用するためのメタプログラミング的な仕組み
検証の度にモデルが実行されているわけではなく、
バリデーターのインスタンスを作成して登録する処理と思えば良い。
モデルのライフサイクル（保存時など）で実際の検証が実行される。

## バリデーションエラー
saveメソッドを呼び出した際にバリデーションに失敗すると、
Railsでは自動的にエラーメッセージが生成されるようになっています。
@post.errors.full_messagesの中に、エラー内容が配列で入ります。

```rb
@post.erros.full_messages do |message|
<%= message %>
```

## 確定後に表示するメッセージなど
```rb
def create
  @post = Post.new(content: params[:content])
  if @post.save
  flash[:notice] = "投稿を作成しました"
  redirect_to("/posts/index")
  else
    render("posts/new")
  end
end
```

```rb
<% if flash[:notice] %>
  <div class="flash">
    <%= flash[:notice] %>
  </div>
```
  
## アクションを経由せずにViewを表示したい場合
renderを使用する。
redirect_toはURLを指定して「redirect_to(URL)」のように書きます。
一方renderはビューファイルを指定して「render("フォルダ名/ファイル名")」とします。

## ログインの保持について
```rb
session[:user_id]
```
- ログイン時、ブラウザに保存される。
- 以降のアクセス、ブラウザから送信される。

この値を元にヘッダ等の画面表示を切り替える。

## 共通のHTML
views/layouts/application.html.erb
top.html.erbなどの各ビューファイルは、この<%= yield %>の部分に代入され、
application.html.erbの一部としてブラウザに表示されていました。
この仕組みによって、ヘッダーなどの共通のレイアウトを1つにまとめることができます。

## 全アクションで共通する処理
全てのコントローラで共通する処理はapplicationコントローラにまとめる

```rb
before_action :set_current_user

def set_current_user
  @current_user = User.find_by(id: session[:user_id])
end
```

## ログインしていない場合のアクセス制限
onlyを用いて各コントローラでbefore_actionを使うことで、指定したアクションでのみそのメソッドを実行することができる
条件に合致する場合にログインページにリダイレクトすることでアクセス制限となる。

```rb
#applicationコントローラ
def authenticate_user
  if @current_user == nil
    flash[:notice] = "ログインが必要です"
    redirect_to("/login")
  end
end
```

```rb
#usersコントローラ
before_action :authenticate_user, {only:[:index, :show, :edit, :update]}
```


## ログインしていない場合のアクション制限
直接URLにアクセスしてアクションを行う場合の対策

```rb
before_action :ensure_correct_user, {only [:edit, :update]}

def ensure_correct_user
  if @current_user.id != params[:id].to_i
    flash[:notice] = "権限がありません"
    redirect_to("/posts/index")
  end
end
```

## コントローラで使用する処理をモデルにメソッドで用意する
Railsではモデル内にインスタンスメソッドを定義することができる。
Postモデル内で定義したインスタンスメソッドは
コントローラーでpostインスタンスに対して用いることができる。

## Gemのインストール
Gemfile
gem 'gemの名前'

```cmd
bundle install
```

### bcrypt ハッシュ化する
```rb
gem 'bcrypt'
```

モデルに「has_secure_password」を追加する。
```rb
class User < ApplicationRecord
  has_secure_password
end
```

内部的には以下の処理と同じ
- passwordとpassword_confirmationという仮想属性（attr_accessor）を追加
- passwordに値が設定されると、それをハッシュ化してpassword_digestカラムに保存するコールバックを追加
- 新規レコード作成時にpasswordの存在を確認(バリデーションの追加)
- password_confirmationが設定されている場合、passwordとの一致を確認
- ユーザー認証のためのauthenticateメソッドを追加

※password_digestをテーブルに用意する必要はある。

### has_secure_passwordのメタプログラミングについて
```rb
# 擬似コード
def self.has_secure_password(options = {})
  # クラスにモジュールを追加
  include InstanceMethodsOnActivation
  
  # 動的にメソッドを定義
  define_method(:authenticate) do |unencrypted_password|
    # 認証ロジック
  end
  
  define_method(:password=) do |unencrypted_password|
    # パスワード設定ロジック
  end
  
  # バリデーションを追加
  validates_presence_of :password, if: -> { password_digest.blank? }
  # その他の設定...
end
```