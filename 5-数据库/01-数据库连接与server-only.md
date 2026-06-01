# 5.1 数据库连接与 server-only

## 一句话
数据库连接必须在服务器端建立，绝不能在客户端代码中暴露敏感信息。

## 为什么需要它

想象一下，你把家门钥匙贴在窗户上——任何路过的人都能拿它开门。在数据库连接中，连接字符串就是那把"钥匙"。如果把它写在客户端代码里，任何人打开浏览器开发者工具就能看到你的数据库密码。这不仅会泄露数据，还可能导致整个数据库被黑客控制。

## 类比

| 概念 | 类比 |
|------|------|
| 数据库连接字符串 | 家门钥匙 |
| 客户端代码 | 大街上 |
| server-only 标记 | 私人保险箱（只能在家打开） |
| 数据库 | 家（存放贵重物品的地方） |
| 服务器 | 你的家（安全的地方） |

## 核心内容

### 数据库连接的基本概念

数据库连接是一个需要认证的网络连接。在 Next.js 中，这个连接必须在服务器端建立，原因如下：

1. **安全性**：连接字符串包含用户名、密码等敏感信息，不能发送到浏览器
2. **性能**：服务器端的连接可以被复用（连接池），客户端每次都重建连接效率低
3. **权限**：服务器可以控制哪些操作被允许，客户端无法绕过这个限制

### Prisma 客户端的正确初始化

Prisma 是一个现代化的数据库 ORM（对象关系映射）工具，它提供类型安全的 API 来操作数据库，无需手写 SQL（这在 5.3 会详细讲解）。在这里，我们使用 Prisma 来建立和管理数据库连接。

```typescript
// lib/db.ts - 这个文件只能在服务器端使用
import { PrismaClient } from '@prisma/client'

// Prisma 客户端单例模式（防止开发环境下创建多个实例）
const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined
}

export const prisma = globalForPrisma.prisma ?? new PrismaClient()

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma

// 使用示例
export async function getUser(id: string) {
  return await prisma.user.findUnique({ where: { id } })
}
```

**代码解读**：
- `PrismaClient` 是 Prisma 提供的数据库访问客户端
- 单例模式确保在开发热重载时不会创建多个连接实例（这会导致数据库连接数耗尽）
- `globalThis` 是全局对象，在 Node.js 环境中用于跨模块共享状态

### server-only 包的作用

`server-only` 是一个特殊的 npm 包，它的作用是在开发环境下抛出错误，提醒开发者这个模块不应该被导入到客户端代码中。

```typescript
// lib/db.ts 的第一行
import 'server-only'

import { PrismaClient } from '@prisma/client'
// ... 其他代码
```

如果这个文件被客户端组件导入：

```typescript
// ❌ 错误示例：在客户端组件中导入数据库代码
'use client'

import { getUser } from '@/lib/db' // 这会在开发环境报错！

export function UserProfile({ userId }: { userId: string }) {
  // ...
}
```

开发环境会看到这个错误：

```
⨯ Error: This module cannot be imported in client components.
It can only be used in Server Components, Server Actions, or Route Handlers.
```

**为什么需要这个保护**：
- 开发时很容易无意中把服务器代码导入到客户端
- `server-only` 提供早期警告，避免安全问题进入生产环境
- 它像一道安全网，防止敏感代码泄露到浏览器

### 数据库连接的正确位置

在 Next.js 中，数据库相关的代码应该放在以下位置：

```
✅ 正确位置：
lib/db.ts              # 数据库客户端初始化
lib/queries/user.ts    # 用户相关的查询函数
app/api/users/route.ts # API 路由（服务器端）
app/users/page.tsx     # Server Component（服务器端）

❌ 错误位置：
components/UserCard.tsx  # 如果这是客户端组件
hooks/useUser.ts         # hooks 通常在客户端使用
```

### 在 Server Component 中使用数据库

```typescript
// app/users/page.tsx - 这是一个 Server Component
import { prisma } from '@/lib/db'

export default async function UsersPage() {
  // ✅ 可以直接在 Server Component 中查询数据库
  const users = await prisma.user.findMany()

  return (
    <div>
      <h1>用户列表</h1>
      <ul>
        {users.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  )
}
```

### 在 Server Action 中使用数据库

```typescript
// app/actions/user.ts
'use server'

import { prisma } from '@/lib/db'
import { z } from 'zod'

const createUserSchema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
})

export async function createUser(formData: FormData) {
  const data = createUserSchema.parse(Object.fromEntries(formData))

  // ✅ 在 Server Action 中可以安全地访问数据库
  const user = await prisma.user.create({
    data: {
      name: data.name,
      email: data.email,
    },
  })

  return user
}
```

### 环境变量的保护

数据库连接字符串应该存储在环境变量中：

```bash
# .env.local（这个文件不应该提交到 Git）
DATABASE_URL="postgresql://user:password@localhost:5432/mydb"
```

```typescript
// lib/db.ts
const prisma = new PrismaClient({
  datasources: {
    db: {
      url: process.env.DATABASE_URL, // 从环境变量读取
    },
  },
})
```

**关键点**：
- `.env.local` 文件只在服务器上存在，浏览器无法访问
- `process.env` 只在服务器端可用，客户端组件中读取它会得到 `undefined`

## 你需要记住的

1. 数据库连接代码必须在服务器端执行，绝不能发送到浏览器
2. `server-only` 包提供开发环境下的安全检查，防止误用
3. 连接字符串等敏感信息存储在环境变量中，不写死在代码里
4. 数据库查询函数应该放在 `lib/` 目录下，便于复用和管理
5. Server Component 和 Server Actions 是访问数据库的正确位置

## AI 代码中的线索

当 AI 生成代码时，以下线索表明它在使用数据库连接：

1. **导入语句**：`import { prisma } from '@/lib/db'` 或 `import { PrismaClient } from '@prisma/client'`
2. **文件位置**：数据库操作通常在 `lib/`、`app/api/` 或带 `'use server'` 的文件中
3. **server-only 导入**：文件开头有 `import 'server-only'`
4. **环境变量使用**：代码中出现 `process.env.DATABASE_URL` 或类似的配置

**危险信号**：
- 客户端组件（有 `'use client'`）中直接导入数据库操作
- 连接字符串硬编码在代码中（如 `const db = new Database('postgresql://...')`）
- 没有 `server-only` 保护的数据库工具函数

## 验证问题

- [ ] 为什么数据库连接代码必须在服务器端执行？如果不这样做会有什么后果？
- [ ] `server-only` 包的作用是什么？它在开发环境和生产环境的行为有何不同？
- [ ] 在 Next.js 中，哪些类型的组件/函数可以安全地访问数据库？
