# React

## Tips

- ctrl + 左クリックで型定義に移動

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
Stateが変わった際に再レンダリングされる
この例だと、setNumが実行されると画面が再レンダリングされる。

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

自分メモ

useEffectは第一引数に、第二引数で変更があった場合に実行する処理を実装する 第一引数は戻り値として、undefinedかクリーンアップ関数が必要

useEffect自体の役割としては、第一引数で受け取った関数を保持していて、 第二引数で変更があった場合に実行する さらに該当のコンポーネントがアンマウントされた場合に、第一引数を実行した時の戻り値として受け取っていたクリーンアップ関数があればそれを実行するって感じ

## メモ化

コンポーネント、変数、関数などの再レンダリング時の制御をするにはメモ化を行う。
前回の処理結果を保持しておくことで、処理を高速化する技術
本来、親のコンポーネントが再レンダリングされると、子も再レンダリングされるが、
それを防ぐことが可能になる。
コンポーネント関数をmemoで囲むだけでPropsに変更がない限り再レンダリングされない。

```jsx
const Component = memo(() => {});
```

### useCallback

関数をPropsに渡す時にコンポーネントをメモ化していても再レンダリングされてしまう原因は、関数の再生成。
関数をPropsとして受け取っている場合、Propsが変化したと判定されて再レンダリングされてしまう。
useCallbackを使用して関数のメモ化を行う必要がある。
useCallbackは第一引数に関数、第二引数に依存配列を設定する。
依存配列を空にすれば最初に作成された関数が使いまわされる。

子コンポーネントにPropsとして渡す関数はuseCallbackを使用してメモ化すると良い。

```jsx
const onClickButton = useCallback(() => {
  alett('ボタンが押されました！');
}, []);
```

## グローバルなState管理

StateをPropsで何段階も渡していくのはやらない。
Contextを使用してStateを管理する。

- React.createContextでContextの器を作成する。

- 作成したContextのProviderでグローバルStateを扱いたいコンポーネントを囲む

  ```jsx
  // AdminFlagProvider.jsx
  import { createContext, useState } from "react";
  
  // Context（状態を共有する箱）を作成
  export const AdminFlagContext = createContext({});
  
  // 関数コンポーネントを定義 propsを受け取る
  export const AdminFlagProvider = props => {
    const { children } = props //子要素の取り出し
    
    //isAdmin という状態変数と、それを変更するsetIsAdmin関数を作成
    const [isAdmin, setIsAdmin] = useState(false);
    
    return (
      // value に isAdmin と setIsAdmin を渡して、子要素からアクセスできるようにする
      // AdminFlagContext.ProviderはvalueというPropsを受け取っている
      <AdminFlagContext.Provider value={{ isAdmin, setIsAdmin}}>
        {children}
      </AdminFlagContext.Provider>
    );
  };
  ```

- Stateを参照したいコンポーネントでReact.useContextを使用する。

  ```jsx
  // EditButton.jsx
  import { useContext } from "react"
  
  import { AdminFlagContext } from "./providers/AdminFlagProvider";
  
  const style = {
    width: "100px",
    padding: "6px",
    borderRadius: "8px"
  };
  
  export const EditButton = () => {
    //Context内のisAdminを取得
    const { isAdmin } = useContext(AdminFlagContext);
    
    return (
      <button style={style} disabled={!isAdmin}>
        編集
      </button>
    );
  };
  ```

  ```jsx
  // App.jsx
  import { useContext } from "react"
  
  import { AdminFlagContext } from "./components/providers/AdminFlagProvider";
  import { Card } from "./components/Card";
  
  export const App = () => {
    //Context内のisAdminと更新関数を取得
    //最も近い <AdminFlagProvider> を見つけて、そのProviderの value プロパティの値を取得している
    const { isAdmin, setIsAdmin } = useContext(AdminFlagContext);
    
    const onClickSwitch = () => setIsAdmin(!isAdmin);
    
    return (
      <div>
        {isAdmin ? <span>管理者です</span> : <span>管理者以外です</span>}
        <button onClick={onClickSwitch}>切り替え</button>
        <Card isAdmin={isAdmin} />
      </div>
    );
  };
  ```

  ```
  //データの流れ
  AdminFlagProvider.jsx
      ↓ value={{ isAdmin, setIsAdmin }} を設定
  AdminFlagContext
      ↓ useContext で取得
  App.jsx
      ↓ isAdmin と setIsAdmin が使用可能
  ```

## カスタムフック

src/hooks
カスタムフックは内部でReact Hooksを使っているかどうか。
React Hooksを使う処理を関数にしたいと思った時にカスタムフックを実装する
状態管理を関数に出すということ

そのため、コンポーネント⇔カスタムフックのインターフェイスは、

##### 入力(引数)

- 初期値は必要になる。（状態管理だから）
- 何らかの設定値など

##### 出力(戻り値)

- 状態
  この状態は最初は引数で与えられた初期値、次は下の状態を操作する関数で変更された状態
- 状態を操作する関数
  基本はReact Hooksで生成される状態を操作する関数
  状態を操作する関数をさらに関数内で使用してその関数を返却することも可能。

```ts
// 検索ロジックのカスタムフック
// src/hooks/useSearch.ts
import { useState, useMemo } from "react";

// 今回作成しいてるカスタムフックのuseSearchの引数の型定義
interface UseSearchProps<T> { // <T>はジェネリック、<>は型が未決定だから
  data: T[]; //検索対象のレコード配列
  searchFields: (keyof T)[]; //検索対象列の配列
}

// 今回作成しいてるカスタムフックのuseSearchの戻り値の型定義
interface UseSearchReturn<T> {
  searchTerm: string; //検索文言
  setSearchTerm: (term: string) => void; //状態を操作する関数
  filteredData: T[]; //検索された結果のレコード配列
}

export function useSearch<T>({
  data, //検索対象のレコード配列
  searchFields //検索対象列の配列
}: UseSearchProps<T>): UseSearchReturn<T> { //ジェネリック。型が未定義だから<T>
  const [searchTerm, setSearchTerm] = useState(""); //カスタムフック内で使うReact Hooks

  const filteredData = useMemo(() => {
    if (!searchTerm.trim()) {
      return data; //検索文言が未設定ならそのまま返却
    }

    return data.filter(item =>
      searchFields.some(field => {
        const value = item[field];
        return typeof value === 'string' &&
               value.toLowerCase().includes(searchTerm.toLowerCase());
      })
    );
  }, [data, searchTerm, searchFields]); //useMemoの依存配列

  return {
    searchTerm, //検索文言。検索文言を渡すのはコンポーネント側では管理できてないから。
    setSearchTerm, //検索文言の状態を操作する関数
    filteredData //検索後のレコード配列
  };
}
```

```tsx
// 上の使用箇所

import { useSearch } from "@/hooks/useSearch";

// 検索機能（全データから検索、通常時は表示データのみ）
  const { searchTerm, setSearchTerm, filteredData: searchResults } = useSearch({
    data: initialData,
    searchFields: ['title', 'themeName']
  });
```
