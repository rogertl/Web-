# 1.4 组件通信与状态提升

## 一句话
组件通信是组件之间数据传递的机制，状态提升是将共享状态移到最近的公共祖先组件中。

## 为什么需要它

不理解组件通信和状态提升，当看到 AI 代码中有多个组件需要同步数据时（比如搜索框和结果列表、过滤器列表和内容展示），你会困惑：为什么有些数据要一层层传递？为什么不在子组件直接管理状态？什么时候应该用状态提升？

状态提升是解决组件协作的基础模式。不掌握它，就无法设计合理的组件数据流，更无法理解后续的状态管理工具。

## 类比

- **状态提升** 就像把共享文件柜搬到办公室中央：所有员工（组件）都能访问，而不是各自保存一份（容易不同步）
- **组件通信** 就像公司内部邮件系统：信息通过明确的路径从一个人传递到另一个人

| 概念 | 类比 |
|------|------|
| 状态提升 | 把共享文件柜移到公共区域（集中管理共享数据） |
| Props drilling | 信息一层层传递的审批流程（逐级传达） |
| 回调函数 | 下级向上级汇报的机制（反向通信） |
| 共享状态 | 多人共用一个文件柜（数据同步） |

## 核心内容

### 组件通信的方式

#### 1. 父→子：Props

```tsx
function Parent() {
  const message = "Hello from parent";
  return <Child message={message} />;
}

function Child({ message }) {
  return <p>{message}</p>;
}
```

#### 2. 子→父：回调函数

```tsx
function Parent() {
  const [count, setCount] = useState(0);

  const handleIncrement = () => {
    setCount((c) => c + 1);
  };

  return (
    <div>
      <h1>Count: {count}</h1>
      <ChildButton onIncrement={handleIncrement} />
    </div>
  );
}

function ChildButton({ onIncrement }) {
  return (
    <button onClick={onIncrement}>
      Increment from child
    </button>
  );
}
```

#### 3. 兄弟组件：状态提升到公共父组件

```tsx
function Parent() {
  const [sharedValue, setSharedValue] = useState("");

  return (
    <div>
      <SiblingA
        value={sharedValue}
        onChange={setSharedValue}
      />
      <SiblingB value={sharedValue} />
    </div>
  );
}

function SiblingA({ value, onChange }) {
  return (
    <input
      type="text"
      value={value}
      onChange={(e) => onChange(e.target.value)}
      placeholder="Type here..."
    />
  );
}

function SiblingB({ value }) {
  return <p>You typed: {value}</p>;
}
```

### 状态提升的原理

当多个组件需要共享同一数据时，将状态提升到它们的公共祖先：

```tsx
// ❌ 不好：各自管理状态，不同步
function TemperatureConverter() {
  return (
    <div>
      <CelsiusInput />
      <FahrenheitInput />
    </div>
  );
}

function CelsiusInput() {
  const [celsius, setCelsius] = useState("");
  return (
    <input
      value={celsius}
      onChange={(e) => setCelsius(e.target.value)}
    />
  );
}

function FahrenheitInput() {
  const [fahrenheit, setFahrenheit] = useState("");
  return (
    <input
      value={fahrenheit}
      onChange={(e) => setFahrenheit(e.target.value)}
    />
  );
}

// ✅ 好：状态提升到父组件
function TemperatureConverter() {
  const [temperature, setTemperature] = useState("");

  const toFahrenheit = (c: string) => {
    const celsius = parseFloat(c);
    if (Number.isNaN(celsius)) return "";
    return ((celsius * 9) / 5 + 32).toString();
  };

  const toCelsius = (f: string) => {
    const fahrenheit = parseFloat(f);
    if (Number.isNaN(fahrenheit)) return "";
    return (((fahrenheit - 32) * 5) / 9).toString();
  };

  const handleChange = (value: string) => {
    setTemperature(value);
  };

  return (
    <div>
      <CelsiusInput
        value={temperature}
        onChange={handleChange}
      />
      <FahrenheitInput
        value={toFahrenheit(temperature)}
        onChange={(v) => handleChange(toCelsius(v))}
      />
    </div>
  );
}
```

### 状态提升的步骤

1. **识别共享数据**：确定哪些组件需要同一份数据
2. **找到公共祖先**：在组件树中找到这些组件的最近公共父组件
3. **将状态移到公共祖先**：用 useState 在父组件中管理状态
4. **通过 props 传递**：父组件向下传递状态和更新函数
5. **子组件接收并使用**：子组件通过 props 使用数据

```tsx
// 场景：Todo 列表，需要添加、删除、切换状态

function App() {
  // 1. 状态提升：所有 Todo 相关状态都在这里
  const [todos, setTodos] = useState([
    { id: 1, text: "Learn React", done: false }
  ]);

  // 2. 更新函数
  const addTodo = (text: string) => {
    setTodos([...todos, { id: Date.now(), text, done: false }]);
  };

  const toggleTodo = (id: number) => {
    setTodos(
      todos.map((todo) =>
        todo.id === id ? { ...todo, done: !todo.done } : todo
      )
    );
  };

  const deleteTodo = (id: number) => {
    setTodos(todos.filter((todo) => todo.id !== id));
  };

  return (
    <div>
      {/* 3. 传递状态和更新函数 */}
      <TodoList
        items={todos}
        onToggle={toggleTodo}
        onDelete={deleteTodo}
      />
      <TodoForm onAdd={addTodo} />
    </div>
  );
}
```

### Props Drilling vs 状态提升

Props drilling 是状态提升的传递方式，但层级深时会很繁琐：

```tsx
// Props drilling 示例
function App() {
  const [theme, setTheme] = useState("light");

  return (
    <Page theme={theme} setTheme={setTheme}>
      <Sidebar theme={theme} setTheme={setTheme}>
        <Settings theme={theme} setTheme={setTheme} />
      </Sidebar>
    </Page>
  );
}
```

后续章节会介绍更优的方案（Context API、状态管理库）。

### 状态提升的局限性

状态提升适合小规模共享状态，但有以下问题：

1. **Props drilling**：层级深时代码冗长
2. **更新逻辑复杂**：多个组件同时修改状态时逻辑分散
3. **性能考虑**：大型应用中需要优化不必要的渲染

```tsx
// 状态提升变得复杂时，考虑其他方案
function ComplexApp() {
  // 当状态和更新逻辑变得复杂时
  // 考虑使用 Context API 或状态管理库
  const [user, setUser] = useState(null);
  const [settings, setSettings] = useState({});
  const [notifications, setNotifications] = useState([]);

  // ... 大量的 props 传递
  return (
    <Layout
      user={user}
      setUser={setUser}
      settings={settings}
      setSettings={setSettings}
      notifications={notifications}
      setNotifications={setNotifications}
    >
      {/* ... */}
    </Layout>
  );
}
```

## 你需要记住的

- 组件通信方式：父→子（Props）、子→父（回调）、兄弟→状态提升
- 状态提升：将共享状态移到公共祖先组件，通过 props 传递
- 状态提升步骤：识别共享数据、找公共祖先、移动状态、传递 props
- Props drilling 是状态提升的传递方式，但层级深时会冗长
- 状态提升适合小规模共享，复杂场景考虑 Context API 或状态管理库

## AI 代码中的线索

以下模式表示正在使用组件通信和状态提升：

```tsx
// 1. 父→子通信
<Child data={parentData} />

// 2. 子→父通信（回调）
<ChildButton onClick={handleClick} />

// 3. 兄弟组件同步（状态提升）
function Parent() {
  const [shared, setShared] = useState('');
  return (
    <>
      <ChildA value={shared} onChange={setShared} />
      <ChildB value={shared} />
    </>
  );
}

// 4. 多个更新函数传递
<TodoList
  items={todos}
  onToggle={handleToggle}
  onDelete={handleDelete}
/>
```

## 验证问题

- [ ] 在什么情况下应该使用状态提升而不是让每个组件自己管理状态？
- [ ] Props drilling 的问题是什么？你能想到什么解决方案吗？
- [ ] 如果两个兄弟组件需要同步数据，应该如何设计组件结构？
