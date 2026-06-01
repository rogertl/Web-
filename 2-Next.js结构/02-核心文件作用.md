# 2.2 核心文件作用：layout.tsx、page.tsx、lib/、components/

## 一句话

Next.js 项目里几个关键文件各司其职：`page.tsx` 写页面内容，`layout.tsx` 写共享布局，`lib/` 放工具函数，`components/` 放 UI 组件。

## 为什么需要它

没有这些约定，你的代码会乱成一团。页面逻辑、UI 组件、工具函数、数据查询全都混在一起，改个功能要翻半天。这些文件和文件夹是"组织原则"，让代码各归各位，维护时一眼就能找到要改的地方。

## 类比

把 Next.js 项目想象成"餐厅厨房"：

| 概念 | 类比 |
|------|------|
| app/ 文件夹 | 餐厅大厅（顾客看到的区域） |
| page.tsx | 菜单（每道菜的说明） |
| layout.tsx | 餐厅装修（桌椅、灯光、音乐，整个餐厅共用） |
| components/ | 半成品食材（切好的蔬菜、腌制好的肉，多处复用） |
| lib/ | 厨房工具（刀具、砧板、调料，不直接给顾客看，但做菜必需） |
| public/ | 餐厅入口（招牌、海报，所有人都能看到的静态资源） |

你不会把生肉、炒锅、餐桌、菜单堆在一个房间。同样的，代码也应该分类存放。

## 核心内容

### page.tsx：页面的"内容"

`page.tsx` 是路由对应的页面内容。它必须**默认导出**一个 React 组件。

```typescript
// app/dashboard/page.tsx
export default function DashboardPage() {
  return (
    <div>
      <h1>仪表盘</h1>
      <p>欢迎回来，用户</p>
    </div>
  );
}
```

**规则**：
- 每个 URL 路径必须有 `page.tsx` 才能访问（否则 404）
- 必须是默认导出（`export default`）
- 可以是 `async` 函数（阶段 3 会详细讲 Server Components）

### layout.tsx：页面的"容器"

`layout.tsx` 定义共享布局。它也必须默认导出一个组件，并且接收 `children` 参数。

```typescript
// app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div>
      <header>
        <h1>我的应用</h1>
        <nav>导航栏</nav>
      </header>
      {children}
      <footer>版权信息</footer>
    </div>
  );
}
```

`children` 是什么？是 Next.js 自动注入的——就是你定义的 `page.tsx` 内容。

**嵌套规则**：
- `app/layout.tsx` → 根布局（作用于整个应用）
- `app/dashboard/layout.tsx` → 只作用于 `/dashboard` 及其子路径
- 子路径的 layout 会**包裹**在父 layout 里面

```
访问 /dashboard/settings：
app/layout.tsx
  └─ app/dashboard/layout.tsx
      └─ app/dashboard/settings/page.tsx
```

### components/：可复用的 UI 组件

`components/` 文件夹放你自己在多个地方用的 UI 组件。

```typescript
// components/Button.tsx
export default function Button({ children, onClick }: {
  children: React.ReactNode;
  onClick?: () => void;
}) {
  return (
    <button onClick={onClick} className="px-4 py-2 bg-blue-500 rounded">
      {children}
    </button>
  );
}

// app/dashboard/page.tsx
import Button from '@/components/Button';

export default function DashboardPage() {
  return (
    <div>
      <h1>仪表盘</h1>
      <Button onClick={() => alert('点击了')}>点击我</Button>
    </div>
  );
}
```

**命名约定**：组件名用大写开头（PascalCase），文件名可以和组件名一样或用 kebab-case。

### lib/：工具函数和数据查询

`lib/` 文件夹放不直接渲染 UI 的代码——工具函数、数据查询、格式化逻辑等。

```typescript
// lib/utils.ts
export function formatDate(date: Date): string {
  return date.toLocaleDateString('zh-CN');
}

export function capitalize(str: string): string {
  return str.charAt(0).toUpperCase() + str.slice(1);
}

// lib/users.ts
export async function getUserById(id: string) {
  const response = await fetch(`https://api.example.com/users/${id}`);
  return response.json();
}

// app/users/[userId]/page.tsx
import { getUserById } from '@/lib/users';

export default async function UserPage({ params }: { params: { userId: string } }) {
  const user = await getUserById(params.userId);

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

**为什么叫 lib？** 是 "library" 的缩写，意思是"代码库"——存放可复用的、非 UI 的代码。

### public/：静态资源

`public/` 文件夹放图片、字体、favicon 等静态文件。这些文件可以通过 `/` 直接访问。

```
public/
├── images/
│   └── logo.png        # 访问：/images/logo.png
├── favicon.ico          # 访问：/favicon.ico
└── robots.txt           # 访问：/robots.txt
```

```typescript
// app/page.tsx
export default function HomePage() {
  return (
    <div>
      <img src="/images/logo.png" alt="Logo" />
    </div>
  );
}
```

**注意**：`public/` 的文件不会经过 Next.js 处理，直接返回原文件。适合不需要优化的资源。

### 完整的项目结构

```
my-nextjs-app/
├── app/                      # App Router（所有页面和路由）
│   ├── layout.tsx           # 根布局（整个应用共用）
│   ├── page.tsx             # 首页 (/)
│   ├── dashboard/           # 仪表盘模块
│   │   ├── layout.tsx       # 仪表盘专用布局
│   │   ├── page.tsx         # /dashboard 页面
│   │   ├── settings/        # /dashboard/settings
│   │   └── analytics/       # /dashboard/analytics
│   └── api/                 # API 路由（4.3 会详细讲）
├── components/              # 自定义 UI 组件
│   ├── Button.tsx
│   ├── Modal.tsx
│   └── Navigation.tsx
├── lib/                     # 工具函数和数据查询
│   ├── utils.ts
│   ├── db.ts               # 数据库连接（阶段 5 会详细讲）
│   └── queries.ts          # 数据查询函数
├── public/                  # 静态资源
│   ├── images/
│   └── favicon.ico
├── package.json            # 项目配置和依赖（0.2 已详细讲）
├── next.config.js          # Next.js 配置文件
└── tsconfig.json            # TypeScript 配置（0.4 已详细讲）
```

## 你需要记住的

1. `page.tsx` = 页面内容（每个路由必须有）。
2. `layout.tsx` = 共享布局（导航、侧边栏、页脚等）。
3. `components/` = 可复用 UI 组件（按钮、卡片、表单等）。
4. `lib/` = 工具函数和数据查询（不直接渲染 UI）。
5. `public/` = 静态资源（图片、字体等，直接通过 `/` 访问）。
6. 这些约定让代码各归各位，维护时快速定位要改的文件。

## AI 代码中的线索

当你在 AI 生成的代码中看到这些模式，说明它在遵循这些约定：

```typescript
// 线索 1：从 @/components 导入 UI 组件
import { Button, Card } from '@/components/ui';

// 线索 2：从 @/lib 导入工具函数
import { formatDate } from '@/lib/utils';
import { db } from '@/lib/db';

// 线索 3：page.tsx 的默认导出
export default function SomePage() {
  // 这是页面组件
}

// 线索 4：layout.tsx 接收 children 参数
export default function SomeLayout({ children }: { children: React.ReactNode }) {
  return <div>{children}</div>;
}

// 线索 5：从 public/ 引用图片
<img src="/images/logo.png" alt="Logo" />

// 线索 6：Server Component 里直接 await（阶段 3 会详细讲）
export default async function UserPage({ params }: { params: { userId: string } }) {
  const user = await getUserById(params.userId); // 从 lib/ 导入的查询函数
  return <div>{user.name}</div>;
}
```

如果看到 `pages/` 文件夹、`getServerSideProps`、或工具函数混在组件文件里，这是旧模式或不规范的结构。

## 验证问题

- [ ] 你在 `app/dashboard/layout.tsx` 写了一个导航栏，它会在 `/dashboard/settings` 页面显示吗？为什么？
- [ ] 以下代码应该放在哪个文件夹？
  ```typescript
  export function calculateDiscount(price: number, percentage: number): number {
    return price * (1 - percentage / 100);
  }
  ```
- [ ] 如果你想在多个页面都显示一个"返回顶部"按钮，应该放在 `components/` 还是 `lib/`？为什么？
