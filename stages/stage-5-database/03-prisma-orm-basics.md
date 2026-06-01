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

### Prisma Schema 装饰符详解

Prisma Schema 中的装饰符（Attributes）用于定义字段和关系的特殊行为。理解每个装饰符的含义，是正确设计数据模型的关键。

#### @id — 主键标识符

**作用**：将字段标记为表的主键，用于唯一标识每条记录。

```prisma
model User {
  id String @id @default(uuid())  // id 是主键，默认值是 UUID
}
```

**关键点**：
- 每个模型必须有且只能有一个 `@id` 字段
- 主键字段自动具有 `@unique` 约束（值不能重复）
- 常见主键类型：`String @default(uuid())` 或 `Int @default(autoincrement())`

**选择主键类型**：

| 类型 | 优势 | 劣势 | 适用场景 |
|------|------|------|---------|
| `String @default(uuid())` | 全局唯一、不暴露记录数量 | 字符串比较稍慢 | 分布式系统、公开 ID |
| `Int @default(autoincrement())` | 性能好、占用空间小 | 容易猜测记录数量 | 内部系统、自增 ID |

#### @default() — 默认值函数

**作用**：为字段设置默认值，当创建记录时不提供该字段时使用。

```prisma
model User {
  id        String   @id @default(uuid())
  name      String   @default("匿名用户")  // 固定默认值
  age       Int      @default(18)          // 数字默认值
  isActive  Boolean  @default(true)        // 布尔默认值
  createdAt DateTime @default(now())       // 函数默认值（当前时间）
  role      String   @default("user")      // 字符串默认值
}
```

**支持的默认值函数**：

| 函数 | 说明 | 示例 |
|------|------|------|
| `uuid()` | 生成 UUID | `@default(uuid())` |
| `autoincrement()` | 自增整数 | `@default(autoincrement())` |
| `now()` | 当前时间戳 | `@default(now())` |
| `cuid()` | 生成 CUID | `@default(cuid())` |

**关键点**：
- `@default()` 只在创建记录时生效，更新时不影响
- 可以使用固定值或函数作为默认值
- 如果字段是可选的（`?`），`@default` 仍然有效

#### @unique — 唯一约束

**作用**：确保字段的值在表中不重复，类似于主键但可以用于多个字段。

```prisma
model User {
  id    String @id @default(uuid())
  email String @unique  // email 必须唯一，不能有两个用户使用相同邮箱
  phone String? @unique // 可选字段也可以唯一
}
```

**关键点**：
- `@unique` 字段会自动创建数据库唯一索引
- 尝试插入重复值会报错（Prisma 抛出 `PrismaClientKnownRequestError`）
- 可以有多个 `@unique` 字段，但只能有一个 `@id`

**常见应用场景**：
- 邮箱地址（`email String @unique`）
- 用户名（`username String @unique`）
- 外部 ID（`stripeCustomerId String @unique`）

#### @relation — 关系定义

**作用**：定义两个模型之间的关系（一对一、一对多、多对多）。

**一对多关系**：

```prisma
model User {
  id    String @id @default(uuid())
  posts Post[] // 一个用户有多篇文章（反向关系）
}

model Post {
  id       String @id @default(uuid())
  title    String
  authorId String  // 外键字段

  // 定义关系：Post 属于 User
  author   User   @relation(fields: [authorId], references: [id])
}
```

**关键参数**：
- `fields`：当前模型的外键字段（`[authorId]`）
- `references`：关联模型的主键字段（`[id]`）

**一对一关系**：

```prisma
model User {
  id      String   @id @default(uuid())
  profile Profile? // 可选关系（用户可能有或没有 Profile）
}

model Profile {
  id    String @id @default(uuid())
  bio   String
  userId String @unique // 一对一关系中外键必须唯一

  user  User   @relation(fields: [userId], references: [id])
}
```

**多对多关系**（Prisma 自动创建中间表）：

```prisma
model Post {
  id         String     @id @default(uuid())
  title      String
  categories Category[] // 隐式关系
}

model Category {
  id    String @id @default(uuid())
  name  String
  posts Post[] // 隐式关系
}
```

**关键点**：
- 多对多关系不需要手动定义中间表，Prisma 自动创建 `_PostToCategory` 表
- 如需自定义中间表（添加额外字段），需要显式定义

#### @updatedAt — 自动更新时间

**作用**：记录每次更新时自动更新为当前时间。

```prisma
model User {
  id        String   @id @default(uuid())
  name      String
  createdAt DateTime @default(now())  // 创建时间不变
  updatedAt DateTime @updatedAt       // 每次更新自动更新
}
```

**关键点**：
- `@updatedAt` 只能用于 `DateTime` 类型字段
- 创建记录时，该字段自动设置为当前时间
- 更新记录时，该字段自动更新为当前时间
- 无需手动管理，适合用于追踪"最后修改时间"

#### 其他常用装饰符

**@map — 自定义表名**：

```prisma
model User {
  id    String @id @default(uuid())
  name  String

  @@map("users") // 在数据库中表名为 "users"（复数）
}
```

**@@index — 复合索引**：

```prisma
model Post {
  id        String   @id @default(uuid())
  title     String
  published Boolean

  @@index([published, createdAt]) // 复合索引，用于加速查询
}
```

**@db.Text — 指定数据库类型**：

```prisma
model Post {
  id      String  @id @default(uuid())
  content String @db.Text // 使用 PostgreSQL 的 TEXT 类型
}
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
