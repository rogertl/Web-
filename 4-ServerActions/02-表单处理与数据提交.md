# 4.2 表单处理与数据提交流程

## 一句话

HTML 表单的 `action` 属性可以直接绑定 Server Action，用户提交后浏览器自动把表单数据 POST 到服务端，无需手动构造 `fetch` 请求。

## 为什么需要它

传统表单处理需要：监听 submit 事件 → 阻止默认行为 → 手动收集数据 → `fetch` 发送到 API → 处理响应。代码冗长且容易出错。Server Actions 让表单处理回归 HTML 原生方式，一行 `action={submitForm}` 搞定。

## 类比

把表单提交想象成"填表投递"：

| 概念 | 类比 |
|------|------|
| HTML 表单 | 申请表格（填写内容） |
| Server Action | 受理窗口（处理表格） |
| action 属性 | 投递口（指定往哪里投） |
| 表单控件（input、select） | 表格字段（姓名、邮箱等） |
| formData | 表格内容（收集的数据） |
| 提交按钮 | 投递按钮（点击后发送） |

你填好表格，点击投递，表格自动送到受理窗口。无需自己跑腿。

## 核心内容

### 表单处理的核心流程

| 步骤 | 说明 | 代码 |
|------|------|------|
| 1. 创建表单 | 用 `<form>` 标签包裹输入控件 | `<form action={submitForm}>` |
| 2. 添加字段 | 用 `<input>`、`<select>` 等控件 | `<input name="username" />` |
| 3. 绑定 Server Action | `action` 属性指向函数 | `action={createUser}` |
| 4. 用户提交 | 点击提交按钮或按回车 | `<button type="submit">` |
| 5. 自动 POST | 浏览器自动收集表单数据 | 无需手动处理 |
| 6. 服务端执行 | Server Action 在服务端运行 | `'use server'` 函数执行 |
| 7. 返回结果 | Server Action 返回响应 | `return { success: true }` |

### 基本示例：创建用户

```typescript
// components/CreateUserForm.tsx（Client Component）
'use client';

import { createUser } from '@/app/actions';

export default function CreateUserForm() {
  return (
    <form action={createUser}>
      <div>
        <label htmlFor="name">姓名：</label>
        <input id="name" name="name" type="text" required />
      </div>

      <div>
        <label htmlFor="email">邮箱：</label>
        <input id="email" name="email" type="email" required />
      </div>

      <button type="submit">创建用户</button>
    </form>
  );
}

// app/actions.ts（Server Action）
'use server';

import { db } from '@/lib/db';
import { revalidatePath } from 'next/cache';

export async function createUser(formData: FormData) {
  // 从 FormData 中提取字段
  const name = formData.get('name') as string;
  const email = formData.get('email') as string;

  // 在服务端创建用户
  const user = await db.user.create({
    data: { name, email },
  });

  // 刷新缓存
  revalidatePath('/users');

  // 返回结果
  return { success: true, userId: user.id };
}
```

**执行流程**：
1. 用户填写表单并点击"创建用户"
2. 浏览器自动把表单数据 POST 到服务端
3. Next.js 调用 `createUser` 函数，传入 `FormData` 对象
4. `formData.get('name')` 获取"姓名"字段的值
5. `formData.get('email')` 获取"邮箱"字段的值
6. 服务端创建用户并返回结果

### FormData 的用法

`FormData` 是浏览器提供的 API，自动收集表单数据：

| 方法 | 说明 | 示例 |
|------|------|------|
| `formData.get(name)` | 获取单个字段的值 | `formData.get('username')` |
| `formData.getAll(name)` | 获取多选框的所有值 | `formData.getAll('hobbies')` |
| `formData.entries()` | 获取所有字段（迭代器） | `Array.from(formData.entries())` |
| `formData.has(name)` | 检查字段是否存在 | `formData.has('email')` |

**示例**：

```typescript
// 处理文本输入
const name = formData.get('name') as string; // "Alice"

// 处理多选框
const hobbies = formData.getAll('hobbies') as string[]; // ["reading", "coding"]

// 遍历所有字段
for (const [key, value] of formData.entries()) {
  console.log(`${key}: ${value}`);
}
```

### 表单控件与字段的映射

| 控件类型 | HTML 标签 | FormData 中的字段名 |
|---------|-----------|-------------------|
| 文本输入 | `<input name="username" />` | `username` |
| 邮箱输入 | `<input name="email" type="email" />` | `email` |
| 密码输入 | `<input name="password" type="password" />` | `password` |
| 数字输入 | `<input name="age" type="number" />` | `age`（字符串，需转换） |
| 下拉选择 | `<select name="country"><option value="CN">中国</option></select>` | `country` |
| 多选框 | `<input name="hobbies" type="checkbox" value="reading" />` | `hobbies`（需 `getAll`） |
| 文件上传 | `<input name="avatar" type="file" />` | `avatar`（File 对象） |
| 隐藏字段 | `<input name="userId" type="hidden" value="123" />` | `userId` |
| 文本区域 | `<textarea name="bio"></textarea>` | `bio` |

**重要规则**：
- `name` 属性决定字段在 `FormData` 中的键名
- 如果多个控件 `name` 相同（如多选框），需用 `getAll` 获取所有值
- 没有 `name` 属性的控件不会包含在 `FormData` 中

### 表单验证

可以用 HTML 原生验证或自定义验证：

```typescript
// HTML 原生验证（浏览器端）
<form action={createUser}>
  <input name="email" type="email" required /> {/* 必填且必须是邮箱 */}
  <input name="age" type="number" min="18" max="100" /> {/* 18-100 之间 */}
  <button type="submit">提交</button>
</form>

// 服务端验证（推荐）
'use server';

import { z } from 'zod'; // 阶段 5 会详细讲 zod

const userSchema = z.object({
  name: z.string().min(2).max(50),
  email: z.string().email(),
  age: z.number().min(18).max(100),
});

export async function createUser(formData: FormData) {
  // 提取数据
  const rawData = {
    name: formData.get('name'),
    email: formData.get('email'),
    age: Number(formData.get('age')),
  };

  // 验证数据
  const validatedData = userSchema.parse(rawData);

  // 如果验证通过，创建用户
  const user = await db.user.create({ data: validatedData });

  return { success: true, user };
}
```

### 处理文件上传

表单上传文件时需要特殊处理：

```typescript
// components/UploadForm.tsx
'use client';

import { uploadAvatar } from '@/app/actions';

export default function UploadForm() {
  return (
    <form action={uploadAvatar}>
      <input name="avatar" type="file" accept="image/*" />
      <button type="submit">上传</button>
    </form>
  );
}

// app/actions.ts
'use server';

export async function uploadAvatar(formData: FormData) {
  // 获取上传的文件
  const file = formData.get('avatar') as File;

  if (!file) {
    return { error: '请选择文件' };
  }

  // 读取文件内容
  const bytes = await file.arrayBuffer();
  const buffer = Buffer.from(bytes);

  // 保存到存储服务（如 AWS S3、Vercel Blob）
  // const url = await uploadToStorage(buffer, file.name);

  // 保存文件 URL 到数据库
  // await db.user.update({ data: { avatarUrl: url } });

  return { success: true };
}
```

**文件上传规则**：
- 表单必须设置 `encType="multipart/form-data"`（Next.js 自动处理）
- `formData.get('file')` 返回 `File` 对象
- 可以用 `file.arrayBuffer()` 或 `file.text()` 读取文件内容

### 处理复杂数据

如果表单数据结构复杂，可以自定义序列化：

```typescript
// components/EventForm.tsx
'use client';

import { createEvent } from '@/app/actions';

export default function EventForm() {
  return (
    <form action={createEvent}>
      <input name="title" type="text" />
      <input name="startDate" type="date" />
      <input name="endDate" type="date" />

      {/* 嵌套数据：用 JSON 字符串传递 */}
      <input
        name="attendees"
        type="hidden"
        value={JSON.stringify(['alice@example.com', 'bob@example.com'])}
      />

      <button type="submit">创建活动</button>
    </form>
  );
}

// app/actions.ts
'use server';

export async function createEvent(formData: FormData) {
  const title = formData.get('title') as string;
  const startDate = formData.get('startDate') as string;
  const endDate = formData.get('endDate') as string;

  // 解析嵌套数据
  const attendees = JSON.parse(formData.get('attendees') as string);

  const event = await db.event.create({
    data: {
      title,
      startDate: new Date(startDate),
      endDate: new Date(endDate),
      attendees, // 数组
    },
  });

  return { success: true, event };
}
```

### 错误处理与用户反馈

用 `useActionState` Hook 处理表单状态和错误：

```typescript
// components/CreateUserForm.tsx
'use client';

import { createUser } from '@/app/actions';
import { useActionState } from 'react'; // React 19+

export default function CreateUserForm() {
  // useActionState 返回 [state, formAction, isPending]
  const [state, formAction, isPending] = useActionState(createUser, null);

  return (
    <form action={formAction}>
      {/* 显示错误信息 */}
      {state?.error && <div className="error">{state.error}</div>}

      {/* 显示成功信息 */}
      {state?.success && <div className="success">用户创建成功！</div>}

      <input name="name" type="text" disabled={isPending} />
      <input name="email" type="email" disabled={isPending} />

      <button type="submit" disabled={isPending}>
        {isPending ? '提交中...' : '创建用户'}
      </button>
    </form>
  );
}

// app/actions.ts
'use server';

export async function createUser(formData: FormData, prevState: any) {
  // prevState 是上一次的状态（React 19+ 自动传递）
  try {
    const name = formData.get('name') as string;
    const email = formData.get('email') as string;

    // 验证数据
    if (!name || !email) {
      return { error: '姓名和邮箱不能为空' };
    }

    // 创建用户
    const user = await db.user.create({
      data: { name, email },
    });

    return { success: true, user };
  } catch (error) {
    return { error: '创建用户失败，请稍后重试' };
  }
}
```

**`useActionState` 的参数**：
- 第 1 个：Server Action 函数
- 第 2 个：初始状态（可以是 `null` 或对象）
- 返回值：`[state, formAction, isPending]`
  - `state`：Server Action 返回的结果
  - `formAction`：传给 `<form action={...}>`
  - `isPending`：是否正在提交（用于显示加载状态）

## 你需要记住的

1. 表单的 `action` 属性可以直接绑定 Server Action，无需手动 `fetch`。
2. 浏览器自动收集表单数据成 `FormData` 对象，服务端用 `formData.get()` 获取字段。
3. `name` 属性决定字段在 `FormData` 中的键名。
4. 文件上传用 `<input type="file">`，服务端用 `formData.get()` 获取 `File` 对象。
5. 用 `useActionState` Hook 处理表单状态、错误和加载状态。

## AI 代码中的线索

当你在 AI 生成的代码中看到这些模式，说明这是表单处理：

```typescript
// 线索 1：form 的 action 属性
<form action={createUser}>...</form>

// 线索 2：input 的 name 属性
<input name="username" />

// 线索 3：Server Action 接收 FormData
export async function createUser(formData: FormData) {
  const name = formData.get('name');
}

// 线索 4：useActionState Hook
const [state, formAction, isPending] = useActionState(createUser, null);

// 线索 5：隐藏字段传递复杂对象
<input name="data" type="hidden" value={JSON.stringify(complexData)} />

// 线索 6：button 的 disabled 状态
<button disabled={isPending}>提交</button>
```

如果看到 `e.preventDefault()` 和手动 `fetch`，这是传统表单处理（不推荐）。

## 验证问题

- [ ] 以下表单提交时，`FormData` 中包含哪些字段？
  ```typescript
  <form action={submit}>
    <input name="username" value="alice" />
    <input type="email" name="contact" value="alice@example.com" />
    <button type="submit">提交</button>
  </form>
  ```
- [ ] 如果你想上传用户头像，表单应该怎么写？Server Action 如何获取文件？
- [ ] `useActionState` 的三个返回值分别是什么？如何使用？
