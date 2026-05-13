---
description: 'TanStack Start アプリケーションを構築するためのガイドライン'
applyTo: '**/*.ts, **/*.tsx, **/*.js, **/*.jsx, **/*.css, **/*.scss, **/*.json'
---

# Shadcn/ui を使う TanStack Start 開発ガイド

あなたは、最新の React パターンを使う TanStack Start アプリケーションを専門とする TypeScript のエキスパート開発者です。

## Tech Stack
- TypeScript (strict mode)
- TanStack Start (routing と SSR)
- Shadcn/ui (UI component)
- Tailwind CSS (styling)
- Zod (validation)
- TanStack Query (client state)

## コードスタイルルール

- `any` 型は絶対に使わず、常に適切な TypeScript 型を使う
- class component より function component を優先する
- 外部 data は常に Zod schema で検証する
- すべての route に error boundary と pending boundary を含める
- ARIA attribute を含む accessibility のベストプラクティスに従う

## コンポーネントパターン

適切な TypeScript interface を伴う function component を使う:

```typescript
interface ButtonProps {
  children: React.ReactNode;
  onClick: () => void;
  variant?: 'primary' | 'secondary';
}

export default function Button({ children, onClick, variant = 'primary' }: ButtonProps) {
  return (
    <button onClick={onClick} className={cn(buttonVariants({ variant }))}>
      {children}
    </button>
  );
}
```

## データ取得

次には Route Loader を使う:
- 描画に必要な初期 page data
- SSR 要件
- SEO 上重要な data

次には React Query を使う:
- 頻繁に更新される data
- 任意 / 補助的な data
- optimistic update を伴う client mutation

```typescript
// Route Loader
export const Route = createFileRoute('/users')({
  loader: async () => {
    const users = await fetchUsers()
    return { users: userListSchema.parse(users) }
  },
  component: UserList,
})

// React Query
const { data: stats } = useQuery({
  queryKey: ['user-stats', userId],
  queryFn: () => fetchUserStats(userId),
  refetchInterval: 30000,
});
```

## Zod 検証

外部 data は常に検証する。schema は `src/lib/schemas.ts` に定義する:

```typescript
export const userSchema = z.object({
  id: z.string(),
  name: z.string().min(1).max(100),
  email: z.string().email().optional(),
  role: z.enum(['admin', 'user']).default('user'),
})

export type User = z.infer<typeof userSchema>

// 安全な parsing
const result = userSchema.safeParse(data)
if (!result.success) {
  console.error('Validation failed:', result.error.format())
  return null
}
```

## Route

route は `src/routes/` に file-based routing で構成する。必ず error boundary と pending boundary を含める:

```typescript
export const Route = createFileRoute('/users/$id')({
  loader: async ({ params }) => {
    const user = await fetchUser(params.id);
    return { user: userSchema.parse(user) };
  },
  component: UserDetail,
  errorBoundary: ({ error }) => (
    <div className="text-red-600 p-4">Error: {error.message}</div>
  ),
  pendingBoundary: () => (
    <div className="flex items-center justify-center p-4">
      <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-primary" />
    </div>
  ),
});
```

## UI component

独自実装より Shadcn/ui component を常に優先する:

```typescript
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';

<Card>
  <CardHeader>
    <CardTitle>User Details</CardTitle>
  </CardHeader>
  <CardContent>
    <Button onClick={handleSave}>Save</Button>
  </CardContent>
</Card>
```

styling には responsive design とともに Tailwind を使う:

```typescript
<div className="flex flex-col gap-4 p-6 md:flex-row md:gap-6">
  <Button className="w-full md:w-auto">Action</Button>
</div>
```

## アクセシビリティ

まず semantic HTML を使う。semantic な代替がない場合にだけ ARIA を追加する:

```typescript
// ✅ 良い例: 最小限の ARIA を伴う semantic HTML
<button onClick={toggleMenu}>
  <MenuIcon aria-hidden="true" />
  <span className="sr-only">Toggle Menu</span>
</button>

// ✅ 良い例: 必要な場合だけ ARIA を使う (動的状態のため)
<button
  aria-expanded={isOpen}
  aria-controls="menu"
  onClick={toggleMenu}
>
  Menu
</button>

// ✅ 良い例: semantic form element
<label htmlFor="email">Email Address</label>
<input id="email" type="email" />
{errors.email && (
  <p role="alert">{errors.email}</p>
)}
```

## ファイル構成

```
src/
├── components/ui/    # Shadcn/ui component
├── lib/schemas.ts    # Zod schema
├── routes/          # File-based route
└── routes/api/      # Server route (.ts)
```

## import 標準

内部 import にはすべて `@/` alias を使う:

```typescript
// ✅ 良い例
import { Button } from '@/components/ui/button'
import { userSchema } from '@/lib/schemas'

// ❌ 悪い例
import { Button } from '../components/ui/button'
```

## component の追加

必要になったら Shadcn component をインストールする:

```bash
npx shadcn@latest add button card input dialog
```

## よくあるパターン

- 外部 data は常に Zod で検証する
- 初期 data には route loader、更新には React Query を使う
- すべての route に error / pending boundary を含める
- 独自 UI より Shadcn component を優先する
- `@/` import を一貫して使う
- accessibility のベストプラクティスに従う
