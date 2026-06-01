# 1.5 React Router（集中式路由）

## 一句话
React Router 是 React 应用的路由库，通过 URL 变化控制显示不同组件，实现单页应用的多页面体验。

## 为什么需要它

不理解 React Router，当看到 AI 生成的代码中 URL 变化但页面不刷新、导航点击后内容切换但浏览器地址栏改变时，你会困惑：这是怎么实现的？为什么不需要服务器返回新页面？路由参数怎么获取？

路由是单页应用的核心。不掌握它，就无法理解多页面应用的导航逻辑，更无法处理动态路由（如 `/user/123`）和嵌套路由。

## 类比

React Router 就像大楼的电梯系统：按下不同楼层按钮（URL 变化）时，电梯（路由器）带你到对应楼层（显示不同组件），而不需要重新建造大楼（页面刷新）。

| 概念 | 类比 |
|------|------|
| 路由（Route） | 楼层（不同的楼层展示不同的内容） |
| 路径（Path） | 楼层按钮（指定要去哪个楼层） |
| 导航（Link/Navigation） | 电梯按钮（触发楼层切换） |
| 路由参数 | 房间号（楼层内的具体位置，如 `/user/123` 的 `123`） |
| 嵌套路由 | 大楼内的楼层分区（主楼层内有子区域） |

## 核心内容

### React Router 基础概念

React Router 是一个独立库，需要单独安装：

```bash
npm install react-router-dom
```

核心组件：
- `BrowserRouter`：路由容器，提供路由上下文
- `Routes` 和 `Route`：定义路径和组件的映射
- `Link` 或 `NavLink`：导航链接（不触发页面刷新）
- `useNavigate` hook：编程式导航

### 基本路由配置

```tsx
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
        <Link to="/contact">Contact</Link>
      </nav>

      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/contact" element={<Contact />} />
      </Routes>
    </BrowserRouter>
  );
}

function Home() {
  return <h1>Home Page</h1>;
}

function About() {
  return <h1>About Page</h1>;
}

function Contact() {
  return <h1>Contact Page</h1>;
}
```

### Link vs a 标签

```tsx
// ❌ 不好：使用 <a> 标签会触发页面刷新
function Nav() {
  return (
    <nav>
      <a href="/about">About</a>
    </nav>
  );
}

// ✅ 好：使用 Link 不刷新页面
function Nav() {
  return (
    <nav>
      <Link to="/about">About</Link>
    </nav>
  );
}
```

### 动态路由参数

```tsx
import { BrowserRouter, Routes, Route, Link, useParams } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<UserList />} />
        {/* :userId 是动态参数 */}
        <Route path="/user/:userId" element={<UserProfile />} />
      </Routes>
    </BrowserRouter>
  );
}

function UserProfile() {
  // 获取路由参数
  const { userId } = useParams<{ userId: string }>();

  return (
    <div>
      <h1>User Profile</h1>
      <p>User ID: {userId}</p>
    </div>
  );
}

function UserList() {
  const users = [1, 2, 3];

  return (
    <ul>
      {users.map((id) => (
        <li key={id}>
          {/* 构建动态链接 */}
          <Link to={`/user/${id}`}>User {id}</Link>
        </li>
      ))}
    </ul>
  );
}
```

### 编程式导航

使用 `useNavigate` hook 进行导航：

```tsx
import { useNavigate } from 'react-router-dom';

function LoginForm() {
  const navigate = useNavigate();

  const handleLogin = async (credentials) => {
    const success = await login(credentials);

    if (success) {
      // 登录成功后跳转
      navigate("/dashboard");
    } else {
      // 登录失败，跳转回登录页
      navigate("/login");
    }
  };

  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      handleLogin({ username, password });
    }}>
      {/* 表单内容 */}
    </form>
  );
}

// 也可以返回上一页
function BackButton() {
  const navigate = useNavigate();

  return (
    <button onClick={() => navigate(-1)}>
      Go Back
    </button>
  );
}
```

### 嵌套路由

```tsx
function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Layout />}>
          {/* 嵌套在 Layout 中的子路由 */}
          <Route index element={<Home />} />
          <Route path="about" element={<About />} />
          <Route path="products" element={<Products />}>
            {/* 更深的嵌套 */}
            <Route path=":id" element={<ProductDetail />} />
          </Route>
        </Route>
      </Routes>
    </BrowserRouter>
  );
}

function Layout() {
  return (
    <div>
      <header>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
        <Link to="/products">Products</Link>
      </header>

      {/* 子路由渲染位置 */}
      <Outlet />
    </div>
  );
}

function Products() {
  return (
    <div>
      <h1>Products</h1>
      {/* 产品的子路由会在这里渲染 */}
      <Outlet />
    </div>
  );
}
```

### 404 页面处理

```tsx
function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />

        {/* 404 页面：匹配所有未定义的路径 */}
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}

function NotFound() {
  const navigate = useNavigate();

  return (
    <div>
      <h1>404 - Page Not Found</h1>
      <button onClick={() => navigate("/")}>
        Go Home
      </button>
    </div>
  );
}
```

### React Router vs Next.js 路由

React Router 是前端路由，而 Next.js 使用文件系统路由：

| 特性 | React Router | Next.js 路由 |
|------|--------------|--------------|
| 路由定义 | 代码配置 | 文件系统（`app/` 目录） |
| 路由类型 | 纯前端路由 | 混合（前端+服务端） |
| 页面刷新 | 不刷新 | 可配置 |
| SEO | 需额外配置 | 原生支持 |

Next.js 的路由会在阶段 2 详细讲解。

## 你需要记住的

- React Router 是单页应用的前端路由库
- `Routes` 和 `Route` 定义路径-组件映射
- `Link` 或 `NavLink` 用于导航，不触发页面刷新
- `useParams` 获取动态路由参数（如 `/user/:userId`）
- `useNavigate` hook 进行编程式导航
- `Outlet` 渲染嵌套子路由
- `path="*"` 处理 404 页面
- React Router vs Next.js 路由：前端路由 vs 文件系统路由

## AI 代码中的线索

以下模式表示正在使用 React Router：

```tsx
// 1. 路由容器
<BrowserRouter>...</BrowserRouter>

// 2. 路由定义
<Route path="/about" element={<About />} />

// 3. 导航链接
<Link to="/dashboard">Dashboard</Link>

// 4. 动态路由参数
<Route path="/user/:userId" element={<User />} />

// 5. 获取路由参数
const { userId } = useParams();

// 6. 编程式导航
const navigate = useNavigate();
navigate("/dashboard");

// 7. 嵌套路由
<Outlet />
```

## 验证问题

- [ ] 为什么单页应用中应该使用 `Link` 而不是 `<a>` 标签？
- [ ] 动态路由参数（如 `/user/:userId`）如何在组件中获取？
- [ ] React Router 和 Next.js 的路由有什么本质区别？
