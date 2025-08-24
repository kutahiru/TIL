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

## 仮想DOM

仮想DOMと実際のDOMを比較し変更箇所のみを実際のDOMに反映することができる。
DOMへの操作を最小限に抑えることが可能になる。

## constとlet

varは再宣言が可能なので使用しない。

- let
  再宣言はできないが、上書きが可能
- const
  再宣言も上書きも不可能
  ただし、配列などのオブジェクトは変更可能
  constが保護するのは**変数とオブジェクトの参照**であって、**オブジェクトの中身**ではないため。
  配列の再代入は不可能だが、中身の変更は可能

## テンプレート文字列

文字列の結合に＋を使う必要がない。
バッククォートで囲む

```js
const message = `私の名前は${name}です。`
```

## アロー関数

従来の関数ではfunctionで定義していた。

```js
function func1(value){
  return value;
};

console.log(func1("func1です。")); //func1です
```

アロー関数では、fucntionは使用しない。
=>：アロー関数の記号です。左辺に引数、右辺に処理を書きます。
{}：処理が1行の場合は省略できます。
return：処理が1行の場合は省略できます。
()：引数が1つの場合は省略できます。

```js
const func2 = (value) => {
  return value;
};

console.log(func2("func2です。")); //func2です
```

### アロー関数の省略記法

- 引数が一つの場合は括弧を省略できる

  ```js
  const func2 = value => {
    return value;
  };
  
  console.log(func2("func2です。")); //func2です
  ```

- 処理を単一行で返却する場合は波括弧とreturnを省略できる
  ```js
  const func4 = (num1, num2) => num1 + num2
  
  console.log(func4(10, 20)); //30
  
  // また、以下の場合も波括弧とreturnを省略
  // ()で囲むことで単一行のようにまとめて返却できる
  const func5 = (val1, val2) => (
    {
      name: val1,
      age: val2,
    }
  )
  
  console.log(func5("主田", 24)); //{name: "主田", age: 24}
  ```

## 分割代入

```js
const myProfile = {
  name: "主田",
  age: 24,
};

const message = `私の名前は${myProfile.name}です。年齢は${myProfile.age}歳です。`;
console.log(message);

// ↑だと、myProfileの部分が冗長なので以下のようにオブジェクトをまとめて代入できる
const {name, age} = myProfile;

const message = `私の名前は${name}です。年齢は${age}歳です。`;
console.log(message);
```

## 引数の分割代入

showDialogの定義で{}が囲われているのは、分割代入のため。
引数の順序を気にせずに一部の引数だけを渡せる。
名前付き引数ができる。

```js
function showDialog({
  content = '',
  title = 'My Dialog',
  width = 100,
  height = 100,
  position = 'center',
  modal = false
}) {	
  console.log(`content: ${content}`);
  console.log(`title: ${title}`);
  console.log(`width: ${width}`);
  console.log(`height: ${height}`);
  console.log(`position: ${position}`);
  console.log(`modal: ${modal}`);
}

showDialog({
  content: 'ダイアログです。',
  modal: true,
});
```

## スプレット構文

...という形でドットを3つ繋げて使用できる。
配列に関して使用することで内部の要素を順番に展開してくれる。

```js
const arr1 = [1, 2];

const summaryFunc = (num1, num2) => console.log(num1 + num2);

//スプレット構文を使用せずに配列の値を渡す場合
summaryFunc(arr1[0], arr[1]); //3

// スプレット構文を使用した場合（順番に展開してくれている）
summaryFunc(arr1...) //3
```

## オブジェクトの省略記法

```js
const name = "主田";
const age = 24;

const user = {
  name: name,
  age: age,
};

console.log(user) //{name: "主田", age: 24}

//上記のユーザーオブジェクトの定義部分を以下のように省略して実装できる
const user = {
  name,
  age,
};

console.log(user) //{name: "主田", age: 24}
```

## モジュール

exportキーワードでモジュールの定義

```js
const AUTHOR = 'YAMADA, Yoshihiro';

export function getTriangleArea(base, height) {
  return base * height / 2;
}

export class Member {
  constructor(name = '名無権兵衛', age = 0) {
    this.name = name;
    this.age = age;
  }

  show() {
    console.log(`私の名前は${this.name}、${this.age}歳です。`);
  }
}
```

import命令でモジュールをインポート

```js
import { getTriangleArea, Member } from './lib/util.js';
// import { Member } from './lib/util.js';

// let path = './lib/util.js';
// import { getTriangleArea, Member } from path;

console.log(getTriangleArea(10, 2));

let m = new Member('佐藤理央', 25);
m.show();

console.log(import.meta);

```

### export default

defaultを一つだけ設定可能

```js
export const VERSION = '2.0';

export default class {
  static circle(radius) {
    return (radius ** 2) * Math.PI;
  }
}
```

規定のエクスポートになるため、関数/クラス名の名前を省略可能。
Areaという名前でアクセスできる。
Areaという名前は自由に命名可能。

```js
import Area from './lib/area.js';

console.log(Area.circle(10));
```

### まとめることもできる

複数の機能をimportしたファイルを一つ用意して、それを複数でimportすれば、各所のimportがすっきりする。
