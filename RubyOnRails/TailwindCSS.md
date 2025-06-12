# TailwindCSS

## インストールについて

### gemを利用

React/Vue等のSPAと組み合わせ等がなく、
シンプルに利用するだけなら以下のgemのみの利用で良い

```
gem 'tailwindcss-rails'
```

```
# gemのコマンドを使用してインストール
bin/rails tailwindcss:install
```

V4 ではtailwind.config.js は不要、V4では以下に配置
app\assets\tailwind\application.css

```
@import "tailwindcss";
```

```
#監視
bin/rails tailwindcss:watch

#ビルド
bin/rails tailwindcss:build
```

## npmを利用

## レスポンシブ対応

- container
  画面サイズに応じて最大幅を自動的に調整してくれる
- mx-auto
  要素を水平方向に中央揃え

```erb
<div class="container mx-auto">
```

### コンテナの視覚的なイメージ

小さい画面では、コンテナは画面の幅いっぱいに広がります（パディングを除く）：

```
|<---- 画面幅 ---->|
|  [container]    |
```

大きな画面では、コンテナはその最大幅に制限され、画面の中央に配置されます：

```
|<------------ 画面幅 (例: 1920px) ------------>|
|                [  container  ]                |
|                 (max-width: 1536px)           |
```

### FlexBox

子要素を横並びに配置するために使用する

```erb
<div class="flex">
    <div class="w-3/5">
        <h2>
            ここに見出しがはいります。
        </h2>
        <p>
            ここは説明の文章が入ります。<br>
            sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS sample texts
        </p>
    </div>
    <div class="w-2/5">
        <img src="img/vince-lee-p6KMIFzWwy8-unsplash.jpg" alt="">
    </div>
</div>
```

```
+--------------------------------------------------+
|                                                  |
|  +-------------------------+ +----------------+  |
|  |                         | |                |  |
|  |  ここに見出しがはいります。 | |                |  |
|  |                         | |                |  |
|  |  ここは説明の文章が入ります。| |      画像      |  |
|  |  sample texts...        | |                |  |
|  |  sample texts...        | |                |  |
|  |                         | |                |  |
|  |      (w-3/5)            | |    (w-2/5)     |  |
|  +-------------------------+ +----------------+  |
|                                                  |
+--------------------------------------------------+
```

- 以下を指定すると余白を埋めるので、詰めて配置されず、横幅に対して子要素が均等間隔で配置される

````
md:ml-auto md:mr-auto
````

### gap

gapを指定することで要素間の余白を設定

```erb
    <div class="flex gap-10 mt-20">
      <div class="w-1/4">
        <%= image_tag "mew_idea.png", alt: "" %>
        <h3>Sample Tomato</h3>
        <p>
          sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS sample texts
        </p>
      </div>
      <div class="w-1/4">
        <%= image_tag "mew_idea.png", alt: "" %>
        <h3>Sample Tomato</h3>
        <p>
          sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS sample texts
        </p>
      </div>
      <div class="w-1/4">
        <%= image_tag "mew_idea.png", alt: "" %>
        <h3>Sample Tomato</h3>
        <p>
          sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS sample texts
        </p>
      </div>
      <div class="w-1/4">
        <%= image_tag "mew_idea.png", alt: "" %>
        <h3>Sample Tomato</h3>
        <p>
          sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS sample texts
        </p>
```

### モバイルファーストの考え方

モバイルファーストのブレイクポイントシステム
最初にモバイルレイアウトを設計
ブレイクポイント名とコロンを付ける方法で「ブラウザ幅が～以上なら適用するスタイル」として設定
「md:w-2/5」は、768px以上なら5/2の幅を適用するという意味

```erb
<div class="md:flex gap-x-10">
    <div class="md:w-2/5">
        <img src="img/vince-lee-p6KMIFzWwy8-unsplash.jpg" alt="">
    </div>
    <div class="md:w-3/5 md:order-first">
        <h2>
            ここに見出しがはいります。
        </h2>
        <p>
            ここは説明の文章が入ります。<br>
            sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS
        </p>
    </div>
</div>
```



![img](https://popshub.s3.amazonaws.com/uploads/ckeditor/pictures/15984/content_image-1696235050222.jpg)

モバイル画面では1列表示（フルワイド）
タブレット以上（md:）では2列表示（各50%幅）
大画面（xl:）では4列表示（各25%幅）

- md:flex-row md:flex-wrap
  横並びで折り返し

-  md:-mx-5

  カード間の間隔調整のために必要
  子要素はmd:px-5でタブレット以上の場合にmarginを設定している。
  　※タブレット以上の場合に横並びになるためmarginを設定しないとくっついてしまう。
  親要素はmd:-mx-5でタブレット以上の場合にマイナスのmarginを設定している。
  　※画面の両端の余白をなくすため。
  　※親要素のマイナスのmarginは子要素のpaddingに重なるということ。

```erb
    <div class="flex flex-col md:flex-row md:flex-wrap md:-mx-5 gap-y-10 mt-20">
      <div class="md:w-1/2 md:px-5 xl:w-1/4">
        <%= image_tag "mew_idea.png", alt: "" %>
        <h3>Sample Tomato</h3>
        <p>
          sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS
        </p>
      </div>
      <div class="md:w-1/2 md:px-5 xl:w-1/4">
        <%= image_tag "mew_idea.png", alt: "" %>
        <h3>Sample Tomato</h3>
        <p>
          sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS
        </p>
      </div>
      <div class="md:w-1/2 md:px-5 xl:w-1/4">
        <%= image_tag "mew_idea.png", alt: "" %>
        <h3>Sample Tomato</h3>
        <p>
          sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS
        </p>
      </div>
      <div class="md:w-1/2 md:px-5 xl:w-1/4">
        <%= image_tag "mew_idea.png", alt: "" %>　
        <h3>Sample Tomato</h3>
        <p>
          sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS sample texts SAMPLE TEXTS
        </p>
      </div>
    </div>
```

## 背景画像 ヘッダ画像

vhはCSSの単位で、「viewport height」
min-h-[45vh]は「最小の高さをビューポートの高さの45%にする」というレスポンシブ対応
少なくともデバイス画面の高さの45%の高さを持つ要素を作る
bg-cover: 背景画像がコンテナ全体をカバーするように設定
bg-center: 背景画像をコンテナの中央に配置するよう指定

```erb
<div class="min-h-[45vh] bg-[url('/assets/idea_header.png')] bg-cover bg-center">
</div>
```

## CSSフィルター　

ぼかし機能など

## デザイン

### 影

```erb
shadow-md shadow-black/60
```

丸み

```
rounded-2xl
```

テキストのぼかし背景

```
backdrop-blur-sm bg-white/60
```

TOPページ
設定したブロックが全画面表示になる。

```
min-h-screen
```

## Alpine.js

```bash
yarn add alpinejs
```

```
#app/javascript/application.js
import Alpine from 'alpinejs'

window.Alpine = Alpine
Alpine.start()
```

```
yarn build
```

## 不具合対応

### 設定が反映されない場合

safelistを設定すると反映される。
ただ、普通は使わないらしい。

## tailwind_merge


使えなかった。動作しなかった。
動的にスタイルを反映したい場合に使用する
`tailwind_merge` メソッドは可変長引数（splat operator `*`）を使って、任意の数のクラス文字列を受け取ります：

```erb
tailwind_merge(class_string1, class_string2, ..., class_stringN)
```

```
gem "tailwind_merge"
```

```ruby
#app/helpers/application_helper.rb
#全体で使用可能にする
module ApplicationHelper
  include TailwindMerge::Rails
end
```

## フォントの変更

GoogleFontで必要な情報を取ってくる
フォントのURLとか、フォント名とか
https://fonts.google.com/

```ruby
# app/views/layouts/application.html.erb
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Zen+Maru+Gothic&display=swap" rel="stylesheet">
```

```css
# app/assets/tailwind/application.css
@theme {
  /* 基本のsans-serifフォントを上書き */
  --font-sans: "Zen Maru Gothic", "Hiragino Sans", "Yu Gothic", sans-serif;
}
```

