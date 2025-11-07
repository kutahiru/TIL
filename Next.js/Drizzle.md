# Drizzle

```bash
npm install drizzle-orm postgres drizzle-kit
```

```ts
// .env.local
DATABASE_URL="postgresql://postgres:password@localhost:5432/myapp_development"
```

```ts
// drizzle.config.ts
import { defineConfig } from 'drizzle-kit';

export default defineConfig({
  schema: './src/db/schema.ts',
  out: './drizzle',
  dialect: 'postgresql',
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
});
```

```ts
// src/db/schema.ts

```

```ts
// Drizzleのクライアントを生成
// src/db/index.ts
import { drizzle } from 'drizzle-orm/postgres-js';
import postgres from 'postgres';
import * as schema from './schema';

const connectionString = process.env.DATABASE_URL!;

// Disable prefetch as it is not supported for "Transaction" pool mode
export const client = postgres(connectionString, { prepare: false });
export const db = drizzle(client, { schema });
```

```json
// package.json
  "scripts": {
    "db:generate": "drizzle-kit generate",
    "db:migrate": "drizzle-kit migrate",
    "db:push": "drizzle-kit push",
    "db:studio": "drizzle-kit studio"
  },

  - npm run db:generate - マイグレーション生成
  - npm run db:migrate - マイグレーション実行
  - npm run db:push - スキーマ直接プッシュ
  - npm run db:studio - GUI管理ツール起動
```

