# TypeScript

## 型定義の管理方法

src/typesで型を一元管理する
```ts
// src/types/user.ts
export type User = {
  id: number;
  name: string;
  age: number;
  personalColor: string;
};
```

```tsx
// App.tsx
import type { User } from ".types/user";

//これで型定義を使用できる
```

