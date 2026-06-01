# 1.7 Client Component 交互逻辑

## 一句话
Client Component 是可以处理用户交互（点击、输入等）的 React 组件，用 `'use client'` 指令标记。

## 为什么需要它

不理解 Client Component，当看到 AI 生成的代码文件顶部有 `'use client'` 指令时，你会困惑：这是干什么用的？什么时候需要加这个指令？为什么有些组件需要它，有些不需要？不加会怎样？

Client Component 是 Next.js App Router 的核心概念。不掌握它，就无法区分哪些代码在客户端运行、哪些在服务端运行，更无法理解后续的 Server Components 和数据获取。

## 类比

- **Client Component** 就像餐厅的餐桌：顾客（用户）在这里直接操作（点菜、用餐），可以随时与菜单（UI）互动
- **Server Component** 就像餐厅的后厨：厨师（服务端）在这里准备菜品（数据），但顾客不能直接进入操作

| 概念 | 类比 |
|------|------|
| Client Component | 餐厅餐桌（用户可以直接互动的区域） |
| Server Component | 后厨准备区（服务端准备数据） |
| 'use client' 指令 | "允许顾客入座"的标识（标记可交互区域） |
| 交互事件 | 顾客操作（点击、输入等用户行为） |
| 客户端渲染 | 在餐桌摆放菜品（在浏览器显示 UI） |

## 核心内容

### 'use client' 指令的作用

`'use client'` 是文件顶部的指令，告诉 Next.js 这个文件是 Client Component：

```tsx
'use client'; // 必须在文件第一行

import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}

export default Counter;
```

关键点：
- `'use client'` 必须在文件第一行（在所有 import 之前）
- 标记后，组件及其子组件都在客户端执行
- 可以使用 hooks（useState、useEffect）、事件处理器

### 什么时候需要 'use client'

需要 `'use client'` 的场景：

1. **使用浏览器 API**：localStorage、window、navigator 等

```tsx
'use client';

import { useState, useEffect } from 'react';

function ThemeToggle() {
  const [theme, setTheme] = useState('light');

  useEffect(() => {
    // 使用 localStorage（浏览器 API）
    const savedTheme = localStorage.getItem('theme') || 'light';
    setTheme(savedTheme);
  }, []);

  const toggleTheme = () => {
    const newTheme = theme === 'light' ? 'dark' : 'light';
    setTheme(newTheme);
    localStorage.setItem('theme', newTheme);
  };

  return (
    <button onClick={toggleTheme}>
      Switch to {theme === 'light' ? 'dark' : 'light'} mode
    </button>
  );
}
```

2. **使用事件处理器**：onClick、onChange 等

```tsx
'use client';

function LoginForm() {
  const [email, setEmail] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    // 表单提交逻辑
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

3. **使用 React Hooks**：useState、useEffect 等

```tsx
'use client';

import { useState, useEffect } from 'react';

function Clock() {
  const [time, setTime] = useState(new Date());

  useEffect(() => {
    const timer = setInterval(() => {
      setTime(new Date());
    }, 1000);

    return () => clearInterval(timer);
  }, []);

  return <div>Current time: {time.toLocaleTimeString()}</div>;
}
```

### 不需要 'use client' 的场景

以下场景不需要 `'use client'`：

1. **纯展示组件**：只渲染 props 传入的数据

```tsx
// ❌ 不需要 'use client'
function UserCard({ name, email }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>{email}</p>
    </div>
  );
}
```

2. **数据获取组件**：在服务端获取数据（后续章节详细讲解）

```tsx
// ❌ 不需要 'use client'
async function BlogPost({ id }) {
  const post = await fetch(`/api/posts/${id}`).then(r => r.json());

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}
```

### Client Component 的边界

`'use client'` 指令有"边界效应"：标记后，其子组件也成为客户端组件：

```tsx
// Parent.tsx
'use client';

import { Child } from './Child';

function Parent() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      {/* Child 即使没有 'use client'，也是客户端组件 */}
      <Child />
    </div>
  );
}

// Child.tsx
// 不需要 'use client'，因为父组件已经是客户端组件
function Child() {
  return <div>Child component</div>;
}
```

### 优化 Client Component 边界

将交互逻辑抽取到独立的客户端组件中，避免大片代码变成客户端组件：

```tsx
// ❌ 不好：整个大组件都是客户端组件
'use client';

function Page() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <Header />
      <MainContent />
      <Sidebar />
      {/* 只有这里需要交互 */}
      <button onClick={() => setCount(count + 1)}>
        Count: {count}
      </button>
      <Footer />
    </div>
  );
}

// ✅ 好：把交互逻辑抽取到小组件
// Page.tsx（服务端组件）
function Page() {
  return (
    <div>
      <Header />
      <MainContent />
      <Sidebar />
      <CounterButton />
      <Footer />
    </div>
  );
}

// CounterButton.tsx（客户端组件）
'use client';

function CounterButton() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

### Client Component vs Server Component

| 特性 | Client Component | Server Component |
|------|------------------|------------------|
| 标记 | `'use client'` | 无标记（默认） |
| 执行位置 | 浏览器（客户端） | 服务器 |
| 可用功能 | hooks、浏览器 API、事件处理 | 数据库调用、文件系统、私密信息 |
| 适用场景 | 交互组件、表单、动画 | 数据获取、静态内容、私密操作 |
| 性能 | 增加客户端 JS | 减少 JS bundle 大小 |

### 常见交互模式

```tsx
'use client';

import { useState } from 'react';

// 1. 展开/收起
function ExpandableContent({ title, content }) {
  const [isExpanded, setIsExpanded] = useState(false);

  return (
    <div>
      <button onClick={() => setIsExpanded(!isExpanded)}>
        {isExpanded ? '▼' : '▶'} {title}
      </button>
      {isExpanded && <p>{content}</p>}
    </div>
  );
}

// 2. 选项卡切换
function Tabs({ items }) {
  const [activeTab, setActiveTab] = useState(0);

  return (
    <div>
      <div>
        {items.map((item, index) => (
          <button
            key={index}
            onClick={() => setActiveTab(index)}
            style={{
              fontWeight: activeTab === index ? 'bold' : 'normal'
            }}
          >
            {item.label}
          </button>
        ))}
      </div>
      <div>{items[activeTab].content}</div>
    </div>
  );
}

// 3. 模态框
function Modal({ isOpen, onClose, children }) {
  if (!isOpen) return null;

  return (
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-content" onClick={(e) => e.stopPropagation()}>
        <button onClick={onClose}>✕</button>
        {children}
      </div>
    </div>
  );
}
```

## 你需要记住的

- `'use client'` 指令标记 Client Component，必须写在文件第一行
- Client Component 可以使用 hooks、事件处理器、浏览器 API
- 需要 `'use client'`：交互逻辑、浏览器 API、状态管理
- 不需要：纯展示、服务端数据获取、静态内容
- `'use client'` 有边界效应，子组件也成为客户端组件
- 把交互逻辑抽取到小组件，优化客户端边界
- Client Component 在浏览器执行，Server Component 在服务器执行

## AI 代码中的线索

以下模式表示正在使用 Client Component：

```tsx
// 1. 文件顶部的 'use client' 指令
'use client';

// 2. 使用 hooks
const [state, setState] = useState();

// 3. 事件处理器
<button onClick={handleClick}>

// 4. 浏览器 API
localStorage.getItem('theme')
window.scrollTo()

// 5. useEffect 副作用
useEffect(() => {
  // 副作用代码
}, []);
```

## 验证问题

- [ ] 为什么有些组件需要 `'use client'` 指令，有些不需要？
- [ ] `'use client'` 指令的"边界效应"是什么？如何优化客户端组件边界？
- [ ] Client Component 和 Server Component 在执行位置和可用功能上有什么区别？
