# 2.1 App Router 项目结构（app/ 文件夹的意义）

## 一句话

App Router 是 Next.js 13+ 的文件系统路由，你在 `app/` 文件夹里按目录结构建文件，就自动变成了网站的 URL 路由和页面。

## 为什么需要它

没有 App Router，你需要手动配置路由、导入组件、嵌套布局。每次加个页面，都要改好几处代码。App Router 让你"文件即路由"，新建文件夹和文件，路由自动生效。这不是懒，是避免在重复的配置上浪费时间。

## 类比

把 `app/` 文件夹想象成"公司组织架构图"：

| 概念 | 类比 |
|------|------|
| app/ 目录 | 公司总部（所有部门的起点） |
| 一级子文件夹（如 app/dashboard/） | 一级部门（如市场部） |
| 二级子文件夹（如 app/dashboard/settings/） | 二级部门（市场部的SEO小组） |
| page.tsx 文件 | 部门职责说明（这个部门做什么） |
| layout.tsx 文件 | 部门装修风格（这个部门的长相） |
| loading.tsx 文件 | 部门前台（等待时显示的加载状态） |
| error.tsx 文件 | 部门应急预案（出错时显示的页面） |

你的公司架构怎么画，网站结构就怎么建。一一对应，不需要额外解释。

## 核心内容

### App Router 的基本规则

Next.js 13+ 使用 App Router（之前叫 Pages Router）。核心规则就 4 条：

| 规则 | 说明 | 示例 |
|------|------|------|
| 每个文件夹 = 一段 URL | `app/dashboard/` 变成 `/dashboard` | `app/users/settings/` → `/users/settings` |
| `page.tsx` = 页面内容 | 这个文件的内容就是页面显示的 | `app/page.tsx` 是首页 `/` |
| `layout.tsx` = 共享布局 | 这个文件夹及子文件夹共用这个布局 | `app/dashboard/layout.tsx` 作用于 `/dashboard` 及其子路径 |
| 文件夹名称支持动态参数 | `[id]` 文件夹匹配任意值 | `app/users/[userId]/page.tsx` → `/users/123` |

### 最小示例：一个首页

```typescript
// app/page.tsx
export default function HomePage() {
  return (
    <div>
      <h1>欢迎来到我的网站</h1>
      <p>这是首页</p>
    </div>
  );
}
```

保存这个文件，访问 `/` 就能看到这段内容。不需要任何路由配置，Next.js 自动识别。

### 添加二级页面

```typescript
// app/dashboard/page.tsx
export default function DashboardPage() {
  return (
    <div>
      <h1>仪表盘</h1>
      <p>这是你的数据中心</p>
    </div>
  );
}
```

访问 `/dashboard` 自动显示这个页面。你只建了个文件，路由自己就有了。

### 添加动态路由

```typescript
// app/users/[userId]/page.tsx
interface Params {
  userId: string;
}

export default function UserPage({ params }: { params: Params }) {
  return (
    <div>
      <h1>用户 {params.userId}</h1>
      <p>正在查看用户详情</p>
    </div>
  );
}
```

访问 `/users/123`，显示"用户 123"；访问 `/users/abc`，显示"用户 abc"。`[userId]` 是个占位符，匹配任意值。

### 添加布局（Layout）

```typescript
// app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div>
      <nav>
        <a href="/dashboard">首页</a>
        <a href="/dashboard/settings">设置</a>
      </nav>
      <main>{children}</main>
      <footer>© 2025 我的公司</footer>
    </div>
  );
}
```

现在 `/dashboard` 和 `/dashboard/settings` 都有这个布局：顶部导航 + 中间内容 + 底部版权。`children` 是 Next.js 自动传入的，就是你定义的 `page.tsx` 内容。

### 文件夹结构一目了然

```
app/
├── page.tsx                  # 首页 /
├── layout.tsx                # 全局布局（作用于整个 app）
├── dashboard/
│   ├── page.tsx             # /dashboard
│   ├── layout.tsx           # /dashboard 的专用布局
│   ├── settings/
│   │   └── page.tsx        # /dashboard/settings
│   └── analytics/
│       └── page.tsx        # /dashboard/analytics
├── users/
│   ├── page.tsx            # /users
│   └── [userId]/
│       └── page.tsx        # /users/123
└── api/
    └── users/
        └── route.ts        # /api/users（API 路由，4.3 会详细讲）
```

看文件夹结构就知道 URL 是什么，不用查配置文件。

### App Router vs Pages Router（旧版本）

你可能在网上看到 Next.js 12 的教程用 `pages/` 文件夹。区别：

| 特性 | App Router (app/) | Pages Router (pages/) |
|------|-------------------|----------------------|
| 布局方式 | 用 `layout.tsx` 嵌套 | 手动组件组合 |
| 数据获取 | 直接在组件里 `async/await` | 用 `getServerSideProps` |
| 文件位置 | 文件系统即路由 | 文件系统即路由 |
| Server Components | 默认支持（阶段 3 会详细讲） | 不支持 |
| 推荐程度 | ✅ 新项目默认用这个 | ⚠️ 旧项目可能还在用 |

本教程专注 App Router，因为它是现代 Next.js 的标准。

## 你需要记住的

1. `app/` 文件夹的文件夹结构 = 网站的 URL 结构。
2. `page.tsx` 文件决定这个路径显示什么内容。
3. `layout.tsx` 文件给这个路径及子路径添加共享布局（导航、侧边栏、页脚）。
4. `[参数名]` 文件夹支持动态路由，参数值从 `params` 对象获取。
5. App Router 是 Next.js 13+ 的默认方式，比旧的 `pages/` 文件夹更现代。

## AI 代码中的线索

当你在 AI 生成的代码中看到这些模式，说明它在用 App Router：

```typescript
// 线索 1：文件路径以 app/ 开头
import { SomeComponent } from '@/components/some-component';

// 线索 2：组件从 app/ 文件夹导出
export default function SomePage() {
  // 这是 app/xxx/page.tsx 的典型写法
}

// 线索 3：组件接收 params 参数（动态路由）
export default function UserDetailPage({ params }: { params: { userId: string } }) {
  // params.userId 是 URL 中的动态部分
}

// 线索 4：async 函数组件（Server Component，阶段 3 会详细讲）
export default async function ProductList() {
  const products = await fetchProducts(); // 直接在组件里 await
  return <div>{...}</div>;
}

// 线索 5：layout.tsx 文件导出默认函数接收 children
export default function SomeLayout({ children }: { children: React.ReactNode }) {
  return <div>{children}</div>;
}
```

相反，如果看到 `pages/` 文件夹或 `getServerSideProps` 函数，这是旧的 Pages Router，不是本教程的重点。

## 验证问题

- [ ] 如果你在 `app/dashboard/analytics/reports/page.tsx` 创建文件，对应的 URL 是什么？
- [ ] `app/users/[userId]/settings/page.tsx` 能匹配 `/users/123/settings` 吗？为什么？
- [ ] 以下代码中，`params` 包含什么值？
  ```typescript
  // app/products/[category]/[id]/page.tsx
  export default function ProductPage({ params }: { params: { category: string; id: string } }) {
    // params = ?
  }
  ```
  访问 URL：`/products/electronics/42`
