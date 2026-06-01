# 5.3 Prisma ORM 基础

## 一句话

Prisma 是一个现代化的数据库 ORM 工具，让你用类型安全的方式操作数据库，不需要手写 SQL 语句。

## 为什么需要它

直接写 SQL 很繁琐且容易出错：你需要处理字符串拼接、类型转换、SQL 注入风险，而且没有类型检查。Prisma 解决了这些问题——它提供一套类型安全的 API 来操作数据库，自动生成 TypeScript 类型，还能在数据库结构变化时自动同步。你用对象的方式思考数据，Prisma 负责转换成数据库能理解的 SQL。

## 类比

| 概念 | 类比 |
|------|------|
| 数据库 | 仓库（实际存放货物的地方） |
| SQL | 用仓库原生语言写的操作手册（难读难写） |
| Prisma | 仓库管理APP（图形界面，自动翻译） |
| Prisma Schema | 货物登记表（定义存储什么类型的东西） |
| Prisma Client | 仓库管理员（你和他打交道，他和仓库打交道） |
| 迁移 | 调整仓库布局的施工方案（自动生成） |

## 核心内容

### 什么是 ORM

ORM（Object-Relational Mapping）对象关系映射，是在编程语言对象和数据库表之间建立映射的技术。Prisma 是 Next.js 生态中最流行的 ORM。

**传统 SQL vs Prisma**：

```typescript
// ❌ 传统 SQL（繁琐、不安全）
const result = await db.query(
  'SELECT * FROM users WHERE email = $1',
  [email]
)

// ✅ Prisma（简洁、类型安全）
const user = await prisma.user.findUnique({
  where: { email },
})
```

### Prisma 的三个核心组件

```
Prisma 生态系统
├── Prisma Schema（schema.prisma）     # 数据模型定义
├── Prisma Client（自动生成）          # 类型安全的数据库客户端
└── Prisma Migrate（迁移工具）         # 数据库结构变更管理
```

### Prisma Schema 基础

```prisma
// prisma/schema.prisma

// 数据库连接配置
datasource db {
  provider = "postgresql"   // 数据库类型
  url      = env("DATABASE_URL") // 从环境变量读取
}

// 生成配置
generator client {
  provider = "prisma-client-js"
  previewFeatures = ["postgresqlExtensions"] // 可选：预览特性
}

// 数据模型定义
model User {
  id        String   @id @default(uuid())
  email     String   @unique
  name      String?
  age       Int?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // 关系定义
  posts     Post[]
}

model Post {
  id        String   @id @default(uuid())
  title     String
  content   String?
  published Boolean  @default(false)
  authorId  String

  // 关系定义
  author    User     @relation(fields: [authorId], references: [id])

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

**字段类型映射**：

| Prisma 类型 | PostgreSQL 类型 | TypeScript 类型 |
|-------------|-----------------|-----------------|
| `String` | TEXT/VARCHAR | `string` |
| `Int` | INTEGER | `number` |
| `Boolean` | BOOLEAN | `boolean` |
| `DateTime` | TIMESTAMPTZ | `Date` |
| `Json` | JSONB | `any` |
| `Decimal` | NUMERIC | `Decimal` (需导入) |

**属性修饰符**：

```prisma
id      String   @id           # 主键
email   String   @unique       # 唯一约束
name    String?                # 可选字段（nullable）
createdAt DateTime @default(now())   # 默认值
updatedAt DateTime @updatedAt   # 自动更新时间
age     Int      @default(18)  # 固定默认值
```

### 数据库关系

Prisma 支持三种基本关系：

```prisma
// 1. 一对一
model User {
  id       String  @id @default(uuid())
  profile  Profile?
}

model Profile {
  id    String  @id @default(uuid())
  user  User    @relation(fields: [userId], references: [id])
  userId String @unique // 外键，必须唯一
}

// 2. 一对多（上面 User ↔ Post 的例子）
model User {
  posts Post[] // 一个用户有多篇文章
}

model Post {
  author    User    @relation(fields: [authorId], references: [id])
  authorId  String  // 外键
}

// 3. 多对多
model Post {
  categories Category[] // Prisma 自动创建中间表
}

model Category {
  posts Post[]
}
```

### Prisma Client 的生成和使用

**生成客户端**：

```bash
npx prisma generate  # 根据 schema 生成 Prisma Client
```

**初始化客户端**：

```typescript
// lib/db.ts
import { PrismaClient } from '@prisma/client'

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined
}

export const prisma = globalForPrisma.prisma ?? new PrismaClient()

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma
```

**为什么需要单例模式**：
- 开发环境下热重载会创建多个 Prisma Client 实例
- 每个实例会创建新的数据库连接池
- 连接数耗尽会导致数据库拒绝服务

### CRUD 操作示例

**Create（创建）**：

```typescript
// 创建单条记录
const user = await prisma.user.create({
  data: {
    email: 'user@example.com',
    name: 'John Doe',
    age: 25,
  },
})

// 创建带关系的记录
const post = await prisma.post.create({
  data: {
    title: 'My First Post',
    content: 'Hello World',
    author: {
      connect: { email: 'user@example.com' }, // 连接已存在的用户
    },
  },
})
```

**Read（读取）**：

```typescript
// 查找唯一记录
const user = await prisma.user.findUnique({
  where: { email: 'user@example.com' },
})

// 查找多条记录
const users = await prisma.user.findMany({
  where: { age: { gte: 18 } }, // 年龄 >= 18
  select: { name: true, email: true }, // 只选择指定字段
  orderBy: { createdAt: 'desc' },
  take: 10, // 限制返回数量
})

// 包含关系
const userWithPosts = await prisma.user.findUnique({
  where: { email: 'user@example.com' },
  include: { posts: true }, // 自动加载用户的所有文章
})
```

**Update（更新）**：

```typescript
// 更新单条记录
const updatedUser = await prisma.user.update({
  where: { email: 'user@example.com' },
  data: { age: 26 },
})

// 批量更新
const result = await prisma.user.updateMany({
  where: { age: { lt: 18 } }, // 年龄 < 18
  data: { isMinor: true },
})

// 更新或创建
const user = await prisma.user.upsert({
  where: { email: 'user@example.com' },
  create: { email: 'user@example.com', name: 'New User' },
  update: { name: 'Updated Name' },
})
```

**Delete（删除）**：

```typescript
// 删除单条记录
const deletedUser = await prisma.user.delete({
  where: { email: 'user@example.com' },
})

// 批量删除
const result = await prisma.user.deleteMany({
  where: { age: { lt: 13 } },
})
```

### 数据库迁移

当 Schema 发生变化时，需要同步数据库结构：

```bash
# 创建迁移文件（自动生成 SQL）
npx prisma migrate dev --name add_user_age

# 应用迁移到数据库
npx prisma migrate deploy

# 重置数据库（开发环境慎用）
npx prisma migrate reset
```

**迁移文件示例**：

```sql
-- prisma/migrations/20240101000000_add_user_age/migration.sql
-- AlterTable
ALTER TABLE "users" ADD COLUMN "age" INTEGER;
```

### 类型安全的优势

Prisma 自动生成 TypeScript 类型：

```typescript
// 自动推断返回类型
const user = await prisma.user.findUnique({
  where: { email: 'test@example.com' },
})
// user 的类型是 User | null

// 类型错误会在编译时捕获
const age: string = user.age // ❌ Type 'number | null' is not assignable to type 'string'

// IDE 自动补全
const users = await prisma.user.findMany({
  where: {
    // IDE 会提示所有可用的字段和操作符
    email: { contains: 'example' },
  },
})
```

## 你需要记住的

1. Prisma Schema 定义数据模型，Prisma Client 提供类型安全的操作 API
2. 使用单例模式初始化 Prisma Client，避免开发环境创建多个实例
3. 关系通过 `@relation` 属性定义，Prisma 自动处理外键约束
4. 迁移文件自动生成 SQL，用于同步数据库结构变化
5. Prisma 自动生成 TypeScript 类型，提供完整的类型检查和 IDE 支持

## AI 代码中的线索

当 AI 生成的代码使用 Prisma 时，你会看到：

1. **导入语句**：`import { prisma } from '@/lib/db'` 或 `import { PrismaClient } from '@prisma/client'`
2. **API 模式**：`prisma.model.operation()` 如 `prisma.user.findMany()`
3. **关系操作**：`include`、`connect`、`disconnect` 等关系操作符
4. **类型推断**：变量类型自动推断为生成的模型类型

**常见模式**：
```typescript
// 查询模式
prisma.user.findUnique({ where: { id } })
prisma.user.findMany({ where: { age: { gte: 18 } } })

// 创建模式
prisma.user.create({ data: { email, name } })
prisma.post.create({ data: { title, author: { connect: { userId } } } } })

// 更新模式
prisma.user.update({ where: { id }, data: { name } })
```

## 验证问题

- [ ] Prisma 相比直接写 SQL 有什么优势？为什么要使用 ORM？
- [ ] 如何在 Prisma Schema 中定义一对多的关系？
- [ ] 为什么需要使用单例模式初始化 Prisma Client？
