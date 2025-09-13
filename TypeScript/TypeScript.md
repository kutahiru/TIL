# TypeScript

## 型定義ありのオブジェクトの定義

```ts
const obj: {
  foo: number;
  bar: string;
} = {
  foo: 123,
  bar: "Hello",
};
```

## Type文

データ型を定義できる

```ts
type FooBarObj = {
  foo: number;
  bar: string;
};
const obj: FooBarObj = {
  foo: 123,
  bar: "Hello"
};
```

## interface宣言

データ型を新規作成する。type文はあくまでも別名を付ける機能。
interface宣言の対象はオブジェクト型のみ。
基本的にtype文を使えばいい。

```ts
interface FooBarObj = {
  foo: number;
  bar: string;
};
const obj: FooBarObj = {
  foo: 0,
  bar: "string"
};
```

## オプショナルなプロパティな宣言

?を付加することで任意のプロパティを設定できる

```ts
type MyObj = {
  foo: boolean;
  bar: boolean;
  baz?: number;
}

const obj: MyObj = { foo: false, bar: true};
const obj2: MyObj = { foo: true, bar: false, baz: 1234};
```

## 部分型

オブジェクトの場合、プロパティの包含関係によって部分型関係を説明できる。
型Sと型Tがオブジェクト型として、以下の条件が満たされれば、SがTの部分型となる。

1. Tが持つプロパティはすべてSにも存在する。
2. 条件1の各プロパティについて、Sにおけるそのプロパティの型はTにおけるプロパティの型の部分型

## **日本語での理解**

- 「HasNameAndAge」は「HasName」よりも**条件が厳しい**
- 条件が厳しい → 該当するオブジェクトが**少ない**
- 少ない集合 → 大きな集合の**「一部分」**
- だから**「部分型」**

```ts
// HumanがAnimalの部分型
type Animal = {
  age: number;
};
type Human = {
  age: number;
  name: string
}
```

```ts
// HumanFamilyがAnimalFamilyの部分型
// motherのプロパティは異なっているが、HumanがAnimalの部分型のため
type AnimalFamily = {
  familyName: string;
  mother: Animal;
  father: Animal;
  child: Animal;
}

type HumanFamily = {
  familyName: string;
  mother: Human;
  father: Human;
  child: Human;
}
```

## 型引数を持つ型

type文で型を作成する時に宣言できる。
以下は型引数Tを持つUser型を定義している。
型名の後に<>で囲う。
型引数Tは宣言の中で型として使うことができ、以下ではchild: Tで使用している。

```ts
type User<T> = {
  name: string;
  child: T;
};

// 型引数を複数宣言することも可能
type Family<Parent, Child> = {
  mother: Parent;
  father: Parent;
  child: Child;
};
```

使用する場合は、<>の中に型を書く

```ts
//使用時
const obj: Family<number, string> = {
  mother: 0,
  father: 100,
  child: "1000"
};
```

## 関数宣言

function 関数名(引数リスト): 戻り値の型 {中身}

```ts
function range(min: number, max: number): number[]{
  const result = [];
  for (let i = min; i <= max, i++){
    result.push(i);
  }
  return result;
}

console.log(range(5, 10)); //[5,6,7,8,9,10]と表示される
```

### アロー関数での宣言

(引数リスト): 戻り値の型 =>  {中身}

```ts
type human = {
  height: number;
  weight: number;
};
const calcBMI = ({height, weight}: human): number => {
  return weight / height ** 2;
};

const uhyo: human = {height: 1.84, weight: 72};
console.log(calcBMI(uhyo);
            
//簡単な処理は省略系が用いられる
//直後にreturnの処理は{}とreturnが省略可能
const calcBMI = ({height, weight}: human): number => weight / height ** 2;
```

## 関数型の記法

```ts
type F = (repeatNum: number) => string; //Fが関数型

// xRepeatにアロー関数を代入している
const xRepeat: F = (num: number): string => "X".repeat(num);
```

### 関数型はコールバック引数として設定可能

(value: number) => numberの部分が関数型の定義
number型の変数を受け取りnumber型の値を返す関数

```ts
function map(array: number[], callback: (value: number) => number): number[]{
  const result: number[] = [];
  for (const elm of array){
    result.push(callback(elm));
  }
  return result;
}

const data = [1, 1, 2, 3, 5, 8, 13];
const result = map(data, (x) => x * 10); //xは仮引数
console.log(result); //10倍された値が表示される
```

## 関数の型引数

関数<型引数たち>(引数たち)
型引数を持つ関数は呼び出すたびに異なる型引数を設定できる。
<>内で定義した型がelementの型になる。
メソッドの内容的に引数のデータ型によって戻り値が変わるため、
戻り値はT[]が設定されている。

```ts
function repeat<T>(element: T, length: number): T[] {
  const result: T[] = [];
  for (let i = 0;i < length; i++){
    result.push(element);
  }
  return result;
}

console.log(repeat<string>("a", 5));
console.log(repeat<number>(123, 5));
```

## クラス

### クラス宣言

コンストラクタを用意しない場合は初期値が必要

```ts
class User{
  name: string = "";
  age: number = 0;
}
```

### メソッド

functionがいらないのはオブジェクトリテラルだから。
メソッド名の直後に括弧と型注釈を書くだけで、JavaScriptエンジンがそれをメソッドとして認識

```ts
class User{
  name: string = "";
  age: number = 0;
  
  isAdult(): boolean {
    return this.age >= 20;
  }
}
```

### コンストラクタ

コンストラクタがあればプロパティの初期値は不要。

```ts
class User{
  name: string;
  age: number;
  
  constructor(name: string, age: number){
    this.name = name;
    this.age = age;
  }
  
  isAdult(): boolean {
    return this.age >= 20;
  }
}

const uhyo = new User("uhyo", 26);
```

### 静的プロパティ、静的メソッド

```ts
class User{
   static adminName: string = "usho";
   static getAdminUser() {
     return new User(User.adminName, 26);
   }
  
  name: string;
  age: number;
  
  constructor(name: string, age: number){
    this.name = name;
    this.age = age;
  }
  
  isAdult(): boolean {
    return this.age >= 20;
  }
}
```

### プライベートプロパティ

以下はどっちもプライベートプロパティだが、privateはTSの機能で、#のほうはJSの機能
#のほうはJSの機能だからこっちを使ったほうが良さそう。

```ts
class User {
  private name: string;
  #age: number
}
```

## export宣言、import宣言

データを提供するモジュール
関数を提供するモジュール

```ts
// uhyo.ts
export const name = "uhyo";
export const age = 26;
```

import時はimport { 変数のリスト } from "モジュール名"

```ts
// index.ts
import { name, age } from "./uhyo.js";

console.log(name, age);
```

### 関数のexport

```ts
// uhyo.ts
export const getUhyoName = () => {
  return "uhyo";
};
```

```ts
// index.ts
import { getUhyoName } from "./uhyo.js"

console.log(`uhyoの名前は${getUhyoNmae()}です`)
```

### defaultについて

基本は使わなくていい。

```ts
// uhyoAge.ts
export default 26;
```

```ts
// index.ts
import uhyoAge from "./uhyoAge.js";

console.log(`uhyoの年齢は${uhyoAge}です`)
```

```ts
// counter.ts
export let value = 0;

export default function increment(){
  return ++value;
}
```

```ts
// index.ts
import increment from "./counter.js"; //片方のインポートの場合
import increment, {value} from "./counter.js"; //両方をインポートしたい場合

console.log(`カウンタの値は${increment()}です`)
console.log(`カウンタの値は${increment()}です`)
```

### 型のexport

```ts
export type Animal = {
  specied: string;
  age: number;
}
```

```ts
type Animal = {
  specied: string;
  age: number;
}

const tama: Animal = {
  specied: "Felis"
  age: 1
}

export { Animal, tama};
```

```ts
import { Animal, tama} from "./animal.js"

const dog: Animal = {
  species: "Canis"
  age: 2
}

console.log(dog, tama);
```

### 一括インポート

変数名で指定された変数に当該モジュールのモジュール名前空間オブジェクトが代入される
そのモジュールからエクスポートされた変数全てをプロパティに持つ特殊なオブジェクト

```TS
// uhyo.ts
export const name = "uhyo";
export const age = 26;
```

```ts
// index.ts
import * as uhyo from "./uhyo.js";

console.log(uhyo.name);
console.log(uhyo.age);
```

### 外部モジュール

プロジェクト内のソースコードに由来しないモジュール

```ts
// これはNode.jsの組み込みモジュール
import { createInterface } from "readline"
```

## Promiseオブジェクト

Promiseベースの非同期処理では非同期処理を行う関数は関数を受け取らずにPromiseオブジェクトを返す。
そのPromiseオブジェクトに対してい終わった後に行う処理を表す関数を登録する

Promise版のfs(ファイル操作)
pの方はPromise<string>
Promiseオブジェクトはthenメソッドを持っている。
このメソッドが引数としてコールバック関数を受け取る。
このメソッドはPromiseオブジェクトの非同期処理が完了したら実行される。

- 従来の非同期処理
  非同期処理を行う関数にコールバック関数を直接渡す
- Promise版の非同期処理
  非同期処理を行う関数はPromiseオブジェクトを返す。
  返されたPromiseオブジェクトにthenでコールバック関数を渡す。

前者の方式だと使いたいAPIごとにどのようにコールバック関数を渡せばいいか調べる必要がある。
後者の方式だとどんな関数でもPromiseオブジェクトを返すので共通の考え方で済む。

```ts
import { readFile } from "fs/promises"

const p = readFile("foo.txt", "utf-8");

p.then((data) => {
  console.log(data);
})
```

## asymc/await構文

asyncの戻り値は必ずPromiseになる。
async/awaitは「**その関数内では順序通りに見える**」が、「**他の処理は並行して動き続ける**」という特徴がある
