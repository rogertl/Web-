# 5.5 zod 数据验证

## 一句话

zod 是一个优先 TypeScript 的数据验证库，让你定义数据 Schema 并自动进行类型检查和验证。

## 为什么需要它

想象你是一家银行的柜员，客户填写存款单。如果客户写错金额、忘记签名、或者填了无效的身份证号，你需要及时发现并拒绝，而不是等到钱已经存入才发现问题。Web 应用也是一样——用户提交的数据可能有各种错误：邮箱格式不对、必填字段为空、数字超出范围。zod 让你在数据进入数据库之前，先通过严格的"安检"，确保数据的正确性和安全性。

## 类比

| 概念 | 类比 |
|------|------|
| 用户数据 | 银行存款单（用户填写的表格） |
| zod Schema | 存单规则（金额、签名、身份证的要求） |
| 验证过程 | 柜员审核（逐项检查） |
| 验证失败 | 退回单据（填写错误，重新填写） |
| 类型推断 | 自动打印的存根（审核通过后生成） |

## 核心

### 什么是 zod

zod 是一个 TypeScript 优先的数据验证库，它的核心特性：

1. **类型安全**：从 Schema 自动推断 TypeScript 类型
2. **链式调用**：流畅的 API 设计
3. **错误处理**：详细的验证错误信息
4. **无依赖**：轻量级，零运行时依赖
5. **支持复杂结构**：嵌套对象、数组、联合类型等

### 基础类型验证

```typescript
import { z } from 'zod'

// 基础类型
const stringSchema = z.string()
const numberSchema = z.number()
const booleanSchema = z.boolean()
const nullSchema = z.null()

// 使用
stringSchema.parse('hello')        // ✅ 'hello'
stringSchema.parse(123)            // ❌ ZodError

// 可选类型
const optionalString = z.string().optional()
optionalString.parse('hello')      // ✅ 'hello'
optionalString.parse(undefined)   // ✅ undefined

// 可空类型
const nullableString = z.string().nullable()
nullableString.parse(null)         // ✅ null
nullableString.parse('hello')      // ✅ 'hello'

// 默认值
const stringWithDefault = z.string().default('hello')
stringWithDefault.parse(undefined) // ✅ 'hello'
```

### 常用验证方法

```typescript
// 字符串验证
const emailSchema = z.string()
  .email('邮箱格式不正确')
  .min(5, '邮箱至少 5 个字符')
  .max(100, '邮箱最多 100 个字符')

// 数字验证
const ageSchema = z.number()
  .min(0, '年龄不能为负')
  .max(120, '年龄不能超过 120')
  .int('年龄必须是整数')

// 枚举验证
const roleSchema = z.enum(['admin', 'user', 'guest'])
roleSchema.parse('admin')   // ✅
roleSchema.parse('superadmin') // ❌ ZodError

// 字符串转数字（处理表单输入）
const ageFromString = z.string()
  .transform(val => Number(val))
  .pipe(z.number().min(0).max(120))
```

### 对象验证

```typescript
// 定义对象 Schema
const userSchema = z.object({
  name: z.string().min(2, '姓名至少 2 个字符'),
  email: z.string().email('邮箱格式不正确'),
  age: z.number().min(0).max(120).optional(),
  role: z.enum(['admin', 'user', 'guest']).default('user'),
})

// 验证数据
const userData = {
  name: 'John Doe',
  email: 'john@example.com',
  age: 25,
}

const validatedUser = userSchema.parse(userData)
// 返回：{ name: 'John Doe', email: 'john@example.com', age: 25, role: 'user' }

// 类型推断
type User = z.infer<typeof userSchema>
// 等价于：
// type User = {
//   name: string
//   email: string
//   age?: number | undefined
//   role: 'admin' | 'user' | 'guest'
// }
```

### 错误处理

```typescript
try {
  userSchema.parse({
    name: 'J', // 姓名太短
    email: 'invalid-email', // 邮箱格式不对
    age: 150, // 年龄超出范围
  })
} catch (error) {
  if (error instanceof z.ZodError) {
    console.log(error.errors)
    // [
    //   {
    //     code: 'too_small',
    //     path: ['name'],
    //     message: '姓名至少 2 个字符',
    //     ...
    //   },
    //   {
    //     code: 'invalid_string',
    //     path: ['email'],
    //     message: '邮箱格式不正确',
    //     ...
    //   },
    //   {
    //     code: 'too_big',
    //     path: ['age'],
    //     message: '数字不能大于 120',
    //     ...
    //   },
    // ]
  }
}
```

### 安全解析（不抛出错误）

```typescript
// safeParse - 返回 { success: true, data } 或 { success: false, error }
const result = userSchema.safeParse(userData)

if (result.success) {
  console.log('验证成功', result.data)
} else {
  console.log('验证失败', result.error.errors)
}

// 实际应用
function validateUser(data: unknown) {
  const result = userSchema.safeParse(data)

  if (!result.success) {
    return {
      success: false,
      errors: result.error.errors.map(e => e.message),
    }
  }

  return { success: true, data: result.data }
}
```

### 在 Server Actions 中使用

```typescript
// app/actions/user.ts
'use server'

import { z } from 'zod'
import { prisma } from '@/lib/db'
import { revalidatePath } from 'next/cache'

// 定义 Schema
const createUserSchema = z.object({
  name: z.string().min(2, '姓名至少 2 个字符'),
  email: z.string().email('邮箱格式不正确'),
  age: z.number().min(0).max(120).optional(),
})

export async function createUser(formData: FormData) {
  // 从 FormData 提取数据
  const rawData = {
    name: formData.get('name'),
    email: formData.get('email'),
    age: formData.get('age') ? Number(formData.get('age')) : undefined,
  }

  // 验证数据
  const result = createUserSchema.safeParse(rawData)

  if (!result.success) {
    // 返回错误信息
    return { errors: result.error.flatten().fieldErrors }
  }

  // 创建用户
  const user = await prisma.user.create({
    data: result.data,
  })

  revalidatePath('/users')
  return { success: true, user }
}
```

**在表单中显示错误**：

```typescript
// app/users/page.tsx
'use client'

import { createUser } from '@/app/actions/user'
import { useFormState } from 'react-dom'

export default function CreateUserForm() {
  const [state, formAction] = useFormState(createUser, null)

  return (
    <form action={formAction}>
      <input name="name" placeholder="姓名" required />
      {state?.errors?.name && (
        <span className="error">{state.errors.name[0]}</span>
      )}

      <input name="email" placeholder="邮箱" required />
      {state?.errors?.email && (
        <span className="error">{state.errors.email[0]}</span>
      )}

      <input name="age" type="number" placeholder="年龄" />
      {state?.errors?.age && (
        <span className="error">{state.errors.age[0]}</span>
      )}

      <button type="submit">创建</button>
    </form>
  )
}
```

### 复杂验证场景

**条件验证**：

```typescript
const passwordSchema = z.string().min(8, '密码至少 8 个字符')
const confirmPasswordSchema = z.string()

const formSchema = z.object({
  password: passwordSchema,
  confirmPassword: confirmPasswordSchema,
}).refine(
  (data) => data.password === data.confirmPassword,
  {
    message: '两次输入的密码不一致',
    path: ['confirmPassword'], // 错误关联到 confirmPassword 字段
  }
)

formSchema.parse({
  password: 'password123',
  confirmPassword: 'password456',
})
// ❌ ZodError: 两次输入的密码不一致
```

**嵌套对象验证**：

```typescript
const addressSchema = z.object({
  street: z.string().min(5, '街道地址至少 5 个字符'),
  city: z.string().min(2, '城市名至少 2 个字符'),
  zipCode: z.string().regex(/^\d{5}$/, '邮编必须是 5 位数字'),
})

const userWithAddressSchema = z.object({
  name: z.string(),
  address: addressSchema,
})

userWithAddressSchema.parse({
  name: 'John Doe',
  address: {
    street: '123 Main St',
    city: 'New York',
    zipCode: '10001',
  },
})
```

**数组验证**：

```typescript
const tagsSchema = z.array(
  z.string().min(1).max(20)
).min(1).max(5)

tagsSchema.parse(['tech', 'coding']) // ✅
tagsSchema.parse([]) // ❌ 至少 1 个标签
tagsSchema.parse(['a', 'b', 'c', 'd', 'e', 'f']) // ❌ 最多 5 个标签
```

### 与 Prisma 结合使用

```typescript
import { z } from 'zod'

// 从 Prisma 类型生成 zod Schema
import { Prisma } from '@prisma/client'

const userCreateSchema: z.ZodType<Prisma.UserCreateInput> = z.object({
  email: z.string().email(),
  name: z.string().min(2),
  posts: z.object({
    create: z.array(
      z.object({
        title: z.string().min(5),
        content: z.string().optional(),
      })
    ).optional(),
  }),
})

// 类型完全匹配 Prisma
const userData: Prisma.UserCreateInput = userCreateSchema.parse({
  email: 'user@example.com',
  name: 'John Doe',
  posts: {
    create: [
      { title: 'My First Post' },
    ],
  },
})
```

## 你需要记住的

1. zod 提供类型安全的数据验证，从 Schema 自动推断 TypeScript 类型
2. 使用 `parse()` 会抛出错误，使用 `safeParse()` 返回结果对象
3. 在 Server Actions 中使用 zod 验证表单数据，防止无效数据进入数据库
4. zod 的错误信息详细，可以直接显示给用户
5. 可以使用 `refine()` 实现自定义验证逻辑（如密码确认）

## AI 代码中的线索

当 AI 生成的代码使用 zod 进行数据验证时，你会看到：

1. **导入语句**：`import { z } from 'zod'`
2. **Schema 定义**：`const schema = z.object({ /* ... */ })`
3. **验证调用**：`schema.parse(data)` 或 `schema.safeParse(data)`
4. **类型推断**：`type Data = z.infer<typeof schema>`
5. **错误处理**：`if (error instanceof z.ZodError) { /* ... */ }`

**常见模式**：
```typescript
// Server Actions 中的验证
const schema = z.object({ /* ... */ })
const result = schema.safeParse(formData)
if (!result.success) return { errors: result.error.flatten().fieldErrors }

// 类型推断
type User = z.infer<typeof userSchema>

// 自定义验证
.refine(data => data.password === data.confirmPassword, {
  message: '密码不一致',
  path: ['confirmPassword'],
})
```

## 验证问题

- [ ] 为什么需要在数据进入数据库之前进行验证？不验证会有什么后果？
- [ ] `parse()` 和 `safeParse()` 有什么区别？分别在什么场景使用？
- [ ] 如何在 Server Actions 中使用 zod 验证表单数据并返回错误信息？
