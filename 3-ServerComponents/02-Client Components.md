# 3.2 Client Components（'use client'）

## 一句话

Client Components 用 `'use client'` 标记，代码在浏览器运行，可以使用 React Hooks、浏览器 API 和交互事件（`onClick`、`onChange`）。

## 为什么需要它

只有 Server Components，你无法处理按钮点击、输入框变化、窗口大小变化等交互。Client Components 让代码在浏览器运行，能响应用户操作。这是 Web 应用的交互层。

## 类比

把 Client Component 想象成"餐桌拼装"：

| 概念 | 类比 |
|------|------|
| Client Component | 餐桌拼装（在顾客面前组装，可以互动） |
| 'use client' 指令 | "请在餐桌操作"的标记 |
| React Hooks | 餐桌工具（刀叉、调料，可以互动） |
| 浏览器 API | 餐桌环境（灯光、温度，顾客能感知） |
| 交互事件 | 顾客动作（加菜、换菜、买单） |

Server Component 是厨房完成的成品，Client Component 是顾客桌上可以互动的部分。

## 核心内容

### Client Components 的核心特征

| 特征 | 说明 | 示例 |
|------|------|------|
| 必须标记 | 文件顶部必须写 `'use client'` 指令 | `'use client'` |
| 在浏览器运行 | 组件代码打包并发送到浏览器执行 | `useState`、`window.innerWidth` |
| 可以使用 Hooks | `useState`、`useEffect`、`useContext` 等 | `const [count, setCount] = useState(0)` |
| 可以使用浏览器 API | `window`、`document`、`navigator` 等 | `window.location.href` |
| 可以处理交互 | `onClick`、`onChange`、`onSubmit` 等事件 | `<button onClick={handleClick}>` |
| 不能直接 async | 组件不能是 `async` 函数（除了 Server Components） | ❌ `export default async function` |
| 需要数据时从服务端获取 | 通过 props 接收数据，或自己 fetch | `<UserCard user={user} />` |

### 基本示例

```typescript
// components/LikeButton.tsx
'use client'; // 必须标记，否则报错

import { useState } from 'react';

export default function LikeButton() {
  const [liked, setLiked] = useState(false);

  const handleClick = () => {
    setLiked(!liked);
  };

  return (
    <button onClick={handleClick}>
      {liked ? '已点赞' : '点赞'}
    </button>
  );
}
```

**执行过程**：
1. 组件代码被打包进客户端 JS
2. 浏览器下载并执行这段代码
3. 用户点击按钮 → `handleClick` 在浏览器执行 → `setLiked` 更新状态
4. React 重新渲染组件，显示新的点赞状态

### 为什么需要 'use client' 指令

Next.js 默认所有组件是 Server Components，必须显式标记哪些是 Client Components。

```typescript
// ❌ 错误：没有 'use client'，但使用了 useState
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0); // 报错：useState 只能在 Client Component 用
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}

// ✅ 正确：添加 'use client' 指令
'use client';
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

### Client Components 的限制

| 限制 | 说明 | 解决方案 |
|------|------|---------|
| 不能直接 async | 组件不能是 `async` 函数 | 用 `useEffect` 获取数据 |
| 不能直连数据库 | 没有访问服务端文件系统的权限 | 通过 API 获取数据 |
| 代码会发送到浏览器 | 组件代码打包进客户端 JS | 保持组件小而专注 |
| SEO 不友好 | 搜索引擎可能看不到动态内容 | Server Components 渲染关键内容 |

### 数据获取：Client Component 的方式

Client Components 不能直接 `await`，需要用 `useEffect`：

```typescript
// components/UserList.tsx
'use client';

import { useState, useEffect } from 'react';

interface User {
  id: number;
  name: string;
}

export default function UserList() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // 浏览器中发起请求
    fetch('/api/users')
      .then(res => res.json())
      .then(data => {
        setUsers(data);
        setLoading(false);
      });
  }, []);

  if (loading) return <div>加载中...</div>;

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

**对比 Server Components**：
- Server Component：`export default async function UserList() { const users = await getUsers(); ... }`
- Client Component：`useEffect(() => { fetch('/api/users').then(...) }, [])`

### Server Components 和 Client Components 的数据传递

最常见的模式：Server Component 获取数据，传给 Client Component。

```typescript
// app/users/page.tsx（Server Component）
import { getUsers } from '@/lib/users';
import UserCard from '@/components/UserCard'; // Client Component

export default async function UsersPage() {
  // 服务端获取数据
  const users = await getUsers();

  return (
    <div>
      <h1>用户列表</h1>
      {users.map(user => (
        // 把数据作为 props 传给 Client Component
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

**优势**：
- 数据获取在服务端（快、安全）
- 交互在客户端（灵活、响应式）
- 客户端 JS 体积小（只打包交互逻辑）

### 边界规则：'use client' 的传染性

**重要规则**：如果一个组件标记了 `'use client'`，所有导入它的组件自动变成 Client Components。

```typescript
// components/Parent.tsx（Server Component，默认）
import Child from './Child';

export default function Parent() {
  return (
    <div>
      <h1>父组件</h1>
      <Child /> {/* 父组件自动变成 Client Component，因为 Child 是 */}
    </div>
  );
}

// components/Child.tsx（Client Component）
'use client';

export default function Child() {
  return <div>子组件</div>;
}
```

**实际影响**：
- `Parent` 和 `Parent` 的父组件，都会被打包进客户端 JS
- 尽量把 `'use client'` 放在最内层的组件（叶子节点）
- 避免在顶层组件标记 `'use client'`

### 嵌套规则：Client Component 可以嵌入 Server Component

```typescript
// app/page.tsx（Server Component）
import InteractiveButton from '@/components/InteractiveButton'; // Client Component

export default function HomePage() {
  return (
    <div>
      <h1>首页</h1>
      {/* Server Component 中嵌入 Client Component */}
      <InteractiveButton />
    </div>
  );
}

// components/InteractiveButton.tsx（Client Component）
'use client';

export default function InteractiveButton() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

**反向不行**：Client Component 中不能直接嵌入 Server Component（需要用 props 传递数据）。

### 常见模式：何时使用 Client Components

| 场景 | 应该用 Client Component | 原因 |
|------|------------------------|------|
| 按钮点击、表单提交 | ✅ | 需要处理 `onClick`、`onSubmit` 等事件 |
| 状态管理（`useState`） | ✅ | 只有 Client Components 能用 Hooks |
| 浏览器 API（`window`） | ✅ | 需要访问浏览器环境 |
| 路由跳转（`useRouter`） | ✅ | `useRouter` 是 Hook，只能在 Client Components 用 |
| 显示列表数据 | ❌ | 用 Server Component 直接 `await` 数据 |
| SEO 关键内容 | ❌ | Server Components 渲染的 HTML 对 SEO 更友好 |

## 你需要记住的

1. Client Components 必须用 `'use client'` 指令标记。
2. Client Components 在浏览器运行，可以使用 React Hooks 和浏览器 API。
3. Client Components 不能是 `async` 函数，需要用 `useEffect` 获取数据。
4. Client Components 的代码会打包并发送到浏览器。
5. `'use client'` 会传染：所有导入它的组件自动变成 Client Components。

## AI 代码中的线索

当你在 AI 生成的代码中看到这些模式，说明这是 Client Component：

```typescript
// 线索 1：文件顶部有 'use client' 指令
'use client';

// 线索 2：使用 React Hooks
import { useState, useEffect } from 'react';
const [count, setCount] = useState(0);

// 线索 3：使用浏览器 API
window.innerWidth;
document.getElementById('...');

// 线索 4：处理交互事件
<button onClick={handleClick}>点击</button>
<input onChange={handleChange} />

// 线索 5：使用 useRouter、useSearchParams 等 Next.js Hooks
import { useRouter } from 'next/navigation';
const router = useRouter();
```

如果看到 `async` 函数组件或数据库查询，这是 Server Component。

## 验证问题

- [ ] 以下组件是 Server Component 还是 Client Component？为什么？
  ```typescript
  'use client';
  import { useState } from 'react';
  export default function Counter() {
    const [count, setCount] = useState(0);
    return <button onClick={() => setCount(count + 1)}>{count}</button>;
  }
  ```
- [ ] 如果你想在组件里使用 `window.innerWidth`，应该用 Server Component 还是 Client Component？为什么？
- [ ] 为什么说 `'use client'` 会传染？这如何影响你的组件设计？
