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

