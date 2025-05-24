# moduleについて

継承を使わずにクラスにインスタンスメソッドを追加する
複数のクラスに対して共通の特異メソッド(クラスメソッド)を追加する
クラス名や定数名の衝突を防ぐために名前空間を作る
関数的メソッドを定義する
シングルトンオブジェクトのように扱って設定値などを保持する。

## include(ミックスイン)

モジュールをクラスにincludeして機能を追加すること
インスタンスメソッドとして追加される
ただし、ミックスイン先のクラスと連携する場合は、特定のインスタンス変数を扱うということはNG
メソッド経由で扱う。
なぜなら、未定義のインスタンス変数を参照可能であったり、エラーに気づかないため。
メソッドであれば未定義のメソッド参照はエラーになる。
セッターメソッド経由であれば問題ないため、self.nameの形式であれば問題なし。@nameが問題。

```ruby
module Loggable
  
  private #モジュール側privateにしておけば、クラスに追加した際もprivateになる
  
  def log(text)
    puts "[LOG] #{text}"
  end
end

class Product
  include Loggable

  def title
    log "title is called"
    "A great movie"
  end
end

class User
  include Loggable

  def name
    log "name is called"
    "Alice"
  end
end

product = Product.new
puts product.title

user = User.new
puts user.name
```

## include先のメソッドを使うモジュール

ダックタイピングの考え方
　※モジュール内には該当のメソッドはない。include先のクラスに該当のメソッドがある。

```ruby
module Taggable
  def price_tag
    "#{price}円" #priceメソッドはinclude先で定義されているという前提の実装
  end
end

class Product
  include Taggable

  def price
    1000
  end
end

product = Product.new
puts product.price_tag
#=> 1000円
```

## extend

モジュールをクラスにextendして機能を追加する。
クラスメソッドとして追加される

```ruby
module Loggable
  def log(text)
    puts "[LOG] #{text}"
  end
end

class Product
  extend Loggable #Loggableモジュールのメソッドをクラスメソッドとして追加する

  def self.create_products(names)
    log "create_product is called" #logメソッドをクラスメソッド内で呼び出す
  end
end

Product.create_products([])

Product.log("hello")
```

## 動的に追加する

※既存のインスタンスも即座に新しいメソッドが使える

```
Product.include Loggable
Product.extend Loggable
```

## 名前空間

クラス名の衝突を避けるために名前空間を使用する。
また、クラスの整理のために利用する。

```ruby
#2塁手のAlice
#BaseBallモジュール内のSecondクラス
BaseBall::Second.new("Alice", 13)

#時計の13秒
#Clockモジュール内のSecondクラス
Clock::Second.new(13)
```

### 名前空間つきクラスの定義方法

2通りの方法がある。

```ruby
# モジュール構文とクラス構文を入れ子にする
module Baseball
  class Second
  end
end

# ::を使ってフラットに書く
class Baseball::Second
end
```

## 関数を定義したモジュール

メソッドの集まりを作るのに向いている

```ruby
module Loggable
  class << self #これは以降がモジュールメソッドの定義となる。 def self.logと一緒
    def log(text)
      puts "[LOG] #{text}"
    end
  end
end

Loggable.log("Hello.")
```

## モジュール関数

ミックスインとしても、モジュールメソッドとしても使えるメソッドを定義できる
このモジュールはincludeして該当のクラスのクラスメソッドとして実行することもできるし、
単純にモジュールのモジュールメソッドとして実行することもできる。

```ruby
module Loggable
  def log(text)
    puts "[LOG] #{text}"
  end
  
  module_function :log
  
  module_function #複数定義する場合は、以降がモジュール関数となる。
end
```

## prepend

includeではなく、prependを使ってミックスインすることも可能
prependを使用してミックスインした場合、同名のメソッドがあった場合に
ミックスイン先のクラスのメソッドよりも優先してモジュールのメソッドが実行される。
使い方はincludeと一緒
コードの変更できないあるクラスにおいて、特定のメソッドを置き換えるなどの利用が可能となる。

## refinements

追加するメソッドの有効範囲を指定できる
以下の例だとStringクラスにメソッド追加しているが、
usingで有効範囲を指定しているため、Userクラス内でのみStringクラスのshuffleメソッドが使用可能になる
無理に既存クラスを拡張する必要があるかの検討は必要となる。

refine クラス名 do
  メソッド定義
end

```ruby
module StringShuffle
  refine String do
    def shuffle
      chars.shuffle.join
    end
  end
end

class User
  #refinementsを有効にする
  using StringShuffle
  
  def initialize(name)
    @name = name
  end
  
  def shuffled_name
    @name.shuffle
  end
end

user = User.new("Alice")
user.shuffled_name #=> "cliAe"

"Alice".shuffle #=>メソッドは使えないためエラー
```

