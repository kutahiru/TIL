# Prisma

```bash
npm install prisma
npm i @prisma/client

npx prisma init
```

```prisma
// prisma/schema.prisma
model Contact {
  id  String  @id @default(cuid())
  name String
  email String  @unique
  createdAt DateTime @default(now())
  updatedAt DateTime  @updatedAt
}
```

```bash
npx prisma migrate dev --name init
```

```bash
// 定義の確認等ができる
npx prisma studio
```

## prismaインスタンスの生成

```ts
// src/lib/prisma.ts
import { PrismaClient } from '@prisma/client'

// グローバルスコープでPrismaインスタンスを保持できる場所を作る
const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined
}
// Prismaインスタンスがあれば使う、なければ作成
export const prisma = globalForPrisma.prisma ?? new PrismaClient()
// 開発環境でのみ使用
if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma
```

## Prismaの操作

prisma.モデル名.メソッドでDB操作できる

```ts
// src/lib/actions/contact.ts
"use server"
import { redirect } from "next/navigation"
import { ContactSchema } from "../../validations/contact"
import { prisma } from '@/lib/prisma';

// ActionStateの型定義
type ActionState = {
  success: boolean;
  errors: {
    name?: string[];
    email?: string[];
  };
  serverError?: string;
}

export async function submitContactForm(
  prevState: ActionState,
  formData: FormData
): Promise<ActionState> 
{
  const name = formData.get("name") as string
  const email = formData.get("email") as string

  // バリデーション
  const validationResult = ContactSchema.safeParse({name, email})

  if(!validationResult.success)
  {
    const errors = validationResult.error.flatten().fieldErrors
    console.log("サーバー側でエラー", errors)
    return {
      success: false,
      errors: {
        name: errors.name || [],
        email: errors.email || [],
      }
    }
  }

  // DB登録
  // メールアドレスの存在確認
  const existingRecord = await prisma.contact.findUnique({ 
    where: {email: email}
  })

  if (existingRecord){
    return {
      success: false,
      errors: {
        name: [],
        email: ["このメールアドレスはすでに登録されています"]
      }
    }
  }

  await prisma.contact.create({
    data: {name, email}
  })

  console.log("送信されたデータ:", {name, email})
  redirect('/contacts/complete')
}

```

## 抽出

```ts
import { getContacts, getContact } from "../../../lib/contact"

export default async function ListPage() {
  const contacts = await getContacts()
  const first = await getContact("1")

  return (
    <div>
      複数
      <ul>
        {contacts.map((contact)=>(
          <li key={contact.id}>
            {contact.name}:{contact.email}
          </li>
        ))}
      </ul>

      1件
      <div>
        { first ? first?.name : "登録されていません"}
      </div>
    </div>
  )
}
```



