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

TypeScript 的价值：**在代码运行之前，编辑器就能告诉你哪里写错了**。

## 类比

| 概念 | 类比 |
|------|------|
| JavaScript | 快递包裹，外面没写是什么东西（拆开才知道） |
| TypeScript | 快递包裹，外面贴了标签（"易碎品"、"书籍"、"3kg"） |
| `: string` | 标签上写"这里面是文字" |
| `interface` | 一张清单，列明包裹里必须有哪些东西 |
| 编辑器报错 | 快递员检查发现清单和实际内容不符，当场退回 |

没有标签，你只能拆开看有没有被坑。有了标签，拿到手里就知道对不对。

## 核心内容

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

**"运行时擦除"是什么意思**：TypeScript 编译后变成 JavaScript，所有类型注解会被移除。`const name: string = 'Tom'` 编译后变成 `const name = 'Tom'`。类型注解只在开发时帮助编辑器检查错误，运行时不存在。

### 2. interface — 描述对象的形状

```ts
interface User {
  id: number
  name: string
  email: string
  avatar?: string        // ? 表示可选属性（有没有都行）
  role: 'admin' | 'user' // 联合类型：只能是这两个字符串之一
}

// 使用
const user: User = {
  id: 1,
  name: 'Tom',
  email: 'tom@example.com',
  role: 'admin'
  // avatar 不写也行，因为有 ?
}
```

### 3. type — 类似 interface，但更灵活

```ts
// 基本用法和 interface 几乎一样
type User = {
  id: number
  name: string
}

// type 能做 interface 做不到的事：组合已有类型
type Status = 'active' | 'inactive' | 'banned'  // 一组可选值
type ID = number | string                         // 可以是多种类型
```

**简单区分**：
- 定义"对象的形状"（有哪些字段）→ 用 `interface`
- 定义"一组值的选项"或"类型的组合" → 用 `type`
- AI 两个都会用，你只需要看懂

### 常见模式（AI 代码中出现频率极高）

```ts
// 1. 组件的 Props 类型（Props 是 React 组件接收参数的方式，阶段 1 会详细讲）
interface ButtonProps {
  text: string
  onClick: () => void           // 一个没有参数、没有返回值的函数
  variant?: 'primary' | 'secondary'
  disabled?: boolean
}

// 2. 可能为空的值
const user: User | null = fetchUser()
//    ↑ User 或者 null（两种可能）

// 3. async 函数返回值
async function getUsers(): Promise<User[]> {
  //                    ↑ 异步函数，最终返回一个 User 数组
  const res = await fetch('/api/users')  // fetch 是浏览器内置的请求函数
  return res.json()
}

// 4. 泛型 — 让类型也能"接收参数"
// ApiResponse<T> 中的 T 是一个类型参数，使用时替换成具体类型
interface ApiResponse<T> {
  data: T               // T 是占位符，实际类型由使用时决定
  error: string | null
  status: number
}
// 使用时：ApiResponse<User> → data 的类型就是 User
// 使用时：ApiResponse<string> → data 的类型就是 string
```

## 你需要记住的

1. `:` 后面的都是类型注解，运行时会被编译器擦除，不会执行
2. `interface` 描述"对象有哪些字段"，`type` 描述"一组值的选项"
3. `?` = 可选，`|` = 或者，`[]` = 数组
4. `Promise<X>` = 异步操作，最终返回 X
5. `<T>` = 泛型，让类型也能接收参数，使用时替换成具体类型

## 读 AI 代码的策略

看到一段 TypeScript 代码，**先看类型定义（interface/type），再看逻辑**。类型定义就是最好的文档。

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

- [ ] `const name: string` 中的 `string` 会在运行时执行吗？为什么？
- [ ] `interface` 和 `type` 有什么区别？各适合什么场景？
- [ ] 看到 `Promise<User[]>` 你能说出这是什么意思吗？`[]` 代表什么？
- [ ] `ApiResponse<T>` 中的 `T` 是什么？为什么需要它？
