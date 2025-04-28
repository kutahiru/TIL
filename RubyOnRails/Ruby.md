# Ruby


### 疑問
- or andについて純粋な制御フローの場合のみ使用するという記載があった。
  実際にはどんな感じなのか。以下はどっちで実装するべき？
```ruby
#正常なユーザーであればメールを送信する。
user.valid? && send_email_to(user)
```

### 便利な書き方

#### 条件分岐で変数に代入
変数への代入自体が戻り値を持つから可能となる。
条件部を==と間違えたかもしれないと推測されることもあるので、この実装はしない。

```ruby
if currency = find_currency(country)
  currency.upcase
end
```

#### nilの時にエラーを回避する方法
&.演算子は、メソッドが呼び出されたオブジェクトがnilの場合、
エラーにせずにnilを返す。


#### nilガード、Or Equals演算子
limitがnilかfalseなら10を代入するコード
```ruby
#以下は全て書き換えなので同じ
#この書き方が一般的
limit ||= 10

#limitがnil以外ならlimit、nilならlimitに10が代入される。
limit || limit = 10

#右辺(limit || 10)が先に評価されるから、limitがnil以外ならlimit、nilなら10が代入される。
limit = limit || 10
```

#### 分岐を伴わないIF文は不要となる。？
```ruby
add = true
a = 1
b = 2

puts add && a + b

#分岐を伴わない条件の場合は上の書き方のほうがシンプル
if add
  puts a + b
end
```

#### 可変長引数

```ruby
def greet(*names)
  names.each do |name|
    puts "Hello, #{name}!"
  end
end
```

### 注意点

#### 真偽値

nilはfalseとして扱われる

### 変数展開

#{変数名}

```ruby
puts "面積は#{area}です"
```

### privateメソッド
クラスの内部からのみ呼び出すことができるメソッド

```ruby
private  # この行以降のメソッドはプライベートになる
```

### protectedメソッド

外部には公開したくないが、同じクラスやサブクラスの中であればレシーバ付きで呼び出せるようにする。
つまりは、他のインスタンスのprotectedメソッドを呼び出せるということ。
privateメソッドは他のインスタンスは呼び出せない。
レシーバ付きっていうのは、「other_user.weight」の部分インスタンス.メソッドのこと。
インスタンス同士で呼び出しが可能ということ。

```ruby
class User

  attr_reader :name

  def initialize(name, weight)
    @name = name
    @weight = weight
  end

  def heavier_than?(other_user)
    other_user.weight < @weight
  end

  protected

  #protectedは同じクラスかサブクラスであればレシーバ付きで呼び出せる
  def weight
    @weight
  end
end

alice = User.new("Alice", 50)
bob = User.new("Bob", 60)

#同じクラスのインスタンスメソッド内であればweightが呼び出せる
puts alice.heavier_than?(bob)
puts bob.heavier_than?(alice)

#クラスの外では呼び出せない。
#alice.weight
```

### オーバーロード

引数のデータ型や個数の違いに応じて同じ名前のメソッドを複数定義できるオーバーロードはない。
引数のデフォルト値や可変長引数を使用することで対応する。

### モジュール

モジュールに定義したメソッドは、
クラスにincludeキーワード、extendキーワードを使って組み込むことができる。

#### include

モジュールのメソッドをクラスの**インスタンスメソッド**として取り込む

```ruby
module Greeting
  def hello
    "Hello!"
  end
end

class Person
  include Greeting
  attr_accessor :name, :age

  def initialize(name, age)
    @name = name
    @age = age
  end
end

person = Person.new("太郎", 30)
puts person.hello
```

#### extend 

モジュールのメソッドをクラスの**クラスメソッド**として取り込む

```ruby
module Greeting
  def hello
    "Hello!"
  end
end

class Person
  extend Greeting
end

puts Person.hello
```

#### 定数

```ruby
module Greeting
  HELLO = "Hello!"
end

class Person
  include Greeting

  def hello
    HELLO
  end
end

person = Person.new
puts person.hello
puts Greeting::HELLO
```

### 名前空間解決演算子

Net`モジュール内の`HTTP`クラスの`get_responseメソッドを呼び出している
Net::HTTP`はNet`モジュール内の`HTTP`クラスを参照している

```ruby
Net::HTTP.get_response(uri)
```

### foreach

```ruby
languages = ["日本語", "英語", "スペイン語"]
# each文を用いて、要素ごとに「○○を話せます」と出力してください
languages.each do |language|
  puts "#{language}を話せます"
end
```

### 入力待ち状態

gets.chomp
gets.chomp.to_i　数値変換

### ヒアドキュメント

複数行にわたる改行の場合に使用する。
「-」は閉じの「SQL」を先頭に置かなくても良くなる。

<<-SQL
SELECT
  *
FROM table
SQL

### 論理演算子 && ||の利用について
最後に評価した式の値を返す。
```ruby
#最初に見つかったユーザーを変数に格納する。
user = find_user("Alice") || find_user("Bob") || find_user("Carol")

#正常なユーザーであればメールを送信する。
user.valid? && send_email_to(user)
```

#### !!を使った論理値の型変換
rubyではnilかfalseは偽、それ以外は真となる。
存在するかどうかでbool値を返すような処理が存在する場合、
IF文を実装せずとも、!!を使用することが可能。
論理値の型変換とは、
文字列型の変数aをnilかどうかで論理値に変換するということ。
最初の!は存在していればfalseになる。この部分が型変換を指している。
!!をつけると、!の状態の存在していればfalseを反転するため、trueになる。
!で型をboolにする。!!で正しいbool値にするということ。

```ruby
nil_value = nil
!nil_value  # => true (nilは偽値なので、その否定はtrue)
!!nil_value # => false (trueの否定はfalse)
```

### or and について
基本的な論理演算子としての機能は同じ
and/or → 制御フロー、条件分岐の記述に
&&/|| → 値の評価や代入を含む式に

and/orは読みやすさと制御の流れを表現するために
&&/||は式の中での論理演算のために

要するに機能的にはほぼ同じで、制御フローの用途として、どちらも使用することは可能だが、
可読性のため、制御フローの用途としてはandとorを使用することが推奨されている。
***純粋な制御フローの場のみ利用***
***代入を伴う場合は使用不可***

### %記法について
文字列は''、""だけでなく、%記法でも定義可能。
%記法は、特殊文字のエスケープが不要となる。
また、複数行の場合の可読性向上

- %q! !
  シングルクォートで囲んだことと同じになる
- %Q! !
  ダブルクォートで囲んだことと同じになる
- %! !
  ダブルクォートで囲んだことと同じになる
- %w! !
  配列の作成
- %s! !
  シンボルの作成
- %i! !
  シンボルの配列の作成

### 代入について
if文、case式の結果を変数に格納することができる。
Rubyが最後に評価した式を戻り値として返す仕様のため。

### !で終わるメソッドは破壊的メソッド
※!がついていないメソッドにも破壊的メソッドはある。
破壊メソッドと非破壊メソッドの同名メソッドがある場合に!がつく。
!は危険という意味

下の例は呼び出した自身の値も変更する。

```ruby
a = "ruby"

a.upcase! #戻り値はRUBY
a #aの値もRUBYに変更される。
```


### ブロックについて
要件によって共通する処理はメソッド、要件によって異なる処理はブロックで分担する。
- 複数行の場合
  do |a|...endの記法
- 1行の場合
  {|a|...}の記法

### ブロックを使う配列のメソッド

#### map
配列の各要素に対して、ブロックを評価した結果を新しい配列として返す。
```ruby
numbers = [1, 2, 3, 4, 5]
new_numbers = numbers.map {|n| n * 10}
```

#### select または filter
配列の各要素に対して、ブロックを評価し、戻り値が真の要素を集めた配列を返す。
```ruby
numbers = [1, 2, 3, 4, 5]
even_numbers = numbers.select {|n| n.even? }
```

#### find
ブロックの戻り値が真になった最初の要素を返す。

#### sum
合計

#### join
配列の要素を連結して一つの文字列にする。
```ruby
chars = ["a", "b", "c"]
chars.join("-") #区切り文字をハイフンにして連結する。
```

sumメソッドを使うと、初期値(先頭の文字)を与えたり、
ブロックないで文字列を加工することができる。
```ruby
chars = ["a", "b", "c"]
chars.sum(">") {|c| c.upcase} 
```

#### each_with_index
何番目の要素を処理しているか判別可能
```ruby
fruits = ["apple", "orange", "melon"]
fruits.each_with_index { |fruit, i| puts "#{i}: #{fruit}" }
```

#### with_index
map等のメソッドで利用可能。また、初期値を指定可能
```ruby
fruits.map.with_index(2) { |fruit, i| puts "#{i}: #{fruit}" }
```

#### sort_by

`sort`と`sort_by`メソッドの違いを理解することは、Rubyでのソート操作において重要です。基本的に、`sort`メソッドはシンプルな配列のソートに適していますが、`sort_by`メソッドはより複雑なデータ構造のソートや、複数の条件に基づくソートに適しています。

```ruby
# 2次元配列をsort_byでソートするサンプル
array = [[3, 5], [1, 2], [4, 1], [2, 3]]

# 第1要素でソート
sorted_by_first = array.sort_by { |element| element[0] }
puts "第1要素でソート:"
p sorted_by_first  # [[1, 2], [2, 3], [3, 5], [4, 1]]

# 第2要素でソート
sorted_by_second = array.sort_by { |element| element[1] }
puts "第2要素でソート:"
p sorted_by_second  # [[4, 1], [1, 2], [2, 3], [3, 5]]

# 複数の条件でソート（第1要素で昇順、第2要素で昇順）
sorted_by_multiple = array.sort_by { |element| [element[0], element[1]] }
puts "複数条件でソート（昇順、昇順）:"
p sorted_by_multiple  # [[1, 2], [2, 3], [3, 5], [4, 1]]

# 複数の条件でソート（第1要素で昇順、第2要素で降順）
sorted_by_mixed = array.sort_by { |element| [element[0], -element[1]] }
puts "複数条件でソート（昇順、降順）:"
p sorted_by_mixed  # [[1, 2], [2, 3], [3, 5], [4, 1]]
```

#### sort

```ruby
# 2次元配列をsortでソートするサンプル
array = [[3, 5], [1, 2], [4, 1], [2, 3]]

# 第1要素でソート
sorted_by_first = array.sort { |a, b| a[0] <=> b[0] }
puts "第1要素でソート:"
p sorted_by_first  # [[1, 2], [2, 3], [3, 5], [4, 1]]

# 第2要素でソート
sorted_by_second = array.sort { |a, b| a[1] <=> b[1] }
puts "第2要素でソート:"
p sorted_by_second  # [[4, 1], [1, 2], [2, 3], [3, 5]]

# 第1要素で昇順、第1要素が同じ場合は第2要素で降順
sorted_complex = array.sort { |a, b| a[0] == b[0] ? b[1] <=> a[1] : a[0] <=> b[0] }
puts "複合条件でソート:"
p sorted_complex  # [[1, 2], [2, 3], [3, 5], [4, 1]]
```

#### ハッシュのeach

ハッシュなので、ブロックパラメータが二つになる。
```ruby
currencies = {japan: "yen", us: "dollar", india: "rupee"}
currencies.each do |key, value|
  puts "#{key}: #{value}"
end
```

### 配列操作

重複を削除した新しい配列を作成

```ruby
numbers = [1, 2, 2, 3, 4, 4, 5, 5, 5]
unique_numbers = numbers.uniq
```

### 範囲オブジェクト

条件の判定や、case文で使える。
```ruby
1..5 #1以上5以下
1...5 #1以上5未満
```


### シンボルについて
- シンボルは整数として管理される。そのため、比較は文字列よりも高速
- 同じシンボルであれば全く同じオブジェクト。メモリ効率が良い。
- イミュータブルなので、勝手に値を変更されることがない。

### ハッシュについて
```ruby
#空のハッシュの宣言
hash = {}

#シンボルを使用したハッシュの宣言
hash = { key1: "value1", key2: "value2"}
```

### ハッシュの操作

#### マージ処理

キーが重複している場合、上書きされる。

```ruby
hash1 = { name: "Alice", age: 30 }
hash2 = { city: "Wonderland" }
hash3 = { name: "Bob", age: 20 }

merged_hash = hash1.merge(hash2)
puts merged_hash

merged_hash_with_conflict = hash1.merge(hash3)
puts merged_hash_with_conflict
```

### SQLへの接続方法

```ruby
require "sqlite3"
db = SQLite3::Database.new 'db/kakeibo.db'
```

select文の実行

```ruby
categories = db.execute('SELECT * FROM categories')
```

### HHTP通信

```ruby
require 'net/http'
require 'uri'

uri = URI.parse("http://www.example.com")
response = Net::HTTP.get_response(uri)

puts response.code
puts response.body
```

### 例外処理

`begin`ブロック内で例外が発生する可能性のある処理を記述
`rescue`ブロックで例外が発生した場合の処理を記述

```ruby
begin
  1 / 0
rescue ZeroDivisionError => e
  puts "ZeroDivisionError: #{e.message}"
end

#↑は特定のエラーなので、全てのエラーをキャッチしたい場合は以下
begin
  # 何らかの処理
rescue => e #発生した例外をeに代入
  puts "エラーが発生しました: #{e.message}"
end
```

### JSONファイルの操作

```ruby
require 'json'

json_string = '{"name": "John", "age": 30, "is_student": false, "courses": ["Math", "Science", "History"]}'
data = JSON.parse(json_string)

puts data["name"]
puts data["age"]
puts data["is_student"]
puts data["courses"]
```

## 自動テストについて

### Minitest
Minitestはクラス内のtest_で始まるメソッドを全て実行する。
「assert_equal」は検証メソッド

```ruby
require "minitest/autorun"
require_relative "../lib/fizz_buzz" #テスト対象のファイル

class FizzBuzzTest < Minitest::Test
  def test_fizz_buzz
    assert_equal "1", fizz_buzz(1)
    assert_equal "2", fizz_buzz(2)
    assert_equal "Fizz", fizz_buzz(3)
    assert_equal "4", fizz_buzz(4)
    assert_equal "Buzz", fizz_buzz(5)
    assert_equal "Fizz", fizz_buzz(6)
    assert_equal "Fizz Buzz", fizz_buzz(15)
  end
end
```

### RSpec
自動テスト。後で調べてみる