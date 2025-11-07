# API

```tsx
// src/app/api/posts/route.ts
import { db } from "@/db"
import { posts, users } from "@/db/schema"
import { desc, eq } from "drizzle-orm"
import { NextRequest, NextResponse } from "next/server"
import { auth } from "@/auth"

// 投稿一覧を取得
export async function GET(request: NextRequest) {
  try {
    const { searchParams } = new URL(request.url)
    const limit = parseInt(searchParams.get('limit') || '12')
    const offset = parseInt(searchParams.get('offset') || '0')

    const allPosts = await db
      .select({
        id: posts.id,
        title: posts.title,
        description: posts.description,
        updated_at: posts.updated_at,
        created_at: posts.created_at,
        user: {
          id: users.id,
          name: users.name,
        },
      })
      .from(posts)
      .leftJoin(users, eq(posts.user_id, users.id))
      .orderBy(desc(posts.created_at))
      .limit(limit)
      .offset(offset)

    return NextResponse.json(allPosts)
  } catch (error) {
    console.error("投稿取得エラー:", error)
    return NextResponse.json(
      { error: "投稿の取得に失敗しました" },
      { status: 500 }
    )
  }
}

// 新規投稿を作成
export async function POST(request: NextRequest) {
  try {
    const { title, description } = await request.json()

    if (!title?.trim()) {
      return NextResponse.json(
        { error: "タイトルは必須です" },
        { status: 400 }
      )
    }

    // セッションからユーザーIDを取得
    const session = await auth()

    if (!session?.user?.id) {
      return NextResponse.json(
        { error: "認証が必要です" },
        { status: 401 }
      )
    }

    const userId = session.user.id

    const newPost = await db
      .insert(posts)
      .values({
        title: title.trim(),
        description: description?.trim() || null,
        user_id: userId,
      })
      .returning()

    return NextResponse.json(newPost[0], { status: 201 })
  } catch (error) {
    console.error("投稿作成エラー:", error)
    return NextResponse.json(
      { error: "投稿の作成に失敗しました" },
      { status: 500 }
    )
  }
}
```

