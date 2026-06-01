# 5.4 CRUD 操作在 Next.js 中的常见位置

## 一句话

CRUD（创建、读取、更新、删除）操作应该集中在服务器端的特定位置，保证安全性和代码一致性。

## 为什么需要它

想象一个餐厅：如果顾客可以直接进入厨房操作厨具，会是什么混乱场面？CRUD 操作就是这些"厨具"——它们应该由专业的"厨师"（服务器代码）来操作，而不是由"顾客"（浏览器）随意摆布。把数据库操作放在错误的位置会导致安全问题、代码混乱和难以维护。Next.js 提供了明确的最佳实践来组织这些操作。

## 类比

| 概念 | 类比 |
|------|------|
| CRUD 操作 | 厨房操作（切菜、烹饪、摆盘） |
| 数据库 | 食材仓库 |
| 客户端代码 | 用餐区（顾客） |
| Server Component | 厨房（厨师操作） |
| Server Action | 点菜单（顾客通过它点菜，厨房执行） |
| Route Handler | 外卖服务（外卖单转给厨房） |
| API 路由 | 自助餐台（半自助服务） |

## 核心内容

### Next.js 中 CRUD 的标准位置

在 Next.js 中，CRUD 操作应该集中在以下三个位置：

```
✅ 推荐位置（按优先级排序）：

1. lib/ 目录（可复用查询函数）
   - lib/queries/user.ts
   - lib/mutations/post.ts

2. Server Actions（交互式操作）
   - app/actions/user.ts
   - app/actions/post.ts

3. Route Handlers（API 路由）
   - app/api/users/route.ts
   - app/api/posts/[id]/route.ts

❌ 避免的位置：
- components/ 目录（组件应该是无状态的展示层）
- hooks/ 目录（客户端 hooks 不应该直接访问数据库）
- 客户端组件内部（带 'use client' 的文件）
```

### 模式 1：lib/ 目录中的查询函数

这是最推荐的方式——将可复用的数据库操作封装在 `lib/` 目录中。

```typescript
// lib/queries/user.ts
import { prisma } from '@/lib/db'
import { cache } from 'react'

/**
 * 获取用户信息（带缓存）
 */
export const getUser = cache(async (id: string) => {
  return await prisma.user.findUnique({
    where: { id },
    select: {
      id: true,
      name: true,
      email: true,
      createdAt: true,
    },
  })
})

/**
 * 获取用户列表（分页）
 */
export async function getUsersPage(page: number = 1, limit: number = 10) {
  const skip = (page - 1) * limit

  const [users, total] = await Promise.all([
    prisma.user.findMany({
      skip,
      take: limit,
      orderBy: { createdAt: 'desc' },
      select: {
        id: true,
        name: true,
        email: true,
      },
    }),
    prisma.user.count(),
  ])

  return {
    users,
    pagination: {
      total,
      pages: Math.ceil(total / limit),
      currentPage: page,
    },
  }
}
```

**在 Server Component 中使用**：

```typescript
// app/users/page.tsx
import { getUsersPage } from '@/lib/queries/user'

export default async function UsersPage({
  searchParams,
}: {
  searchParams: { page?: string }
}) {
  const page = Number(searchParams.page) || 1
  const { users, pagination } = await getUsersPage(page)

  return (
    <div>
      <h1>用户列表</h1>
      <ul>
        {users.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
      <div>
        当前页：{pagination.currentPage} / {pagination.pages}
      </div>
    </div>
  )
}
```

**为什么使用 `cache` 函数**：
- React 的 `cache` 函数确保相同的参数只查询一次
- 在同一个请求中多次调用同一查询时，直接返回缓存结果
- 减少数据库压力，提高性能

### 模式 2：Server Actions 中的写操作

对于需要用户交互的写操作（创建、更新、删除），使用 Server Actions。

```typescript
// app/actions/user.ts
'use server'

import { prisma } from '@/lib/db'
import { revalidatePath } from 'next/cache'
import { redirect } from 'next/navigation'

/**
 * 创建新用户
 */
export async function createUser(formData: FormData) {
  const name = formData.get('name') as string
  const email = formData.get('email') as string

  // ✅ 在 Server Action 中可以安全地访问数据库
  const user = await prisma.user.create({
    data: { name, email },
  })

  // 重新验证缓存（刷新页面数据）
  revalidatePath('/users')

  // 可选：重定向到新用户页面
  redirect(`/users/${user.id}`)
}

/**
 * 更新用户信息
 */
export async function updateUser(id: string, formData: FormData) {
  const name = formData.get('name') as string

  await prisma.user.update({
    where: { id },
    data: { name },
  })

  revalidatePath('/users')
  revalidatePath(`/users/${id}`)
}

/**
 * 删除用户
 */
export async function deleteUser(id: string) {
  await prisma.user.delete({
    where: { id },
  })

  revalidatePath('/users')
  redirect('/users')
}
```

**在表单中使用**：

```typescript
// app/users/page.tsx
import { createUser, deleteUser } from '@/app/actions/user'

export default function UsersPage() {
  return (
    <div>
      {/* 创建用户表单 */}
      <form action={createUser}>
        <input name="name" placeholder="姓名" required />
        <input name="email" placeholder="邮箱" required />
        <button type="submit">创建</button>
      </form>

      {/* 用户列表 + 删除按钮 */}
      <UserList />
    </div>
  )
}

function UserList() {
  const users = [] // 假设从某处获取

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>
          {user.name}
          <form action={deleteUser.bind(null, user.id)}>
            <button type="submit">删除</button>
          </form>
        </li>
      ))}
    </ul>
  )
}
```

### 模式 3：Route Handlers（API 路由）

当需要为第三方提供 API 或实现特定的 REST 端点时使用。

```typescript
// app/api/users/route.ts
import { prisma } from '@/lib/db'
import { NextResponse } from 'next/server'

/**
 * GET /api/users - 获取用户列表
 */
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url)
  const page = Number(searchParams.get('page')) || 1
  const limit = Number(searchParams.get('limit')) || 10

  const users = await prisma.user.findMany({
    skip: (page - 1) * limit,
    take: limit,
  })

  return NextResponse.json({ users })
}

/**
 * POST /api/users - 创建新用户
 */
export async function POST(request: Request) {
  const body = await request.json()
  const user = await prisma.user.create({
    data: {
      name: body.name,
      email: body.email,
    },
  })

  return NextResponse.json(user, { status: 201 })
}
```

```typescript
// app/api/users/[id]/route.ts
import { prisma } from '@/lib/db'
import { NextResponse } from 'next/server'

/**
 * GET /api/users/:id - 获取单个用户
 */
export async function GET(
  request: Request,
  { params }: { params: { id: string } }
) {
  const user = await prisma.user.findUnique({
    where: { id: params.id },
  })

  if (!user) {
    return NextResponse.json({ error: 'User not found' }, { status: 404 })
  }

  return NextResponse.json(user)
}

/**
 * PUT /api/users/:id - 更新用户
 */
export async function PUT(
  request: Request,
  { params }: { params: { id: string } }
) {
  const body = await request.json()
  const user = await prisma.user.update({
    where: { id: params.id },
    data: body,
  })

  return NextResponse.json(user)
}

/**
 * DELETE /api/users/:id - 删除用户
 */
export async function DELETE(
  request: Request,
  { params }: { params: { id: string } }
) {
  await prisma.user.delete({
    where: { id: params.id },
  })

  return NextResponse.json({ success: true })
}
```

### 三种模式的选择指南

| 场景 | 推荐方式 | 原因 |
|------|---------|------|
| 在多个 Server Component 中复用查询 | lib/queries/ | 避免代码重复，便于缓存 |
| 表单提交、用户交互操作 | Server Actions | 原生表单支持，自动处理状态 |
| 为第三方提供 API | Route Handlers | 标准 REST API，便于外部调用 |
| 需要复杂的权限控制 | Route Handlers | 灵活的请求/响应处理 |
| 简单的 CRUD | Server Actions | 更少的代码，更好的性能 |

### 常见错误示例

**❌ 错误 1：在客户端组件中直接访问数据库**

```typescript
'use client'

import { prisma } from '@/lib/db' // 这会导致运行时错误

export function UserList() {
  // ❌ 客户端组件无法访问服务器端的数据库
  const users = await prisma.user.findMany()
  return <div>{/* ... */}</div>
}
```

**❌ 错误 2：在组件中混合查询和业务逻辑**

```typescript
// app/users/page.tsx
export default async function UsersPage() {
  // ❌ 应该把查询逻辑提取到 lib/queries/user.ts
  const users = await prisma.user.findMany({
    where: { age: { gte: 18 } },
    orderBy: { createdAt: 'desc' },
  })

  const total = await prisma.user.count({ where: { age: { gte: 18 } } })

  return <div>{/* ... */}</div>
}
```

**❌ 错误 3：在客户端 hooks 中访问数据库**

```typescript
// hooks/useUser.ts
export function useUser(id: string) {
  // ❌ hooks 在客户端执行，无法访问数据库
  const [user, setUser] = useState(null)

  useEffect(() => {
    async function fetchUser() {
      const response = await fetch(`/api/users/${id}`)
      const data = await response.json()
      setUser(data)
    }
    fetchUser()
  }, [id])

  return user
}
```

### 项目结构推荐

```
your-app/
├── lib/
│   ├── db.ts                    # Prisma Client 初始化
│   ├── queries/                 # 读操作
│   │   ├── user.ts
│   │   └── post.ts
│   └── mutations/               # 写操作（可选）
│       ├── user.ts
│       └── post.ts
├── app/
│   ├── actions/                # Server Actions
│   │   ├── user.ts
│   │   └── post.ts
│   ├── api/                     # Route Handlers
│   │   ├── users/
│   │   │   ├── route.ts
│   │   │   └── [id]/
│   │   │       └── route.ts
│   │   └── posts/
│   └── users/
│       ├── page.tsx            # 使用 lib/queries/user.ts
│       └── [id]/
│           └── page.tsx
└── components/
    └── UserCard.tsx             # 纯展示组件，无数据操作
```

## 你需要记住的

1. CRUD 操作应该集中在服务器端的特定位置，避免散落在各处
2. lib/ 目录用于可复用的查询函数，Server Actions 用于交互操作，Route Handlers 用于 API 端点
3. 客户端组件和 hooks 不应该直接访问数据库
4. 使用 `cache` 函数包装查询函数以提高性能
5. Server Actions 会自动处理 `revalidatePath` 和 `redirect`，简化状态管理

## AI 代码中的线索

当 AI 生成的代码包含 CRUD 操作时，查看以下线索判断位置是否正确：

**正确模式**：
```typescript
// lib/queries/ - 查询函数
export async function getUser(id: string) { /* ... */ }

// app/actions/ - Server Actions
export async function createUser(formData: FormData) { /* ... */ }

// app/api/ - Route Handlers
export async function GET(request: Request) { /* ... */ }
```

**危险信号**：
```typescript
// ❌ 在客户端组件中导入 Prisma
'use client'
import { prisma } from '@/lib/db'

// ❌ 在 hooks 中直接查询数据库
export function useUser(id: string) {
  const user = await prisma.user.findUnique({ where: { id } })
}

// ❌ 在组件中混合业务逻辑
export default async function Page() {
  const users = await prisma.user.findMany()
  // 应该提取到 lib/queries/
}
```

## 验证问题

- [ ] 为什么不应该在客户端组件中直接访问数据库？这会导致什么问题？
- [ ] lib/queries/、Server Actions、Route Handlers 这三种位置分别适用于什么场景？
- [ ] 如何判断 AI 生成的代码是否把 CRUD 操作放在了正确的位置？
