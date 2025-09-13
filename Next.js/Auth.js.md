# Auth.js

## インストール

```bash
npm install next-auth
```

## 環境変数

```ts
// .env.local
AUTH_GOOGLE_ID={CLIENT_ID}
AUTH_GOOGLE_SECRET={CLIENT_SECRET}
```

## オプション設定

Authは環境変数を自動的に取得するため、手動で渡す必要はない。
Drizzle ORM Adapterを使用する場合
https://authjs.dev/getting-started/adapters/drizzle

```ts
// app/lib/next-auth/option.ts
import NextAuth from "next-auth"
import Google from "next-auth/providers/google"
import { DrizzleAdapter } from "@auth/drizzle-adapter"
import { db, users, accounts, sessions, verificationTokens } from "@/lib/db/schema"

export const { handlers, auth, signIn, signOut } = NextAuth({
  adapter: DrizzleAdapter(db, {
    usersTable: users,
    accountsTable: accounts,
    sessionsTable: sessions,
  }),
  providers: [
    Google,
  ],
})
```

```tsx
// ./components/サインイン.tsx
import { signIn } from "@/auth"
 
export default function SignIn() {
  return (
    <form
      action={async () => {
        "use server"
        await signIn("google")
      }}
    >
      <button type="submit">Signin with Google</button>
    </form>
  )
} 
```

