# 5.6 错误处理与安全性

## 一句话

正确处理错误并防止常见安全漏洞是生产级应用的基础，保护用户数据和系统安全。

## 为什么需要它

想象你是一家银行的安保负责人。如果客户在 ATM 机上操作时系统出错，屏幕只显示"Error 500"，客户不知道发生了什么，可能会反复操作导致资金异常。更危险的是，如果有人故意输入特殊字符来欺骗系统，或者通过错误信息推测出系统的内部结构，银行的安全就会受到威胁。Web 应用也是如此——良好的错误处理既提供好的用户体验，又防止信息泄露；安全防护则阻挡恶意攻击，保护系统和用户。

## 类比

| 概念 | 类比 |
|------|------|
| 错误处理 | 银行异常处理（操作失败时显示友好提示） |
| 安全漏洞 | ATM 安全漏洞（有人试图破解） |
| SQL 注入 | 填写虚假存款单（试图欺骗银行） |
| XSS 攻击 | 伪造银行通知（用户点击恶意链接） |
| 错误信息泄露 | 在大屏幕上显示内部账目（信息泄露） |
| 参数验证 | 检查存款单的完整性 |

## 核心内容

### 错误处理最佳实践

**服务器端错误处理**：

```typescript
// lib/errors.ts
/**
 * 自定义错误类
 */
export class AppError extends Error {
  constructor(
    message: string,
    public statusCode: number = 500,
    public code?: string
  ) {
    super(message)
    this.name = 'AppError'
  }
}

export class NotFoundError extends AppError {
  constructor(resource: string) {
    super(`${resource} not found`, 404, 'NOT_FOUND')
  }
}

export class ValidationError extends AppError {
  constructor(message: string) {
    super(message, 400, 'VALIDATION_ERROR')
  }
}

export class AuthenticationError extends AppError {
  constructor(message: string = 'Unauthorized') {
    super(message, 401, 'AUTHENTICATION_ERROR')
  }
}
```

**在 Server Actions 中使用**：

```typescript
// app/actions/user.ts
'use server'

import { prisma } from '@/lib/db'
import { revalidatePath } from 'next/cache'
import { NotFoundError, ValidationError } from '@/lib/errors'

export async function updateUser(id: string, formData: FormData) {
  try {
    // 检查用户是否存在
    const user = await prisma.user.findUnique({
      where: { id },
    })

    if (!user) {
      throw new NotFoundError('User')
    }

    // 验证数据
    const name = formData.get('name') as string
    if (!name || name.length < 2) {
      throw new ValidationError('Name must be at least 2 characters')
    }

    // 更新用户
    const updatedUser = await prisma.user.update({
      where: { id },
      data: { name },
    })

    revalidatePath('/users')
    return { success: true, user: updatedUser }

  } catch (error) {
    // 处理不同类型的错误
    if (error instanceof NotFoundError) {
      return {
        success: false,
        error: {
          code: error.code,
          message: error.message,
        },
      }
    }

    if (error instanceof ValidationError) {
      return {
        success: false,
        error: {
          code: error.code,
          message: error.message,
        },
      }
    }

    // 记录未知错误
    console.error('Unexpected error:', error)

    // 返回通用错误信息（不暴露内部细节）
    return {
      success: false,
      error: {
        code: 'INTERNAL_ERROR',
        message: 'An unexpected error occurred',
      },
    }
  }
}
```

**在 Route Handlers 中使用**：

```typescript
// app/api/users/[id]/route.ts
import { prisma } from '@/lib/db'
import { NextResponse } from 'next/server'
import { NotFoundError } from '@/lib/errors'

export async function GET(
  request: Request,
  { params }: { params: { id: string } }
) {
  try {
    const user = await prisma.user.findUnique({
      where: { id: params.id },
    })

    if (!user) {
      throw new NotFoundError('User')
    }

    return NextResponse.json(user)

  } catch (error) {
    if (error instanceof NotFoundError) {
      return NextResponse.json(
        { error: error.message },
        { status: error.statusCode }
      )
    }

    console.error('Unexpected error:', error)

    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    )
  }
}
```

### SQL 注入防护

**什么是 SQL 注入**：

SQL 注入是一种攻击方式，恶意用户通过在输入字段中插入 SQL 代码来操纵数据库查询。

**不安全的示例（伪代码，Prisma 不会发生这种情况）**：

```typescript
// ❌ 危险：直接拼接 SQL（这是伪代码，Prisma 不会这样工作）
const email = userInput
const query = `SELECT * FROM users WHERE email = '${email}'`
// 如果 userInput = "' OR '1'='1"
// 查询变成：SELECT * FROM users WHERE email = '' OR '1'='1'
// 这会返回所有用户！
```

**Prisma 自动防护 SQL 注入**：

```typescript
// ✅ 安全：Prisma 自动处理参数化查询
const user = await prisma.user.findUnique({
  where: { email: userInput },
})

// Prisma 在底层使用参数化查询，防止 SQL 注入
// 即使 userInput 包含恶意 SQL 代码，也不会被执行
```

**使用 Prisma 的安全原则**：

1. **始终使用 Prisma 的查询 API**，不要手动拼接 SQL
2. **使用参数化查询**，Prisma 自动处理
3. **验证用户输入**，在使用前用 zod 验证

```typescript
import { z } from 'zod'

// 先验证输入
const emailSchema = z.string().email()
const validatedEmail = emailSchema.parse(userInput)

// 再查询
const user = await prisma.user.findUnique({
  where: { email: validatedEmail },
})
```

### XSS（跨站脚本攻击）防护

**什么是 XSS**：

XSS 攻击者通过在网页中注入恶意脚本代码，窃取用户数据或进行其他恶意操作。

**XSS 攻击示例**：

```typescript
// ❌ 危险：直接渲染用户输入的内容
function UserBio({ bio }: { bio: string }) {
  return <div>{bio}</div>
}

// 如果 bio = "<script>alert('XSS')</script>"
// 页面会执行这个脚本
```

**React/Next.js 自动防护**：

```typescript
// ✅ 安全：React 自动转义内容
function UserBio({ bio: string }) {
  return <div>{bio}</div>
  // React 会将 < 转义为 &lt;，> 转义为 &gt;
  // 所以 <script> 不会被执行
}
```

**需要注意的场景**：

```typescript
// ❌ 危险：使用 dangerouslySetInnerHTML
function PostContent({ content }: { content: string }) {
  return <div dangerouslySetInnerHTML={{ __html: content }} />
  // 如果 content 包含恶意脚本，它会被执行！
}

// ✅ 安全：使用 DOMPurify 清理 HTML
import DOMPurify from 'isomorphic-dompurify'

function PostContent({ content }: { content: string }) {
  const cleanContent = DOMPurify.sanitize(content)
  return <div dangerouslySetInnerHTML={{ __html: cleanContent }} />
}
```

**URL 注入防护**：

```typescript
// ❌ 危险：直接使用用户输入作为 URL
<a href={userInputWebsite}>Visit Website</a>
// 如果 userInputWebsite = "javascript:alert('XSS')"

// ✅ 安全：验证 URL 格式
import { z } from 'zod'

const urlSchema = z.string().url()
const validatedUrl = urlSchema.parse(userInputWebsite)
<a href={validatedUrl}>Visit Website</a>
```

### CSRF（跨站请求伪造）防护

**什么是 CSRF**：

攻击者诱导用户在已认证的状态下执行非预期操作，比如发送转账请求。

**Next.js 的防护**：

```typescript
// Next.js 使用内置的 CSRF 保护
// 在 Server Actions 和 Route Handlers 中自动启用

// ✅ 安全：Next.js 自动验证 CSRF token
'use server'

export async function transferMoney(formData: FormData) {
  const to = formData.get('to') as string
  const amount = Number(formData.get('amount'))

  // Next.js 自动验证请求来源
  // 恶意网站无法伪造有效的请求
}
```

### 输入验证和清理

**全面的输入验证策略**：

```typescript
import { z } from 'zod'

// 定义严格的验证 Schema
const createUserSchema = z.object({
  name: z.string()
    .min(2, '姓名至少 2 个字符')
    .max(50, '姓名最多 50 个字符')
    .transform(val => val.trim()), // 去除前后空格

  email: z.string()
    .email('邮箱格式不正确')
    .toLowerCase(), // 统一转换为小写

  age: z.string()
    .optional()
    .transform(val => val ? Number(val) : undefined)
    .pipe(
      z.number()
        .min(0, '年龄不能为负')
        .max(120, '年龄不能超过 120')
        .int('年龄必须是整数')
    ),

  bio: z.string()
    .max(500, '简介最多 500 个字符')
    .optional(),
})

export async function createUser(formData: FormData) {
  // 验证并清理输入
  const result = createUserSchema.safeParse(Object.fromEntries(formData))

  if (!result.success) {
    return { errors: result.error.flatten().fieldErrors }
  }

  // 使用清理后的数据
  const user = await prisma.user.create({
    data: result.data, // 数据已经被验证和清理
  })

  return { success: true, user }
}
```

### 错误信息的处理

**不要泄露敏感信息**：

```typescript
// ❌ 危险：返回详细的内部错误信息
return NextResponse.json({
  error: 'Database connection failed: Connection refused at localhost:5432',
  status: 500,
})

// ✅ 安全：返回通用错误信息
return NextResponse.json({
  error: 'Internal server error',
  status: 500,
})

// 同时记录详细的错误到服务器日志
console.error('Database connection failed:', {
  message: error.message,
  stack: error.stack,
  timestamp: new Date().toISOString(),
})
```

### 常见安全检查清单

**环境变量安全**：

```typescript
// ✅ 检查必需的环境变量
const requiredEnvVars = [
  'DATABASE_URL',
  'API_SECRET_KEY',
  'JWT_SECRET',
] as const

const missingEnvVars = requiredEnvVars.filter(
  envVar => !process.env[envVar]
)

if (missingEnvVars.length > 0) {
  throw new Error(
    `Missing required environment variables: ${missingEnvVars.join(', ')}`
  )
}
```

**数据库权限最小化**：

```sql
-- 不要使用超级用户权限
-- ✅ 创建应用专用用户
CREATE USER app_user WITH PASSWORD 'secure_password';
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_user;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO app_user;
```

**HTTPS 和安全头**：

```typescript
// next.config.js
const nextConfig = {
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          {
            key: 'X-Frame-Options',
            value: 'DENY',
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff',
          },
          {
            key: 'Referrer-Policy',
            value: 'strict-origin-when-cross-origin',
          },
          {
            key: 'Permissions-Policy',
            value: 'camera=(), microphone=(), geolocation=()',
          },
        ],
      },
    ]
  },
}
```

## 你需要记住的

1. 错误处理要区分用户友好信息和系统日志，不泄露内部实现细节
2. Prisma 自动防护 SQL 注入，但仍需验证用户输入
3. React/Next.js 自动防护 XSS，但使用 dangerouslySetInnerHTML 时要清理 HTML
4. 使用 zod 验证所有用户输入，确保数据格式正确
5. 检查环境变量是否完整，使用最小权限的数据库用户

## AI 代码中的线索

当 AI 生成的代码包含错误处理和安全措施时，你会看到：

**错误处理模式**：
```typescript
// 自定义错误类
class AppError extends Error { /* ... */ }

// 在 Action/Handler 中捕获错误
try {
  // 操作
} catch (error) {
  if (error instanceof AppError) {
    return { error: error.message }
  }
  console.error('Unexpected error:', error)
  return { error: 'Internal error' }
}
```

**安全验证模式**：
```typescript
// 输入验证
const schema = z.object({ /* ... */ })
const result = schema.safeParse(data)

// Prisma 查询（自动防注入）
const user = await prisma.user.findUnique({ where: { email } })

// HTML 清理
const clean = DOMPurify.sanitize(userContent)
```

**危险信号**：
```typescript
// ❌ 直接拼接 SQL（伪代码）
const query = `SELECT * FROM users WHERE email = '${email}'`

// ❌ 返回详细的内部错误
return { error: `Database error: ${error.message}` }

// ❌ 不清理用户输入就渲染
<div dangerouslySetInnerHTML={{ __html: userInput }} />
```

## 验证问题

- [ ] 为什么不能把详细的内部错误信息直接返回给用户？这会有什么安全风险？
- [ ] Prisma 如何防护 SQL 注入攻击？还需要手动做哪些防护？
- [ ] 在哪些情况下需要使用 DOMPurify 清理 HTML 内容？
