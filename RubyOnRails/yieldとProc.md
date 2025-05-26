# yieldとProc

## yield

- ブロックを呼び出すことができる。

```ruby
def greet
  puts "おはよう"
  yield
  puts "こんばんは"
end

greet do
   puts "こんにちは"
end

#=>
# おはよう
# こんにちは
# こんばんは
```

- ブロックに引数を渡したり、ブロックの戻り値を受け取ったりできる

```ruby
def greet
  puts "おはよう"
  text = yield "こんにちは"
  puts "こんばんは"
end

greet do |text|
  text * 2
end
```

- ブロックを引数として明示的に受け取る

```ruby
def greet(&block) #引数名の前に&をつける
  puts "おはよう"
  text = block.call("こんにちは")
  puts text
  puts "こんばんは"
end

greet do |text|
  text * 2
end
```

