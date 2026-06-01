# 4.1 Server Actions（'use server'）

## 一句话

Server Actions 用 `'use server'` 标记，让客户端代码（如按钮点击）直接调用服务端函数，无需手动创建 API 路由，简化表单提交和数据修改。

## 为什么需要它

没有 Server Actions，客户端的交互（如按钮点击）需要：创建 API 路由 → 客户端 fetch → 处理响应。代码分散在多个文件，难以维护。Server Actions 让客户端直接调用服务端函数，一行代码搞定。

## 类比

把 Server Actions 想象成"餐厅呼叫器"：

| 概念 | 类比 |
|------|------|
| Client Component（按钮） | 餐桌上的呼叫器（顾客操作） |
| Server Action（服务端函数） | 厨房指令（后厨执行） |
| 'use server' 指令 | "传到厨房"标记（指定执行位置） |
| 表单提交 | 顾客点菜（呼叫器 → 厨房） |
| 数据修改 | 厨师烹饪（服务端执行） |

顾客按呼叫器（按钮点击），指令直接传到厨房（服务端），无需服务员（API）中转。

## 核心内容

### Server Actions 的核心特征

| 特征 | 说明 | 示例 |
|------|------|------|
| 用 `'use server'` 标记 | 文件顶部或函数顶部添加指令 | `'use server'` |
| 在服务端执行 | 函数代码在 Node.js 环境运行 | `await db.user.create(...)` |
| 客户端直接调用 | Client Components 可以直接调用 | `<form action={createUser} />` |
| 无需手动创建 API | 不需要创建 `app/api/` 路由 | 自动生成 HTTP 端点 |
| 支持序列化参数 | 参数自动序列化（JSON） | 传递对象、数组等 |
| 支持返回值 | 可以返回数据给客户端 | `return { success: true }` |

### 基本示例

```typescript
// app/actions.ts（服务端函数）
'use server';

import { db } from '@/lib/db';
import { revalidatePath } from 'next/cache';

export async function createUser(formData: FormData) {
  // 在服务端执行
  const name = formData.get('name') as string;
  const email = formData.get('email') as string;

  // 直接连接数据库
  const user = await db.user.create({
    data: { name, email },
  });

  // 清除缓存（阶段 5 会详细讲）
  revalidatePath('/users');

  return { success: true, userId: user.id };
}

// components/CreateUserForm.tsx（Client Component）
'use client';

import { createUser } from '@/app/actions';

export default function CreateUserForm() {
  // Client Component 直接调用服务端函数
  async function handleSubmit(formData: FormData) {
    const result = await createUser(formData);
    if (result.success) {
      alert(`用户创建成功，ID：${result.userId}`);
    }
  }

  return (
    <form action={handleSubmit}>
      <input name="name" type="text" placeholder="姓名" />
      <input name="email" type="email" placeholder="邮箱" />
      <button type="submit">创建用户</button>
    </form>
  );
}
```

**执行流程**：
1. 用户点击"创建用户"按钮
2. 浏览器自动把表单数据 POST 到服务端
3. Next.js 执行 `createUser` 函数（在服务端）
4. 函数返回结果给浏览器
5. **注意**：无需手动创建 `/api/users` 路由

### Server Actions 的两种写法

#### 写法 1：独立文件（推荐）

```typescript
// app/actions.ts
'use server'; // 文件顶部标记，文件内所有函数都是 Server Actions

export async function createUser(formData: FormData) {
  // 服务端代码
}

export async function updateUser(id: string, formData: FormData) {
  // 服务端代码
}

export async function deleteUser(id: string) {
  // 服务端代码
}
```

#### 写法 2：内联函数

```typescript
// components/UserForm.tsx
'use client';

import { createUser } from '@/app/actions';

export default function UserForm() {
  // 直接在组件里定义 Server Action
  async function localAction(formData: FormData) {
    'use server'; // 函数顶部标记

    // 服务端代码
    const name = formData.get('name') as string;
    // ...
  }

  return <form action={localAction}>...</form>;
}
```

**推荐**：用独立文件，所有 Server Actions 集中管理，便于复用和维护。

### Server Actions vs API 路由

| 特性 | Server Actions | API 路由（Route Handlers） |
|------|----------------|--------------------------|
| 文件位置 | 任意 `actions.ts` 文件 | 必须在 `app/api/` 文件夹 |
| 标记方式 | `'use server'` 指令 | 自动识别 `route.ts` |
| 调用方式 | 直接调用函数 | `fetch('/api/xxx')` |
| 参数序列化 | 自动（支持 FormData） | 手动（JSON.stringify） |
| 表单处理 | 原生支持（`action` 属性） | 需要手动构造请求 |
| 类型安全 | ✅ TypeScript 全程类型检查 | ❌ fetch 返回 `any` |
| 适用场景 | 表单提交、数据修改 | 通用 API、第三方调用 |

**何时用 Server Actions**：
- ✅ 表单提交（创建、更新、删除）
- ✅ 按钮触发的数据修改
- ✅ 需要类型安全的交互

**何时用 API 路由**：
- ✅ 第三方服务调用（如 Webhook）
- ✅ RESTful API（如移动端后端）
- ✅ 需要完全自定义 HTTP 行为

### 表单处理（原生支持）

Server Actions 原生支持 HTML 表单：

```typescript
// components/LoginForm.tsx
'use client';

import { login } from '@/app/actions';

export default function LoginForm() {
  return (
    <form action={login}>
      <input name="username" type="text" />
      <input name="password" type="password" />
      <button type="submit">登录</button>
    </form>
  );
}

// app/actions.ts
'use server';

export async function login(formData: FormData) {
  const username = formData.get('username') as string;
  const password = formData.get('password') as string;

  // 验证用户（阶段 5 会详细讲 zod 数据验证）
  const user = await authenticateUser(username, password);

  if (!user) {
    return { error: '用户名或密码错误' };
  }

  // 设置 Session（阶段 6 会详细讲）
  return { success: true, user };
}
```

**优势**：
- 无需手动构造 `fetch` 请求
- 无需手动处理 `formData`
- 原生支持，浏览器兼容性好

### 按钮触发（非表单）

不是表单的交互也能用 Server Actions：

```typescript
// components/LikeButton.tsx
'use client';

import { likePost } from '@/app/actions';

export default function LikeButton({ postId }: { postId: string }) {
  async function handleLike() {
    // 直接调用 Server Action
    const result = await likePost(postId);
    if (result.success) {
      alert('点赞成功');
    }
  }

  return <button onClick={handleLike}>点赞</button>;
}

// app/actions.ts
'use server';

export async function likePost(postId: string) {
  await db.post.update({
    where: { id: postId },
    data: { likes: { increment: 1 } },
  });

  revalidatePath('/posts'); // 刷新缓存

  return { success: true };
}
```

### 错误处理

Server Actions 的错误会自动传递给客户端：

```typescript
// app/actions.ts
'use server';

export async function createUser(formData: FormData) {
  try {
    const user = await db.user.create({
      data: {
        name: formData.get('name') as string,
        email: formData.get('email') as string,
      },
    });

    return { success: true, user };
  } catch (error) {
    // 错误自动传递给客户端
    return { success: false, error: '创建用户失败' };
  }
}

// components/CreateUserForm.tsx
'use client';

import { createUser } from '@/app/actions';
import { useActionState } from 'react'; // React 19+ 的 Hook

export default function CreateUserForm() {
  const [state, formAction] = useActionState(createUser, null);

  return (
    <form action={formAction}>
      {state?.error && <div className="error">{state.error}</div>}
      <input name="name" type="text" />
      <input name="email" type="email" />
      <button type="submit">创建用户</button>
    </form>
  );
}
```

**错误处理规则**：
- Server Action 返回的对象（如 `{ success: false, error: '...' }`）会自动传递给客户端
- 客户端用 `useActionState` Hook 接收状态（React 19+）
- 如果 Server Action 抛出异常，Next.js 自动显示错误页面

### 重新验证缓存（revalidatePath）

数据修改后，通常需要刷新相关页面的缓存：

```typescript
'use server';

import { revalidatePath } from 'next/cache';

export async function updateUser(id: string, formData: FormData) {
  await db.user.update({
    where: { id },
    data: { name: formData.get('name') as string },
  });

  // 刷新 /users 页面的缓存
  revalidatePath('/users');
  // 或刷新特定路径
  revalidatePath(`/users/${id}`);

  return { success: true };
}
```

**为什么需要**：
- Next.js 默认缓存 Server Components 的数据获取（3.4 已详细讲）
- 数据修改后，缓存过期，需要手动刷新
- `revalidatePath` 告诉 Next.js 重新获取该路径的数据

### Server Actions 的安全性

Server Actions 在服务端执行，天然比客户端代码安全：

| 安全特性 | 说明 |
|---------|------|
| 代码不暴露 | Server Actions 的代码不发送到浏览器 |
| 数据验证 | 可以用 zod 验证输入（阶段 5 会详细讲） |
| 权限检查 | 可以在执行前检查用户权限（阶段 6 会详细讲） |
| SQL 注入防护 | 用 Prisma ORM 自动防护（阶段 5 会详细讲） |

**安全最佳实践**：
- ✅ 用 zod 验证输入（防止恶意数据）
- ✅ 检查用户权限（防止未授权操作）
- ✅ 用 ORM 参数化查询（防止 SQL 注入）
- ❌ 不要信任客户端传来的数据（始终验证）

## 你需要记住的

1. Server Actions 用 `'use server'` 标记，让客户端直接调用服务端函数。
2. Server Actions 在服务端执行，可以连接数据库、修改数据。
3. Server Actions 原生支持表单提交，无需手动创建 API 路由。
4. Server Actions 返回的对象会自动传递给客户端，用 `useActionState` 接收。
5. 数据修改后，用 `revalidatePath` 刷新相关页面的缓存。

## AI 代码中的线索

当你在 AI 生成的代码中看到这些模式，说明这是 Server Action：

```typescript
// 线索 1：'use server' 指令
'use server';

// 线索 2：函数接收 FormData 参数
export async function createUser(formData: FormData) {
  // 服务端代码
}

// 线索 3：表单的 action 属性指向函数
<form action={createUser}>...</form>

// 线索 4：直接调用服务端函数
const result = await createUser(formData);

// 线索 5：revalidatePath 调用
revalidatePath('/users');

// 线索 6：从 @/app/actions 导入
import { createUser } from '@/app/actions';
```

如果看到 `fetch('/api/xxx')`，这是传统 API 调用，不是 Server Action。

## 验证问题

- [ ] 以下代码是 Server Action 还是普通函数？为什么？
  ```typescript
  export async function createUser(formData: FormData) {
    await db.user.create({ data: { name: formData.get('name') } });
    return { success: true };
  }
  ```
- [ ] 如果你想在按钮点击时修改数据库，应该用 Server Action 还是 API 路由？为什么？
- [ ] Server Actions 的代码会发送到浏览器吗？这对安全性有什么影响？
