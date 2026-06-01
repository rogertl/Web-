# 0.4 TypeScript 基础：类型注解、interface、type

## 一句话

**TypeScript 给 JavaScript 加了"说明书"，让你（和 AI）一眼看出每个变量是什么、函数要什么参数。**

## 为什么需要它

JavaScript 的问题：变量可以是任何类型，你不知道一个函数需要传什么参数。

```js
// JavaScript — 你不知道 user 是什么形状
function greet(user) {
  return 'Hello, ' + user.name
}
// 传 { name: 'Tom' } → 正常
// 传 { age: 25 } → 报错（但 JS 不会提前告诉你）
// 传 null → 崩溃
```

```ts
// TypeScript — 明确说明 user 必须有什么
interface User {
  name: string
  age: number
}

function greet(user: User): string {
  return 'Hello, ' + user.name
}
// 传 { name: 'Tom', age: 25 } → ✅
// 传 { age: 25 } → ❌ 编辑器立刻标红：缺少 name
// 传 null → ❌ 编辑器立刻标红：类型不对
```

## 类比

| 概念 | 类比 |
|------|------|
| JavaScript | 快递包裹，外面没写是什么东西（拆开才知道） |
| TypeScript | 快递包裹，外面贴了标签（"易碎品"、"书籍"、"3kg"） |
| `: string` | 标签上写"这里面是文字" |
| `interface` | 一张清单，列明包裹里必须有哪些东西 |
| 编辑器报错 | 快递员检查发现清单和实际内容不符，当场退回 |

没有标签，你只能拆开看有没有被坑。有了标签，拿到手里就知道对不对。

## 三个核心语法

### 1. 类型注解 — 给变量贴标签

```ts
const name: string = 'Tom'
const age: number = 25
const isActive: boolean = true
const items: string[] = ['a', 'b', 'c']  // 字符串数组

// 函数的参数和返回值也可以标注
function add(a: number, b: number): number {
  return a + b
}
```

### 2. interface — 描述对象的形状

```ts
interface User {
  id: number
  name: string
  email: string
  avatar?: string        // ? 表示可选属性
  role: 'admin' | 'user' // 联合类型：只能是这两个值之一
}

// 使用
const user: User = {
  id: 1,
  name: 'Tom',
  email: 'tom@example.com',
  role: 'admin'
}
```

### 3. type — 类似 interface，但更灵活

```ts
// 基本用法和 interface 几乎一样
type User = {
  id: number
  name: string
}

// type 能做 interface 做不到的事
type Status = 'active' | 'inactive' | 'banned'
type ID = number | string            // 可以是数字或字符串
```

**简单区分**：定义"对象的形状"用 `interface`，定义"一组值的选项"或"类型的组合"用 `type`。AI 两个都会用，你只需要看懂。

## 常见模式（AI 代码中出现频率极高）

```ts
// 1. API 响应类型
interface ApiResponse<T> {
  data: T
  error: string | null
  status: number
}

// 2. 组件 Props 类型
interface ButtonProps {
  text: string
  onClick: () => void
  variant?: 'primary' | 'secondary'
  disabled?: boolean
}

// 3. 可能为空的值
const user: User | null = fetchUser()
//    ↑ 注意这里：User 或者 null

// 4. async 函数返回值
async function getUsers(): Promise<User[]> {
  //                    ↑ 异步函数，最终返回 User 数组
  const res = await fetch('/api/users')
  return res.json()
}
```

## 你需要记住的

1. `:` 后面的都是类型注解，不是代码逻辑，运行时会被擦除
2. `interface` 和 `type` 都是描述"数据长什么样"
3. `?` 表示可选，`|` 表示或者，`[]` 表示数组
4. 看到 `Promise<X>` → 这是一个异步操作，最终会返回 X

## 读 AI 代码的策略

看到一段 TypeScript 代码，先看类型定义（interface/type），再看逻辑。**类型定义就是最好的文档**。

```ts
// 先看这个 — 你就知道这个函数要什么、返回什么
async function createPost(
  title: string,
  content: string,
  tags: string[]
): Promise<Post> {

// 再看具体实现
  const response = await fetch('/api/posts', {
    method: 'POST',
    body: JSON.stringify({ title, content, tags })
  })
  return response.json()
}
```

## 验证问题

- [ ] `const name: string` 中，`string` 会在运行时执行吗？
- [ ] `interface` 和 `type` 有什么区别？什么时候用哪个？
- [ ] 看到 `Promise<User[]>` 你能说出这是什么意思吗？
