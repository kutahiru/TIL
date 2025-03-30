# CSS_default

- [CSS\_default](#css_default)
  - [共通](#共通)
    - [html](#html)
    - [body](#body)
    - [img](#img)
    - [メディアクエリのブレイクポイント](#メディアクエリのブレイクポイント)
  - [header](#header)
    - [wrapper](#wrapper)
    - [.header](#header-1)
    - [displayプロパティについて](#displayプロパティについて)
    - [レスポンシブ対応](#レスポンシブ対応)
    - [要素の固定](#要素の固定)


## 共通
```css
@charset "UTF-8";
html {
  font-size: 100%;
}
body {
  color: #707070;
  font-family: 'YuGothic', 'Yu Gothic', sans-serif;
}
img {
  max-width: 100%;
  vertical-align: bottom;
}
li {
  list-style: none;
}
a {
  color: #707070;
  text-decoration: none;
}
a:hover {
  opacity: 0.7;
}

/*-------------------------------------------
スマートフォン
-------------------------------------------*/
@media screen and (max-width: 767px) {
}
```

### html
フォントサイズを100%にしておくことで、
ユーザーがブラウザで設定したフォントサイズが設定される。
```css
html {
  font-size: 100%;
}
```


### body
サイト全体の基本を設定する。
```css
body {
  color: #707070;
  font-family: 'YuGothic', 'Yu Gothic', sans-serif;
}
```

### img
レスポンシブで作成する場合は最大幅を100%にしておくことで
親コンテンツからはみ出すことを防げる。
「vertical-align: bottom;」は画面下の隙間を消すため。
```css
img {
  max-width: 100%;
  vertical-align: bottom;
}
```

### メディアクエリのブレイクポイント
- 基本がPCでスマホ用のブレイクポイントを設定する場合は、「767px」
- 基本はスマホでPC用のブレイクポイントを設定する場合は、「1025px

## header

### wrapper
- 「padding: 0 20px」は画面幅を狭めた際の両サイドの余白
これがあることで、狭い画面でもコンテンツをブラウザの端から離すことができる。
レスポンシブデザインの手法。
- 「margin: 0 auto」は横幅を設定したボックスを中央に配置する。

この組み合わせがレスポンシブデザインの手法。
```css
.wrapper {
  max-width: 1000px;
  padding: 0 20px;
  margin: 0 auto;
}
```

### .header

- 「display: flex」要素が横並びになる。
- 「align-items: center」要素が縦方向の中心に配置される。
- 「justify-content: space-between」横方向に均等割り付けされる。
これでヘッダ画像が左端にグローバルナビゲーションが右側に表示される。

```css
#header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-top: 35px;
  padding-bottom: 35px;
}
```

### displayプロパティについて
- inline
要素は横に並ぶ。横幅や高さは指定できない。主に文章の一部として使用する。
- inline-block
要素は横に並ぶ。横幅、高さ、余白の調整が可能。
- block
要素は縦に並ぶ。横幅、高さ、予約の調整が可能。
inlineの要素を縦に並べたい場合に使用。
- flex
要素は横に並ぶ。
- grid
格子状のマス目を使ってレイアウトを組み立てていく手法
- none
要素を非表示。
要素の表示切替で使用する。

### レスポンシブ対応
- 余白の調整
```css
margin-bottom: 50px;
```

- フォントサイズと位置の調整
```css
font-size: 16px;
text-align: left;
```

- 横並びだったものを縦並びに変更
以下は表示順も逆順に変更している
```css
flex-direction: column-reverse;
```

- 画像等のサイズを変更
```css
width: 100%;
```


### 要素の固定
- 左上を基準に上から100px,左から200pxの位置に固定される。
レスポンシブ対応ではtop: 10px、left: 20px
```css
position: fixed
top: 100px
left: 200px
```

- 元の位置を基準に上から100px,左から200px移動する。
```css
position: relative
top: 100px
left: 200px
```

- ウィンドウを基準に上から100px,左から200px移動する。fixedと異なり固定されない。
※親要素にrelativeの指定なし。
```css
position: absolute
top: 100px
left: 200px
```

- 親要素を基準に上から100px,左から200px移動する。
※親要素にrelativeの指定あり。
```css
position: absolute
top: 100px
left: 200px
```
