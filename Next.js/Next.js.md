# Next.js

React.jsを基にしたWebアプリケーション開発のためのフレームワーク

## プライベートフォルダについて

App Router では、アンダースコア（_）で始まるフォルダは「プライベートフォルダ」として扱われる。
URL ルーティングの対象にならない
ページの部品（コンポーネント）としてのみ利用される

- src/components/（共通コンポーネント）
  複数ページで共通して使うコンポーネントを置く場所です。
  ヘッダーやフッター、共通ボタン、モーダル、レイアウト部品をまとめます。
- src/app/blog/_components/（ページ専用コンポーネント）
  そのページ専用のコンポーネントを置く場所です。
  投稿一覧（PostList）、投稿の詳細カード、検索フォームなどを配置します。

## src/types/（型定義）について

アプリ全体で利用する TypeScriptの型定義 をまとめる場所です。
投稿データの型（Post）、ユーザー情報の型（User）、API レスポンスの型などを定義します

## src/lib/（ロジック・ユーティリティ）について

UI に依存しないビジネスロジックやユーティリティ関数を置く場所です。
データ取得関数（DB や API 呼び出し）、日付フォーマット関数、文字列変換などの共通処理をまとめます。

## Server ComponentとClient Component

先頭に何も書かないとServer Componentとして扱われる。
Server ComponentからClient Componentを実行するのは普通。
　※逆はできない。
だから基本はServer Componentでユーザーの操作が入る部分だけClient Componentとする、。

### Server Component

サーバー側で実行される React コンポーネント。
fetch や DBアクセスなどの重たい処理に適しており、パフォーマンスやセキュリティに優れる。

### Client Component

useStateやuseEffectなどのReactフックを利用して動的に状態を管理・更新できる。
ファイルの先頭にuse clientを書く必要がある。

## useActionState による状態管理

フォームを送信したときの状態（成功・失敗・処理中）を管理できる React のフック
次のような利点があります。

- サーバーで処理された結果（成功・エラー）をクライアントに自然に返せる
- 状態をリアクティブに反映でき、UIとの親和性が高い
- 送信中 (`isPending`) やバリデーションエラーを自然に表示できる

```tsx
const [state, formAction, isPending] = useActionState(action, initialState, permalink?);
```

**基本的な使い方**

- 第1引数にサーバー側で処理する関数、第2引数に状態の初期値を指定します。第3引数はオプションでページの URL などを表す文字列を指定します。
- permalink は、フォーム送信後にブラウザの「戻る」ボタンやリロード時の動作を正しく保つために使う URLです。
- 返り値は以下です。
  - state: エラーやサーバー処理後の状態
  - formAction: フォームの `action` に渡す関数。これが submit 時に呼ばれます。
  - isPending: 処理中かどうかを示すフラグ

## ServerAction

Server Actionsは、サーバー上で実行される非同期関数
use serverを先頭に記述することで Server Action になります。
クライアントの <form> から送信された FormData を、Server Action 側で受け取ります。

Promise<FormState>について、
Promiseはジェネリック。ジェネリックは型を後から決められる、型の入れ物。Arrayとかと一緒。
戻り値がFormStateだけど、非同期だからPromiseのジェネリックに入っている。
後で入れて返却する約束。

```ts
// my-app/src/app/actions/post.ts
'use server'

import { redirect } from 'next/navigation'
import { promises as fs } from 'fs'
import path from 'path'
import { Post } from '@/types/post'

const filePath = path.join(process.cwd(), 'src/data/posts.json')

type FormState = {
  error?: string
}

// Server Actionが非同期だからasyncを書く必要がある
export async function createPost(
  _prevState: FormState,
  formData: FormData,
): Promise<FormState> { //このPromise<FormState>が戻り値の定義
  const title = formData.get('title') as string
  const content = formData.get('content') as string

  if (!title || !content) {
    return { error: 'タイトルと内容は必須です。' }
  }

  // 現在の投稿一覧を読み込み
  const data = await fs.readFile(filePath, 'utf-8')
  const posts: Post[] = JSON.parse(data)

  // 新規投稿を作成
  const newPost: Post = {
    id: String(posts.length + 1),
    title,
    content,
    date: new Date().toISOString().slice(0, 10),
  }

  // 新しい投稿を追加してファイルに書き込み
  posts.push(newPost)
  try {
    await fs.writeFile(filePath, JSON.stringify(posts, null, 2), 'utf-8')
  } catch {
    return { error: '保存に失敗しました。' }
  }

  redirect('/blog')
}
```

### FormData APIとinputの連携

- `<input name='title' />` のように name 属性を付けると、`formData.get('title')` で取得できます。
- name属性が `FormData` と連携するため必須です。

```ts
const title = formData.get('title') as string
const content = formData.get('content') as string
```

## Reactのメソッドについて

```ts
```

## loading.tsx の導入

ページの読み込み中に白画面ではなく「読み込み中です…」と表示することができる
```tsx
// src/app/loading.tsx
export default function Loading() {
  return <div className="p-6 text-center text-gray-500">読み込み中です…</div>
}
```

## error.tsx の導入

エラー発生時にユーザーに明確な通知と再試行ボタンを提供

```tsx
// src/app/error.tsx
'use client'

export default function Error({
  error,
  reset,
}: {
  error: Error
  reset: () => void
}) {
  return (
    <div className="p-6 text-center text-red-600">
      エラーが発生しました：{error.message}
      <br />
      <button
        onClick={() => reset()}
        className="mt-4 rounded bg-red-500 px-4 py-2 text-white"
      >
        再読み込み
      </button>
    </div>
  )
}
```

