# 1.3 Hooks 基础（useState、useEffect）

## 一句话
Hooks 是 React 函数组件中处理状态和副作用的特殊函数。

## 为什么需要它

不理解 Hooks，当看到 AI 代码中组件内部有 `useState`、`useEffect` 等"魔法"调用时，你会完全迷失：这些函数从哪来？为什么组件一渲染就会执行？为什么 `u[...]

Hooks 是现代 React 开发的核心。不掌握 useState 和 useEffect，就无法理解组件如何响应数据变化、如何管理副作用（API 调用、订阅等）。

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

### useState 中变量的生命周期

这是初学者最容易困惑的地方。让我用一个真实的例子来说明：

```tsx
function Counter() {
  const [count, setCount] = useState(0);
  
  // 这个普通变量，每次组件渲染都会重新创建
  const multiplied = count * 2;  // ❌ 每次渲染都是新对象
  
  console.log('Component rendered, multiplied =', multiplied);

  return (
    <div>
      <p>Count: {count}</p>
      <p>Multiplied: {multiplied}</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

**完整的生命周期流程：**

```
【第一次渲染】
1. 函数执行：Counter()
2. useState(0) 被调用
   └─ React 在内存中为这个组件实例创建一个"状态存储"，存储 count = 0
3. const multiplied = count * 2  → multiplied = 0
4. 组件返回 JSX，React 渲染到 DOM
5. 控制台输出：'Component rendered, multiplied = 0'

【用户点击按钮】
6. onClick 触发 → setCount(1) 被调用
7. React 更新内存中的状态：count = 1
8. React **立即重新调用** Counter() 函数
   └─ 之前的 multiplied 变量被丢弃

【第二次渲染】
9. 函数执行：Counter()（从头开始！）
10. useState(0) 被调用
    └─ React 不会再次初始化，而是从状态存储中返回最新值：count = 1
    └─ const [count, setCount] = [1, setCount函数]
11. const multiplied = count * 2  → multiplied = 2（新对象！）
12. 组件返回 JSX，React 渲染到 DOM
13. 控制台输出：'Component rendered, multiplied = 2'

【继续点击...】
14. 每次点击都重复 6-13 的流程
```

**关键理解点：**

| 概念 | 生命周期 | 说明 |
|------|---------|------|
| **useState 状态变量** | 持久存储 | 在组件卸载前一直存活，跨越多次渲染 |
| **普通变量** | 每次重建 | 组件每次渲染都会重新创建，前一个被销毁 |
| **setState 函数** | 持久存储 | React 确保在整个生命周期中是同一个函数引用 |

```tsx
function Demo() {
  const [count, setCount] = useState(0);
  const temp = Math.random();  // 每次渲染都是不同的数字！
  
  // ✅ 这个 if 永远为 true，因为每次都是新对象
  if (temp !== temp) {
    console.log('不可能执行');
  }

  return (
    <div>
      <p>Count: {count}</p>
      <p>Random: {temp}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}

// 【第一次渲染】temp = 0.523
// 【第二次渲染】temp = 0.841（完全不同的数字！前一个被销毁）
// 【第三次渲染】temp = 0.276
```

**为什么这很重要？**

1. **避免在组件函数体中创建引用类型**
```tsx
// ❌ 不好：每次渲染都创建新数组/对象
function BadComponent() {
  const [items, setItems] = useState([]);
  const defaultOptions = { color: 'red' };  // 每次渲染都是新对象！
  
  // 如果在 useEffect 中使用 defaultOptions，会导致无限循环
  useEffect(() => {
    console.log('Options changed');
  }, [defaultOptions]);  // 依赖总是在改变！
}

// ✅ 好：使用 useState 或 useCallback
function GoodComponent() {
  const [items, setItems] = useState([]);
  
  // 方式1：用 useState 存储
  const [defaultOptions] = useState({ color: 'red' });
  
  // 方式2：用 useCallback（后续章节）
  // const getOptions = useCallback(() => ({ color: 'red' }), []);
  
  useEffect(() => {
    console.log('Options changed');
  }, [defaultOptions]);  // 依赖保持不变
}
```

2. **理解为什么状态更新会触发重新渲染**
```tsx
function Counter() {
  const [count, setCount] = useState(0);

  // 每次点击按钮：
  // 1. setCount(count + 1) 被调用
  // 2. React 更新内存中的状态值
  // 3. React 重新调用整个 Counter 函数
  // 4. useState(0) 返回最新的 count 值
  // 5. 函数体的代码再执行一遍
  // 6. UI 更新显示新的 count
  
  const handleClick = () => {
    setCount(count + 1);
    // 这里 count 仍然是旧值！新值在下一次渲染才会生效
    console.log(count);  // 总是比点击的慢一步
  };

  return (
    <div>
      <p>{count}</p>
      <button onClick={handleClick}>Click</button>
    </div>
  );
}
```

3. **为什么 useEffect 的依赖数组很关键**
```tsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  // ❌ 没有依赖数组：每次渲染都执行
  useEffect(() => {
    fetchUser(userId);  // API 被无限调用！
  });

  // ✅ 空依赖数组：只在首次渲染执行
  useEffect(() => {
    fetchUser(userId);
  }, []);

  // ✅ 依赖 userId：userId 变化时执行
  useEffect(() => {
    fetchUser(userId);
  }, [userId]);
}

// 【第一次渲染】
// - useEffect 执行 → 发起 API 请求
// 
// 【数据到达，调用 setUser】
// - 触发第二次渲染
// - useEffect 再次检查依赖
//   - 如果没有依赖数组 → 再次发起 API 请求（无限循环！）
//   - 如果依赖 userId 且 userId 没变 → 不执行
```

### useState 的使用场景

表单输入、切换状态、累积数据等：

```tsx
function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [isLoading, setIsLoading] = useState(false);

  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    // e.preventDefault() 阻止表单默认提交行为（防止页面刷新）
    e.preventDefault();

    // 更新状态触发 UI 更新
    setIsLoading(true);  // 按钮显示"提交中..."

    try {
      // 异步操作（如 API 调用）
      await login(email, password);

      // 登录成功后可以重定向或显示成功消息
    } catch (error) {
      console.error('登录失败:', error);
    } finally {
      // 恢复状态
      setIsLoading(false); // 按钮恢复可点击
    }
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

**这段表单代码中的关键概念**：

| 概念 | 说明 | 示例 |
|------|------|------|
| **React.FormEvent** | TypeScript 表单事件类型 | `e: React.FormEvent<HTMLFormElement>` |
| **e.preventDefault()** | 阻止表单默认提交（防止页面刷新） | 在提交处理开始时调用 |
| **setIsLoading(true)** | 更新状态，触发组件重新渲染 | 状态变化 → UI 立即响应 |
| **await login()** | 等待异步操作完成 | 必须在 `async` 函数中使用 |
| **try/catch/finally** | 错误处理和资源清理 | `finally` 中恢复加载状态 |
| **e.target.value** | 获取输入框的当前值 | `onChange` 事件的目标元素 |
| **disabled={isLoading}** | 根据状态禁用按钮 | 加载中时禁止重复提交 |
| **value={email}** | 受控组件：值由状态决定 | 输入框显示 `email` 状态的值 |

**什么是受控组件？**
```tsx
// 受控组件：值由 React 状态控制
<input value={email} onChange={(e) => setEmail(e.target.value)} />
// ↓
// 用户输入 → onChange 触发 → setEmail 更新状态 → value 更新 → 显示新值

// 非受控组件（不推荐）：值由 DOM 自己管理
<input defaultValue={email} />
```

**表单处理的完整流程：**
1. 用户输入 → `onChange` 事件触发 → `setEmail` 更新状态 → 输入框显示新值
2. 用户提交 → `onSubmit` 事件触发 → `handleSubmit` 执行
3. `e.preventDefault()` 阻止页面刷新
4. `setIsLoading(true)` 更新状态 → 按钮变为"登录中..."
5. `await login()` 执行异步操作
6. `setIsLoading(false)` 恢复状态 → 按钮恢复可点击

---

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

---

## 组件完整的渲染生命周期

理解 Hooks 的关键是理解组件的渲染流程。以 `UserProfile` 组件为例：

```
【第一次渲染】
1. 函数组件执行
   ├─ useState(null) → user = null
   └─ useState(true) → loading = true
   
2. 组件渲染到 DOM
   └─ 页面显示 "Loading..."
   
3. 组件已经完全渲染完毕，**现在**才执行 useEffect
   └─ 开始 fetchUser() 异步操作（不阻塞渲染）

【等待中...】
用户看到 "Loading..." 几秒钟

【数据到达，状态更新】
4. fetchUser() 完成，调用 setUser(data)
   └─ React 触发重新渲染

【第二次渲染】
5. 函数组件再次执行
   ├─ useState(null) → user = { name: '张三', email: '...' }
   └─ useState(true) → loading = false
   
6. 组件渲染到 DOM
   └─ 页面显示用户信息（不再是"Loading..."）
   
7. useEffect 检查依赖数组 [userId]
   └─ userId 没有变化，所以不再执行
```

**核心点：**
- ✅ `useState` 初始化状态，状态变化触发重新渲染
- ✅ `useEffect` 在渲染**之后**执行，用来做数据获取、订阅等副作用
- ✅ 副作用中的 `setUser()` 会再次触发渲染，形成循环直到数据完全加载

**为什么要这样设计？**
1. **渲染优先**：先渲染页面骨架（显示 Loading），再去获取数据
2. **非阻塞**：useEffect 不会延迟页面显示，提升用户体验
3. **自动同步**：数据到了自动更新状态，状态变化自动重新渲染

---

## 你需要记住的

- `useState` 返回 `[状态值, 更新函数]`，状态变化触发重新渲染
- **useState 的状态是持久的**，但普通变量在每次渲染都会重新创建
- `useEffect` 在渲染**之后**执行副作用（API 调用、订阅等）
- useEffect 的依赖数组控制何时重新执行
- useEffect 可以返回清理函数，用于解除副作用
- Hooks 必须在函数顶层调用，不能在条件或循环中使用
- 只在 React 函数组件或自定义 Hook 中调用 Hooks
- **组件渲染流程**：初始化状态 → 渲染 UI → 执行 useEffect → 数据回来后状态更新 → 重新渲染

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
- [ ] 组件首次渲染到显示 "Loading..."，然后数据到达会经历什么过程？
- [ ] useState 中的状态变量和普通变量的生命周期有什么区别？
