# TailwindCSS

```
gem 'tailwindcss-rails'
```

app\assets\stylesheets\application.tailwind.css

```
@import "tailwindcss";
@tailwind base;
@tailwind components;
@tailwind utilities;
```

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
