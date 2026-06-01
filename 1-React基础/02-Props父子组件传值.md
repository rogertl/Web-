# 1.2 Props（父子组件传值）

## 一句话
Props 是父组件向子组件传递数据的单向数据流机制。

## 为什么需要它

不理解 Props，当看到 AI 代码中组件之间传递各种参数时，你会困惑：这些数据从哪里来？子组件如何接收？能不能修改传入的数据？

Props 是组件通信的基础。不掌握它，就无法理解组件之间如何协作。

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

Props 通过组件的参数传递：

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
      <Button text="保存" color="green" />
      <Button text="取消" color="red" />
      <Button text="帮助" />
    </div>
  );
}
```

关键点：
- `text` 和 `color` 是 props 的名称
- `color = 'blue'` 是默认值，父组件不传时使用
- 父组件传递时用 `属性名={值}` 的语法

---

### Props 传递的括号规则

在 JSX 中传递 props 时，括号的使用很重要：

| 传递的数据类型 | 写法 | 说明 |
|-------------|------|------|
| 字符串 | `title="Hello"` | 直接写，不用括号 |
| 数字 | `count={42}` | 必须用 `{}` |
| 变量 | `name={userName}` | 必须用 `{}` |
| 对象 | `user={{ name: "张三" }}` | 双层 `{{}}`：外层 JSX，内层对象 |
| 函数 | `onClick={() => {}}` | 必须用 `{}` |

```tsx
// ❌ 错误示例
<Button text=textValue />          // 变量不用 {} 会当作字符串
<Button count=42 />                // 数字不用 {} 会报错
<Button style={color: "red"} />    // 对象不用外层 {} 会解析错误

// ✅ 正确示例
<Button text={textValue} />        // 变量必须用 {}
<Button count={42} />              // 数字必须用 {}
<Button style={{ color: "red" }} /> // 对象必须用 {{}}
```

**记忆口诀**：除了字符串字面量，其他所有值都必须用花括号 `{}` 包裹。

---

### Props 的只读性

Props 是只读的，子组件不能修改传入的 props：

```tsx
// ❌ 错误：直接修改 props
function UserCard({ name }) {
  name = name.toUpperCase(); // 不要这样做！
  return <h1>{name}</h1>;
}

// ✅ 正确：使用本地变量
function UserCard({ name }) {
  const displayName = name.toUpperCase();
  return <h1>{displayName}</h1>;
}
```

为什么只读？
- 保证数据流的可预测性
- 避免意外的副作用
- 让数据来源清晰（修改只发生在数据产生的地方）

---

### Props 的类型定义

用 TypeScript 定义 Props 类型：

```tsx
// 定义 props 类型
interface CardProps {
  title: string;
  content: string;
  author?: string; // 可选属性
  likes: number;
}

// 使用类型注解
function Card({ title, content, author, likes }: CardProps) {
  return (
    <article className="card">
      <h2>{title}</h2>
      <p>{content}</p>
      {author && <p>作者：{author}</p>}
      <p>❤️ {likes}</p>
    </article>
  );
}

// 使用组件
function BlogList() {
  return (
    <Card
      title="理解 Props"
      content="Props 是组件间传递数据的主要方式..."
      author="小明"
      likes={42}
    />
  );
}
```

Interface 的价值：
- ✅ IDE 自动提示：输入 `<Card` 时，IDE 会列出所有 props
- ✅ 类型检查：漏传必需字段会立即报错
- ✅ 文档作用：interface 就是最好的文档

---

### children prop — 组件嵌套

`children` 是 React 中的特殊 prop，用于组件嵌套：

```tsx
// 接收子内容
function Card({ children }) {
  return (
    <div className="card">
      {children}  {/* 渲染父组件传入的子内容 */}
    </div>
  );
}

// 使用：标签之间的内容就是 children
<Card>
  <h2>标题</h2>
  <p>内容</p>
</Card>
```

**children 的关键点**：
- `children` 是保留的 prop 名称，不用显式传递
- 可以是任何 JSX 元素、文本、或组件
- 用于组件嵌套和组合

---

### Props 的展开语法

传递多个 props 时用展开语法：

```tsx
function UserCard({ name, age, email }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>年龄：{age}</p>
      <p>邮箱：{email}</p>
    </div>
  );
}

function App() {
  const user = {
    name: '张三',
    age: 25,
    email: 'zhang@example.com'
  };

  // 等价于 <UserCard name={user.name} age={user.age} email={user.email} />
  return <UserCard {...user} />;
}
```

---

### 组件组合示例

当多个组件组合时，它们的 props 需要配合：

```tsx
// 1. 头像组件
interface AvatarProps {
  src: string;
  alt: string;
}

function Avatar({ src, alt }: AvatarProps) {
  return <img src={src} alt={alt} className="avatar" />;
}

// 2. 用户信息组件
interface UserInfoProps {
  avatar: { src: string; alt: string };
  name: string;
}

function UserInfo({ avatar, name }: UserInfoProps) {
  return (
    <div className="user-info">
      <Avatar {...avatar} />
      <h4>{name}</h4>
    </div>
  );
}

// 3. 使用组合
function UserProfile() {
  const user = {
    name: '张三',
    avatar: { src: '/avatar.jpg', alt: '张三的头像' }
  };

  return <UserInfo {...user} />;
}
```

**组合的关键点**：
- 每个组件的 interface 描述它需要什么数据
- 父组件负责把数据拆解成子组件需要的部分
- 用 `{...props}` 传递匹配的属性

---

### Props 与回调函数

子组件通过回调函数与父组件通信：

```tsx
interface ButtonProps {
  text: string;
  onClick: () => void; // 回调函数类型
}

function Button({ text, onClick }: ButtonProps) {
  return <button onClick={onClick}>{text}</button>;
}

// 使用
function Toolbar() {
  const handleClick = () => {
    console.log('按钮被点击');
  };

  return <Button text="点击我" onClick={handleClick} />;
}
```

**关键点**：
- 子组件不能直接修改父组件的数据
- 通过回调函数，子组件通知父组件"发生了什么"
- 父组件决定如何响应

---

## 进阶提示（可选阅读）

以下内容对初学者来说可能有点复杂，可以先跳过，等后续章节遇到再回来查看。

### props 对象转发

有时候不解构，直接使用 `props` 对象：

```tsx
// 转发所有 props 到子元素
function InputWrapper({ label, ...props }) {
  return (
    <div className="input-wrapper">
      <label>{label}</label>
      <input {...props} />  {/* 转发剩余的 props */}
    </div>
  );
}

// 使用
<InputWrapper
  label="用户名"
  type="text"
  placeholder="请输入用户名"
/>
```

### Props drilling（多层传递）

当多个层级需要传递数据时，会出现 props drilling：

```tsx
function Page({ user }) {
  return (
    <Layout user={user}>
      <Sidebar user={user} />
      <Content user={user} />
    </Layout>
  );
}
```

Props drilling 是数据传递的基础方式，但层级深时会冗长。后续章节会讨论更优的方案（如状态管理）。

---

## 你需要记住的

- Props 是父组件向子组件传递数据的单向机制
- Props 是只读的，子组件不能直接修改
- 用 TypeScript interface 定义 props 类型，提高代码安全性
- Interface 是组件的**契约**，告诉使用者组件需要什么输入
- 可选 props 用 `?` 标记，并提供默认值
- 展开语法 `{...obj}` 可传递多个 props
- children prop 用于组件嵌套和组合
- **Props 传递括号规则**：除了字符串，所有值都用 `{}` 包裹；对象用 `{{}}`

## AI 代码中的线索

以下模式表示正在使用 Props：

```tsx
// 1. 组件参数解构
function Component({ prop1, prop2 }) {}

// 2. 传递字符串（不用括号）
<Card title="Hello" />

// 3. 传递变量（用括号）
<Card count={userCount} />

// 4. 传递对象（双层括号）
<UserCard user={{ name: "张三", age: 25 }} />

// 5. 传递函数
<Button onClick={handleSubmit} />

// 6. 展开语法传递
<User {...userData} />

// 7. TypeScript props 类型
interface Props { name: string; }
function Card({ name }: Props) {}

// 8. children prop
function Wrapper({ children }) {
  return <div className="wrap">{children}</div>;
}
```

## 验证问题

- [ ] 为什么 Props 是只读的？
- [ ] 你能区分 `title="Hello"` 和 `title={title}` 的区别吗？
- [ ] 什么时候应该用 TypeScript 定义 Props 类型？
- [ ] children prop 的作用是什么？
