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

## API利用サンプル

```javascript
window.addEventListener('load', () => {
  const button = document.querySelector("button")
  const repositorySpace = document.querySelector("#repository_list")
  button.addEventListener('click', () => {
    const githubName = document.querySelector("#github_name").value
    fetch(`https://api.github.com/users/${githubName}/repos`)
    .then((response) => response.json())
    .then((data) => {
      //for文を使ってdataの配列の要素をrepositoryInfoという変数に一回一回格納する
      for(repositoryInfo of data){
        const li = document.createElement("li")

        li.innerHTML = repositoryInfo["name"]
        //リポジトリの名前の取得を確認
        repositorySpace.appendChild(li)
      }
    });
  })
})
```

## Promiseについて

Promiseはresolve()やreject()という決まった関数が実行されてから、次の処理を動かす仕組み
async/awaitを使った方法でも同じような処理をわかりやすいコードで記載することができる

```js
new Promise((resolve, reject) => {
  // 非同期で処理したいことを記述
  // 成功したらresolve()を呼ぶ
  resolve()
  // 失敗したらreject()を呼ぶ
  reject()
}).then(() => {
  // 上のresolve()が実行された後の処理
  })
  .catch(() => {
  // 上のreject()が実行された後の処理
  })

new Promise((resolve, reject) => {
  resolve()
}).then(() => {
    console.log("resolveが実行されてthenの処理が動きました")
  })
  .catch(() => {
    console.log("rejectが実行されてcatchの処理が動きました")
  })
```