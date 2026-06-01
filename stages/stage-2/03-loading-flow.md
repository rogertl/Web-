# 2.3 代码加载流程（从浏览器请求到页面显示的全链路）

## 一句话

你在浏览器输入 URL 按回车，Next.js 按固定顺序处理请求：找到对应的 `page.tsx`，加载数据，套用 layout，渲染成 HTML 返回给浏览器。

## 为什么需要它

不知道这个流程，遇到页面空白或报错时，你会像在黑盒子里操作——不知道问题出在哪。懂了流程，你能快速定位是路由错误、数据查询失败，还是组件渲染问题。这是调试的基础。

## 类比

把页面加载想象成"餐厅上菜"流程：

| 步骤 | 技术概念 | 类比 |
|------|---------|------|
| 1. 用户请求 URL | 浏览器发起 HTTP 请求 | 顾客点菜 |
| 2. Next.js 找到对应文件 | 路由匹配 | 服务员找到对应菜单 |
| 3. 加载数据（如数据库查询） | Server Component 数据获取 | 厨师准备食材 |
| 4. 应用 Layout | 嵌套布局渲染 | 摆盘、装饰 |
| 5. 生成 HTML | 服务端渲染 | 装盘完成 |
| 6. 返回给浏览器 | HTTP 响应 | 端给顾客 |
| 7. 浏览器显示页面 | 渲染 HTML | 顾客看到菜 |

你在第 6 步之前看不到任何东西，但后端已经完成了 5 个步骤。任何一个步骤出错，都影响最终显示。

## 核心内容

### 完整加载流程

以访问 `/dashboard/settings` 为例：

```
浏览器输入 URL → HTTP GET /dashboard/settings
           ↓
Next.js 接收请求，开始处理
           ↓
1. 路由匹配：找到 app/dashboard/settings/page.tsx
           ↓
2. 加载布局树：
   app/layout.tsx（根布局）
   └─ app/dashboard/layout.tsx（仪表盘布局）
      └─ app/dashboard/settings/page.tsx（页面内容）
           ↓
3. 渲染 Server Components（阶段 3 会详细讲）：
   - 执行 app/dashboard/layout.tsx 的代码（生成导航栏）
   - 执行 app/dashboard/settings/page.tsx 的代码（如果这是 async 函数，会 await 数据）
           ↓
4. 生成 HTML：把所有组件渲染成的 HTML 合并
           ↓
5. 返回响应：HTTP 200 + HTML 内容
           ↓
浏览器接收 HTML，解析并显示页面
```

### 关键时间点

| 时间点 | 发生在 | 说明 |
|--------|--------|------|
| T0：用户输入 URL | 浏览器 | 用户行为 |
| T1：Next.js 接收请求 | 服务端 | 请求到达 Next.js 服务器 |
| T2：路由匹配完成 | 服务端 | Next.js 找到了对应的 `page.tsx` |
| T3：数据加载完成 | 服务端 | 如果 `page.tsx` 是 `async`，这里等待数据 |
| T4：HTML 生成完成 | 服务端 | Next.js 把所有组件渲染成 HTML |
| T5：响应返回浏览器 | 浏览器 | 浏览器开始渲染 HTML |
| T6：页面完全显示 | 浏览器 | 用户看到最终页面 |

T0 到 T5 在服务端完成（对用户不可见），T5 到 T6 在浏览器完成。

### 如果某个步骤出错

| 出错点 | 典型原因 | 用户看到 |
|--------|---------|---------|
| T2 路由匹配失败 | `page.tsx` 不存在 | 404 页面 |
| T3 数据加载超时 | 数据库慢、API 无响应 | 加载状态（如果定义了 `loading.tsx`）或超时错误 |
| T3 数据返回错误 | 查询语法错误、API 500 | 错误页面（如果定义了 `error.tsx`） |
| T4 渲染错误 | 组件代码有 bug | 错误页面或白屏 |

这就是为什么 Next.js 提供了 `loading.tsx` 和 `error.tsx`——让 T3 出错时用户能看到友好的提示。

### 实例：一个带数据的页面

```typescript
// app/users/[userId]/page.tsx
import { getUserById } from '@/lib/users';

export default async function UserPage({ params }: { params: { userId: string } }) {
  // T3：这里会等待数据
  const user = await getUserById(params.userId);

  // T4：数据拿到后，渲染 JSX
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

浏览器请求 `/users/123` 时的实际流程：

```
GET /users/123 → Next.js 接收
           ↓
找到 app/users/[userId]/page.tsx
           ↓
params = { userId: '123' }
           ↓
执行 getUserById('123') → 等待数据库响应
           ↓
拿到数据：{ name: 'Alice', email: 'alice@example.com' }
           ↓
渲染 JSX 成 HTML：<div><h1>Alice</h1><p>alice@example.com</p></div>
           ↓
套用 layout（如果有），生成完整 HTML
           ↓
返回给浏览器
```

### loading.tsx 和 error.tsx 的角色

```typescript
// app/users/[userId]/loading.tsx
export default function Loading() {
  return <div>加载中...</div>;
}

// app/users/[userId]/error.tsx
export default function Error({ error }: { error: Error }) {
  return <div>出错了：{error.message}</div>;
}
```

当 `getUserById` 耗时超过 1 秒：
- Next.js 先显示 `loading.tsx` 的内容
- 数据返回后，替换成实际的 `page.tsx` 内容

当 `getUserById` 抛出错误：
- Next.js 显示 `error.tsx` 的内容
- 不会把错误堆栈直接暴露给用户（安全性）

### 客户端导航的情况

你在页面里点击 `<Link href="/dashboard">`，不是浏览器刷新，而是 Next.js 的**客户端导航**：

```
点击 Link → JavaScript 拦截（不刷新浏览器）
         ↓
Next.js 在后台发起请求 /dashboard
         ↓
收到 HTML 后，用 JavaScript 替换当前页面内容
         ↓
用户看到新页面，但 URL 和浏览历史都更新了
```

这就是 SPA（单页应用）的感觉（0.3 已详细讲），但 Next.js 帮你处理了路由和渲染。

### Next.js vs 传统 SPA 的加载对比

| 特性 | Next.js（SSR） | 传统 SPA（如纯 React） |
|------|---------------|---------------------|
| 首次加载 | 服务端渲染 HTML，直接显示 | 浏览器下载空 HTML + JS，执行 JS 后才显示 |
| SEO（搜索引擎优化） | 搜索引擎直接读到内容 | 搜索引擎只能读到空壳 |
| 首屏速度 | 快（直接显示 HTML） | 慢（等待 JS 执行） |
| 后续导航 | 客户端导航，快速切换 | 客户端路由，快速切换 |

Next.js 结合了两者的优点：首屏像传统网站一样快，后续导航像 SPA 一样流畅。

## 你需要记住的

1. 浏览器请求 URL → Next.js 找到对应的 `page.tsx` → 加载数据 → 渲染 HTML → 返回给浏览器。
2. 如果 `page.tsx` 是 `async` 函数，Next.js 会等待数据返回再继续渲染。
3. `loading.tsx` 在数据加载时显示，`error.tsx` 在出错时显示。
4. 整个流程在服务端完成（T1-T5），浏览器只负责最后渲染（T5-T6）。
5. 客户端导航（Link 组件）不走浏览器刷新，在后台请求数据并用 JavaScript 替换页面内容。

## AI 代码中的线索

当你在 AI 生成的代码中看到这些模式，说明它在利用这个加载流程：

```typescript
// 线索 1：async page 组件（利用 T3 数据加载）
export default async function UserPage({ params }: { params: { userId: string } }) {
  const user = await getUserById(params.userId); // T3 等待数据
  return <div>{user.name}</div>;
}

// 线索 2：loading.tsx（优化 T3 用户体验）
export default function Loading() {
  return <Skeleton />; // 显示骨架屏
}

// 线索 3：error.tsx（处理 T3 错误）
export default function Error({ error }: { error: Error }) {
  return <div>错误：{error.message}</div>;
}

// 线索 4：Link 组件（客户端导航）
import Link from 'next/link';
<Link href="/dashboard">仪表盘</Link> // 不刷新浏览器，后台请求

// 线索 5：useRouter 钩子（编程式客户端导航）
import { useRouter } from 'next/navigation';
const router = useRouter();
router.push('/settings'); // 等同于点击 Link
```

如果看到数据获取在 `useEffect` 里做，这是旧模式（客户端渲染），不是 Next.js 推荐的做法。

## 验证问题

- [ ] 你访问 `/users/123`，页面一直显示"加载中..."。最可能的问题出在加载流程的哪个步骤？为什么？
- [ ] 以下代码中，`console.log` 会按什么顺序输出？
  ```typescript
  // app/page.tsx
  export default async function HomePage() {
    console.log('1: 组件开始执行');
    const data = await fetchData();
    console.log('2: 数据已加载');
    return <div>{data.name}</div>;
  }
  ```
- [ ] 为什么 Next.js 首屏加载比纯 React SPA 快？从加载流程的角度解释。
