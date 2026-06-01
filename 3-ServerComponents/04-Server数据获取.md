# 3.4 数据获取在 Server Component 中的写法（async/await）

## 一句话

Server Components 可以直接写成 `async` 函数，用 `await` 获取数据，Next.js 在服务端执行并渲染成 HTML，数据获取和渲染一次完成。

## 为什么需要它

传统方式（客户端数据获取）需要：浏览器加载 JS → 执行代码 → 发请求 → 等数据 → 渲染。用户看到长时间空加载。Server Components 让数据获取移到服务端：服务端获取数据 → 渲染 HTML → 直接返回。用户瞬间看到内容。

## 类比

把数据获取想象成"点菜方式"：

| 方式 | 类比 | 流程 | 速度 |
|------|------|------|------|
| Server Component 数据获取 | 餐厅提前备菜 | 厨房做好 → 直接端上来 | 快（直接成品） |
| 客户端数据获取 | 餐厅现做 | 点菜 → 厨房做 → 端上来 | 慢（等待） |
| 数据库 | 食材仓库 | 厨房直连，餐桌不能直连 | - |
| async/await | 厨师等待 | 等食材准备好再烹饪 | - |

Server Components 是"提前备菜"，客户端获取是"现点现做"。

## 核心内容

### Server Components 数据获取的核心特征

| 特征 | 说明 | 示例 |
|------|------|------|
| 直接 `async` | 组件可以是 `async` 函数 | `export default async function Page() { ... }` |
| 使用 `await` | 可以直接 `await` 数据获取 | `const users = await getUsers()` |
| 在服务端执行 | 数据获取在 Node.js 环境，不暴露给浏览器 | `db.user.findMany()` |
| 一次渲染 | 数据获取和渲染在服务端一次完成 | 用户直接看到最终 HTML |
| 减少客户端 JS | 数据获取代码不发送到浏览器 | 减少客户端体积 |
| 错误自动处理 | `await` 失败时触发 `error.tsx` | 无需手动 try-catch |

### 基本示例

```typescript
// app/users/page.tsx
import { getUsers } from '@/lib/users';

// Server Component 可以直接 async
export default async function UsersPage() {
  // 直接 await 数据获取
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

**执行流程**：
1. 浏览器请求 `/users`
2. Next.js 在服务端执行 `UsersPage` 组件
3. `getUsers()` 在服务端查询数据库
4. Next.js 等待数据返回（`await`）
5. 数据返回后，Next.js 渲染 JSX 成 HTML
6. HTML 返回给浏览器显示
7. **注意**：`UsersPage` 和 `getUsers` 的代码不会发送到浏览器

### 数据获取函数的位置

数据获取函数通常放在 `lib/` 文件夹：

```typescript
// lib/users.ts
import { db } from '@/lib/db'; // 数据库连接（阶段 5 会详细讲）

export async function getUsers() {
  // 服务端直接查询数据库
  const users = await db.user.findMany();
  return users;
}

export async function getUserById(id: string) {
  const user = await db.user.findUnique({
    where: { id: parseInt(id) },
  });
  return user;
}
```

**为什么放 `lib/`**：
- `lib/` 是工具函数和数据查询的约定位置（2.2 已详细讲）
- 数据获取逻辑和 UI 组件分离，便于复用
- 服务端专属代码（数据库查询）不应该放在组件里

### 并行数据获取

如果需要获取多个独立数据，用 `Promise.all` 并行获取：

```typescript
// app/dashboard/page.tsx
import { getUsers } from '@/lib/users';
import { getProducts } from '@/lib/products';
import { getOrders } from '@/lib/orders';

export default async function DashboardPage() {
  // 并行获取三个独立数据
  const [users, products, orders] = await Promise.all([
    getUsers(),
    getProducts(),
    getOrders(),
  ]);

  return (
    <div>
      <h1>仪表盘</h1>
      <Stats users={users} products={products} orders={orders} />
    </div>
  );
}
```

**为什么并行**：
- 如果串行（`await getUsers(); await getProducts();`），总耗时 = 各个耗时之和
- 如果并行（`await Promise.all([...])`），总耗时 = 最慢的那个
- 三个各 1 秒的请求：串行 3 秒，并行 1 秒

### 依赖数据获取：串行

如果第二个数据依赖第一个数据的结果，必须串行：

```typescript
// app/users/[userId]/posts/page.tsx
import { getUserById } from '@/lib/users';
import { getPostsByUserId } from '@/lib/posts';

export default async function UserPostsPage({ params }: { params: { userId: string } }) {
  // 先获取用户（验证用户存在）
  const user = await getUserById(params.userId);
  if (!user) return <div>用户不存在</div>;

  // 再获取该用户的文章（依赖第一步的结果）
  const posts = await getPostsByUserId(user.id);

  return (
    <div>
      <h1>{user.name} 的文章</h1>
      <ul>
        {posts.map(post => <li key={post.id}>{post.title}</li>)}
      </ul>
    </div>
  );
}
```

**为什么串行**：
- `getPostsByUserId` 需要 `user.id`，必须等 `getUserById` 返回
- 不能并行，因为第二步依赖第一步

### 错误处理：自动触发 error.tsx

Server Components 的 `await` 失败时，Next.js 自动触发 `error.tsx`：

```typescript
// app/users/page.tsx
export default async function UsersPage() {
  // 如果 getUsers() 抛出错误，Next.js 自动显示 error.tsx
  const users = await getUsers();

  return <div>{...}</div>;
}

// app/users/error.tsx
'use client'; // error.tsx 必须是 Client Component

export default function Error({ error, reset }: { error: Error; reset: () => void }) {
  return (
    <div>
      <h2>出错了：{error.message}</h2>
      <button onClick={reset}>重试</button>
    </div>
  );
}
```

**无需手动 try-catch**：Next.js 自动捕获错误并显示 `error.tsx`。如果你想自定义错误处理：

```typescript
export default async function UsersPage() {
  let users;
  try {
    users = await getUsers();
  } catch (error) {
    return <div>加载失败，请稍后重试</div>;
  }

  return <div>{...}</div>;
}
```

### 数据缓存：避免重复请求

Next.js 自动缓存 Server Components 的数据获取：

```typescript
// app/users/page.tsx
export default async function UsersPage() {
  // 第一次请求：查询数据库
  const users = await getUsers();

  return (
    <div>
      <UserList users={users} />
      {/* 如果 UserList 也调用 getUsers()，Next.js 用缓存，不重复查询 */}
    </div>
  );
}
```

**缓存规则**：
- 同一个请求周期内，相同的数据获取函数会被缓存
- 可以用 `revalidatePath` 或 `revalidateTag` 手动清除缓存（阶段 5 会详细讲）
- 开发环境（`npm run dev`）默认关闭缓存，生产环境默认开启

### 动态数据 vs 静态数据

| 数据类型 | 特征 | 示例 |
|---------|------|------|
| 动态数据 | 每次请求都重新获取 | 用户数据、库存数量 |
| 静态数据 | 构建时获取一次，之后复用 | 产品信息、博客文章 |

Next.js 默认所有 Server Components 都是动态的（每次请求都重新获取数据）。如果你想静态化：

```typescript
// export const revalidate = 3600; // 每 3600 秒（1小时）重新获取一次
// 或 export const dynamic = 'force-static'; // 完全静态，构建时获取一次

export default async function ProductsPage() {
  const products = await getProducts(); // 静态数据
  return <div>{...}</div>;
}
```

**什么时候静态化**：
- 数据不经常变化（产品列表、文档）
- 数据量大，不想每次都查询
- 需要全球 CDN 加速

### 与 Client Components 数据获取的对比

| 特性 | Server Components 数据获取 | Client Components 数据获取 |
|------|-------------------------|-------------------------|
| 代码位置 | `async` 函数组件，直接 `await` | `useEffect` + `fetch` |
| 执行环境 | 服务端（Node.js） | 浏览器 |
| 数据源 | 可以直连数据库 | 只能通过 API |
| 首屏速度 | 快（直接返回 HTML） | 慢（等待 JS 加载和执行） |
| SEO | 友好（搜索引擎能读取 HTML） | 不友好（搜索引擎看到空壳） |
| 客户端 JS 体积 | 小（数据获取代码不发送） | 大（包含数据获取逻辑） |

**示例对比**：

```typescript
// Server Components（推荐）
export default async function UsersPage() {
  const users = await getUsers(); // 服务端获取
  return <UserList users={users} />;
}

// Client Components（不推荐用于首屏数据）
'use client';
export default function UsersPage() {
  const [users, setUsers] = useState([]);
  useEffect(() => {
    fetch('/api/users').then(res => res.json()).then(setUsers); // 浏览器获取
  }, []);
  return users.length ? <UserList users={users} /> : <div>加载中</div>;
}
```

## 你需要记住的

1. Server Components 可以是 `async` 函数，直接 `await` 数据获取。
2. 数据获取在服务端执行，不暴露给浏览器，安全且快速。
3. 多个独立数据用 `Promise.all` 并行获取，依赖数据串行获取。
4. Next.js 自动缓存数据获取结果，避免重复查询。
5. Server Components 数据获取比 Client Components 快且 SEO 友好。

## AI 代码中的线索

当你在 AI 生成的代码中看到这些模式，说明这是 Server Components 数据获取：

```typescript
// 线索 1：async 函数组件
export default async function UsersPage() {
  // Server Components 数据获取
}

// 线索 2：直接 await 数据获取
const users = await getUsers();
const products = await getProducts();

// 线索 3：Promise.all 并行获取
const [users, products] = await Promise.all([
  getUsers(),
  getProducts(),
]);

// 线索 4：从 lib/ 导入数据获取函数
import { getUsers } from '@/lib/users';

// 线索 5：数据库查询（Prisma）
const users = await db.user.findMany();

// 线索 6：fetch 到外部 API（也在服务端执行）
const data = await fetch('https://api.example.com/data', {
  cache: 'no-store', // 可选：禁用缓存
});
```

如果看到 `useEffect` + `fetch`，这是 Client Components 数据获取（不推荐用于首屏）。

## 验证问题

- [ ] 以下代码是并行还是串行数据获取？为什么？
  ```typescript
  export default async function DashboardPage() {
    const user = await getUser();
    const posts = await getPostsByUserId(user.id);
    return <div>{...}</div>;
  }
  ```
- [ ] 如果你想同时获取用户列表和产品列表（两者独立），应该用什么方式？为什么？
- [ ] Server Components 数据获取和 Client Components 数据获取，哪个对 SEO 更友好？为什么？
