# 1.3 Hooks 基础（useState、useEffect）

## 一句话
Hooks 是 React 函数组件中处理状态和副作用的特殊函数。

## 为什么需要它

不理解 Hooks，当看到 AI 代码中组件内部有 `useState`、`useEffect` 等"魔法"调用时，你会完全迷失：这些函数从哪来？为什么组件一渲染就会执行？为什么 `useEffect` 里面可以写异步代码？依赖数组是干什么的？

Hooks 是现代 React 开发的核心。不掌握 useState 和 useEffect，就无法理解组件如何响应数据变化、如何处理副作用（API 调用、订阅等）。

## 类比

- **useState** 就像电视遥控器的频道切换器：按下按钮切换频道，屏幕立即显示新内容（状态改变触发 UI 更新）
- **useEffect** 就像闹钟的定时任务：在特定条件满足时（依赖变化）自动执行某些操作（副作用）

| 概念 | 类比 |
|------|------|
| useState | 遥控器频道按钮（状态管理） |
| useEffect | 闹钟定时任务（副作用处理） |
| 依赖数组 | 闹钟的触发条件（什么时候执行） |
| 副作用 | 闹钟响铃时的操作（API 调用、订阅等） |

## 核心内容

### useState：管理组件内部状态

`useState` 让函数组件拥有内部状态，状态变化会触发组件重新渲染：

```tsx
import { useState } from 'react';

function Counter() {
  // count 是当前状态，setCount 是更新状态的函数
  // 0 是初始值
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Current count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
      <button onClick={() => setCount(0)}>
        Reset
      </button>
    </div>
  );
}
```

关键点：
- `useState` 返回一个数组：`[当前值, 更新函数]`
- 解构赋值命名：`const [state, setState] = useState(initialValue)`
- 调用 `setState` 会触发组件重新渲染
- 初始值只在组件首次渲染时使用

### useState 的使用场景

表单输入、切换状态、累积数据等：

```tsx
function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [isLoading, setIsLoading] = useState(false);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsLoading(true);

    // 模拟 API 调用
    await login(email, password);

    setIsLoading(false);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
      />
      <button type="submit" disabled={isLoading}>
        {isLoading ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
}
```

### useEffect：处理副作用

`useEffect` 在组件渲染后执行副作用（API 调用、订阅、手动 DOM 操作等）：

```tsx
import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // 副作用：获取用户数据
    async function fetchUser() {
      try {
        const response = await fetch(`/api/users/${userId}`);
        const data = await response.json();
        setUser(data);
      } catch (error) {
        console.error('Failed to fetch user:', error);
      } finally {
        setLoading(false);
      }
    }

    fetchUser();
  }, [userId]); // 依赖数组：userId 变化时重新执行

  if (loading) return <div>Loading...</div>;
  if (!user) return <div>User not found</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>Email: {user.email}</p>
    </div>
  );
}
```

### useEffect 的执行时机

```tsx
function EffectDemo() {
  const [count, setCount] = useState(0);

  // 1. 每次渲染后都执行
  useEffect(() => {
    console.log('Component rendered');
  });

  // 2. 只在首次渲染后执行（空依赖数组）
  useEffect(() => {
    console.log('Component mounted');
  }, []);

  // 3. count 变化时执行
  useEffect(() => {
    console.log('Count changed:', count);
  }, [count]);

  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

### useEffect 的清理函数

返回的函数会在组件卸载或下次 effect 执行前调用：

```tsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    // 建立连接
    const connection = createChatConnection(roomId);

    connection.on('message', (msg) => {
      setMessages((prev) => [...prev, msg]);
    });

    // 清理函数：断开连接
    return () => {
      connection.disconnect();
    };
  }, [roomId]);

  return (
    <ul>
      {messages.map((msg) => (
        <li key={msg.id}>{msg.text}</li>
      ))}
    </ul>
  );
}
```

### Hooks 的规则

Hooks 有严格的调用规则，违反规则会导致 bug：

```tsx
function BadExample() {
  const [count, setCount] = useState(0);

  if (count > 5) {
    // ❌ 错误：不能在条件语句中调用 Hook
    const [data, setData] = useState(null);
  }

  const handleClick = () => {
    // ❌ 错误：不能在普通函数中调用 Hook
    const [name, setName] = useState('');
  };

  useEffect(() => {
    // ✅ 正确：在函数顶层调用
    console.log('Effect runs');
  });

  return <div>Count: {count}</div>;
}

// ✅ 正确示例
function GoodExample() {
  const [count, setCount] = useState(0);
  const [data, setData] = useState(null);

  useEffect(() => {
    console.log('Effect runs');
  }, [count]);

  return <div>Count: {count}</div>;
}
```

**Hooks 规则总结：**
1. 只在函数顶层调用 Hooks（不在循环、条件、嵌套函数中调用）
2. 只在 React 函数组件或自定义 Hook 中调用 Hooks

为什么？React 依赖 Hooks 的调用顺序来正确关联状态和副作用。违反规则会破坏这个顺序。

## 你需要记住的

- `useState` 返回 `[状态值, 更新函数]`，状态变化触发重新渲染
- `useEffect` 在渲染后执行副作用（API 调用、订阅等）
- useEffect 的依赖数组控制何时重新执行
- useEffect 可以返回清理函数，用于解除副作用
- Hooks 必须在函数顶层调用，不能在条件或循环中使用
- 只在 React 函数组件或自定义 Hook 中调用 Hooks

## AI 代码中的线索

以下模式表示正在使用 Hooks：

```tsx
// 1. useState 状态管理
const [count, setCount] = useState(0);

// 2. useEffect 副作用
useEffect(() => {
  // 副作用代码
}, [dependency]);

// 3. 清理函数
useEffect(() => {
  const sub = subscribe();
  return () => sub.unsubscribe();
}, []);

// 4. 表单状态
const [value, setValue] = useState('');

// 5. 加载状态
const [loading, setLoading] = useState(false);
```

## 验证问题

- [ ] 为什么 useState 更新状态会触发组件重新渲染？
- [ ] useEffect 的依赖数组为空 `[]` 和有依赖 `[count]` 有什么区别？
- [ ] 为什么 Hooks 必须在函数顶层调用，不能在条件语句中使用？
