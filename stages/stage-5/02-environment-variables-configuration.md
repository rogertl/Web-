# 5.2 环境变量与配置

## 一句话

环境变量是存储敏感配置和运行时参数的安全方式，它们在服务器端可用，客户端无法访问。

## 为什么需要它

想象你正在开发一个应用，它需要连接数据库、调用第三方 API、使用密钥进行身份验证。如果你把这些敏感信息直接写在代码里，每次部署到不同环境（开发、测试、生产）都要修改代码，更危险的是——这些密钥会被提交到 Git 仓库，任何能访问代码的人都能看到。环境变量解决了这个问题：它让配置和代码分离，不同的环境使用不同的配置值。

## 类比

| 概念 | 类比 |
|------|------|
| 环境变量 | 更衣室的储物柜钥匙（不同的人用不同的柜子） |
| 代码 | 更衣室的布局（固定不变） |
| 配置 | 柜子里的物品（根据需求变化） |
| .env 文件 | 储物柜钥匙清单（不应该公开） |
| process.env | 柜台服务员的钥匙册（只有工作人员能看） |

## 核心内容

### 什么是环境变量

环境变量是操作系统级别键值对存储，在 Node.js 运行时可以通过 `process.env` 访问。它们的主要用途：

1. **存储敏感信息**：数据库密码、API 密钥、JWT 密钥
2. **区分运行环境**：开发环境用测试数据库，生产环境用真实数据库
3. **配置可变参数**：端口、域名、功能开关

### Next.js 中的环境变量系统

Next.js 提供了一套完整的环境变量管理机制：

```typescript
// 读取环境变量
const dbUrl = process.env.DATABASE_URL
const apiKey = process.env.API_KEY

// 类型安全的方式（Next.js 9.4+）
import { env } from '@/env.js' // 后续会讲到类型定义
```

**重要特性**：
- 服务器端：可以访问所有环境变量
- 客户端：只能访问以 `NEXT_PUBLIC_` 开头的变量
- 构建时：`NEXT_PUBLIC_*` 变量会被打包进客户端代码

### .env 文件的优先级

Next.js 按以下顺序加载环境变量（后加载的会覆盖前面的）：

```
1. .env.local          # 本地开发专用（不提交到 Git）
2. .env.development     # 开发环境
3. .env.test           # 测试环境
4. .env.production     # 生产环境
5. .env                # 所有环境的默认值
```

**示例**：

```bash
# .env.local（包含本地敏感信息，不提交）
DATABASE_URL="postgresql://localhost:5432/myapp_dev"
API_KEY="sk_test_123456789"

# .env.development（开发环境配置，可以提交）
NODE_ENV="development"
NEXT_PUBLIC_APP_URL="http://localhost:3000"

# .env.production（生产环境配置，可以提交）
NODE_ENV="production"
NEXT_PUBLIC_APP_URL="https://myapp.com"
```

**关键点**：
- `.env.local` 用于本地敏感信息，应该加入 `.gitignore`
- `.env.*.local` 文件永远不会被提交到 Git
- 其他 `.env` 文件可以提交，但不应该包含真实密钥

### 客户端可访问的变量

以 `NEXT_PUBLIC_` 开头的变量会暴露到浏览器：

```bash
# .env.local
NEXT_PUBLIC_ANALYTICS_ID="G-XXXXXXXXXX"
DATABASE_URL="postgresql://..." # 这个变量不会发送到浏览器
```

```typescript
// 客户端组件中
'use client'

export function Analytics() {
  // ✅ 可以访问 NEXT_PUBLIC_ 变量
  const analyticsId = process.env.NEXT_PUBLIC_ANALYTICS_ID

  // ❌ undefined（这个变量不会发送到客户端）
  const dbUrl = process.env.DATABASE_URL

  return <div>{analyticsId}</div>
}
```

**重要安全原则**：
- 永远不要把密钥、密码、API Token 放在 `NEXT_PUBLIC_*` 变量中
- 这些变量会被硬编码到客户端 JavaScript 中，任何人都能看到

### 最佳实践：类型安全的环境变量

为了提供类型检查和自动补全，推荐创建一个专门的配置文件。这里我们使用 zod 来定义和验证环境变量的 Schema。zod 是一个优先 TypeScript 的数据验证库，让你定义数据 Schema 并自动进行类型检查（这在 5.5 会详细讲解）。

```typescript
// env.ts
import { z } from 'zod'

const envSchema = z.object({
  // 服务器端变量
  DATABASE_URL: z.string().url(),
  API_SECRET: z.string().min(32),

  // 客户端变量
  NEXT_PUBLIC_APP_URL: z.string().url(),
  NEXT_PUBLIC_ANALYTICS_ID: z.string(),
})

// 验证环境变量（应用启动时执行）
const parsed = envSchema.safeParse(process.env)

if (!parsed.success) {
  console.error('❌ Invalid environment variables:', parsed.error.format())
  throw new Error('Invalid environment variables')
}

export const env = parsed.data

// 类型导出
export type Env = z.infer<typeof envSchema>
```

**使用方式**：

```typescript
// 任何文件中
import { env } from '@/env'

// ✅ 类型安全，IDE 自动补全
const dbUrl = env.DATABASE_URL
const appId = env.NEXT_PUBLIC_ANALYTICS_ID

// ❌ 编译错误：TypeScript 会检查拼写错误
const apiKe = env.API_KEY // Property 'API_KEY' does not exist
```

**优势**：
- 启动时就验证配置，而不是运行时才发现
- TypeScript 提供类型检查和自动补全
- 明确哪些变量可用，减少拼写错误

### 在不同环境中的使用

**开发环境**：

```bash
# .env.local
DATABASE_URL="postgresql://localhost:5432/myapp_dev"
NEXT_PUBLIC_APP_URL="http://localhost:3000"
```

**生产环境（Vercel 部署）**：

```bash
# 在 Vercel Dashboard 中配置
DATABASE_URL="postgresql://prod-server:5432/myapp_prod"
NEXT_PUBLIC_APP_URL="https://myapp.com"
```

**Vercel 环境变量设置**：
1. 打开项目设置 → Environment Variables
2. 添加变量名和值
3. 选择环境（Production、Preview、Development）
4. 重新部署后生效

### Server Actions 中的使用

```typescript
'use server'

import { env } from '@/env'

export async function createPayment(data: PaymentData) {
  // ✅ 可以安全地使用服务器端环境变量
  const stripe = new Stripe(env.STRIPE_SECRET_KEY)

  // ❌ 错误：不要在客户端使用的 Action 中暴露密钥
  return { stripeKey: env.STRIPE_SECRET_KEY }
}
```

### 常见陷阱

1. **在客户端访问服务器变量**：

```typescript
'use client'

// ❌ undefined
const dbUrl = process.env.DATABASE_URL
```

2. **把密钥放在 NEXT_PUBLIC_ 变量中**：

```bash
# ❌ 危险！任何用户都能看到这个密钥
NEXT_PUBLIC_STRIPE_SECRET_KEY="sk_live_123456"
```

3. **忘记提交 .env.example**：

```bash
# .env.example（这个文件应该提交到 Git）
DATABASE_URL="postgresql://user:password@localhost:5432/myapp"
API_KEY="your_api_key_here"

# .env.local（真实值，不提交）
DATABASE_URL="postgresql://real_user:real_pass@localhost:5432/real_app"
API_KEY="real_api_key_123456"
```

## 你需要记住的

1. 环境变量用于存储敏感配置，不应该提交到 Git 仓库
2. `NEXT_PUBLIC_*` 变量会暴露到浏览器，不要放密钥
3. `.env.local` 包含本地敏感信息，必须加入 `.gitignore`
4. 使用 zod 创建类型安全的配置文件，提前验证环境变量
5. 服务器端可以访问所有变量，客户端只能访问 `NEXT_PUBLIC_*` 变量

## AI 代码中的线索

当 AI 生成使用环境变量的代码时，你会看到：

1. **读取方式**：`process.env.VARIABLE_NAME` 或 `import { env } from '@/env'`
2. **公开变量标识**：`NEXT_PUBLIC_` 前缀表示这个变量会发送到客户端
3. **配置文件**：`env.ts` 或 `config/` 目录下的类型定义文件
4. **启动验证**：应用启动时检查环境变量的存在性和有效性

**危险信号**：
- 在客户端组件中访问 `process.env.DATABASE_URL` 等敏感变量
- 把 API 密钥硬编码在代码中而不是使用环境变量
- 没有 `.env.example` 文件说明需要哪些环境变量

## 验证问题

- [ ] 为什么需要把数据库连接字符串放在环境变量中，而不是直接写在代码里？
- [ ] `NEXT_PUBLIC_` 前缀的作用是什么？为什么不应该把敏感信息放在这些变量中？
- [ ] 在 Next.js 中，如何实现类型安全的环境变量访问？
