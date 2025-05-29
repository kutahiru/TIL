# DBと紐づかないモデル

### 前段

ユースケースとは何らかの目的を達成するために行われるユーザーとアプリケーションの間の一連のやり取りを表したもの
RailsではこのやりとりはHTTPを介して行われる。
すなわちresourcesメソッドによるリソースベースのルーティング
そして、RailsはURLで表されるリソースをデータベースのテーブルと1対1に対応させており、
これらのCRUD操作を通じてユーザーとやりとりする。

RailsではモデルのCRUD操作の前後に実行される処理を追加することで、
ユースケースを実現するというアプローチをとっている。
このアプローチを実現するための機能がバリデーションとコールバック。
ただし、アプリの機能要求が複雑になるとこのアプローチは機能しなくなる。
URLで表されるリソースとデータベースのテーブルの1対1関係も崩れる。
対応するテーブルが存在しないケースも存在する。

この問題に対する対処法
新しいレイヤーを導入してユースケースのロジックをモデルから分離する
具体的には素のRubyクラスを定義してそこにユースケースのロジックを実装する
その際にActiveModelを利用したほうが良い。
ActiveModelを利用することで、モデルと連携するメソッドやビューのメソッド(form_withなど)を利用できる。

## ActiveModel

ActiveModelを利用することで、自分で定義した素のRubyクラスにもActiveRecordと同等のインターフェイスや機能を追加できる。

### ActiveModel::Attributes

型を持つ属性の定義を容易にしてくれるモジュール
型変換、コードの冗長、デフォルト値の管理、仮想属性の実装などの恩恵

```ruby
class Person
  include ActiveModel::Attributes
  
  attribute :name, :string
  attribute :age, integer
end
```

### ActiveModel::Callbacks

コールバック機能の実装を容易にしてくれるモジュール
define_model_callbacksメソッドは、コールバックの定義をするため。
コールバックの実行タイミングは、「run_callbacks :コールバック名 do」の部分

```ruby
class Person
  extend ActiveModel::Callbacks
  
  attr_accessoor :created_at, :updated_at
  
  define_model_callbacks :save #コールバックの対象となるメソッドを指定
  
  before_save :record_timestamps
  
  def save
    puts "saveメソッド開始"
    
    run_callbacks :save do  # ← ここでコールバック実行
      puts "実際の保存処理"
      true
    end
    
    puts "saveメソッド終了"
  end
  
  private
  
  def record_timestamps
    current_time = Time.current
    
    self.created_at ||= current_time
    self.updated_at = current_time
  end
end
```

```ruby
define_model_callbacks :save
で
before_save, around_save, after_saveメソッドが利用可能になる
特定のものだけが必要な場合はonlyオプションを設定する
define_model_callbacks :save, only: %i[before after]
```

### ActiveModel::Model

複数のモデルActiveModel内の複数のモジュールが組み合わさっているモデル
ActiveModel::Modelをincludeするだけで、オブジェクトをコントローラやビューのメソッドで利用できるようになる。
モデル的なRubyクラスを実装する場合に使用する

### ActiveModel::EachValidator

カスタムバリデーターを作成できる
ある一つの属性のバリデーションルールを定義するときに利用する

### ActiveModel::Validator

複数の属性を組み合わせたバリデーションルールなど、複雑なルールを定義する時に利用する

