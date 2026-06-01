# 00 基础概念预热

## 一句话
代码世界的命名规则和简洁写法，让代码易读易懂。

## 为什么需要它
想象你在看一本外文书，如果作者用的拼写混乱、缩写随意，你读起来会非常吃力。代码也是如此——命名规范和语法糖让代码"说同一种语言"，减少理解成本，避免"写的人懂，过一周自己都看不懂"的尴尬。

## 类比

### 命名规范 vs 英文大小写规则
| 技术概念 | 生活类比 |
|---------|---------|
| 驼峰命名 | iPhone、iPad——首词小写，后续每个单词首字母大写 |
| 帕斯卡命名 | MacBook、JavaScript——每个单词首字母都大写 |
| 蛇形命名 | user_name、first_name——单词间用下划线连接 |
| kebab-case | css-class-name、url-slug——单词间用连字符连接 |

### 语法糖 vs 快捷键
| 技术概念 | 生活类比 |
|---------|---------|
| 语法糖 | 键盘快捷键（Cmd+C 复制）——用简写代替繁琐操作 |
| async/await | "等咖啡做好后再喝"——把异步操作写成看起来像同步的代码 |
| 箭头函数 | 缩写英语（"thx" 代替 "thanks"）——用 `=>` 省略 `function` 关键字 |
| 解构 | 点餐时说"要套餐A"——自动包含汉堡+薯条+饮料，不用逐个说 |

## 核心内容

### 一、命名规范

#### 驼峰命名（camelCase）
第一个单词首字母小写，后续每个单词首字母大写。

```typescript
// 变量、函数名通常用驼峰
let userName = "张三";
let isLoggedIn = true;

function getUserInfo() { }
function calculateTotalPrice() { }
```

#### 帕斯卡命名（PascalCase）
每个单词首字母都大写，也叫"大驼峰"。

```typescript
// 类名、接口名、组件名通常用帕斯卡
class UserController { }
interface UserProfile { }
type OrderStatus = "pending" | "completed";

// React 组件名也用帕斯卡
function HomePage() { }
const UserProfile = () => { };
```

#### 蛇形命名（snake_case）
所有字母小写，单词间用下划线连接。

```typescript
// 数据库字段、环境变量常用蛇形
let user_id = 123;
let created_at = "2025-01-01";

// 环境变量也常用蛇形（但全大写）
const API_KEY = "your-secret-key";
const DB_HOST = "localhost";
```

#### kebab-case
所有字母小写，单词间用连字符连接。

```typescript
// 常用于 URL 路径、文件名、CSS 类名
let url-slug = "hello-world";  // 注意：TypeScript 变量名不能用 kebab-case，这是伪代码

// 实际场景：
// 文件名：user-profile.tsx
// URL：/api/user-profile
// CSS 类名：.user-profile
```

---

### 二、语法糖是什么

**语法糖**是编程语言提供的"简化写法"，它不改变功能本质，只是让代码更简洁、更易读。就像英语中的缩写（"don't" 代替 "do not"）。

#### 为什么要用语法糖？
- **减少样板代码**：不用重复写固定模式
- **提高可读性**：简洁的写法更容易理解意图
- **降低出错概率**：写的代码越少，越不容易写错

---

### 三、常见语法糖示例

#### 1. async/await —— 异步操作的同步写法

传统写法（Promise 链）：
```typescript
// 传统写法：回调地狱
function getUserData() {
  fetch("/api/user")
    .then(response => response.json())
    .then(data => {
      console.log(data);
    })
    .catch(error => {
      console.error(error);
    });
}
```

语法糖写法（async/await）：
```typescript
// async/await 写法：看起来像同步代码
async function getUserData() {
  try {
    const response = await fetch("/api/user");
    const data = await response.json();
    console.log(data);
  } catch (error) {
    console.error(error);
  }
}
```

**关键点**：
- `async` 声明函数是异步的
- `await` 等待异步操作完成（会暂停函数执行，直到结果返回）
- 必须配合 `try/catch` 处理错误

---

#### 2. 类属性简写 —— 省略构造函数赋值

传统写法：
```typescript
class User {
  name: string;
  age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
}
```

语法糖写法：
```typescript
class User {
  constructor(
    public name: string,
    public age: number
  ) { }
}
```

**关键点**：
- 在构造函数参数中加 `public`/`private`/`protected`
- TypeScript 自动创建同名属性并赋值

---

#### 3. 箭头函数 —— 省略 `function` 关键字

传统写法：
```typescript
function add(a: number, b: number): number {
  return a + b;
}

const multiply = function(a: number, b: number): number {
  return a + b;
};
```

语法糖写法：
```typescript
const add = (a: number, b: number): number => {
  return a + b;
};

// 单行可以省略大括号和 return（隐式返回）
const multiply = (a: number, b: number): number => a * b;

// 单参数可以省略括号
const double = (n: number) => n * 2;
```

**关键点**：
- `=>` 是箭头函数的标志
- 单行表达式可以省略 `return`（自动返回）
- `this` 绑定与普通函数不同（这在阶段 1 会详细讲解）

---

#### 4. 解构 —— 从对象/数组中提取数据

传统写法：
```typescript
const user = { name: "张三", age: 25, city: "北京" };
const name = user.name;
const age = user.age;
```

语法糖写法：
```typescript
const user = { name: "张三", age: 25, city: "北京" };

// 对象解构
const { name, age } = user;
// 相当于：const name = user.name; const age = user.age;

// 数组解构
const numbers = [1, 2, 3];
const [first, second] = numbers;
// 相当于：const first = numbers[0]; const second = numbers[1];
```

**关键点**：
- 用 `{}` 解构对象，用 `[]` 解构数组
- 变量名必须与对象的属性名匹配
- 可以重命名：`const { name: userName } = user;`

---

#### 5. 展开运算符 —— 复制/合并对象和数组

传统写法：
```typescript
// 合并数组
const arr1 = [1, 2];
const arr2 = [3, 4];
const combined = arr1.concat(arr2);

// 复制对象
const original = { name: "张三", age: 25 };
const copy = Object.assign({}, original);
```

语法糖写法：
```typescript
// 展开数组
const arr1 = [1, 2];
const arr2 = [3, 4];
const combined = [...arr1, ...arr2];  // [1, 2, 3, 4]

// 展开对象
const original = { name: "张三", age: 25 };
const copy = { ...original };
const updated = { ...original, age: 26 };  // 覆盖 age 属性
```

**关键点**：
- `...` 是展开运算符
- 对象展开创建新对象（浅拷贝）
- 可以用来合并、覆盖属性

---

#### 6. 可选链 —— 安全访问嵌套属性

传统写法：
```typescript
const user = {
  profile: {
    address: {
      city: "北京"
    }
  }
};

// 需要层层判断避免报错
const city = user && user.profile && user.profile.address && user.profile.address.city;
```

语法糖写法：
```typescript
// 可选链 `?.` —— 如果中间某层不存在，返回 undefined 而不是报错
const city = user?.profile?.address?.city;
```

**关键点**：
- `?.` 是可选链操作符
- 如果左边是 `null` 或 `undefined`，直接返回 `undefined`
- 避免深层属性访问时的报错

---

#### 7. 空值合并 —— 为 null/undefined 提供默认值

传统写法：
```typescript
const userName = user.name !== null && user.name !== undefined ? user.name : "访客";
```

语法糖写法：
```typescript
const userName = user.name ?? "访客";
```

**关键点**：
- `??` 是空值合并操作符
- 只在左边是 `null` 或 `undefined` 时才返回右边
- 注意：`0`、`false`、`""` 不会被当作"空值"（这与 `||` 不同）

---

## 你需要记住的

1. **命名规范**：变量用驼峰（`userName`），类/组件用帕斯卡（`UserController`），数据库字段用蛇形（`user_id`）
2. **async/await**：让异步代码读起来像同步代码，必须配合 `try/catch` 使用
3. **箭头函数**：`=>` 简化函数写法，单行表达式可省略 `return`
4. **解构**：用 `const { name } = user` 从对象中提取属性
5. **可选链 `?.`**：安全访问嵌套属性，避免深层访问报错

## AI 代码中的线索

当 AI 生成的代码中出现这些模式时，你应该知道正在使用什么：

| 线索 | 含义 |
|-----|------|
| `async function name() {` 或 `const name = async () => {` | 这是异步函数，内部可以使用 `await` |
| `await fetch(...)` 或 `await somePromise` | 等待异步操作完成，代码会暂停在这里直到结果返回 |
| `const { name, age } = user` | 从 `user` 对象中提取 `name` 和 `age` 属性 |
| `const copy = { ...original }` | 复制 `original` 对象的所有属性到新对象 |
| `user?.profile?.address` | 安全访问嵌套属性，如果中间某层不存在会返回 `undefined` 而不是报错 |
| `const value = data ?? "default"` | 如果 `data` 是 `null` 或 `undefined`，使用 `"default"` 作为默认值 |
| `const name = (arg) => arg * 2` | 箭头函数，单参数、单表达式的简化写法 |
| `class User { constructor(public name: string) { } }` | TypeScript 类属性简写，自动创建 `name` 属性 |

**实战提示**：当你看到 AI 代码中有 `async`、`await`、解构、展开运算符时，不要被吓到——它们只是让代码更简洁的"语法糖"，本质逻辑可以用传统写法表达。

## 验证问题

- [ ] 我能区分驼峰命名和帕斯卡命名的使用场景吗？
- [ ] 我能解释 `async/await` 为什么被称为"语法糖"吗？
- [ ] 我能在 AI 生成的代码中识别出解构和展开运算符的用法吗？
