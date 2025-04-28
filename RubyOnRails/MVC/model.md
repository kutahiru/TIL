# model

### ActiveRecordのコールバック機能

```
1.before_validation: バリデーションが実行される前に呼び出されます。
2.after_validation: バリデーションが実行された後に呼び出されます。
3.before_save: レコードが保存される前に呼び出されます。
4.after_save: レコードが保存された後に呼び出されます。
5.before_destroy: レコードが削除される前に呼び出されます。
6.after_destroy: レコードが削除された後に呼び出されます。
```

```ruby
class User < ApplicationRecord
  before_save :capitalize_name

  private

  def capitalize_name
    self.name = name.capitalize
  end
end
```

### ActiveRecordのメソッド

```ruby
user = User.new(name: "Runteq", email: "runteq@example.com")

user = User.new(name: "Runteq", email: "runteq@example.com")
user.save

user = User.create(name: "Runteq", email: "runteq@example.com")

users = User.all

# すべてのユーザーを取得し、名前の昇順に並べる
sorted_users = User.all.order(:name)

user = User.find(1)

user = User.find_by(name: "Runteq", email: "runteq@example.com")

users = User.where(name: "Runteq")

user = User.find(1) # IDが1のユーザーを取得
user.update(name: "Runtekun") # 名前を"Runtekun"に更新

user = User.find(1)  # IDが1のユーザーを取得
user.destroy # ユーザーを削除

users = User.order(:name) # 名前の昇順でユーザーを取得
users_desc = User.order(name: :desc)  # 名前の降順でユーザーを取得

# 年齢が18歳以上のユーザーの数をカウント
adult_users_count = User.where("age >= ?", 18).count
```

### バリデーションはモデルで設定する

```ruby
validates(:content, {presence: true})
validates(:content, {presence: true, length: {maximum: 140}})
numericality #数値
uniqueness: true #重複チェック
```

validatesメソッドは可変長引数、第2引数以降はハッシュ形式の検証オプション
検証の度にモデルが実行されているわけではなく、
バリデーターのインスタンスを作成して登録する処理と思えば良い。
モデルのライフサイクル（保存時など）で実際の検証が実行される。

### バリデーションエラー

エラー表示ロジックは共通処理のviewに記載するのが良い。
app/views/shared/_error_messages.html.erb
saveメソッドを呼び出した際にバリデーションに失敗すると、
Railsでは自動的にエラーメッセージが生成されるようになっています。
@post.errors.full_messagesの中に、エラー内容が配列で入ります。

```rb
@post.erros.full_messages do |message|
<%= message %>
```

### scopeの定義

利用頻度の高い検索条件に名前をつけることができる。

```ruby
class Book < ApplilactionRecord
  scope :costly, -> { where("price > ?", 3000)}
  scope :written, ->(theme){ where("name like ?", "%#{theme}")} #引数を受け取るスコープ
end

Book.costly #実行時
Book.costly.writtrn_about("java") #3000円以上で末尾がjavaの本を抽出
```

#### 注意点

該当するレコードが存在しない場合に0の配列ではなくnilを返却するような関数は使用しない。
不正な結果になるため。

```ruby
class Book < ApplilactionRecord
  scope :find_price, ->(price){ find_by(price: price)} #不正な実装
  scope :with_price, ->(price){ where(price: price) } #あるべき実装
end

Book.find_price(10000) #条件が無視され全件が抽出される
Book.with_price(10000) #正常
```

### ActiceRecord::Enum

DBに区分を持つ方法

```ruby
class Book < Application
  enum sales_status: {
    reservation: 0 #予約受付
    now_on_sales: 1 #発売中
    end_of_print: 2 #販売終了
  }
```

## アソシエーション

Railsにおけるアソシエーション（Association）は、ActiveRecordモデル間の関連付けを表現するための機能
データベースのテーブル間の関連性を表現し、異なるモデル間でのデータの取得や操作を容易にする。

1. 一対多の関係（One-to-Many Association）: あるモデルが他のモデルに対して1つ以上の関連を持つ場合に使用されます。例えば、1つのブログ記事には複数のコメントがある場合などです。このような場合、ブログ記事モデルはコメントモデルとの一対多の関係を定義します。

2. 多対多の関係（Many-to-Many Association）: 2つのモデルが相互に複数の関連を持つ場合に使用されます。例えば、ユーザーとロール（権限）の関係があり、1人のユーザーが複数のロールを持ち、1つのロールには複数のユーザーが属する場合などです。このような場合、中間テーブル（ジョインテーブル）を介して関連を定義します。

3. 一対一の関係（One-to-One Association）: あるモデルが他のモデルと1対1の関係を持つ場合に使用されます。例えば、ユーザープロフィールや設定情報を別のモデルで管理する場合などです。このような場合、各モデル間での一対一の関連を定義します。

### アソシエーションの種類

- belongs_to :単数形
  1つのモデルが他のモデルに属する場合に使用します。例えば、掲示板がユーザーに属する場合、Boardモデルに対してUserモデルを belongs_to :user と定義します。これにより、Boardモデルのインスタンスから投稿者であるユーザー情報を簡単に取得できます。
- has_one：
  1つのモデルが他のモデルと1対1で関連付けられる場合に使用します。例えば、ユーザーが1つのプロフィールを持つ場合、Userモデルに対してProfileモデルをhas_one :profile と定義します。これにより、Userモデルのインスタンスから関連づいたプロフィール情報を簡単に取得できます。
- has_many :複数形
  1つのモデルが複数の他のモデルと関連付けられる場合に使用します。例えば、ユーザーが複数の掲示板を持つ場合、Userモデルに対してBoardモデルを has_many :boards と定義します。これにより、Userモデルのインスタンスからそのユーザーが投稿した複数の掲示板情報を簡単に取得できます。
- has_many :bookmark_boards, through: :bookmarks, source: :board

  has_many :関連名, through: :中間テーブル名, source: :最終的に参照したいテーブル名の単数形
  中間モデルを介して多対多の関係を設定。
  Userが複数のbookmarkを持っていて、各bookmarkはboardに関連付けられている
  **これはユーザーがブックマークした掲示板を扱いたいために設定している**

  ```ruby
  class User < ApplicationRecord
    has_many :bookmarks, dependent: :destroy
    has_many :bookmark_boards, through: :bookmarks, source: :board
  end
  
  #生成されるメソッド
  user.bookmark_boards #ユーザーがブックマークした掲示板の一覧
  ```

  **実装時の考え方**

  Userモデルのhas_many :bookmarksにより、ユーザーからブックマークへのアクセスが可能
  Bookmarkモデルのbelongs_to :boardにより、ブックマークから掲示板へのアクセスが可能
  Userモデルのhas_many :bookmark_boards, through: :bookmarks, source: :boardにより、上記2つの関連付けを組み合わせて、ユーザーから直接掲示板へのアクセスが可能になるあるユーザーのブックマークした掲示板の一覧を参照可能なページを作成する。

  

  **実装時の考え方2**
  あるユーザーなので、userモデルに定義する必要がある。
  まずはユーザーのブックマーク一覧を取得するので、bookmarksの参照が必要。
  これだと、掲示板のidのみで中身を取得できないので、bookmarksを元にboardsの参照が必要
  ということは、
  through: :bookmarks　#中間テーブル(厳密にはuserモデルで定義しているアソシエーション名)
  source: :board　#最終的に参照するモデルの単数形(厳密にはbookmarkモデルで定義しているアソシエーション名)
  has_many :bookmarks_board
  命名規約的におかしいので、単数_複数の形にする
  has_many :bookmark_boards
- has_one :through：中間モデルを介して一対一の関係を設定。例：SupplierがAccountを通じてAccountHistoryと関係する。
- has_and_belongs_to_many：中間モデルなしで多対多の関係を設定。例：AssemblyとPartが互いに多対多の関係を持つ。

belongs_toとhas_oneの選択は外部キーの位置によって決まり、has_many :throughとhas_and_belongs_to_manyの選択は中間モデルの必要性によって決まります。

### 定義

```ruby
class Board < ApplicationRecord
  ... 省略 ...
  belongs_to :user
end

class User < ApplicationRecord
  ... 省略 ...
  has_many :boards, dependent: :destroy #destroy 時に関連づけられたモデルに対して destroy が実行される
end
```

### 具体的な利点

- N+1問題の対応

  ```ruby
  posts = Post.includes(:user).all  # ユーザーデータも同時に取得
  posts.each do |post|
    puts post.user.name  # キャッシュされたデータを使用（追加クエリなし）
  end
  ```

- データの整合性の維持

- コードの簡素化と開発効率の向上
  ```ruby
  user = User.find(1)
  user.posts #userの投稿一覧
  
  #関連付けの外部キー（user_id）が自動的
  #newもbuildも機能は同じだが、関連付けコンテキストではbuildを使う
  user.posts.new(title: "新規投稿")
  user.posts.build(title: "構築のみ")
  
  #即時にデータベースに保存します
  user.posts.create(title: "即保存")  # 作成と同時にuser_idが設定される
  
  ```


### includesについて

内部的にはpreloadとeager_loadが選択される実行計画とパフォーマンスの比較

- preload
  - クエリ数: 複数（関連モデルごとに1つ＋親モデル用に1つ）
  - メモリ使用量: 通常少ない
  - ネットワーク: 複数回の通信が必要
  - 適している場合: 関連レコード数が多く、関連テーブルに条件を指定しない場合
- eager_load
  - クエリ数: 常に1つ
  - メモリ使用量: JOINにより大きくなる可能性あり
  - ネットワーク: 1回の通信
  - 適している場合: 関連レコード数が少なく、関連テーブルに条件を指定する場合

```sql
--preloadの場合
SELECT "users".* FROM "users"
SELECT "posts".* FROM "posts" WHERE "posts"."user_id" IN (1, 2, 3, ...)
```

```sql
--eager_load
--JOINを使うため、関連テーブルにも条件を指定できる
SELECT "users"."id" AS t0_r0, "users"."name" AS t0_r1, ..., 
       "posts"."id" AS t1_r0, "posts"."title" AS t1_r1, ... 
FROM "users" 
LEFT OUTER JOIN "posts" ON "posts"."user_id" = "users"."id"
```

