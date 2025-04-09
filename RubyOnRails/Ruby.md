# Ruby

- [Ruby](#ruby)
    - [疑問](#疑問)
    - [便利な書き方](#便利な書き方)
      - [条件分岐で変数に代入](#条件分岐で変数に代入)
      - [nilの時にエラーを回避する方法](#nilの時にエラーを回避する方法)
      - [nilガード、Or Equals演算子](#nilガードor-equals演算子)
      - [分岐を伴わないIF文は不要となる。？](#分岐を伴わないif文は不要となる)
    - [ヒアドキュメント](#ヒアドキュメント)
    - [論理演算子 \&\& ||の利用について](#論理演算子--の利用について)
      - [!!を使った論理値の型変換](#を使った論理値の型変換)
    - [or and について](#or-and-について)
    - [%記法について](#記法について)
    - [代入について](#代入について)
    - [!で終わるメソッドは破壊的メソッド](#で終わるメソッドは破壊的メソッド)
    - [ブロックについて](#ブロックについて)
    - [ブロックを使う配列のメソッド](#ブロックを使う配列のメソッド)
      - [map](#map)
      - [select](#select)
      - [find](#find)
      - [sum](#sum)
      - [join](#join)
      - [each\_with\_index](#each_with_index)
      - [with\_index](#with_index)
      - [ハッシュのeach](#ハッシュのeach)
    - [範囲オブジェクト](#範囲オブジェクト)
    - [シンボルについて](#シンボルについて)
    - [ハッシュについて](#ハッシュについて)
  - [自動テストについて](#自動テストについて)
    - [Minitest](#minitest)
    - [RSpec](#rspec)


### 疑問
- or andについて純粋な制御フローの場合のみ使用するという記載があった。
  実際にはどんな感じなのか。以下はどっちで実装するべき？
```rb
#正常なユーザーであればメールを送信する。
user.valid? && send_email_to(user)
```

### 便利な書き方

#### 条件分岐で変数に代入
変数への代入自体が戻り値を持つから可能となる。
条件部を==と間違えたかもしれないと推測されることもあるので、この実装はしない。
```rb
if currency = find_currency(country)
  currency.upcase
end
```

#### nilの時にエラーを回避する方法
&.演算子は、メソッドが呼び出されたオブジェクトがnilの場合、
エラーにせずにnilを返す。


#### nilガード、Or Equals演算子
limitがnilかfalseなら10を代入するコード
```rb
#以下は全て書き換えなので同じ
#この書き方が一般的
limit ||= 10

#limitがnil以外ならlimit、nilならlimitに10が代入される。
limit || limit = 10

#右辺(limit || 10)が先に評価されるから、limitがnil以外ならlimit、nilなら10が代入される。
limit = limit || 10
```

#### 分岐を伴わないIF文は不要となる。？
```rb
add = true
a = 1
b = 2

puts add && a + b

#分岐を伴わない条件の場合は上の書き方のほうがシンプル
if add
  puts a + b
end
```

### ヒアドキュメント
複数行にわたる改行の場合に使用する。
<<SQL
SELECT
  *
FROM table
SQL

### 論理演算子 && ||の利用について
最後に評価した式の値を返す。
```rb
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
```rb
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

```rb
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
```rb
numbers = [1, 2, 3, 4, 5]
new_numbers = numbers.map {|n| n * 10}
```

#### select
配列の各要素に対して、ブロックを評価し、戻り値が真の要素を集めた配列を返す。
```rb
numbers = [1, 2, 3, 4, 5]
even_numbers = numbers.select {|n| n.even? }
```

#### find
ブロックの戻り値が真になった最初の要素を返す。

#### sum
合計

#### join
配列の要素を連結して一つの文字列にする。
```rb
chars = ["a", "b", "c"]
chars.join("-") #区切り文字をハイフンにして連結する。
```

sumメソッドを使うと、初期値(先頭の文字)を与えたり、
ブロックないで文字列を加工することができる。
```rb
chars = ["a", "b", "c"]
chars.sum(">") {|c| c.upcase} 
```

#### each_with_index
何番目の要素を処理しているか判別可能
```rb
fruits = ["apple", "orange", "melon"]
fruits.each_with_index { |fruit, i| puts "#{i}: #{fruit}" }
```

#### with_index
map等のメソッドで利用可能。また、初期値を指定可能
```rb
fruits.map.with_index(2) { |fruit, i| puts "#{i}: #{fruit}" }
```

#### ハッシュのeach
ハッシュなので、ブロックパラメータが二つになる。
```rb
currencies = {japan: "yen", us: "dollar", india: "rupee"}
currencies.each do |key, value|
  puts "#{key}: #{value}"
end
```

### 範囲オブジェクト
条件の判定や、case文で使える。
```rb
1..5 #1以上5以下
1...5 #1以上5未満
```


### シンボルについて
- シンボルは整数として管理される。そのため、比較は文字列よりも高速
- 同じシンボルであれば全く同じオブジェクト。メモリ効率が良い。
- イミュータブルなので、勝手に値を変更されることがない。

### ハッシュについて
```rb
#空のハッシュの宣言
hash = {}

#シンボルを使用したハッシュの宣言
hash = { key1: "value1", key2: "value2"}
```

## 自動テストについて

### Minitest
Minitestはクラス内のtest_で始まるメソッドを全て実行する。
「assert_equal」は検証メソッド
```rb
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
自動テスト。後で調べてみる。