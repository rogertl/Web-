# 5.0 数据库基础概念

## 一句话

关系型数据库使用表、行、列来组织数据，通过主键、外键、索引来优化查询和数据完整性，通过事务来保证数据一致性。

## 为什么需要它

学习数据库操作之前，需要理解数据库的基本概念。就像学开车前需要知道方向盘、油门、刹车是什么。不理解主键、外键、索引、事务，就无法设计合理的数据结构，写出高效的查询代码。

## 类比

把数据库想象成"档案室"：

| 概念 | 生活类比 |
|------|---------|
| 数据库 | 档案室（存放所有档案的地方） |
| 表 | 档案柜（存放特定类型的档案） |
| 行 | 一份完整档案（一个人的所有信息） |
| 列 | 档案中的一个字段（如"姓名"、"年龄"） |
| 主键 | 身份证号（唯一标识一个人） |
| 外键 | 档案编号（关联到其他档案柜） |
| 索引 | 档案目录（快速找到目标档案） |
| 事务 | 搬家打包（要么全部搬完，要么保持原状） |

## 核心内容

### 关系型数据库的基本结构

关系型数据库使用**表**（Table）来组织数据，类似于 Excel 表格：

```
┌─────────────────────────────────────────┐
│          users 表（用户表）               │
├──────────┬─────────────┬────────┬────────┤
│  id      │  name       │  email  │  age   │  ← 列（Column）
├──────────┼─────────────┼────────┼────────┤
│    1     │  张三       │  ...    │   25   │  ← 行（Row）
│    2     │  李四       │  ...    │   30   │
│    3     │  王五       │  ...    │   28   │
└──────────┴─────────────┴────────┴────────┘
```

| 概念 | 说明 | 生活类比 |
|------|---------|---------|
| **表（Table）** | 存储特定类型数据的结构 | 文件柜（存放同类型档案） |
| **行（Row）** | 一条完整的记录 | 一份档案（一个人的完整信息） |
| **列（Column）** | 数据的一个属性 | 档案字段（如"姓名"） |
| **主键（Primary Key）** | 唯一标识每条记录的列 | 身份证号（唯一标识） |

### 主键（Primary Key）

**作用**：唯一标识表中的每一条记录，不能重复，不能为空。

```sql
-- PostgreSQL 示例
CREATE TABLE users (
  id SERIAL PRIMARY KEY,  -- id 是主键，自动递增
  name VARCHAR(100),
  email VARCHAR(100) UNIQUE
);
```

**生活类比**：
- **身份证号**：每个人都有唯一的身份证号，不会重复
- **学号**：学校里每个学生有唯一的学号
- **车牌号**：每辆车有唯一的车牌号

**Prisma 中的表示**：
```prisma
model User {
  id String @id @default(uuid())  // id 是主键，默认值是 UUID
}
```

**为什么需要主键**：
- 确保每条记录可以被唯一标识
- 作为外键关联的目标
- 提高查询效率（数据库自动为主键创建索引）

### 外键（Foreign Key）

**作用**：建立两个表之间的关联，确保数据的完整性。

```sql
-- 创建用户表
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100)
);

-- 创建文章表，author_id 是外键，关联到 users 表的 id
CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  title VARCHAR(200),
  author_id INTEGER REFERENCES users(id)  -- 外键
);
```

**生活类比**：
- **订单的客户ID**：订单中的"客户ID"关联到客户表，确保每个订单都对应一个真实客户
- **员工的部门ID**：员工表中的"部门ID"关联到部门表，确保每个员工都属于一个真实存在的部门
- **书的ISBN**：图书借阅记录中的"ISBN"关联到图书表

**Prisma 中的表示**：
```prisma
model User {
  id    String @id @default(uuid())
  posts Post[] // 反向关系（一个用户有多篇文章）
}

model Post {
  id       String @id @default(uuid())
  title    String
  authorId String // 外键字段

  // 定义外键关系：Post 属于 User
  author   User   @relation(fields: [authorId], references: [id])
}
```

**外键的约束**：
- **引用完整性**：外键的值必须在被引用的表中存在
- **级联删除**：删除被引用的记录时，可以自动删除引用它的记录
- **防止孤立数据**：确保不会有"没有作者的"

### 索引（Index）

**作用**：加速数据查询，类似于书籍的目录。

```sql
-- 在 email 字段上创建索引
CREATE INDEX idx_user_email ON users(email);

-- 复合索引（多个字段）
CREATE INDEX idx_post_published_date ON posts(published, created_at);
```

**生活类比**：
- **书籍目录**：让你快速找到章节，而不需要翻遍整本书
- **字典索引**：让你快速查找单词，而不需要从头读到尾
- **电话黄页**：按姓氏排列，快速找到目标电话

**为什么需要索引**：
- **没有索引**：数据库需要扫描全表（全表扫描），慢
- **有索引**：数据库直接定位到目标行，快

**性能对比**：
```
假设有 100 万条用户记录

没有索引：
SELECT * FROM users WHERE email = 'user@example.com';
-- 需要扫描 100 万行，耗时：约 1-5 秒

有索引：
SELECT * FROM users WHERE email = 'user@example.com';
-- 直接定位到目标行，耗时：约 0.001 秒
```

**Prisma 中的索引**：
```prisma
model Post {
  id        String   @id @default(uuid())
  title     String
  published Boolean

  @@index([published, createdAt]) // 创建复合索引
}
```

**索引的代价**：
| 优势 | 代价 |
|------|------|
| ✅ 加速查询 | ❌ 降低写入性能（每次插入/更新都要更新索引） |
| ✅ 支持快速排序 | ❌ 占用存储空间 |
| ✅ 支持快速筛选 | ❌ 不是越多越好（需要权衡） |

**最佳实践**：
- 为经常用于查询条件（`WHERE`）、排序（`ORDER BY`）、连接（`JOIN`）的字段创建索引
- 避免为很少查询的字段创建索引
- 避免为经常更新的字段创建过多索引

### 事务（Transaction）

**作用**：确保多个操作要么全部成功，要么全部失败，保持数据一致性。

**生活类比：银行转账**

```
转账流程：
1. 从账户 A 扣除 100 元
2. 向账户 B 增加 100 元

如果第 1 步成功但第 2 步失败（例如系统崩溃）：

没有事务：
  账户 A 少了 100 元，账户 B 没收到
  → 数据不一致，钱消失了！

有事务：
  自动回滚，账户 A 恢复原值
  → 两个账户都没变，数据一致
```

**SQL 示例**：
```sql
BEGIN; -- 开始事务

UPDATE accounts SET balance = balance - 100 WHERE id = 1; -- 扣款
UPDATE accounts SET balance = balance + 100 WHERE id = 2; -- 入账

COMMIT; -- 提交事务（两个操作都生效）

-- 如果中间出错，执行 ROLLBACK; 回滚事务（两个操作都撤销）
```

**Prisma 中的事务**：
```typescript
await prisma.$transaction(async (tx) => {
  // 从账户 A 扣款
  await tx.account.update({
    where: { id: 1 },
    data: { balance: { decrement: 100 } }
  });

  // 向账户 B 入账
  await tx.account.update({
    where: { id: 2 },
    data: { balance: { increment: 100 } }
  });
});
// 如果中间出错，Prisma 自动回滚
```

**事务的 ACID 特性**：

| 特性 | 说明 | 生活类比 |
|------|------|---------|
| **原子性（A）** | 操作要么全成功，要么全失败 | 搬家打包（要么全搬走，要么留原处） |
| **一致性（C）** | 事务前后数据保持一致 | 转账前后总金额不变 |
| **隔离性（I）** | 多个事务互不干扰 | 两个人同时转账，互不影响 |
| **持久性（D）** | 事务提交后，数据永久保存 | 确认收货后，订单永久生效 |

**常见应用场景**：
- 银行转账（扣款+入账）
- 电商订单（扣库存+创建订单+扣款）
- 用户注册（创建用户+初始化设置+发送欢迎邮件）

### 关系类型

| 关系类型 | 说明 | 示例 |
|---------|------|------|
| **一对一** | A 表的一条记录对应 B 表的一条记录 | 用户 ↔ 档案（一个用户一个档案） |
| **一对多** | A 表的一条记录对应 B 表的多条记录 | 用户 ↔ 文章（一个用户多篇文章） |
| **多对多** | A 表的多条记录对应 B 表的多条记录 | 文章 ↔ 标签（一篇文章多个标签） |

**一对多（最常见）**：
```
用户（1）─────文章（N）
  张三      →  文章1、文章2、文章3
  李四      →  文章4、文章5
```

**Prisma 一对多示例**：
```prisma
model User {
  id    String @id @default(uuid())
  posts Post[] // 一个用户有多篇文章
}

model Post {
  id       String @id @default(uuid())
  title    String
  authorId String

  // 多篇文章属于一个用户
  author   User   @relation(fields: [authorId], references: [id])
}
```

**多对多（需要中间表）**：
```
文章（N）─────标签（N）
  文章1      →  标签A、标签B
  文章2      →  标签B、标签C
  文章3      →  标签A、标签C
```

**Prisma 多对多示例**：
```prisma
model Post {
  id         String     @id @default(uuid())
  title      String
  categories Category[] // Prisma 自动创建中间表
}

model Category {
  id    String @id @default(uuid())
  name  String
  posts Post[] // 隐式关系
}
```

**一对一（外键必须唯一）**：
```prisma
model User {
  id      String   @id @default(uuid())
  profile Profile? // 可选关系
}

model Profile {
  id    String @id @default(uuid())
  bio   String
  userId String @unique // 一对一关系中外键必须唯一

  user  User   @relation(fields: [userId], references: [id])
}
```

## 你需要记住的

1. **主键**：唯一标识每条记录，不能重复，一个表只能有一个主键
2. **外键**：建立表之间的关联，确保数据完整性，可以有一对多、一对一、多对多关系
3. **索引**：加速查询，但会降低写入性能，需要权衡
4. **事务**：确保多个操作要么全成功，要么全失败，保持数据一致性
5. **关系类型**：一对一、一对多、多对多，一对多是最常见的关系

## AI 代码中的线索

当 AI 生成的代码涉及数据库操作时，你会看到：

1. **Prisma Schema 中的主键**：`id String @id @default(uuid())`
2. **Prisma Schema 中的外键**：`author User @relation(fields: [authorId], references: [id])`
3. **Prisma Schema 中的索引**：`@@index([published, createdAt])`
4. **Prisma 事务**：`prisma.$transaction(async (tx) => { ... })`
5. **关系操作**：`include: { posts: true }`、`connect: { userId }`

**常见模式**：
```typescript
// 主键查询
prisma.user.findUnique({ where: { id } })

// 外键关系查询
prisma.post.findMany({ include: { author: true } })

// 事务操作
prisma.$transaction([prisma.user.update(...), prisma.post.create(...)])
```

## 验证问题

- [ ] 什么是主键？为什么每个表都需要主键？
- [ ] 外键的作用是什么？它如何保证数据的完整性？
- [ ] 什么情况下需要创建索引？索引有什么代价？
- [ ] 什么是事务？为什么需要事务？（用银行转账举例）
- [ ] 一对多、一对一、多对多关系有什么区别？各举一个例子。
