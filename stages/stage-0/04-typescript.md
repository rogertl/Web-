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
type Nullable<T> = T | null          // 泛型工具类型
```

**经验法则**：AI 代码中 `interface` 和 `type` 都会出现，功能类似，不用纠结用哪个。

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
  //                    ↑ 异步函数返回 Promise，里面是 User 数组
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
