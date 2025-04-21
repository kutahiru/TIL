# Classについて

## initialize(コンストラクタ)

```ruby
#インスタンス作成時は、initializeが実行される。
User.new
```

### インスタンスメソッド

インスタンス（オブジェクト）に対して呼び出されるメソッド
***インスタンス変数にアクセスできる***

### クラスメソッド

クラス自体に対して呼び出されるメソッド
***インスタンス変数にはアクセスできない***
インスタンスの状態に依存しない処理
def self.メソッド名

```ruby
class User
  def initialize(name)
    @name = name
  end
  
  #クラスメソッド
  def self.create_users(names)
    names.map do |name|
      User.new(name)
    end  
  end
end
names = ["alice", "bob"]
User.create_users(names)
```

###　インスタンスメソッドからクラスメソッドを呼び出す場合

クラス名.メソッド名

```ruby
class Product
  #クラス変数
  def self.format_price(price)
    "#{price}円"
  end

  #インスタンス変数
  def to_s
    formattred_price = Product.format_price(price) #クラス変数のCALL
  end
end
```

### インスタンス変数

インスタンス内部で共有される変数
@で始める
インスタンス変数はクラス外部から直接の参照不可

```ruby
@name = name
```

### アクセサメソッド

外部からインスタンス変数にアクセスするメソッド

```ruby
attr_accessor :name #単純な読み書きのみの場合
attr_reader :name #読み取り専用の場合
attr_writer :name #書き込み専用の場合

#ゲッターメソッド
def name
  @name
end

#セッターメソッド
def name=(value)
  @name = value
end
```

## 継承

### 継承

```ruby
class DVD < Product
end
```

### Superクラスのメソッド呼び出し

親クラスと引数が同じ場合、
子クラスで引数無しの「super」実行時は、自分に渡された引数を親クラス渡す。
子クラスで親クラスと同じプロパティへの代入処理を実装しなくて済む。
そもそも子クラスで独自のinitializeが不要なら、実装しなければ親クラスのinitializeが実行される。

### オーバーライド

同名メソッドを作るだけ。
Productクラスのto_sはObjectクラスのto_sのオーバーライド
DVDクラスのto_sはProductクラスのto_sのオーバーライド
DVD.to_sメソッドではsepurを実行している。
これは、親クラスのProduct.to_sを実行している。

```ruby
#製品クラス
class Product
  attr_reader :name, :price

  def initialize(name, price)
    @name = name
    @price = price
  end

  def to_s #Objectクラスのto_sのオーバーライド
    "name: #{name}, price: #{price}"
  end
end

#DVDクラス
class DVD < Product
  attr_reader :running_time

  def initialize(name, price, running_time)
    super(name, price)
    @running_time = running_time
  end

  def to_s #Productクラスのto_sのオーバーライド
    "#{super}, running_time: #{running_time}"
  end
end

dvd = DVD.new("A Great movie", 1000, 120)
puts dvd.to_s
```

### 引数転送

子クラスで引数を使用しない場合、「...」で親クラスに引数をそのまま渡せるので、
定義する必要がなくなる。
ただし、子クラスで独自処理がなければ、initializeを定義しなければ良いだけではある。

```ruby
def initialize(...)
  supur
end
```

## 用語

### レシーバー

レシーバとはメソッドの対象になったオブジェクトのこと
以下の場合、first_nameメソッドのレシーバーはuserということになる。

```ruby
user.first_name
```
