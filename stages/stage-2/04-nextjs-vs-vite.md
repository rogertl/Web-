# 2.4 Next.js vs Vite 项目差异

## 一句话

Vite 是纯前端开发工具（只处理浏览器端代码），Next.js 是全栈框架（前端 + 服务端）。选哪个取决于你是否需要服务端能力。

## 为什么需要它

很多教程混用这两个工具，你可能会困惑：为什么有些项目用 Vite，有些用 Next.js？不懂差异，你可能会在需要后端的项目里用 Vite，导致后续要大量重构。或者在全栈项目里错过 Next.js 的便利，自己造轮子。

## 类比

把开发工具想象成"交通工具"：

| 工具 | 类比 |
|------|------|
| Vite | 城市自行车（适合城市通勤，灵活快速） |
| Next.js | 越野车（能跑公路，也能跑山路、沙漠） |

如果你只在城市里跑（纯前端项目），自行车够用且便宜。如果你要越野（需要后端、数据库、API），自行车不行，得换越野车。

## 核心内容

### 核心差异一览

| 特性 | Vite（纯前端） | Next.js（全栈） |
|------|---------------|----------------|
| **用途** | 只处理浏览器端代码 | 前端 + 服务端 |
| **运行环境** | 只能在浏览器运行 | 浏览器 + Node.js 服务端 |
| **代码位置** | `src/` 文件夹 | `app/` 文件夹（App Router） |
| **路由** | 需要手动配置（如 React Router） | 文件系统即路由 |
| **数据获取** | 在浏览器用 `fetch` | 在服务端用 `async/await` |
| **API** | ❌ 不能写后端接口 | ✅ 可以写 API 路由 |
| **数据库** | ❌ 不能直连数据库 | ✅ 可以在服务端连接 |
| **构建产物** | 静态文件（HTML/CSS/JS） | 静态文件 + 服务端代码 |
| **部署** | 任何静态托管（Vercel、Netlify） | 需要 Node.js 运行时（Vercel、自建服务器） |
| **学习曲线** | 低（只需懂前端） | 中（需懂前端 + 后端概念） |

### 什么时候用 Vite

**适合场景**：
- 纯前端应用（如个人博客、静态官网）
- 不需要后端逻辑（数据通过第三方 API 获取）
- 快速原型开发
- 学习 React/JavaScript 基础

**Vite 项目示例**：

```bash
# 创建 Vite + React 项目
npm create vite@latest my-app -- --template react
cd my-app
npm install
npm run dev
```

项目结构：
```
my-app/
├── src/
│   ├── main.jsx         # 入口文件（相当于 React 项目的根）
│   ├── App.jsx          # 主组件
│   └── assets/          # 静态资源
├── index.html           # HTML 模板
└── package.json
```

**Vite 数据获取**（在浏览器执行）：

```javascript
// src/App.jsx
import { useState, useEffect } from 'react';

function App() {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    // 浏览器里发起请求
    fetch('https://api.example.com/users')
      .then(res => res.json())
      .then(data => setUsers(data));
  }, []);

  return (
    <div>
      {users.map(user => <div key={user.id}>{user.name}</div>)}
    </div>
  );
}
```

### 什么时候用 Next.js

**适合场景**：
- 需要后端 API（如用户认证、数据库操作）
- 需要 SEO（搜索引擎优化）
- 需要服务端渲染（首屏速度）
- 全栈应用（前后端都写）

**Next.js 项目示例**：

```bash
# 创建 Next.js 项目
npx create-next-app@latest my-app
cd my-app
npm run dev
```

项目结构：
```
my-app/
├── app/                    # App Router（2.1 已详细讲）
│   ├── layout.tsx
│   ├── page.tsx
│   └── api/              # API 路由
├── components/
├── lib/
└── package.json
```

**Next.js 数据获取**（在服务端执行）：

```typescript
// app/users/page.tsx
import { getUsers } from '@/lib/users'; // 服务端查询函数

export default async function UsersPage() {
  // 服务端直接连接数据库，不走浏览器
  const users = await getUsers();

  return (
    <div>
      {users.map(user => <div key={user.id}>{user.name}</div>)}
    </div>
  );
}
```

### 迁移成本：Vite → Next.js

如果你在 Vite 项目里写了数据获取逻辑，迁移到 Next.js 时要改：

| Vite 写法 | Next.js 写法 | 变化 |
|----------|-------------|------|
| 数据在 `useEffect` 里获取 | 数据在 `async` 组件里获取 | 从客户端迁移到服务端 |
| 用 `fetch` 调用外部 API | 用 ORM（如 Prisma）直连数据库 | 跳过 HTTP 请求，更快 |
| 路由用 React Router | 用文件系统路由 | 删除路由配置 |
| 环境变量用 `import.meta.env` | 用 `process.env` | 语法不同 |

**迁移示例**：

```javascript
// Vite（客户端获取）
function UsersPage() {
  const [users, setUsers] = useState([]);
  useEffect(() => {
    fetch('/api/users').then(res => res.json()).then(setUsers);
  }, []);
  return <div>{users.map(...)}</div>;
}

// Next.js（服务端获取）
async function UsersPage() {
  const users = await db.user.findMany(); // 直连数据库
  return <div>{users.map(...)}</div>;
}
```

### 迁移成本：Next.js → Vite

从 Next.js 迁移到 Vite 更复杂：

| Next.js 特性 | Vite 对应方案 | 缺失 |
|-------------|-------------|------|
| Server Components | ❌ 没有对应概念 | 需要改回客户端渲染 |
| API 路由（`app/api/`） | ❌ 需要单独后端服务 | 需要新建 Express/Fastify 服务 |
| 服务端渲染 | ❌ 只有客户端渲染 | SEO 受影响 |
| 文件系统路由 | 需要手动配置 React Router | 增加配置复杂度 |

**建议**：如果你的项目已经用了 Next.js 的服务端特性，迁移到 Vite 基本等于重写。

### 如何选择：决策树

```
需要后端（数据库、API、认证）？
├─ 是 → 用 Next.js
└─ 否 → 需要 SEO？
    ├─ 是 → 用 Next.js（静态导出也能做，但 Next.js 更方便）
    └─ 否 → 项目复杂度？
        ├─ 高（多页面、状态管理复杂） → 用 Vite（灵活、社区大）
        └─ 低 → 任意，看团队熟悉度
```

**经验法则**：
- 新手学全栈 → Next.js（一站式，减少决策疲劳）
- 已有后端服务 → Vite（前端独立部署，解耦）
- 企业级应用 → Next.js（统一技术栈，减少运维复杂度）

### 混合使用：可以同时用吗？

可以，但一般没必要。典型混合场景：
- 主站用 Next.js（SEO + 服务端）
- 管理后台用 Vite（纯前端，交互复杂）

但这样会增加技术栈复杂度，小团队慎选。

## 你需要记住的

1. Vite = 纯前端，Next.js = 全栈（前端 + 后端）。
2. 如果需要数据库、API、认证 → 用 Next.js。
3. 如果只是纯前端展示 → Vite 够用，且启动更快。
4. Next.js 项目迁移到 Vite 基本等于重写（放弃服务端能力）。
5. Vite 项目迁移到 Next.js 相对容易（主要是改数据获取方式）。

## AI 代码中的线索

当你在 AI 生成的代码中看到这些模式，说明它是 Next.js 项目：

```typescript
// 线索 1：app/ 文件夹（App Router）
export default function SomePage() {
  // 这是 Next.js App Router
}

// 线索 2：async page 组件（服务端渲染）
export default async function UsersPage() {
  const users = await getUsers(); // 服务端获取
  return <div>{...}</div>;
}

// 线索 3：API 路由
export async function GET() {
  return Response.json({ data: '...' });
}

// 线索 4：从 @/lib 导入数据库操作
import { db } from '@/lib/db';
```

当看到这些模式，说明是 Vite 项目：

```javascript
// 线索 1：src/ 文件夹
// 线索 2：main.jsx 或 main.tsx 入口文件
// 线索 3：React Router 配置
// 线索 4：数据获取在 useEffect 里
```

## 验证问题

- [ ] 你在做一个需要用户登录、显示个性化数据的项目。应该用 Vite 还是 Next.js？为什么？
- [ ] 以下代码在 Next.js 中能正常运行吗？如果不能，需要改什么？
  ```javascript
  // src/App.jsx（Vite 写法）
  function App() {
    const [data, setData] = useState(null);
    useEffect(() => {
      fetch('/api/data').then(res => res.json()).then(setData);
    }, []);
    return data ? <div>{data.name}</div> : <div>加载中</div>;
  }
  ```
- [ ] 为什么说"Next.js 项目迁移到 Vite 基本等于重写"？从服务端渲染的角度解释。
