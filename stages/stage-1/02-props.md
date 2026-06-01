# 1.2 Props（父子组件传值）

## 一句话
Props 是父组件向子组件传递数据的单向数据流机制。

## 为什么需要它

不理解 Props，当看到 AI 代码中组件之间传递各种参数时，你会困惑：这些数据从哪里来？子组件如何接收？能不能修改传入的数据？为什么有时候数据传了但组件没有变化？

Props 是组件通信的基础。不掌握它，就无法理解组件之间如何协作，更无法理解后续的状态管理和数据流。

## 类比

Props 就像快递包裹：寄件人（父组件）把物品（数据）打包给收件人（子组件），收件人只能查看和使用，不能改变包裹里的内容。

| 概念 | 类比 |
|------|------|
| Props | 快递包裹（从父组件传给子组件的数据） |
| 父组件 | 寄件人（准备和发送数据） |
| 子组件 | 收件人（接收和使用数据） |
| 单向数据流 | 快递只能寄件人→收件人，不能反过来 |
| Props 只读 | 收件人不能修改包裹里的物品 |

## 核心内容

### Props 的基本用法

Props 通过组件的参数传递，解构赋值时常见：

```tsx
// 子组件接收 props
function Button({ text, color = 'blue' }) {
  return (
    <button
      style={{ backgroundColor: color }}
      onClick={() => alert(`Clicked: ${text}`)}
    >
      {text}
    </button>
  );
}

// 父组件传递 props
function Toolbar() {
  return (
    <div>
      <Button text="Save" color="green" />
      <Button text="Cancel" color="red" />
      <Button text="Help" />
    </div>
  );
}
```

关键点：
- `text` 和 `color` 是 props 的名称
- `color = 'blue'` 是默认值，父组件不传时使用
- 父组件传递时用 `属性名={值}` 的语法

### Props 的只读性

Props 是只读的，子组件不能修改传入的 props：

```tsx
// ❌ 错误：直接修改 props
function UserCard({ name }) {
  name = name.toUpperCase(); // 不要这样做！
  return <h1>{name}</h1>;
}

// ✅ 正确：使用本地状态或派生值
function UserCard({ name }) {
  const displayName = name.toUpperCase();
  return <h1>{displayName}</h1>;
}
```

为什么只读？
- 保证数据流的可预测性
- 避免意外的副作用
- 让数据来源清晰（修改只发生在数据产生的地方）

### Props 的类型检查

用 TypeScript 定义 Props 类型：

```tsx
// 定义 props 类型
interface CardProps {
  title: string;
  content: string;
  author?: string; // 可选属性
  likes: number;
  tags?: string[]; // 可选数组
}

// 使用类型注解
function Card({ title, content, author, likes, tags = [] }: CardProps) {
  return (
    <article className="card">
      <h2>{title}</h2>
      <p>{content}</p>
      {author && <p>By {author}</p>}
      <div className="meta">
        <span>❤️ {likes}</span>
        {tags.map((tag) => (
          <span key={tag} className="tag">
            #{tag}
          </span>
        ))}
      </div>
    </article>
  );
}

// 使用组件
function BlogList() {
  return (
    <Card
      title="Understanding Props"
      content="Props are the primary way to pass data..."
      author="Alice"
      likes={42}
      tags={['react', 'beginner']}
    />
  );
}
```

### Props 的展开语法

传递多个 props 时用展开语法：

```tsx
function UserCard({ name, age, email, city }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>Age: {age}</p>
      <p>Email: {email}</p>
      <p>City: {city}</p>
    </div>
  );
}

function App() {
  const user = {
    name: 'Bob',
    age: 28,
    email: 'bob@example.com',
    city: 'Tokyo'
  };

  // 等价于 <UserCard name={user.name} age={user.age} ... />
  return <UserCard {...user} />;
}
```

### Props 的向下传递（Props Drilling）

当多个层级需要传递数据时，会出现 props drilling：

```tsx
function Page({ user }) {
  return (
    <Layout user={user}>
      <Sidebar user={user} />
      <Content>
        <Header user={user} />
        <Body user={user} />
      </Content>
    </Layout>
  );
}
```

Props drilling 是数据传递的基础方式，但层级深时会冗长。后续章节（1.4 状态提升）会讨论更优的方案。

### Props 与组件复用

合理的 props 设计让组件更可复用：

```tsx
// ❌ 不好：硬编码，不灵活
function AlertBox() {
  return (
    <div className="alert alert-error">
      Error: Operation failed!
    </div>
  );
}

// ✅ 好：通过 props 配置
function AlertBox({ type, message, onClose }) {
  return (
    <div className={`alert alert-${type}`}>
      <span>{message}</span>
      {onClose && <button onClick={onClose}>✕</button>}
    </div>
  );
}

// 使用
<AlertBox
  type="success"
  message="Your changes have been saved!"
  onClose={() => setShowAlert(false)}
/>
```

## 你需要记住的

- Props 是父组件向子组件传递数据的单向机制
- Props 是只读的，子组件不能直接修改
- 用 TypeScript interface 定义 props 类型，提高代码安全性
- 可选 props 用 `?` 标记，并提供默认值
- 展开语法 `{...obj}` 可传递多个 props
- Props drilling 是多层级数据传递的基础方式
- 良好的 props 设计让组件更可复用

## AI 代码中的线索

以下模式表示正在使用 Props：

```tsx
// 1. 组件参数解构
function Component({ prop1, prop2 }) {}

// 2. 传递字符串
<Card title="Hello" />

// 3. 传递表达式（注意花括号）
<Card count={user.items.length} />

// 4. 传递函数
<Button onClick={handleSubmit} />

// 5. 展开语法传递
<User {...userData} />

// 6. TypeScript props 类型
interface Props { name: string; }
function Card({ name }: Props) {}
```

## 验证问题

- [ ] 为什么 Props 是只读的？如果子组件需要修改数据应该怎么做？
- [ ] 在什么情况下应该使用 TypeScript 定义 Props 类型？
- [ ] Props drilling 的问题是什么？你能否想到更好的数据传递方式？
