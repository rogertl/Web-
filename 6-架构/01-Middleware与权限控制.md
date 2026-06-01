# 6.1 Middleware 与权限控制

## 一句话

Middleware 是 Next.js 的中间件，在请求到达页面之前拦截，用于权限验证、身份识别、重定向等，文件名必须是 `middleware.ts`。

## 为什么需要它

没有 Middleware，权限检查需要写在每个页面组件里，代码重复且容易遗漏。Middleware 统一在入口处拦截，未授权请求直接拒绝，不进入后续流程。

## 类比

把 Middleware 想象成"小区门卫"：

| 概念 | 类比 |
|------|------|
| Middleware | 门卫（检查通行证） |
| 权限验证 | 检查业主卡（只有业主能进） |
| 重定向 | 拦截外来人员（请到登记处） |
| request 对象 | 来访者信息（身份证、访客记录） |
| NextResponse | 放行指令（通过或拒绝） |
| `middleware.ts` | 门卫室（固定位置） |

门卫在小区入口统一检查，不用每栋楼都安排保安。

## 核心内容

### Middleware 的核心特征

| 特征 | 说明 |
|------|------|
| 文件名必须是 `middleware.ts` | 放在项目根目录或 `app/` 文件夹 |
| 拦截所有请求 | 页面、API、静态资源都会经过 Middleware |
| 运行在 Edge | 不支持 Node.js API（如 `fs`、`process.env` 直接访问） |
| 必须返回 NextResponse | 用 `NextResponse.next()` 继续或 `redirect()` 重定向 |
| 支持匹配路径 | 用 `matcher` 配置哪些路径需要拦截 |

### 基本示例：权限验证

```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // 获取 Session（阶段 6.2 会详细讲认证系统）
  const session = request.cookies.get('session');

  // 如果没有 Session，重定向到登录页
  if (!session) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  // 有 Session，继续请求
  return NextResponse.next();
}

// 配置拦截路径
export const config = {
  matcher: '/dashboard/:path*', // 只拦截 /dashboard 及其子路径
};
```

**执行流程**：
1. 用户访问 `/dashboard/settings`
2. Next.js 先执行 `middleware(request)`
3. Middleware 检查 Cookie 中的 `session`
4. 如果没有 `session`，重定向到 `/login`
5. 如果有 `session`，继续请求，进入 `/dashboard/settings` 页面

### matcher 配置：匹配路径

| 匹配模式 | 说明 | 示例 |
|---------|------|------|
| `/dashboard/:path*` | 匹配 `/dashboard` 及所有子路径 | ✅ `/dashboard`、`/dashboard/settings` |
| `/api/:path*` | 匹配所有 API 路由 | ✅ `/api/users`、`/api/posts/123` |
| `/((?!api).+` | 排除 `/api`，匹配其他所有路径 | ✅ `/dashboard`、`/users`；❌ `/api/xxx` |

**示例**：

```typescript
export const config = {
  // 只拦截 /dashboard 及其子路径
  matcher: '/dashboard/:path*',

  // 拦截多个路径
  matcher: ['/dashboard/:path*', '/admin/:path*'],

  // 排除 API 路由和静态资源
  matcher: '/((?!api|_next/static|_next/image|favicon.ico).*)',
};
```

### 路径信息和重定向

```typescript
export function middleware(request: NextRequest) {
  // 获取当前路径
  const path = request.nextUrl.pathname;

  // 根据路径处理
  if (path.startsWith('/admin')) {
    // 检查管理员权限
    const role = request.cookies.get('role');

    if (role !== 'admin') {
      return NextResponse.redirect(new URL('/unauthorized', request.url));
    }
  }

  if (path.startsWith('/api')) {
    // API 路由的额外检查
    const apiKey = request.headers.get('x-api-key');

    if (!apiKey) {
      return NextResponse.json({ error: 'API Key 缺失' }, { status: 401 });
    }
  }

  return NextResponse.next();
}
```

### 响应操作

| 操作 | 说明 | 示例 |
|------|------|------|
| `NextResponse.next()` | 继续请求 | `return NextResponse.next()` |
| `NextResponse.redirect(url)` | 重定向到其他页面 | `return NextResponse.redirect(new URL('/login', request.url))` |
| `NextResponse.rewrite(url)` | 内部重写（不改变 URL） | `return NextResponse.rewrite(new URL('/dashboard', request.url))` |
| `NextResponse.json(data)` | 返回 JSON（常用于 API） | `return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })` |

### 设置和修改响应

```typescript
export function middleware(request: NextRequest) {
  const response = NextResponse.next();

  // 设置自定义响应头
  response.headers.set('X-Frame-Options', 'DENY');
  response.headers.set('X-Content-Type-Options', 'nosniff');

  // 设置 Cookie
  response.cookies.set('theme', 'dark', {
    maxAge: 60 * 60 * 24 * 30, // 30 天
    httpOnly: true,
  });

  return response;
}
```

### 常见模式：登录保护

```typescript
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

// 不需要登录的路径
const publicPaths = ['/login', '/register', '/forgot-password'];

export function middleware(request: NextRequest) {
  const path = request.nextUrl.pathname;
  const session = request.cookies.get('session');

  // 如果是公开路径，直接放行
  if (publicPaths.includes(path)) {
    return NextResponse.next();
  }

  // 如果没有 Session，重定向到登录页
  if (!session) {
    const loginUrl = new URL('/login', request.url);
    loginUrl.searchParams.set('redirect', path); // 记住原始路径
    return NextResponse.redirect(loginUrl);
  }

  // 有 Session，继续请求
  return NextResponse.next();
}

export const config = {
  matcher: '/((?!api|_next/static|_next/image|favicon.ico).*)', // 排除 API 和静态资源
};
```

### Middleware 的限制

| 限制 | 说明 | 解决方案 |
|------|------|---------|
| 不支持 Node.js API | 不能用 `fs`、`process.env`、数据库 | 用环境变量、Headers、Cookies |
| 运行在 Edge | 执行环境受限 | 避免复杂计算 |
| 不能访问 Route Params | 无法获取 `[userId]` 等动态参数 | 从 URL 手动解析 |

## 你需要记住的

1. Middleware 用 `middleware.ts` 文件名，必须放在项目根目录或 `app/` 文件夹。
2. Middleware 拦截所有请求，必须在 `matcher` 中配置路径。
3. 用 `NextResponse.next()` 继续，用 `NextResponse.redirect()` 重定向。
4. Middleware 运行在 Edge，不支持 Node.js API。
5. Middleware 适合权限验证、身份识别、重定向，不适合复杂逻辑。

## AI 代码中的线索

当你在 AI 生成的代码中看到这些模式，说明这是 Middleware：

```typescript
// 线索 1：文件名是 middleware.ts
// middleware.ts

// 线索 2：export middleware 函数
export function middleware(request: NextRequest) { ... }

// 线索 3：返回 NextResponse
return NextResponse.next();
return NextResponse.redirect(...);

// 线索 4：config.matcher 配置
export const config = { matcher: '/dashboard/:path*' };

// 线索 5：操作 Cookie
const session = request.cookies.get('session');
response.cookies.set('theme', 'dark');
```

## 验证问题

- [ ] 如果你想保护 `/admin` 路径，只有 `admin` 角色能访问，Middleware 应该怎么写？
- [ ] `NextResponse.redirect()` 和 `NextResponse.rewrite()` 有什么区别？
- [ ] 为什么 Middleware 不能用 Node.js API？
