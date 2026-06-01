# 4.3 Route Handlers（API 路由）

## 一句话

Route Handlers 是 Next.js 的 API 路由，在 `app/api/` 文件夹创建 `route.ts` 文件，导出 `GET`、`POST` 等函数，自动映射到 HTTP 端点。

## 为什么需要它

Server Actions 适合表单提交，但有时需要标准 REST API（如第三方 Webhook、移动端后端）。Route Handlers 提供传统 API 路由方式，支持自定义 HTTP 方法、响应头和错误处理。

## 类比

把 Route Handlers 想象成"餐厅外卖窗口"：

| 概念 | 类比 |
|------|------|
| Route Handler | 外卖窗口（接受订单） |
| HTTP 方法 | 订单类型（GET = 查菜单，POST = 下订单） |
| 请求参数 | 订单详情（要什么菜、几点送） |
| 响应数据 | 外卖内容（菜品、小票） |
| `app/api/` 文件夹 | 外卖区（专门处理外卖的地方） |
| `route.ts` 文件 | 窗口编号（每个窗口一个编号） |

Server Actions 是堂食（直接在餐厅吃），Route Handlers 是外卖（打包带走）。

## 核心内容

### Route Handlers 的核心特征

| 特征 | 说明 | 示例 |
|------|------|------|
| 文件位置 | 必须在 `app/api/` 文件夹 | `app/api/users/route.ts` |
| 文件名 | 必须是 `route.ts`（或 `route.js`） | `app/api/users/route.ts` |
| 导出函数 | 导出 HTTP 方法名（大写） | `export async function GET()` |
| URL 映射 | 文件路径自动映射到 URL | `app/api/users/route.ts` → `/api/users` |
| 请求对象 | 自动注入 `Request` 对象 | `GET(request: Request)` |
| 响应对象 | 返回 `Response` 对象 | `return Response.json(data)` |
| 支持所有方法 | GET、POST、PUT、PATCH、DELETE、HEAD、OPTIONS | `export async function DELETE()` |

### 基本示例：用户列表 API

```typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { db } from '@/lib/db';

// GET /api/users → 获取所有用户
export async function GET(request: NextRequest) {
  try {
    // 查询数据库
    const users = await db.user.findMany();

    // 返回 JSON 响应
    return NextResponse.json({ users });
  } catch (error) {
    // 错误处理
    return NextResponse.json(
      { error: '获取用户列表失败' },
      { status: 500 }
    );
  }
}

// POST /api/users → 创建用户
export async function POST(request: NextRequest) {
  try {
    // 解析请求体
    const body = await request.json();
    const { name, email } = body;

    // 验证数据
    if (!name || !email) {
      return NextResponse.json(
        { error: '姓名和邮箱不能为空' },
        { status: 400 }
      );
    }

    // 创建用户
    const user = await db.user.create({
      data: { name, email },
    });

    return NextResponse.json({ user }, { status: 201 });
  } catch (error) {
    return NextResponse.json(
      { error: '创建用户失败' },
      { status: 500 }
    );
  }
}
```

**执行流程**：
1. 客户端请求 `GET /api/users`
2. Next.js 找到 `app/api/users/route.ts`
3. Next.js 调用 `GET(request)` 函数
4. 函数查询数据库并返回 `NextResponse.json({ users })`
5. Next.js 自动设置 `Content-Type: application/json` 并返回响应

### HTTP 方法与语义

| 方法 | 语义 | 示例 URL | 典型用途 |
|------|------|----------|----------|
| GET | 获取资源 | `GET /api/users` | 查询数据 |
| POST | 创建资源 | `POST /api/users` | 创建新记录 |
| PUT | 完整更新 | `PUT /api/users/123` | 替换整个资源 |
| PATCH | 部分更新 | `PATCH /api/users/123` | 更新资源的一部分 |
| DELETE | 删除资源 | `DELETE /api/users/123` | 删除记录 |
| HEAD | 获取头信息 | `HEAD /api/users` | 检查资源是否存在 |
| OPTIONS | 获取支持的方法 | `OPTIONS /api/users` | CORS 预检请求 |

### 动态路由：带参数的 API

Route Handlers 支持动态路由（和页面路由一样）：

```typescript
// app/api/users/[userId]/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { db } from '@/lib/db';

// GET /api/users/123 → 获取单个用户
export async function GET(
  request: NextRequest,
  { params }: { params: { userId: string } }
) {
  try {
    const userId = parseInt(params.userId);

    const user = await db.user.findUnique({
      where: { id: userId },
    });

    if (!user) {
      return NextResponse.json(
        { error: '用户不存在' },
        { status: 404 }
      );
    }

    return NextResponse.json({ user });
  } catch (error) {
    return NextResponse.json(
      { error: '获取用户失败' },
      { status: 500 }
    );
  }
}

// PATCH /api/users/123 → 更新用户
export async function PATCH(
  request: NextRequest,
  { params }: { params: { userId: string } }
) {
  try {
    const userId = parseInt(params.userId);
    const body = await request.json();

    const user = await db.user.update({
      where: { id: userId },
      data: body, // 只更新提供的字段
    });

    return NextResponse.json({ user });
  } catch (error) {
    return NextResponse.json(
      { error: '更新用户失败' },
      { status: 500 }
    );
  }
}

// DELETE /api/users/123 → 删除用户
export async function DELETE(
  request: NextRequest,
  { params }: { params: { userId: string } }
) {
  try {
    const userId = parseInt(params.userId);

    await db.user.delete({
      where: { id: userId },
    });

    return NextResponse.json({ success: true });
  } catch (error) {
    return NextResponse.json(
      { error: '删除用户失败' },
      { status: 500 }
    );
  }
}
```

**注意**：
- 动态参数 `[userId]` 从第二个参数 `{ params }` 获取
- `params` 是对象，键名和路由参数名匹配（`params.userId`）

### 请求处理：获取请求信息

`NextRequest` 对象扩展了标准的 Web Request API：

```typescript
export async function GET(request: NextRequest) {
  // 获取 URL 参数
  const url = new URL(request.url);
  const page = parseInt(url.searchParams.get('page') || '1');
  const limit = parseInt(url.searchParams.get('limit') || '10');

  // 获取请求头
  const authHeader = request.headers.get('authorization');

  // 获取 Cookie
  const token = request.cookies.get('token');

  // 获取请求体（POST/PUT/PATCH）
  // const body = await request.json();

  return NextResponse.json({ page, limit });
}
```

### 响应处理：自定义响应

`NextResponse` 提供多种响应方法：

```typescript
// 返回 JSON
export async function GET() {
  return NextResponse.json({ users: [...] });
}

// 返回文本
export async function GET() {
  return new NextResponse('Hello World', { status: 200 });
}

// 返回自定义状态码
export async function POST() {
  return NextResponse.json(
    { error: 'Invalid input' },
    { status: 400 }
  );
}

// 设置响应头
export async function GET() {
  return NextResponse.json(
    { data: '...' },
    {
      headers: {
        'Cache-Control': 'max-age=3600',
        'X-Custom-Header': 'value',
      },
    }
  );
}

// 设置 Cookie
export async function POST() {
  const response = NextResponse.json({ success: true });
  response.cookies.set('token', 'abc123', {
    httpOnly: true,
    secure: true,
    maxAge: 3600,
  });
  return response;
}
```

### CORS：跨域请求处理

如果前端和后端不在同一域名，需要处理 CORS：

```typescript
// app/api/users/route.ts
export async function OPTIONS(request: NextRequest) {
  return new NextResponse(null, {
    status: 200,
    headers: {
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
      'Access-Control-Allow-Headers': 'Content-Type, Authorization',
    },
  });
}

// 所有请求都添加 CORS 头
export async function GET(request: NextRequest) {
  const data = await db.user.findMany();

  return NextResponse.json(
    { users: data },
    {
      headers: {
        'Access-Control-Allow-Origin': '*',
      },
    }
  );
}
```

**注意**：生产环境不要用 `Access-Control-Allow-Origin: *`，应该指定具体域名。

### Route Handlers vs Server Actions

| 特性 | Route Handlers | Server Actions |
|------|----------------|----------------|
| 文件位置 | `app/api/xxx/route.ts` | 任意 `actions.ts` 文件 |
| 调用方式 | `fetch('/api/xxx')` | 直接调用函数 |
| 参数处理 | 手动解析 `request.json()` | 自动序列化参数 |
| 类型安全 | ❌ `fetch` 返回 `any` | ✅ TypeScript 全程类型检查 |
| 适用场景 | REST API、Webhook、第三方调用 | 表单提交、按钮交互、内部调用 |

**何时使用 Route Handlers**：
- ✅ 第三方服务调用（如 Webhook）
- ✅ 移动端后端（需要标准 REST API）
- ✅ 需要完全自定义 HTTP 行为
- ✅ 公开 API（任何人都可以调用）

**何时使用 Server Actions**：
- ✅ 表单提交（创建、更新、删除）
- ✅ 按钮触发的交互
- ✅ 需要类型安全的内部调用

### Webhook 处理示例

Route Handlers 特别适合处理 Webhook：

```typescript
// app/api/webhooks/stripe/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { headers } from 'next/headers';

export async function POST(request: NextRequest) {
  // 获取 Webhook 签名（用于验证）
  const signature = headers().get('stripe-signature');

  // 读取请求体
  const body = await request.text();

  // 验证 Webhook 签名（防止伪造请求）
  // const event = stripe.webhooks.constructEvent(body, signature, webhookSecret);

  // 解析事件
  // const event = JSON.parse(body);

  // 根据事件类型处理
  // switch (event.type) {
  //   case 'payment_intent.succeeded':
  //     await handlePaymentSucceeded(event.data.object);
  //     break;
  // }

  // 返回 200（告诉 Stripe 接收成功）
  return new NextResponse(null, { status: 200 });
}
```

**Webhook 规则**：
- 必须快速返回 200 响应（避免发送方重试）
- 通常需要验证签名（防止恶意请求）
- 异步处理业务逻辑（不阻塞响应）

### 错误处理最佳实践

```typescript
export async function GET(request: NextRequest) {
  try {
    // 获取参数
    const url = new URL(request.url);
    const id = parseInt(url.pathname.split('/').pop() || '0');

    // 查询数据
    const user = await db.user.findUnique({
      where: { id },
    });

    // 404：资源不存在
    if (!user) {
      return NextResponse.json(
        { error: '用户不存在' },
        { status: 404 }
      );
    }

    // 200：成功
    return NextResponse.json({ user });
  } catch (error) {
    // 500：服务器错误
    console.error('获取用户失败：', error);
    return NextResponse.json(
      { error: '服务器内部错误' },
      { status: 500 }
    );
  }
}
```

**HTTP 状态码规范**：
- 200：成功
- 201：创建成功
- 400：客户端错误（如参数缺失）
- 401：未认证
- 403：无权限
- 404：资源不存在
- 500：服务器错误

## 你需要记住的

1. Route Handlers 在 `app/api/` 文件夹创建 `route.ts` 文件。
2. 导出 HTTP 方法（`GET`、`POST` 等）自动映射到对应 HTTP 请求。
3. 动态参数 `[userId]` 从第二个参数 `{ params }` 获取。
4. 用 `NextResponse.json()` 返回 JSON 响应，用 `request.json()` 获取请求体。
5. Route Handlers 适合 REST API、Webhook、第三方调用；Server Actions 适合表单提交和内部交互。

## AI 代码中的线索

当你在 AI 生成的代码中看到这些模式，说明这是 Route Handler：

```typescript
// 线索 1：app/api/ 文件夹
// app/api/users/route.ts

// 线索 2：导出 HTTP 方法（大写）
export async function GET(request: NextRequest) { ... }
export async function POST(request: NextRequest) { ... }

// 线索 3：返回 NextResponse
return NextResponse.json({ data: '...' });

// 线索 4：动态参数
export async function GET(request, { params }) {
  const id = params.userId;
}

// 线索 5：解析请求体
const body = await request.json();

// 线索 6：设置响应头
return NextResponse.json({ data }, { headers: { 'X-Custom': 'value' } });
```

如果看到 `'use server'` 指令或 `action={submitForm}`，这是 Server Action。

## 验证问题

- [ ] 以下文件对应的 HTTP 端点是什么？
  ```typescript
  // app/api/products/[productId]/reviews/route.ts
  export async function GET() { ... }
  ```
- [ ] 如果你想创建一个用户更新 API，应该用哪个 HTTP 方法？为什么？
- [ ] Route Handlers 和 Server Actions 的主要区别是什么？何时用哪个？
