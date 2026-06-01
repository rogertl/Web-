# 1.1 React 组件与 JSX

## 一句话
React 组件是可复用的 UI 构建块，JSX 是在 JavaScript 中写 HTML-like 语法的方式。

## 为什么需要它

如果不理解 React 组件和 JSX，当看到 AI 生成的代码中大量的函数返回 HTML-like 语法时，你会完全搞不懂：这不是标准的 JavaScript，为什么能运行？为什么有些地方用 `className` 而不是 `class`？组件之间是如何组合的？

React 组件让 UI 开发变成搭积木，JSX 让这种搭积木的方式直观易懂。不掌握它们，就无法理解任何 React/Next.js 项目的基本结构。

## 类比

React 组件就像乐高积木：每个积木（组件）是一个独立的、有特定形状和功能的单元，可以把它们拼在一起组成更大的结构（页面）。

| 概念 | 类比 |
|------|------|
| React 组件 | 乐高积木块（标准件，可重复使用） |
| JSX | 乐高拼装说明书（用视觉化的方式描述积木如何组合） |
| Props | 传递给下一个积木块的参数（比如颜色、尺寸） |
| 组件组合 | 把多个小积木拼成一个模型（从小组件拼成页面） |

## 核心内容

### React 组件是什么

React 组件是一个返回 UI 的 JavaScript 函数。组件接收输入（Props），返回描述 UI 的结构（JSX）。

```tsx
// 这是一个最简单的 React 组件
function Welcome() {
  return <h1>Hello, World!</h1>;
}

// 也可以用箭头函数
const Welcome = () => {
  return <h1>Hello, World!</h1>;
};
```

关键特征：
- 组件名必须大写开头（`Welcome` 而不是 `welcome`）
- 组件返回 JSX（看起来像 HTML 的语法）
- 组件可以像 HTML 标签一样使用：`<Welcome />`

### JSX 是什么

JSX 是 JavaScript 的语法扩展，允许在 JS 中写类似 HTML 的代码。它不是 HTML，也不是字符串，而是语法糖——编译器会把它转换成标准的 JavaScript 调用。

```tsx
// 你写的 JSX
const element = <h1>Hello, World!</h1>;

// 编译后的实际 JavaScript（简化版）
const element = React.createElement('h1', null, 'Hello, World!');
```

JSX 的规则：
1. 必须有一个根元素（或者用 Fragment `<>...</>` 包裹）
2. 所有标签必须闭合（`<img />` 而不是 `<img>`）
3. 使用 `className` 而不是 `class`（因为 `class` 是 JS 关键字）
4. 属性名用 camelCase（`onClick` 而不是 `onclick`）
5. `{}` 在 JSX 中表示 JavaScript 表达式

```tsx
function Card({ title, content }) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <p>{content}</p>
      <button onClick={() => alert('Clicked!')}>
        Click me
      </button>
    </div>
  );
}
```

### 组件的组合

组件可以被其他组件使用，形成组件树：

```tsx
function Button({ text }) {
  return <button className="btn">{text}</button>;
}

function Card({ title, content }) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <p>{content}</p>
      <Button text="Learn More" />
    </div>
  );
}

function Page() {
  return (
    <div className="page">
      <Card
        title="React Components"
        content="Components are the building blocks of React apps."
      />
    </div>
  );
}
```

### Fragment 的使用

当不需要额外的 DOM 元素时，用 Fragment 避免嵌套：

```tsx
// ❌ 不必要的 div
function List() {
  return (
    <div>
      <li>Item 1</li>
      <li>Item 2</li>
    </div>
  );
}

// ✅ 使用 Fragment
function List() {
  return (
    <>
      <li>Item 1</li>
      <li>Item 2</li>
    </>
  );
}
```

### JSX 中的条件渲染

JSX 中的条件渲染用三元运算符或逻辑与（`&&`）：

```tsx
function Greeting({ isLoggedIn }) {
  return (
    <div>
      {isLoggedIn ? <WelcomeUser /> : <LoginPrompt />}
    </div>
  );
}

function ShowDetails({ hasPermission }) {
  return (
    <div>
      <h1>Dashboard</h1>
      {hasPermission && <AdminPanel />}
    </div>
  );
}
```

### JSX 中的列表渲染

列表渲染用 `map()`，注意每个元素需要唯一的 `key`：

```tsx
function TodoList({ items }) {
  return (
    <ul>
      {items.map((item) => (
        <li key={item.id}>
          {item.text}
        </li>
      ))}
    </ul>
  );
}
```

`key` 的重要性：
- 帮助 React 识别哪些元素改变了、添加了或删除了
- 应该使用稳定的、唯一的值（如 ID），而不是数组索引
- 索引作为 key 会导致列表顺序变化时的性能问题或状态错乱

## 你需要记住的

- React 组件是返回 JSX 的函数，组件名必须大写开头
- JSX 是语法糖，会被编译成 `React.createElement()` 调用
- JSX 中 `class` 写成 `className`，属性用 camelCase
- `{}` 在 JSX 中执行 JavaScript 表达式
- 组件可以嵌套组合，形成组件树
- 列表渲染时每个元素需要唯一的 `key` 属性
- 用 Fragment（`<>...</>`）避免不必要的 DOM 嵌套

## AI 代码中的线索

在 AI 生成的代码中，以下模式表示正在使用 React 组件和 JSX：

```tsx
// 1. 函数返回 JSX
function Component() {
  return <div>...</div>;
}

// 2. 大写开头的标签使用
<Card title="..." />

// 3. className 而不是 class
<div className="container">...</div>

// 4. {} 中的表达式
<h1>{user.name}</h1>

// 5. 列表 map
{items.map(item => <Item key={item.id} />)}
```

当你看到这些模式时，这个文件一定是 React 组件代码。

## 验证问题

- [ ] 能否解释为什么 JSX 中的 `class` 要写成 `className`？
- [ ] 在什么情况下应该使用 Fragment 而不是 `div`？
- [ ] 为什么列表渲染需要 `key`，它解决了什么问题？
