# JavaScript

## 公式Doc
https://developer.mozilla.org/ja/docs/Web/JavaScript

## JavaScriptをWebページに組み込む方法

### HTMLファイル内に記述
```html
  <script>
    alert('Hello, JavaScript!');
  </script>
```

### HTMLで外部ファイルとして読み込む
```html
<script src="script.js"></script>
```

### インラインスクリプトとして記述する
```html
<button onclick="alert('Hello, JavaScript!')">Click me!</button>
```

## アロー関数
=>：アロー関数の記号です。左辺に引数、右辺に処理を書きます。
{}：処理が1行の場合は省略できます。
return：処理が1行の場合は省略できます。
()：引数が1つの場合は省略できます。
```js
const greet = (name) => "Hello, " + name + "!";
```

## DOM操作
```js
document.getElementById('changeTextButton').addEventListener('click', function() {
  let header = document.getElementById('header');
  header.textContent = 'Text Changed!';
});
```


## イベント
よく使われるイベント
click：ユーザーが要素をクリックしたときに発生します。
mouseover：ユーザーが要素にマウスを乗せたときに発生します。
mouseout：ユーザーが要素からマウスを離したときに発生します。
keydown：ユーザーがキーを押したときに発生します。
load：ページや画像が読み込まれたときに発生します。
change：フォームの値が変更されたときに発生します。


## イベントハンドラをボタンに設定する
※alert() はブラウザに組み込まれたJavaScript APIの一部
addEventListnerメソッドの第1引数にはイベント名、第2引数にはイベントハンドラ
```html
<!DOCTYPE html>
<html>
<head>
    <title>addEventListenerでのイベントハンドラ</title>
</head>
<body>
    <button id="myButton">Click me</button>
    <script>
        // alertを呼び出す関数
        function showAlert() {
            alert('Button clicked!');
        }
        let button = document.getElementById('myButton');
        button.addEventListener('click', showAlert);
    </script>
</body>
</html>
```

## クライアントサイドWebAPI
ブラウザ内で動作するJavaScriptを使ってWebページにインタラクティブな機能を追加するためのAPI
https://developer.mozilla.org/ja/docs/Web/API