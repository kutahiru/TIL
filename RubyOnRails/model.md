# model

## コールバック

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

