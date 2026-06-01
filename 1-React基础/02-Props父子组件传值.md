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

### Interface 与组件组合

Interface 不仅仅是类型检查工具，更是**组件契约**和**组合规则**的文档。良好的 interface 设计能让组件组合更清晰、更安全。

#### Interface 作为组件契约

**组件契约**是指组件对外承诺的输入输出规范：

```tsx
// 定义组件契约：告诉使用者"这个组件需要什么，承诺返回什么"
interface UserCardProps {
  // 必需字段：组件必须有这些数据才能正常工作
  user: {
    id: string;
    name: string;
    email: string;
  };
  
  // 可选配置：不提供也能工作，但可以自定义行为
  variant?: 'default' | 'compact' | 'detailed';
  showEmail?: boolean;
  
  // 回调函数：组件向外通信的通道
  onEdit?: (userId: string) => void;
  onDelete?: (userId: string) => void;
}

// 组件实现：按照契约接收数据
function UserCard({ user, variant = 'default', showEmail = true, onEdit, onDelete }: UserCardProps) {
  return (
    <div className={`user-card user-card--${variant}`}>
      <h3>{user.name}</h3>
      {showEmail && <p>{user.email}</p>}
      
      <div className="actions">
        {onEdit && <button onClick={() => onEdit(user.id)}>编辑</button>}
        {onDelete && <button onClick={() => onDelete(user.id)}>删除</button>}
      </div>
    </div>
  );
}

// 使用组件：必须满足契约要求
function UserList() {
  const users = [
    { id: '1', name: '张三', email: 'zhang@example.com' },
    { id: '2', name: '李四', email: 'li@example.com' }
  ];

  const handleEdit = (id: string) => {
    console.log('编辑用户:', id);
  };

  return (
    <div>
      {users.map(user => (
        <UserCard
          key={user.id}
          user={user}                              // ✅ 满足必需字段
          variant="detailed"                       // ✅ 提供可选配置
          showEmail={true}                          // ✅ 自定义显示
          onEdit={handleEdit}                       // ✅ 提供回调
          // onDelete 未提供，但契约中标记为可选，所以 ✅ 合法
        />
      ))}
    </div>
  );
}
```

**契约的价值**：
- ✅ IDE 自动提示：输入 `<UserCard` 时，IDE 会列出所有必需和可选的 props
- ✅ 类型检查：漏传必需字段会立即报错，而不是运行时才发现
- ✅ 文档作用：interface 就是最好的文档，告诉别人如何使用这个组件

#### 组件组合中的 Interface 协作

当多个组件组合时，它们的 interface 需要配合：

```tsx
// 1. 定义基础组件的接口
interface AvatarProps {
  src: string;
  alt: string;
  size?: 'small' | 'medium' | 'large';
}

function Avatar({ src, alt, size = 'medium' }: AvatarProps) {
  const sizeClass = `avatar-${size}`;
  return <img src={src} alt={alt} className={sizeClass} />;
}

// 2. 定义使用 Avatar 的组件接口
interface UserInfoProps {
  // 组合模式：UserInfo 需要 Avatar 需要的数据
  avatar: {
    src: string;
    alt: string;
  };
  name: string;
  role?: string;
}

function UserInfo({ avatar, name, role }: UserInfoProps) {
  return (
    <div className="user-info">
      {/* 组件组合：UserInfo 把数据传给 Avatar */}
      <Avatar {...avatar} size="small" />
      <div>
        <h4>{name}</h4>
        {role && <span className="role">{role}</span>}
      </div>
    </div>
  );
}

// 3. 更高层组件组合
interface UserProfileProps {
  user: {
    id: string;
    name: string;
    email: string;
    avatar: {
      src: string;
      alt: string;
    };
    role?: string;
  };
  onEdit?: (id: string) => void;
}

function UserProfile({ user, onEdit }: UserProfileProps) {
  return (
    <div className="user-profile">
      {/* 组件组合：把大对象拆解成各组件需要的部分 */}
      <UserInfo 
        avatar={user.avatar}    // 传递 avatar 部分
        name={user.name}         // 传递 name
        role={user.role}          // 传递 role
      />
      <p>{user.email}</p>
      {onEdit && (
        <button onClick={() => onEdit(user.id)}>编辑资料</button>
      )}
    </div>
  );
}
```

**组合时的数据流**：

```
UserProfile 接收完整 user 对象
    │
    ├─→ UserInfo 需要 avatar + name + role
    │       │
    │       └─→ Avatar 需要 src + alt
    │
    └─→ 直接显示 email
```

#### Interface 继承与组件扩展

通过 interface 继承，可以建立组件之间的类型关系：

```tsx
// 基础按钮接口
interface BaseButtonProps {
  children: React.ReactNode;
  disabled?: boolean;
}

// 扩展基础接口
interface ClickableButtonProps extends BaseButtonProps {
  onClick: () => void;
  variant?: 'primary' | 'secondary';
}

// 链接按钮接口
interface LinkButtonProps extends BaseButtonProps {
  href: string;
  target?: '_blank';
}

// 组合接口（联合类型）
type ButtonProps = ClickableButtonProps | LinkButtonProps;

function Button(props: ButtonProps) {
  // 类型守卫：区分不同类型的按钮
  if ('href' in props) {
    // LinkButtonProps
    return <a href={props.href} target={props.target}>{props.children}</a>;
  } else {
    // ClickableButtonProps
    const { onClick, variant = 'primary', children, disabled = false } = props;
    return (
      <button onClick={onClick} disabled={disabled}>
        {children}
      </button>
    );
  }
}
```

#### 组件组合的最佳实践

| 原则 | 说明 | 示例 |
|------|------|------|
| **单一职责** | 每个 interface 只描述一个组件的契约 | `UserCardProps` 只描述 `UserCard` 需要什么 |
| **可组合性** | 设计 props 时考虑如何与其他组件配合 | `UserInfoProps` 包含 `AvatarProps` 需要的字段 |
| **明确必需/可选** | 必需字段强制提供，可选字段提供默认值 | `user` 必需，`variant` 可选 |
| **类型复用** | 通过类型共享减少重复定义 | `type UserId = string;` 多个组件共用 |
| **渐进增强** | 基础组件简单，高层组件复杂 | `Avatar` → `UserInfo` → `UserProfile` |

#### 实战示例：卡片组件系统

```tsx
// 1. 定义通用类型
type CardVariant = 'default' | 'bordered' | 'shadow';
type CardSize = 'small' | 'medium' | 'large';

// 2. 基础卡片接口
interface BaseCardProps {
  children: React.ReactNode;
  variant?: CardVariant;
  size?: CardSize;
  className?: string;
}

// 3. 带标题的卡片接口
interface TitledCardProps extends BaseCardProps {
  title: string;
  subtitle?: string;
  action?: React.ReactNode;  // 标题右侧的操作按钮
}

// 4. 带头部的卡片接口
interface HeaderCardProps extends BaseCardProps {
  header: React.ReactNode;  // 自定义头部内容
  footer?: React.ReactNode;  // 自定义底部内容
}

// 5. 组合接口（联合类型）
type CardProps = BaseCardProps | TitledCardProps | HeaderCardProps;

// 6. 组件实现：根据不同 props 渲染不同结构
function Card(props: CardProps) {
  const baseClasses = 'card';
  
  if ('title' in props) {
    // TitledCardProps
    const { title, subtitle, action, children, variant = 'default', size = 'medium', className } = props;
    return (
      <div className={`${baseClasses} card--${variant} card--${size} ${className || ''}`}>
        <div className="card__header">
          <div>
            <h3>{title}</h3>
            {subtitle && <p>{subtitle}</p>}
          </div>
          {action && <div className="card__action">{action}</div>}
        </div>
        <div className="card__body">{children}</div>
      </div>
    );
  } else if ('header' in props) {
    // HeaderCardProps
    const { header, footer, children, variant = 'default', size = 'medium', className } = props;
    return (
      <div className={`${baseClasses} card--${variant} card--${size} ${className || ''}`}>
        <div className="card__header">{header}</div>
        <div className="card__body">{children}</div>
        {footer && <div className="card__footer">{footer}</div>}
      </div>
    );
  } else {
    // BaseCardProps
    const { children, variant = 'default', size = 'medium', className } = props;
    return (
      <div className={`${baseClasses} card--${variant} card--${size} ${className || ''}`}>
        {children}
      </div>
    );
  }
}

// 7. 使用示例
function Dashboard() {
  return (
    <div>
      {/* 基础卡片 */}
      <Card variant="bordered">
        <p>简单内容</p>
      </Card>

      {/* 带标题的卡片 */}
      <Card 
        title="用户统计" 
        subtitle="2024年1月"
        action={<button>导出</button>}
        variant="shadow"
      >
        <p>活跃用户：1,234</p>
      </Card>

      {/* 自定义头部的卡片 */}
      <Card
        header={
          <div className="custom-header">
            <img src="/logo.png" alt="Logo" />
            <h3>自定义标题</h3>
          </div>
        }
        footer={<button>查看详情</button>}
      >
        <p>卡片内容</p>
      </Card>
    </div>
  );
}
```

**关键点**：
- Interface 继承（`extends`）建立组件之间的类型关系
- 联合类型（`|`）让一个组件支持多种使用方式
- 类型守卫（`'title' in props`）区分不同的 props 结构
- 良好的 interface 设计让组件既灵活又类型安全

## 你需要记住的

- Props 是父组件向子组件传递数据的单向机制
- Props 是只读的，子组件不能直接修改
- 用 TypeScript interface 定义 props 类型，提高代码安全性
- Interface 是组件的**契约**，告诉使用者组件需要什么输入
- 组件组合时，不同组件的 interface 需要配合（数据结构匹配）
- 通过 interface 继承（`extends`）建立组件之间的类型关系
- 可选 props 用 `?` 标记，并提供默认值
- 展开语法 `{...obj}` 可传递多个 props
- Props drilling 是多层级数据传递的基础方式
- 良好的 props 设计让组件更可复用、更易组合

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
