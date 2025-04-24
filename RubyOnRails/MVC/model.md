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

```

validatesメソッドは可変長引数、第2引数以降はハッシュ形式の検証オプション
検証の度にモデルが実行されているわけではなく、
バリデーターのインスタンスを作成して登録する処理と思えば良い。
モデルのライフサイクル（保存時など）で実際の検証が実行される。

### バリデーションエラー

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

- belongs_to：モデルが他のモデルに従属する場合に使用。例：BookモデルがAuthorに属する。
- has_one：一方のモデルが他方のモデルを所有する場合に使用。例：SupplierがAccountを持つ。
- has_many：モデルが複数の他モデルを所有する場合に使用。例：Authorが複数のBookを持つ。
- has_many :through：中間モデルを介して多対多の関係を設定。例：Physicianがappointmentsを通じてPatientと関係する。
- has_one :through：中間モデルを介して一対一の関係を設定。例：SupplierがAccountを通じてAccountHistoryと関係する。
- has_and_belongs_to_many：中間モデルなしで多対多の関係を設定。例：AssemblyとPartが互いに多対多の関係を持つ。

belongs_toとhas_oneの選択は外部キーの位置によって決まり、has_many :throughとhas_and_belongs_to_manyの選択は中間モデルの必要性によって決まります。

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
  
  #関連付け特有の機能を利用可能
  user.posts.new(title: "新規投稿")  # user_idが自動でセットされる
  user.posts.create(title: "即保存")  # 作成と同時にuser_idが設定される
  user.posts.build(title: "構築のみ")  # インスタンス生成のみ
  ```

  
