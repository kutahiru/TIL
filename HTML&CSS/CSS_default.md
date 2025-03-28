# CSS_default

```css
@charset "UTF-8";

html {
  font-size: 100%;
}
body {
  color: #707070;
  font-family: sans-serif;
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
  font-family: sans-serif;
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
- 基本はスマホでPC用のブレイクポイントを設定する場合は、「1025px」


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