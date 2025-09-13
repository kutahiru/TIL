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

## オブジェクトリテラル

jsのオブジェクトは連想配列、辞書とかハッシュとかと同じもの。
const obj = {}がオブジェクトリテラル

```js
const name = "主田";
const age = 24;

const user = {
  name: name,
  age: age,
};

console.log(user) //{name: "主田", age: 24}
```

### オブジェクトリテラルの省略記法

```js
// 上記のユーザーオブジェクトの定義部分を以下のように省略して実装できる
// 同一の変数が存在しているから利用できる
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

## Fetch API

FetchAPIを利用することでサーバに要求を投げたり、
応答を受け取ったりといった通信部分をJavaScriptで実装できるようになる。
非同期で実行される。
fetchメソッドの戻り値はresとしているが、実態はPromise<Response>オブジェクト
res.josn()メソッドの戻り値はPromise<object>
流れとして、以下が定番
fetch + then(レスポンス処理) + then(結果値処理)

```json
// book1.json
{
  "title": "TypeScript入門",
  "price": 2948,
  "publisher": "技術評論社"
}
```

```js
// jsonからタイトルを取得する
fetch('book.json')
  .then(res => res.json()) //fetch('book.json')の戻り値がres
  .then(data => console.log(data.title)); //res.jsonの戻り値がdata

// 上の通信エラー時の対応を入れたバージョン
fetch('book.json')
  .then(res => {
    if (res.ok) {
      return res.json();
    }
    throw new Error('指定のリソースが無効です。');
  })
  .then(data => console.log(data.title))
  .catch(e => window.alert(`問題発生：${e.message}`));

```

## Promiseオブジェクト

以下は文字列が渡されると500ミリ秒後に入力値〇〇というメッセージを表示する。
文字列が空の場合は、入力は空ですのエラーを返す非同期処理
**Promiseを使うことで非同期処理を同期処理のように実装できる。**

非同期で処理すべき内容を関数としてまとめる。
以下の例だとasyncProcessのこと。
この関数が戻り値として返すのがPromiseオブジェクト

Promiseは非同期処理の状態を監視するためのオブジェクト
コンストラクターには実行すべき非同期処理を関数リテラルかアロー関数で実装する。

関数の引数であるresolve,rejectは非同期処理の成功と失敗を通知するための関数

自分の理解
Promise自体は非同期処理と直接の関係はなし。
setTimeout自体が元から非同期処理になっている。
setTimeout(関数, 実行タイミング)メソッド。
Promiseオブジェクトのコンストラクターでは関数を引数に取る。
以下の例ではアロー関数を引数に設定している。
resolveとrejectはPromiseオブジェクトが用意してして、Promiseオブジェクトのコンストラクターに渡す関数は、
resolveとrejectを引数に取る。
resolveは成功状態にする関数、rejectをは失敗状態にする関数。

```js
function asyncProcess(value) {
  // asyncProcessは即座にPromiseをreturnするメソッド
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (value) {
        resolve(`入力値： ${value}`);
      } else {
        reject('入力は空です');
      }
    }, 500);
  });
}

asyncProcess('トクジロウ')
  .then(response => console.log(response))
  .catch(error => console.log(`エラー： ${error}`))
  // .then(
  //   response => console.log(response),
  //   error => console.log(`エラー： ${error}`)
  // )
  .finally(() => console.log('処理終了'));

```

### Promiseを使わない場合

コールバックを利用する必要がある

```js
function asyncProcess(value, callback) {
  setTimeout(() => {
    if (value) {
      callback(null, `入力値： ${value}`); // 成功: (エラーなし, 結果)
    } else {
      callback('入力は空です', null);      // 失敗: (エラー, 結果なし)
    }
  }, 500);
}

asyncProcess('テスト', (error, result) => {
  if (error) {
    console.log(error);  // エラー処理
  } else {
    console.log(result); // 成功処理
  }
});

// 複数の非同期処理を連続で行う場合に以下のように入れ子になってしまう
asyncProcess('値1', (error1, result1) => {
  if (error1) return console.log(error1);
  
  asyncProcess(result1, (error2, result2) => {
    if (error2) return console.log(error2);
    
    asyncProcess(result2, (error3, result3) => {
      if (error3) return console.log(error3);
      
      console.log('最終結果:', result3);
      // さらに深くなる...
    });
  });
});

//Promise版なら以下で実現できる。
//thenが1個前の結果を取得してアロー関数の引数にしている。
//このthenはPromiseオブジェクトのメソッド promise.then()
//asyncProcessの戻り値がpromiseオブジェクトだから使える。
asyncProcess('値1')
  .then(result1 => asyncProcess(result1))
  .then(result2 => asyncProcess(result2))
  .then(result3 => console.log('最終結果:', result3))
  .catch(error => console.log(error));
```

## 非同期処理の並列処理

Promise.allメソッドを利用することで複数の非同期処理を並列に実行し、その全てが成功した場合に処理を実行する。
全てが成功した時のみthenメソッドが実行される。

```js
function asyncProcess(value) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (value) {
        resolve(`入力値： ${value}`);
      } else {
        reject('入力は空です');
      }
    }, 500);
  });
}

Promise.all([
  asyncProcess('トクジロウ'),
  asyncProcess('ニンザブロウ'),
  asyncProcess('リンリン')
])
  .then(response => console.log(response))
  .catch(error => console.log(`エラー： ${error}`));
```

allSettledメソッドは、成否にかかわらず全ての非同期処理を実行する。

```js
function asyncProcess(value) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (value) {
        resolve(`入力値： ${value}`);
      } else {
        reject('入力は空です');
      }
    }, 500);
  });
}

Promise.allSettled([
  asyncProcess('トクジロウ'),
  asyncProcess('ニンザブロウ'),
  asyncProcess('リンリン')
])
  .then(response => console.log(response))
  .catch(error => console.log(`エラー： ${error}`));

```

## Promiseの処理を同期的に記述する

async/await構文を利用することで同期的に表現できる。
ここでいう同期的とはmainの中は同期的という意味
mainの外部は非同期で実行される。
async/awaitは「**その関数内では順序通りに見える**」が、「**他の処理は並行して動き続ける**」という特徴がある

```js
function asyncProcess(value) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (typeof value === 'number') {
        resolve(value ** 2);
      } else {
        reject('引数valueは数値でなければいけません。');
      }
    }, 500);
  });
}
async function main() {
  let result1 = await asyncProcess(2);
  let result2 = await asyncProcess(result1);
  let result3 = await asyncProcess(result2);
  return result3;
}

main()
  .then(response => console.log(response))
  .catch(error => console.log(`エラー： ${error}`));

```

