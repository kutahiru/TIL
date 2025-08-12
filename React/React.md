# React

npm create vite@latest . --template react-ts

## スタイルの適用

CSS Modulesを利用してhome.module.cssファイルのスタイルを読み込む
CSS Modules を使うことで、home.module.css に定義されたクラス名がローカルスコープに限定され、
他のコンポーネントとクラス名が衝突することを防ぐことができる

```tsx
import { Link } from "react-router-dom";
import styles from "../../styles/home.module.css"; //スタイルの読み込み

export default function HomePage() {
  return (
    <div className={styles.container}>
      <h1 className={styles.title}>タスク管理アプリ</h1>
      <p className={styles.description}>
        React×Vite×React Routerで作るタスク管理アプリです。
      </p>
      <Link to="/tasks" className={styles.link}>
        タスク一覧へ
      </Link>
    </div>
  );
}
```

```css
/* src/styles/home.module.css */
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
  text-align: center;
  font-family: Arial, sans-serif;
}

.title {
  font-size: 2.5rem;
  color: #333;
  margin-bottom: 1.5rem;
}

.description {
  font-size: 1.2rem;
  color: #666;
  margin-bottom: 2rem;
}

.link {
  display: inline-block;
  background-color: #0070f3;
  color: white;
  padding: 0.8rem 1.5rem;
  border-radius: 5px;
  text-decoration: none;
  font-size: 1rem;
  transition: background-color 0.3s ease;
}

.link:hover {
  background-color: #0051bb;
}
```

## コンポーネント化

Appという名前のReactコンポーネントを作成している
return以降が複数行の場合は()で囲む。
divタグで囲んでいるのは、jsxのルールでreturn以降は1つのタグで囲まれている必要があるから。
空のタグで囲んでもいい。※Fragmentの省略記法

```jsx
// src/app.jsx
export const App = () => {
  return (
    <div> 
      <h1>Hello, World!</h1>
      <p>お元気ですか？</p>
    </div>
  );
};
```

```jsx
// src/main.jsx
import ReactDOM from "react-dom/client";
import { App } from "./app.jsx"; //拡張子は省略可能

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(<App />); //関数名をタグで囲むことで、コンポーネントとして扱うことができる
```

## イベント

JSXで書いているHTMLのタグの中では、{}で囲むことでJavaScriptを記述できる

```jsx
// src/app.jsx
export const App = () => {
  //ボタンを押した時に実行する関数を定義
  const onClickbutton = () => {
    alert("aa");
  }

  return (
    <div>
      <h1>Hello, World!</h1>
      <p>お元気ですか？</p>
      <button onClick={onClickbutton}>ボタン</button>
    </div>
  );
};
```

## スタイル

```jsx
// src/app.jsx
export const App = () => {
  //ボタンを押した時に実行する関数を定義
  const onClickbutton = () => {
    alert("aa");
  }

  return (
    <div>
      <h1 style={{ color: "red" }}>Hello, World!</h1>
      <p>お元気ですか？</p>
      <button onClick={onClickbutton}>ボタン</button>
    </div>
  );
};
```

### CSSオブジェクトを利用する方法

```jsx
// src/app.jsx
export const App = () => {
  //ボタンを押した時に実行する関数を定義
  const onClickbutton = () => {
    alert("aa");
  }

 const contentStyle = {
  color: "blue", //JavaScriptのオブジェクトを指定しているので、""が必要になる
  fontSize: "20px"
 }

  return (
    <div>
      <h1 style={ contentStyle }>Hello, World!</h1>
      <p>お元気ですか？</p>
      <button onClick={onClickbutton}>ボタン</button>
    </div>
  );
};
```

## Props

コンポーネントに渡す引数のようなもの。
コンポーネントは値に応じて表示するスタイルや内容を変化させる。

```jsx
// src/app.jsx
import { ColoredMessage } from "./components/ColoredMessage";

export const App = () => {
  //ボタンを押した時に実行する関数を定義
  const onClickbutton = () => {
    alert("aa");
  }

  return (
    <div>
      <h1 style={{ color: "red" }}>Hello, World!</h1>
      <ColoredMessage color="blue">お元気ですか？</ColoredMessage>
      <ColoredMessage color="pink">元気です！</ColoredMessage>
      <button onClick={onClickbutton}>ボタン</button>
    </div>
  );
};
```

childrenはコンポーネントのタグで囲まれているものを渡している。
ColoredMessageタグで囲まれているメッセージを取得するのにprops.childrenを参照している。
単純な文字列だけではなく、タグで囲んだ要素を丸ごと渡すこともできる。

```jsx
// src/components/ColoredMessage.jsx
export const ColoredMessage = (props) => {
  const contentStyle = {
    color: props.color,
    fontSize: "20px"
  };

  return <p style={contentStyle}>{props.children}</p>
};
```

### 分割代入の利用

props.は冗長なので、分割代入を利用すると良い
```jsx
const {color, children} = props;
```

さらにcontentStyleの定義時にも省略できるようになる。

```jsx
  const contentStyle = {
    color, // color: colorなので、省略できる
    fontSize: "20px"
  };
```

また、引数の段階で展開することも可能
```jsx
export const ColoredMessage = ( { color, childred } ) => {
```

## State

画面に表示するデータや可変の状態を全てStateとして管理している
useStateという関数を使用してStateを扱う
const [num, setNum] = useState(0);
numが状態を持った変数、setNumが状態を更新する関数

```jsx
// src/app.jsx
import { useState } from "react"; //useStateをimport
import { ColoredMessage } from "./components/ColoredMessage";

export const App = () => {
  //stateの定義
  const [num, setNum] = useState(0); //定義して分割代入

  //ボタンを押した時に実行する関数を定義
  const onClickbutton = () => {
    setNum(num + 1)
  }

  return (
    <div>
      <h1 style={{ color: "red" }}>Hello, World!</h1>
      <ColoredMessage color="blue">お元気ですか？</ColoredMessage>
      <ColoredMessage color="pink">元気です！</ColoredMessage>
      <button onClick={onClickbutton}>ボタン</button>
      <p>{num}</p>
    </div>
  );
};
```

## useEffect

useEffect( 実行する関数, [依存する値]);
「ある値が変わった時に限り、ある処理を実行する」機能
下の例だと、numというStateの値が変わった時にアラートを表示したい
第一引数にはアロー関数で処理を記述
第二引数には配列で依存する値を指定

不要な再レンダリングによる負荷を避けるために必要な機能
また、依存する値を未指定にすることで初回のみ実行するような処理を設定することもできる。

```jsx
export const App = () => {
  useEffect( () = => {
    alert();
  }, [num]); 
  
  return (
    //省略
  );
};
```

