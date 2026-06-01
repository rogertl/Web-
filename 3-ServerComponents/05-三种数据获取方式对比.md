# 3.5 三种数据获取方式对比

## 一句话

Next.js 中有三种数据获取方式：Server Components 直接 `await`、Client Components `useEffect` + `fetch`、以及 SWR/React Query 客户端库。理解它们的区别和适用场景，是写出高性能应用的关键。

## 为什么需要它

很多开发者困惑：为什么有了 Server Components 还要学 SWR/React Query？什么时候用哪种方式？三种方式各有优劣，选错了会导致性能差、代码复杂、甚至无法实现需求。理解三种方式的特点和适用场景，才能做出正确的技术选择。

## 类比

把数据获取想象成"取快递"：

| 方式 | 类比 | 流程 | 优缺点 |
|------|------|------|--------|
| Server Components await | 快递直接送到家 | 快递员 → 送到家门口 → 签收 | 快（无需出门）、省事（不用跑）、但无法查看快递状态 |
| Client Components useEffect | 自己去快递柜取 | 接通知 → 走到快递柜 → 输入码 → 取件 | 慢（需要出门）、累（要走路）、但能看到快递状态 |
| SWR/React Query | 快递柜+自动刷新 | 快递柜取件 + 自动通知新快递 | 兼顾两者：既有实时状态，又有自动更新 |

**核心区别**：
- **Server Components**：服务端取数据，一次性返回，没有客户端状态
- **Client Components fetch**：客户端取数据，有状态管理，但性能差
- **SWR/React Query**：客户端取数据，但自动缓存、自动刷新、功能强大

## 核心内容

### 三种方式概览

| 方式 | 代码位置 | 执行环境 | 首屏速度 | 实时更新 | 缓存 | 适用场景 |
|------|---------|---------|---------|---------|------|---------|
| **Server Components await** | `async` 函数组件 | 服务端 | ⚡ 快 | ❌ 无（需刷新） | ✅ 服务端缓存 | 首屏数据、SEO |
| **Client Components useEffect** | `useEffect` + `fetch` | 浏览器 | 🐌 慢 | ✅ 手动实现 | ❌ 无 | 简单交互数据 |
| **SWR/React Query** | 自定义 Hook | 浏览器 | ⚡ 快（缓存） | ✅ 自动刷新 | ✅ 智能缓存 | 复杂交互数据 |

### 方式 1：Server Components 直接 await（推荐用于首屏）

#### 代码示例

```typescript
// app/users/page.tsx
import { getUsers } from '@/lib/users';

export default async function UsersPage() {
  // 服务端直接 await
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

#### 执行流程

```
1. 浏览器请求 /users
2. Next.js 在服务端执行 UsersPage
3. getUsers() 查询数据库
4. Next.js 渲染 JSX → HTML
5. HTML 返回给浏览器
6. 浏览器显示最终内容
```

#### 优势

| 优势 | 说明 |
|------|------|
| **首屏速度快** | 直返 HTML，无需等待 JS 加载和执行 |
| **SEO 友好** | 搜索引擎能直接读取 HTML |
| **减少客户端 JS** | 数据获取代码不发送到浏览器 |
| **安全性高** | 数据库连接不暴露给浏览器 |

#### 劣势

| 劣势 | 说明 | 影响 |
|------|------|------|
| **无实时更新** | 数据不会自动更新 | 用户看到的是页面加载时的数据 |
| **依赖页面刷新** | 获取最新数据需刷新页面 | 用户体验不如实时更新 |
| **无客户端状态** | 无法实现"加载中"、"错误重试"等交互 | 需要配合其他方案 |

#### 适用场景

✅ **推荐用于**：
- 首屏数据展示（用户列表、产品详情）
- SEO 关键页面（博客、营销页）
- 不需要实时更新的数据（配置信息、静态内容）

❌ **不推荐用于**：
- 需要实时更新的数据（在线人数、库存数量）
- 频繁变化的数据（聊天消息、通知）
- 需要复杂状态管理的数据（分页、筛选、排序）

---

### 方式 2：Client Components useEffect + fetch（传统方案）

#### 代码示例

```typescript
// app/users/page.tsx
'use client';

import { useState, useEffect } from 'react';

export default function UsersPage() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    async function fetchUsers() {
      try {
        setLoading(true);
        const res = await fetch('/api/users');
        const data = await res.json();
        setUsers(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }

    fetchUsers();
  }, []);

  if (loading) return <div>加载中...</div>;
  if (error) return <div>错误：{error}</div>;

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

#### 执行流程

```
1. 浏览器请求 /users
2. Next.js 返回 HTML（但只有加载状态骨架）
3. 浏览器加载并执行 JS
4. useEffect 触发 fetch
5. 浏览器发请求到 /api/users
6. API 查询数据库并返回 JSON
7. setUsers 更新状态，重新渲染
8. 浏览器显示最终内容
```

#### 优势

| 优势 | 说明 |
|------|------|
| **完全控制** | 可以精确控制加载、错误、重试逻辑 |
| **实时更新** | 可以手动触发 refetch 获取最新数据 |
| **客户端状态** | 可以实现复杂的交互状态（分页、筛选） |

#### 劣势

| 劣势 | 说明 | 影响 |
|------|------|------|
| **首屏慢** | 需要等待 JS 加载、执行、请求数据 | 用户看到长时间加载状态 |
| **SEO 不友好** | 搜索引擎看到的是空壳 | 影响搜索引擎排名 |
| **代码复杂** | 需要手动管理状态、错误、加载 | 容易出现 bug |
| **无缓存** | 每次组件挂载都重新请求 | 性能差，服务器压力大 |

#### 适用场景

✅ **推荐用于**：
- 需要实时更新的数据（在线状态、通知数量）
- 用户操作后的数据刷新（提交表单后重新获取）
- 无法在服务端获取的数据（浏览器专属数据）

❌ **不推荐用于**：
- 首屏数据展示（性能差）
- SEO 关键页面（搜索引擎看不到）
- 简单的数据展示（代码过于复杂）

---

### 方式 3：SWR/React Query（现代方案）

#### 代码示例（SWR）

```typescript
// app/users/page.tsx
'use client';

import useSWR from 'swr';

const fetcher = (url: string) => fetch(url).then(res => res.json());

export default function UsersPage() {
  // SWR 自动管理数据获取、缓存、刷新
  const { data: users, error, isLoading } = useSWR('/api/users', fetcher);

  if (isLoading) return <div>加载中...</div>;
  if (error) return <div>错误：{error.message}</div>;

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

#### 代码示例（React Query / TanStack Query）

```typescript
// app/users/page.tsx
'use client';

import { useQuery } from '@tanstack/react-query';

export default function UsersPage() {
  // React Query 自动管理数据获取、缓存、刷新
  const { data: users, error, isLoading } = useQuery({
    queryKey: ['users'],
    queryFn: () => fetch('/api/users').then(res => res.json()),
  });

  if (isLoading) return <div>加载中...</div>;
  if (error) return <div>错误：{error.message}</div>;

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

#### 执行流程

```
1. 浏览器请求 /users
2. Next.js 返回 HTML（但只有加载状态骨架）
3. 浏览器加载并执行 JS
4. SWR/React Query 自动发起请求
5. 如果有缓存，立即返回缓存数据（瞬间显示）
6. 后台自动刷新获取最新数据
7. 数据更新后，自动重新渲染
```

#### 优势

| 优势 | 说明 |
|------|------|
| **智能缓存** | 自动缓存数据，避免重复请求 |
| **自动刷新** | 可配置自动重新获取（窗口聚焦、定时刷新） |
| **代码简洁** | 无需手动管理状态，几行代码搞定 |
| **功能强大** | 支持分页、无限滚动、乐观更新等 |
| **缓存共享** | 多个组件共享同一数据源，自动去重 |

#### 劣势

| 劣势 | 说明 | 影响 |
|------|------|------|
| **需要学习** | 理解缓存策略、配置选项需要时间 | 学习曲线 |
| **额外依赖** | 需要安装 SWR 或 React Query | 增加打包体积 |
| **SEO 问题** | 仍然是客户端渲染，搜索引擎可能看到空壳 | 需要配合 SSR |

#### 适用场景

✅ **推荐用于**：
- 需要实时更新的数据（在线用户、库存数量）
- 复杂的数据交互（分页、筛选、排序）
- 多组件共享数据（用户信息、购物车）
- 需要乐观更新的操作（点赞、收藏）

❌ **不推荐用于**：
- 简单的首屏数据（用 Server Components 更简单）
- SEO 关键页面（搜索引擎看不到）
- 一次性数据（不需要缓存和刷新）

---

### 三种方式对比表格

| 维度 | Server Components await | Client Components useEffect | SWR/React Query |
|------|------------------------|----------------------------|-----------------|
| **代码位置** | `async` 函数组件 | `useEffect` + `fetch` | 自定义 Hook |
| **执行环境** | 服务端 | 浏览器 | 浏览器 |
| **首屏速度** | ⚡ 快（直返 HTML） | 🐌 慢（等待 JS） | ⚡ 快（有缓存时） |
| **SEO** | ✅ 友好 | ❌ 不友好 | ❌ 不友好 |
| **客户端 JS** | ✅ 小（代码不发送） | ❌ 大（包含逻辑） | ❌ 大（包含库） |
| **实时更新** | ❌ 无（需刷新） | ✅ 手动实现 | ✅ 自动刷新 |
| **缓存** | ✅ 服务端缓存 | ❌ 无 | ✅ 智能客户端缓存 |
| **代码复杂度** | ✅ 简单 | ❌ 复杂（手动管理状态） | ✅ 简单 |
| **数据源** | 可直连数据库 | 必须通过 API | 必须通过 API |
| **适用场景** | 首屏、SEO | 简单交互 | 复杂交互、实时数据 |

### 选择决策树

```
需要数据获取
    │
    ├─ 是否是首屏数据？
    │   ├─ 是 → 是否需要 SEO？
    │   │   ├─ 是 → Server Components await ✅
    │   │   └─ 否 → 继续判断
    │   └─ 否 → 继续判断
    │
    ├─ 数据是否需要实时更新？
    │   ├─ 是 → 是否复杂交互（分页、筛选、无限滚动）？
    │   │   ├─ 是 → SWR/React Query ✅
    │   │   └─ 否 → Client Components useEffect（简单场景）
    │   └─ 否 → 继续判断
    │
    ├─ 数据是否需要在多个组件间共享？
    │   ├─ 是 → SWR/React Query ✅
    │   └─ 否 → 继续判断
    │
    └─ 默认推荐顺序：
        1. Server Components await（首屏）
        2. SWR/React Query（交互数据）
        3. Client Components useEffect（简单场景）
```

### 实战示例：混合使用

最佳实践是**混合使用**三种方式，各取所长：

```typescript
// app/dashboard/page.tsx（Server Component - 首屏数据）
import { getUserStats } from '@/lib/stats';
import { OnlineUsers } from '@/components/OnlineUsers'; // Client Component

export default async function DashboardPage() {
  // 方式 1：服务端获取静态数据（用户统计）
  const stats = await getUserStats();

  return (
    <div>
      <h1>仪表盘</h1>

      {/* Server Component：显示静态数据 */}
      <div>
        <h2>用户统计</h2>
        <p>总用户数：{stats.totalUsers}</p>
        <p>活跃用户：{stats.activeUsers}</p>
      </div>

      {/* Client Component：显示实时数据（在线用户） */}
      {/* 方式 3：SWR/React Query 自动刷新 */}
      <OnlineUsers />
    </div>
  );
}

// components/OnlineUsers.tsx（Client Component - 实时数据）
'use client';

import useSWR from 'swr';

const fetcher = (url: string) => fetch(url).then(res => res.json());

export function OnlineUsers() {
  // 方式 3：SWR 自动缓存和刷新（每 5 秒）
  const { data: users, mutate } = useSWR('/api/online-users', fetcher, {
    refreshInterval: 5000, // 每 5 秒自动刷新
  });

  return (
    <div>
      <h2>在线用户（{users?.length || 0}）</h2>
      <button onClick={() => mutate()}>立即刷新</button>
      <ul>
        {users?.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

**数据流分析**：
1. **首屏加载**：Server Component 在服务端获取静态数据，直返 HTML
2. **实时数据**：Client Component 在浏览器中用 SWR 自动刷新
3. **最佳性能**：静态数据不走客户端 JS，实时数据自动更新

### 常见错误

#### 错误 1：首屏数据用 Client Components useEffect

```typescript
// ❌ 错误：首屏数据用客户端获取
'use client';
export default function UsersPage() {
  const [users, setUsers] = useState([]);
  useEffect(() => {
    fetch('/api/users').then(setUsers);
  }, []);
  return <div>{users.length > 0 ? <UserList users={users} /> : <div>加载中</div>}</div>;
}

// ✅ 正确：首屏数据用 Server Components
export default async function UsersPage() {
  const users = await getUsers();
  return <UserList users={users} />;
}
```

**问题**：首屏慢、SEO 不友好、用户体验差

#### 错误 2：实时数据用 Server Components

```typescript
// ❌ 错误：实时数据用服务端获取（无法自动更新）
export default async function OnlineUsersPage() {
  const users = await getOnlineUsers(); // 只获取一次
  return <div>{users.map(u => <div key={u.id}>{u.name}</div>)}</div>;
}

// ✅ 正确：实时数据用 SWR/React Query
'use client';
export default function OnlineUsersPage() {
  const { data: users } = useSWR('/api/online-users', fetcher, {
    refreshInterval: 5000,
  });
  return <div>{users?.map(u => <div key={u.id}>{u.name}</div>)}</div>;
}
```

**问题**：数据不会自动更新，用户看到的是过期数据

#### 错误 3：简单交互用 SWR/React Query

```typescript
// ❌ 错误：一次性数据也用 SWR
'use client';
export default function UserProfile({ userId }: { userId: string }) {
  const { data: user } = useSWR(`/api/users/${userId}`, fetcher);
  return <div>{user?.name}</div>;
}

// ✅ 正确：一次性数据用 Server Components
export default async function UserProfile({ userId }: { userId: string }) {
  const user = await getUserById(userId);
  return <div>{user.name}</div>;
}
```

**问题**：引入不必要的依赖和复杂度

## 你需要记住的

1. **Server Components await**：首选方案，适合首屏数据和 SEO 关键页面。
2. **Client Components useEffect**：传统方案，只用于简单交互数据，性能差。
3. **SWR/React Query**：现代方案，适合实时数据、复杂交互、多组件共享数据。
4. **最佳实践**：混合使用，Server Components 负责首屏，SWR/React Query 负责实时数据。
5. **决策依据**：看数据是否需要实时更新、是否需要 SEO、交互复杂度。

## AI 代码中的线索

当你在 AI 生成的代码中看到这些模式，判断使用的数据获取方式：

```typescript
// 线索 1：async 函数组件 + await → Server Components 数据获取
export default async function UsersPage() {
  const users = await getUsers(); // 服务端获取
}

// 线索 2：useEffect + fetch → Client Components 数据获取
'use client';
useEffect(() => {
  fetch('/api/users').then(setUsers);
}, []);

// 线索 3：useSWR 或 useQuery → SWR/React Query
const { data } = useSWR('/api/users', fetcher);
const { data } = useQuery({ queryKey: ['users'], queryFn: fetchUsers });

// 线索 4：refreshInterval / revalidateOnFocus → 实时数据配置
useSWR(url, fetcher, { refreshInterval: 5000 }); // 每 5 秒刷新

// 线索 5：mutate → 手动触发刷新
const { data, mutate } = useSWR(url, fetcher);
mutate(); // 立即刷新
```

## 验证问题

- [ ] 你的项目中，首屏数据是用 Server Components 还是 Client Components 获取的？为什么？
- [ ] 如果需要显示"在线用户数"这种实时数据，你会选择哪种方式？为什么？
- [ ] SWR/React Query 相比 useEffect + fetch，最大的优势是什么？
- [ ] 什么时候应该用 Server Components 获取数据，什么时候应该用 SWR/React Query？
