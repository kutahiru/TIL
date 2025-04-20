# model

### コールバック

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

### バリデーションはモデルで設定する

```ruby
validates(:content, {presence: true})
validates(:content, {presence: true, length: {maximum: 140}})
numericality #数値

```

validates メソッドは、指定された属性（フィールド）に対して、様々な検証ルールを適用するためのメタプログラミング的な仕組み
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



