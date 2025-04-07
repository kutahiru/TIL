# Ruby

### 疑問
- or andについて純粋な制御フローの場合のみ使用するという記載があった。
  実際にはどんな感じなのか。以下はどっちで実装するべき？
```rb
#正常なユーザーであればメールを送信する。
user.valid? && send_email_to(user)
```

### ヒアドキュメント
複数行にわたる改行の場合に使用する。
<<SQL
SELECT
  *
FROM table
SQL

### && ||の利用について
最後に評価した式の値を返す。
```rb
#最初に見つかったユーザーを変数に格納する。
user = find_user("Alice") || find_user("Bob") || find_user("Carol")

#正常なユーザーであればメールを送信する。
user.valid? && send_email_to(user)
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


### 範囲オブジェクト
条件の判定や、case文で使える。
```rb
1..5 #1以上5以下
1...5 #1以上5未満
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