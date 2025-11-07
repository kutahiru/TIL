# Auth.js

## インストール

```bash
npm install next-auth@beta
```

## 環境変数

google利用時はGoogle Cloud Platformで作成する

```ts
// .env.local
AUTH_SECRET="" //npx auth secretを実行する

AUTH_GOOGLE_ID=""
AUTH_GOOGLE_SECRET=""
```

## オプション設定

Authは環境変数を自動的に取得するため、手動で渡す必要はない。
Drizzle ORM Adapterを使用する場合
https://authjs.dev/getting-started/adapters/drizzle

```bash
npm install @auth/drizzle-adapter
```

```ts
// src/auth.ts
import NextAuth from "next-auth"
import Google from "next-auth/providers/google"
import { DrizzleAdapter } from "@auth/drizzle-adapter"
import { db } from "@/db"
//sessionsはJWTでするし、verificationTokensは不要
//import { users, accounts, sessions, verificationTokens } from "@/db/schema"
import { users, accounts } from "@/db/schema"

export const { handlers, auth, signIn, signOut } = NextAuth({
  adapter: DrizzleAdapter(db, {
    usersTable: users,
    accountsTable: accounts,
  }),
  session: { strategy: "jwt" },
  providers: [
    Google({
      clientId: process.env.AUTH_GOOGLE_ID!,
      clientSecret: process.env.AUTH_GOOGLE_SECRET!,
    }),
  ],
  pages: {
    signIn: '/auth/signin',
  },
  callbacks: {
    jwt({ token, user, trigger, session }) {
      if (user) {
        token.id = user.id
        token.name = user.name
      }

      // update()が呼ばれた時の処理
      if (trigger === "update" && session?.name) {
        token.name = session.name
      }

      return token
    },
    session({ session, token }) {
      if (token.id) {
        session.user.id = token.id as string
      }
      if (token.name) {
        session.user.name = token.name as string
      }
      return session
    },
  },
})
```

このエンドポイントで認証周りの API リクエストをキャッチし、
サインイン/サインアウト処理、セッション管理、コールバック処理などを行う。

```ts
// src/app/api/auth/[...nextauth]/route.ts
import { handlers } from "@/auth"

export const { GET, POST } = handlers
```

```tsx
// src/app/auth/signin/page.tsx
import { signIn, getProviders } from 'next-auth/react'

・
・
・

<button
  onClick={() => signIn('google', { callbackUrl: '/' })}
  className="・・・"
>ログインボタン</button>
```

