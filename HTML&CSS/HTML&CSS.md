# 勉強メモHTML&CSS


## 疑問
- Webデザインとかするときはコード書きつつ画面見ながらとか？  
どうやってるのか。

## ボックスモデル
- HTMLの全ての要素には、border（初期状態では表示されない）があり、  
その外側の余白はmargin, 内側の余白はpaddingです。

## 枠組み
- 大きな枠組み
body
　header
　main
　footer

- sectionの枠組み
section
　div
　　h2
　　div


## divタグとsectionタグの違い
sectionタグは文章やコンテンツの一つのまとまりを作るために使用する。
通常はそのまとまりに対して見出しをつける場合に使用する。

## Flexbox
### 基本
- 横並びにしたい場合、親要素に設定する。  
```css
display: flex;
```

- 縦ならびにしたい場合、要素の親要素に設定する  
```css
flex-direction: column
```

- 親要素の幅に合わせて伸縮したい場合、子要素に設定する。  
```css
flex: auto;
```

### 折り返し
- 折り返したい要素の親要素に設定する。  
```css
flex-wrap: wrap  
```

- 折り返したい要素自身には列数に応じたwidth  
```css
width: 50%
```

## Flexbox使用時の子要素の配置
- 垂直方向の中央
```css
align-items: center
```

- 水平方向の中央
```css
justify-content: center;
```

- コンテナ内のアイテムを均等に配置する
```css
#footer .inner {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
```


## containerクラスが重要

## インライン要素について
- インライン要素(初期状態の`<a>`タグ)はテキストの流れの中に配置できる。  
- ブロック要素は前後に改行が入り、1行全体を占める  
- インラインブロック要素はテキストの流れの中に配置でき、幅や高さを設定できる

### 使い分け
- 文章や段落：ブロック要素が自然（幅いっぱいに広がり、前後に改行）
- ナビゲーションリンク：インラインまたはインラインブロックが適切（横並び）
- ボタン：インラインブロックが便利（サイズ指定可能でテキスト内に配置可能）

## ブロック要素を中央寄せにする方法
containerクラスのように、広い範囲を囲うようなブロック要素の場合はmargin
```css
margin: 0 auto
```

テキストやボタンのようなインライン要素・インラインブロック要素の場合はtext-alignを使用する。
```css
text-align: center
```

## box-sizing: border-boxについて
要素の幅(width)の合計にpaddingとborderが含まれるようになる。

## メディアクエリ
ブラウザの画面サイズに応じてCSSのスタイルを設定できる手法
```css
@media (max-width: 1000px) {
  .lesson {
    width: 50%;
    margin-bottom: 50px;
  }
}
```

## 子要素が全てfloatでも、親要素が高さを持つように設定できる。
空タグを用意して設定する。
```css
clear: left;
```

## 埋め込みコード(ERBタグ)
・<%= %>  
出力ありの埋め込みRubyコード  
実行結果がHTMLとして出力される

・<% %>  
出力なしの埋め込みRubyコード  
Rubyコードは実行されるが、結果はHTMLに出力されない

## フォントファミリーを指定する際のルール
- フォント名はOS毎に指定する。
- 最後に総称フォントを必ず指定する。
- 総称フォント名は引用符で囲まない。

「'YuGothic'」・・・Macのフォント
「'Yu Gothic'」・・・Windowsのフォント
「sans-serif」・・・総称フォント

```css
font-family: 'YuGothic', 'Yu Gothic', sans-serif;
```

## positionプロパティについて
要素を固定させたり、重ね合わせる際に使用する。
また、以下とセットで使用されることが多い。
「位置を指定するためのtop,right,bottom,left」
「重なり順序を指定するためのz-indexプロパティ」

- fixed
要素は指定された位置で固定される。
- relative
元の位置を基準に指定された数値の分だけ移動する。
- absolute
ウィンドウを基準に指定された数値の分だけ移動する。
親要素にrelativeが指定されている場合は、親要素の位置が基準となる。
- static
初期値、元の位置に配置される。リセットしたい場合に使用する。
- sticky
親要素の中で固定されて表示される。
サイドバーの追従バナー等で使用される。

## 要素の固定
- 左上を基準に上から100px,左から200pxの位置に固定される。
レスポンシブ対応ではtop: 10px、left: 20px
またはレスポンシブ対応のため%で指定する。
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

- リセットする場合
```css
position: static;
```

## absoluteにより高さがなくなる
子要素が全てabsoluteの場合、親要素の高さが0になってしまう。
そのため親要素に明示的に高さを設定する必要がある。


## z-indexプロパティについて
要素の重なり順序を制御している。
基本は10毎に採番する。後でメンテしやすいため。

## object-fitプロパティについて
画像をトリミングしたりする際に利用される。
画像や動画がゆがんだり不自然に引き伸ばされたりすることを避けるために使用する機能
- contain
縦横比を維持したまま、ボックスに収まるように拡大縮小される。
- cover
縦横比を維持したまま、ボックスからはみ出した部分はトリミングされる。
幅か高さが決まっている場合に効果がある。
決まっていない場合は拡大縮小するためトリミングが必要なくなる。
```css
.new-type .img img {
  height: 480px;
  object-fit: cover;
  object-position: left;
}
```

- color(表示位置を指定)
「object-position」プロパティとセットで使用することで、
表示位置を固定することができる。
左寄せでトリミングを行うなどが可能となる。


## 複数行の日本語表記をきれいに揃える
```css
text-align: justify;
```

## displayプロパティについて
- inline
要素は横に並ぶ。横幅や高さは指定できない。主に文章の一部として使用する。
- inline-block
要素は横に並ぶ。横幅、高さ、余白の調整が可能。
大きなボタンを作ることができる。
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

## 透過について
rgbaで透過した場合は指定の色だけが透過するので背景のみ透過し、文字を透過させないことができる。
opacityで透過した場合は文字も含めて透過する。

## 透過を利用した画像の合成
画像の上に半透明の白色レイヤーを重ねている。
```css
background-image: url(../img/bg.jpg);
background-color: rgba(255, 255, 255, 0.5);
background-blend-mode: lighten;
```

## 要素を左右どちらかに詰めて配置したい場合
要素の左側のマージンを自動的に計算して、利用可能なスペースをすべて左マージンに割り当てる。
そのため、要素は右側に配置される。
幅を何らかの方法で明示的に制限しないと横幅いっぱいに広がるため、意味がなくなる。
```css
margin-left: auto;
max-width: 520px
```

## 疑似要素
要素にスタイルをつける。
- ::bofore 要素の直前にコンテンツを追加する。
contentプロパティに追加するテキストを設定する。画像を追加することもできる。

```css
/*テキストの直前に「・」を追加する*/
.sample ::before{
  content: "・";
}
```

- ::after 要素の直後にコンテンツを追加する。

```css
/*ボタンの右端に黒いラインをひく*/
.online-store .box .btn::after {
  content: "";
  width: 40px;
  height: 1px;
  background-color: #333;
  position: absolute;
  top: 20px;
  right: -20px;
}
```

## 斜めにラインの入った背景を設定する
```css
.mainvisual {
  background-color: #ffed58;
  clip-path: polygon(0 0, 100% 0, 100% 80%, 0 100%);
  padding: 70px 0 160px;
  margin-bottom: 40px;
}
```


## 改行をメディアクエリで制御する。
brのクラスを設定することで制御する。

```html
<span>プログラミングで</span><br><span>未来の扉を</span><br class="pc"><span>ひらこう</span>
```

```css
  .pc {
    display: none;
  }
```

## jQueryの利用


```html
<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.7.1/jquery.min.js"></script>
<script src="js/main.js"></script>
```

## ハンバーガーメニュー
スマートフォン利用時のヘッダ部にある3本線のこと。

```html
<div class="hamburger">
  <span></span>
  <span></span>
  <span></span>
</div>
<nav class="navi">
  <ul class="menu">
    <li><a href="index.html">トップ</a></li>
    <li><a href="service.html">サービス</a></li>
    <li><a href="price.html">料金</a></li>
    <li><a href="contact.html">お問い合わせ</a></li>
  </ul>
</nav>
```

```css
@media screen and (max-width: 767px) {
  #header .hamburger {
    width: 50px;
    height: 50px;
    cursor: pointer;
    position: fixed;
    top: 5px;
    right: 10px;
    z-index: 30;
  }
  #header .hamburger span {
    width: 30px;
    height: 3px;
    background-color: #000;
    display: inline-block;
    position: absolute;
    left: 10px;
    transition: all 0.4s;
  }
  #header .hamburger span:nth-of-type(1) {
    top: 16px; 
  }
  #header .hamburger span:nth-of-type(2) {
    top: 25px;
  }
  #header .hamburger span:nth-of-type(3) {
    top: 34px;
  }
  #header .navi.active {
    right: 0;
  }

  #header .navi {
    width: 80%;
    height: 100vh;
    background-color: #fff;
    position: fixed;
    top: 0;
    right: -80%;
    z-index: 20;
    transition: all 0.6s;
  }

  #header .navi .menu {
    width: 100%;
    height: 100vh;
    flex-direction: column;
    padding: 60px 0;
    overflow: auto;
  }

  #header .navi .menu li {
    padding: 10px 0;
    margin-left: 0;
  }
}
```

- ボタンを×にするCSS(ボタンのアニメーション)
```css
  #header .hamburger.active span:nth-of-type(1) {
    top: 24px;
    transform: rotate(-45deg);
  }
  #header .hamburger.active span:nth-of-type(2) {
    opacity: 0;
  }
  #header .hamburger.active span:nth-of-type(3) {
    top: 24px;
    transform: rotate(45deg);
  }
```

- jQueryでactiveクラスの追加、削除を行う処理を作成
```js
$(function(){
  /*-------------------------------
  ハンバーガーメニュー
  ---------------------------------*/
  $(".hamburger").click(function () {
    $(this).toggleClass("active");
    $("#header .navi").toggleClass("active");
  });
  
  $(".navi a").click(function () {
    $(".hamburger").removeClass("active");
    $("#header .navi").removeClass("active");
  });
});
```

## toggleClassとaddClassとremoveClassについて
- toggleClass
  対象となる要素に指定したクラスが存在していれば削除、存在しなければ追加を行う。

- addClass
  対象の要素に指定したクラスを追加する。

- removeClass
  対象の要素から指定したクラスを削除する。


## アイコンAwesome  
・Font Awesome  
https://fontawesome.com 
```html
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css">
```

## CSSフレームワーク
Tailwind CSS
Bootstrap、Tailwind CSS、Foundationなど

CDNを使用してTailwind CSSを読み込む方法
```html
<head>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
```
公式Doc
https://tailwindcss.com/docs/installation/using-vite

## 開発者モード
- 左上の矢印で選択モード
- StylesタブでCSSを確認できる。  
上のほうのhovにてホバー時のCSSを確認することもできる。  
上のほうの＋マークにてCSSを追加することもできる。