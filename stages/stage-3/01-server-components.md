# 3.1 Server Components（RSC）核心概念

## 一句话

Server Components 是 Next.js 13+ 的默认组件类型，只在服务端运行，可以直接写 `async/await` 获取数据，不能使用浏览器 API（如 `window`、`onClick`）。

## 为什么需要它

没有 Server Components，数据获取必须在客户端完成。浏览器发请求 → 等待响应 → 显示内容，用户看到空加载状态。Server Components 让数据获取移到服务端，直接渲染成 HTML 返回，用户瞬间看到内容。这不是性能优化，是渲染模型的根本改变。

## 类比

把组件渲染想象成"餐厅上菜"：

| 概念 | 类比 |
|------|------|
| Server Component | 后厨烹饪（在厨房完成，直接上成品菜） |
| Client Component | 餐桌拼装（在顾客面前组装，有互动过程） |
| 服务端 | 厨房（看不见的地方，但能做复杂操作） |
| 浏览器 | 餐桌（顾客能看到的地方，但只能做简单操作） |
| 数据库 | 食材仓库（厨房直接连接，餐桌不能直接连） |
| HTML 成品 | 成品菜（厨房做好后直接端上桌） |

Server Component 是厨房完成的菜，Client Component 是顾客桌上自己拼的菜。前者快，后者能互动。

## 核心内容

### Server Components 的核心特征

| 特征 | 说明 | 示例 |
|------|------|------|
| 默认类型 | Next.js 中所有组件默认是 Server Components | `export default function Users() { ... }` |
| 只在服务端运行 | 组件代码在 Node.js 环境执行，不发送到浏览器 | `await db.user.findMany()` 可直接写 |
| 可以 async | 组件可以是 `async` 函数，直接 `await` 数据 | `export default async function Users() { ... }` |
| 不能使用浏览器 API | 没有 `window`、`document`、`onClick` 等 | 报错：`window is not defined` |
| 不发送 JS 到浏览器 | 组件代码不会打包进客户端 JS | 减少客户端 JS 体积 |
| 支持服务端专属功能 | 可以直连数据库、读写文件系统 | `fs.readFile()`、`prisma.user.findMany()` |

### 基本示例

```typescript
// app/users/page.tsx
import { getUsers } from '@/lib/users';

// Server Component（默认，无需标记）
export default async function UsersPage() {
  // 直接在服务端获取数据
  const users = await getUsers();

  return (
    <div>
      <h1>用户列表</h1>
      <ul>
        {users.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

这段代码怎么执行？
1. 浏览器请求 `/users`
2. Next.js 在服务端执行 `UsersPage` 组件
3. `getUsers()` 在服务端查询数据库
4. Next.js 把 JSX 渲染成 HTML
5. HTML 返回给浏览器显示
6. **注意**：`UsersPage` 组件的代码不会发送到浏览器

### 为什么不能使用浏览器 API

Server Components 在 Node.js 环境运行，不是浏览器环境：

```typescript
// ❌ 错误：Server Component 中使用浏览器 API
export default function UsersPage() {
  const width = window.innerWidth; // 报错：window is not defined

  return <div>窗口宽度：{width}</div>;
}

// ✅ 正确：把需要浏览器 API 的部分拆成 Client Component
'use client'; // 3.2 会详细讲

import { useState, useEffect } from 'react';

export default function WindowWidth() {
  const [width, setWidth] = useState(0);

  useEffect(() => {
    setWidth(window.innerWidth);
  }, []);

  return <div>窗口宽度：{width}</div>;
}
```

### Server Components 的优势

| 优势 | 说明 | 传统方案的问题 |
|------|------|--------------|
| **数据获取在服务端** | 直接查询数据库，无需额外的 API 层 | 需要写 API 路由 → 客户端 fetch → 等待 |
| **减少客户端 JS** | 组件代码不发送到浏览器 | 所有组件代码都要打包，体积大 |
| **SEO 友好** | 直接渲染 HTML，搜索引擎能读取 | 客户端渲染的页面，搜索引擎看到空壳 |
| **首屏速度快** | 直返 HTML，无需等待 JS 执行 | 等待 JS 加载、执行、请求数据 |
| **安全性高** | 敏感代码（数据库连接）不暴露给浏览器 | API 密钥、数据库连接可能泄露 |

### Server Components 的限制

| 限制 | 说明 | 解决方案 |
|------|------|---------|
| 不能使用 React Hooks | `useState`、`useEffect` 等只能用于 Client Components | 拆成 Client Component |
| 不能使用浏览器 API | `window`、`document`、`onClick` 等不能用 | 拆成 Client Component |
| 不能处理交互 | 按钮点击、表单提交等需要浏览器处理 | 用 Server Actions（阶段 4 会详细讲） |
| 不能使用 Context | Server Components 无法读写 Context | 在 Client Component 中使用 |

### 混合使用：Server Components 嵌套 Client Components

这是 Next.js 的核心模式：外层 Server Component 负责数据，内层 Client Component 负责交互。

```typescript
// app/users/page.tsx（Server Component）
import { getUsers } from '@/lib/users';
import UserCard from '@/components/UserCard'; // Client Component

export default async function UsersPage() {
  const users = await getUsers();

  return (
    <div>
      <h1>用户列表</h1>
      {users.map(user => (
        // Server Component 可以嵌入 Client Component
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  );
}

// components/UserCard.tsx（Client Component）
'use client';

import { useState } from 'react';

export default function UserCard({ user }: { user: { id: number; name: string } }) {
  const [liked, setLiked] = useState(false);

  return (
    <div>
      <h2>{user.name}</h2>
      <button onClick={() => setLiked(!liked)}>
        {liked ? '已点赞' : '点赞'}
      </button>
    </div>
  );
}
```

**数据流**：
1. `UsersPage`（Server）在服务端获取所有用户数据
2. `UsersPage` 把每个 `user` 对象作为 props 传给 `UserCard`
3. `UserCard`（Client）在浏览器中运行，处理点击交互
4. **注意**：`UserCard` 的代码会打包并发送到浏览器

### 常见模式：什么时候拆 Client Component？

| 场景 | 应该是 Server Component | 应该是 Client Component |
|------|------------------------|------------------------|
| 显示数据列表 | ✅ 直接 `await` 数据并渲染 | ❌ 需要先 fetch 再显示 |
| 表单提交 | ✅ 用 Server Actions（阶段 4） | ❌ 需要客户端状态管理 |
| 按钮交互 | ❌ 需要处理 `onClick` | ✅ 可以用 `onClick` |
| 路由跳转 | ✅ 用 `<Link>` 组件 | ❌ 需要用 `useRouter` |
| 访问浏览器 API | ❌ 没有 `window`/`document` | ✅ 可以访问浏览器 API |
| 使用 React Hooks | ❌ `useState`、`useEffect` 等不能用 | ✅ 可以用所有 Hooks |

### Server Components 不是"服务端渲染"的同义词

重要区分：
- **服务端渲染（SSR）**：渲染过程发生在服务端（生成 HTML），但组件代码可能还是发送到浏览器（用于水合 hydration）
- **Server Components**：组件代码完全不发送到浏览器，只在服务端运行

Next.js 13+ 的默认模式：Server Components + SSR。组件在服务端渲染成 HTML，且组件代码不发送到浏览器。

## 你需要记住的

1. Next.js 13+ 的组件默认是 Server Components，只在服务端运行。
2. Server Components 可以是 `async` 函数，直接 `await` 数据获取。
3. Server Components 不能使用浏览器 API（`window`、`document`）和 React Hooks（`useState`、`useEffect`）。
4. Server Components 的代码不发送到浏览器，减少客户端 JS 体积。
5. 需要交互的部分必须拆成 Client Components（用 `'use client'` 标记）。

## AI 代码中的线索

当你在 AI 生成的代码中看到这些模式，说明这是 Server Component：

```typescript
// 线索 1：没有 'use client' 指令（默认就是 Server Component）
export default function UsersPage() {
  // 这是 Server Component
}

// 线索 2：async 函数组件（只有 Server Components 可以 async）
export default async function UsersPage() {
  const users = await getUsers(); // 直接 await
  return <div>{...}</div>;
}

// 线索 3：直接使用数据库查询（服务端专属）
import { db } from '@/lib/db';
const users = await db.user.findMany();

// 线索 4：没有浏览器 API 和 Hooks
// 没有 useState、useEffect、window、document

// 线索 5：从 app/ 文件夹导出（App Router 默认 Server Component）
export default function SomePage() {
  // Server Component
}
```

如果看到 `'use client'` 指令，说明这是 Client Component（3.2 会详细讲）。

## 验证问题

- [ ] 以下组件是 Server Component 还是 Client Component？为什么？
  ```typescript
  export default async function ProductsPage() {
    const products = await getProducts();
    return <div>{products.map(p => <div key={p.id}>{p.name}</div>)}</div>;
  }
  ```
- [ ] 如果你想在组件里处理按钮点击事件，应该保持 Server Component 还是拆成 Client Component？为什么？
- [ ] Server Components 的代码会发送到浏览器吗？这对客户端 JS 体积有什么影响？
