# 3.3 Server Component 与 Client Component 的协作规则

## 一句话

Server Components 获取数据并传给 Client Components，Client Components 处理交互。外层 Server，内层 Client，数据通过 props 单向流动。

## 为什么需要它

如果你搞混了 Server 和 Client 的边界，要么数据获取出错（Client Component 不能 `await`），要么交互失效（Server Component 不能用 `onClick`）。理解协作规则，才能正确拆分组件，让数据层和交互层各司其职。

## 类比

把 Server 和 Client 协作想象成"餐厅分工"：

| 角色 | 类比 | 职责 |
|------|------|------|
| Server Component | 后厨 | 烹饪、准备食材（数据获取） |
| Client Component | 餐厅服务员 | 服务顾客、处理请求（交互） |
| props 传递 | 传菜口 | 后厨把成品菜传给服务员 |
| 单向数据流 | 菜品流向 | 只能从后厨 → 服务员 → 顾客，不能反向 |
| 边界清晰 | 厨房门 | 厨房和餐厅明确分开，不能混淆 |

后厨（Server）负责准备数据，服务员（Client）负责互动。两者通过传菜口（props）传递数据。

## 核心内容

### 核心协作规则

| 规则 | 说明 | 示例 |
|------|------|------|
| **外层 Server，内层 Client** | Server Component 嵌套 Client Component | `app/page.tsx`（Server） → `<UserCard />`（Client） |
| **数据通过 props 传递** | Server 把数据作为 props 传给 Client | `<UserCard user={user} />` |
| **单向数据流** | 数据只能从 Server → Client，不能反向 | Client 不能直接修改 Server 的数据 |
| **Client 需要数据时从 props 获取** | Client 不能自己 `fetch` 数据 | `function UserCard({ user }) { ... }` |
| **'use client' 会传染** | 导入 Client Component 的组件自动变成 Client | Server Component 导入 Client Component 后自己也变 Client |
| **边界要清晰** | Server 和 Client 的职责要明确分开 | 数据在 Server，交互在 Client |

### 规则 1：外层 Server，内层 Client

最常见的模式：外层负责数据，内层负责交互。

```typescript
// app/users/page.tsx（Server Component）
import { getUsers } from '@/lib/users';
import UserList from '@/components/UserList'; // Client Component

export default async function UsersPage() {
  // 外层：Server Component 获取数据
  const users = await getUsers();

  return (
    <div>
      <h1>用户列表</h1>
      {/* 内层：Client Component 处理交互 */}
      <UserList users={users} />
    </div>
  );
}

// components/UserList.tsx（Client Component）
'use client';

import { useState } from 'react';

export default function UserList({ users }: { users: Array<{ id: number; name: string }> }) {
  const [filter, setFilter] = useState('');

  // 内层：Client Component 处理交互（过滤）
  const filteredUsers = users.filter(user => user.name.includes(filter));

  return (
    <div>
      <input
        type="text"
        value={filter}
        onChange={e => setFilter(e.target.value)}
        placeholder="搜索用户"
      />
      <ul>
        {filteredUsers.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

**为什么这样设计**：
- Server Component（`UsersPage`）在服务端获取数据，快且安全
- Client Component（`UserList`）在浏览器处理搜索交互，响应式
- 数据通过 `props` 传递：`users` 从 Server → Client

### 规则 2：数据通过 props 传递

Server Components 不能"传函数"给 Client Components（因为函数会在浏览器执行，无法访问服务端），只能传数据。

```typescript
// ✅ 正确：传数据
export default async function UsersPage() {
  const users = await getUsers();
  return <UserList users={users} />; // 传数据
}

// ❌ 错误：传函数（函数会在浏览器执行，无法访问服务端）
export default async function UsersPage() {
  const getUsers = async () => { /* ... */ };
  return <UserList getUsers={getUsers} />; // 这行不通
}
```

如果 Client Component 需要获取新数据，用 Server Actions（阶段 4 会详细讲）或 API 调用。

### 规则 3：单向数据流

数据从 Server → Client 单向流动，Client 不能直接修改 Server 的数据。

```typescript
// app/users/[userId]/page.tsx（Server Component）
import { getUserById } from '@/lib/users';
import UserProfile from '@/components/UserProfile'; // Client Component

export default async function UserPage({ params }: { params: { userId: string } }) {
  // Server 获取数据
  const user = await getUserById(params.userId);

  return <UserProfile user={user} />;
}

// components/UserProfile.tsx（Client Component）
'use client';

import { useState } from 'react';

export default function UserProfile({ user }: { user: { id: number; name: string } }) {
  const [name, setName] = useState(user.name);

  const handleSave = async () => {
    // Client 不能直接修改 Server 的数据
    // 需要通过 Server Action 或 API 提交更改
    await fetch(`/api/users/${user.id}`, {
      method: 'PATCH',
      body: JSON.stringify({ name }),
    });
  };

  return (
    <div>
      <input value={name} onChange={e => setName(e.target.value)} />
      <button onClick={handleSave}>保存</button>
    </div>
  );
}
```

**数据流**：
1. Server（`UserPage`）获取 `user` 数据
2. Server 把 `user` 作为 props 传给 Client（`UserProfile`）
3. Client 修改本地状态（`name`），但不影响 Server 的数据
4. Client 通过 Server Action 或 API 提交更改到服务端

### 规则 4：'use client' 的传染性

一旦组件标记了 `'use client'`，所有导入它的组件都会变成 Client Components。

```typescript
// components/GrandParent.tsx（原本是 Server Component）
import Parent from './Parent';

export default function GrandParent() {
  return (
    <div>
      <h1>祖组件</h1>
      <Parent /> {/* GrandParent 自动变成 Client Component */}
    </div>
  );
}

// components/Parent.tsx（Client Component）
'use client';
import Child from './Child';

export default function Parent() {
  return <Child />; {/* Child 自动变成 Client Component */}
}

// components/Child.tsx（Client Component）
'use client';

export default function Child() {
  return <div>子组件</div>;
}
```

**实际影响**：
- `GrandParent`、`Parent`、`Child` 都会被打包进客户端 JS
- 尽量把 `'use client'` 放在最内层（叶子节点），减少客户端 JS 体积

**优化策略**：把 Server Component 放在外层，Client Component 放在内层。

```typescript
// ✅ 优化后：Server Component 在外层
// app/page.tsx（Server Component）
import InteractiveButton from '@/components/InteractiveButton'; // Client Component

export default async function HomePage() {
  const data = await getData(); // Server 获取数据
  return (
    <div>
      <h1>首页</h1>
      <InteractiveButton data={data} /> {/* 只有 InteractiveButton 打包 */}
    </div>
  );
}
```

### 规则 5：边界要清晰

Server Components 和 Client Components 的职责要明确分开：

| 职责 | Server Component | Client Component |
|------|------------------|-------------------|
| 数据获取 | ✅ 直接 `await` 数据 | ❌ 需要 `useEffect` + `fetch` |
| 数据库操作 | ✅ 直接连接数据库 | ❌ 不能直连，需 API |
| 交互处理 | ❌ 不能用 `onClick` | ✅ 可以用所有事件 |
| 状态管理 | ❌ 不能用 `useState` | ✅ 可以用所有 Hooks |
| 浏览器 API | ❌ 没有 `window`/`document` | ✅ 可以访问浏览器环境 |
| 代码体积 | 不发送到浏览器 | 打包并发送到浏览器 |

### 协作模式示例：一个完整的页面

```typescript
// app/products/[productId]/page.tsx（Server Component）
import { getProductById } from '@/lib/products';
import AddToCartButton from '@/components/AddToCartButton'; // Client Component
import ProductReviews from '@/components/ProductReviews'; // Client Component

export default async function ProductPage({ params }: { params: { productId: string } }) {
  // Server：获取产品数据
  const product = await getProductById(params.productId);

  return (
    <div>
      <h1>{product.name}</h1>
      <p>价格：${product.price}</p>
      <p>描述：{product.description}</p>

      {/* Client 1：处理加购物车交互 */}
      <AddToCartButton productId={product.id} />

      {/* Client 2：处理评论展示和交互 */}
      <ProductReviews productId={product.id} />
    </div>
  );
}

// components/AddToCartButton.tsx（Client Component）
'use client';

import { useState } from 'react';

export default function AddToCartButton({ productId }: { productId: string }) {
  const [added, setAdded] = useState(false);

  const handleClick = () => {
    // TODO: 通过 Server Action 添加到购物车（阶段 4 会详细讲）
    setAdded(true);
  };

  return (
    <button onClick={handleClick} disabled={added}>
      {added ? '已加入购物车' : '加入购物车'}
    </button>
  );
}

// components/ProductReviews.tsx（Client Component）
'use client';

import { useState, useEffect } from 'react';

export default function ProductReviews({ productId }: { productId: string }) {
  const [reviews, setReviews] = useState([]);

  useEffect(() => {
    // Client：获取评论数据
    fetch(`/api/products/${productId}/reviews`)
      .then(res => res.json())
      .then(data => setReviews(data));
  }, [productId]);

  return (
    <div>
      <h2>用户评论</h2>
      {reviews.map((review: any) => (
        <div key={review.id}>{review.content}</div>
      ))}
    </div>
  );
}
```

**数据流**：
1. Server（`ProductPage`）获取产品数据
2. Server 把 `productId` 作为 props 传给 Client Components
3. Client 1（`AddToCartButton`）处理加购物车交互
4. Client 2（`ProductReviews`）自己获取评论数据（也可以改成 Server 传）

### 常见错误：混淆边界

| 错误 | 问题 | 修正 |
|------|------|------|
| 在 Client Component 里 `await` 数据 | Client Components 不能 `async` | 改用 `useEffect` + `fetch` |
| 在 Server Component 里用 `onClick` | Server Components 不能处理交互 | 拆成 Client Component |
| 把 `'use client'` 放在外层组件 | 导致整个组件树打包到浏览器 | 把 `'use client''` 放在内层 |
| Client Component 传函数给 Server | 函数无法在服务端执行 | 用 Server Actions（阶段 4） |

## 你需要记住的

1. 外层 Server Component 负责数据，内层 Client Component 负责交互。
2. 数据通过 props 从 Server 传递给 Client，单向流动。
3. `'use client'` 会传染：所有导入它的组件自动变成 Client Components。
4. Client Components 不能直接修改 Server 的数据，需要通过 Server Actions 或 API。
5. 边界要清晰：Server 获取数据，Client 处理交互，职责分开。

## AI 代码中的线索

当你在 AI 生成的代码中看到这些模式，说明这是 Server 和 Client 协作：

```typescript
// 线索 1：Server Component 获取数据，传给 Client
export default async function Page() {
  const data = await getData(); // Server 获取
  return <ClientComponent data={data} />; // 传给 Client
}

// 线索 2：Client Component 通过 props 接收数据
'use client';
export default function ClientComponent({ data }) {
  // 处理交互
}

// 线索 3：Server Component 嵌套多个 Client Components
export default async function Page() {
  return (
    <div>
      <ClientComponent1 />
      <ClientComponent2 />
    </div>
  );
}

// 线索 4：Client Component 自己 fetch 数据（也可以，但不如 Server 传）
'use client';
useEffect(() => {
  fetch('/api/data').then(...);
}, []);
```

如果看到 Server Component 里用 `onClick` 或 `useState`，这是错误边界。

## 验证问题

- [ ] 以下代码的边界划分正确吗？为什么？
  ```typescript
  // app/page.tsx
  'use client';
  import { useState } from 'react';
  export default function Page() {
    const [data, setData] = useState(null);
    useEffect(() => {
      fetch('/api/data').then(res => res.json()).then(setData);
    }, []);
    return data ? <div>{data.name}</div> : <div>加载中</div>;
  }
  ```
  应该如何改进？
- [ ] 如果你想在页面上显示一个按钮（点击后弹出提示），应该把按钮组件放在 Server Component 还是 Client Component？为什么？
- [ ] 为什么说 `'use client'` 会传染？这对组件设计有什么影响？
