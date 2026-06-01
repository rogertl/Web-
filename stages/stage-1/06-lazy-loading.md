# 1.6 懒加载与 Suspense

## 一句话
懒加载是延迟加载组件代码的技术，Suspense 是处理加载状态的机制。

## 为什么需要它

不理解懒加载和 Suspense，当看到 AI 生成的代码中组件用 `lazy()` 包装、外围有 `<Suspense>` 标签时，你会困惑：为什么不能直接 import 组件？Suspense 到底在等什么？fallback 显示什么内容？

懒加载是性能优化的核心手段。不掌握它，就无法理解如何减少初始加载时间、如何处理异步组件的加载状态。

## 类比

- **懒加载** 就像快递点自提：不是所有包裹一次性送到家，而是按需领取（只加载当前需要的组件代码）
- **Suspense** 就像快递点的"正在整理"告示牌：包裹还在整理时显示（fallback），整理好后替换为实际内容

| 概念 | 类比 |
|------|------|
| 懒加载（lazy） | 快递自提（按需领取，减少一次性配送） |
| 代码分割 | 把大包裹分成小包裹（减小单次配送量） |
| Suspense | "正在整理"告示牌（显示加载状态） |
| fallback | 临时占位内容（等待时显示的加载提示） |
| 动态 import | 需要时才去仓库取包裹（按需加载代码） |

## 核心内容

### 为什么需要懒加载

单页应用的所有代码通常打包成一个 JS 文件，随着项目增长会很大：

```tsx
// ❌ 不使用懒加载：所有组件代码都在一个文件
import { Home, About, Products, Contact } from './pages';

function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/about" element={<About />} />
      <Route path="/products" element={<Products />} />
      <Route path="/contact" element={<Contact />} />
    </Routes>
  );
}
```

问题：
- 首次加载需要下载所有组件代码
- 用户可能只访问一个页面，但下载了全部
- 初始加载慢，影响用户体验

### React.lazy：懒加载组件

`React.lazy` 动态导入组件，只在需要时加载：

```tsx
import { lazy, Suspense } from 'react';

// ✅ 使用懒加载：组件代码按需加载
const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Products = lazy(() => import('./pages/Products'));
const Contact = lazy(() => import('./pages/Contact'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/products" element={<Products />} />
        <Route path="/contact" element={<Contact />} />
      </Routes>
    </Suspense>
  );
}
```

关键点：
- `import('./pages/Home')` 是动态 import，返回 Promise
- `React.lazy()` 接收这个函数，返回一个懒加载组件
- 懒加载组件使用时必须包裹在 `<Suspense>` 中

### Suspense：处理加载状态

`Suspense` 的 `fallback` 属性指定加载时显示的内容：

```tsx
import { lazy, Suspense } from 'react';

const Dashboard = lazy(() => import('./pages/Dashboard'));

function App() {
  return (
    <Suspense
      fallback={
        <div className="loading-spinner">
          <span>Loading dashboard...</span>
        </div>
      }
    >
      <Dashboard />
    </Suspense>
  );
}
```

fallback 可以是任何 JSX：
- 简单文本：`<div>Loading...</div>`
- 加载动画：`<Spinner />`
- 骨架屏：`<Skeleton />`

### 代码分割（Code Splitting）

懒加载实现代码分割，构建工具（如 Webpack、Vite）会把懒加载的组件打包成单独的文件：

```tsx
// 不使用懒加载：所有代码在一个文件（bundle.js）
// 使用懒加载：代码分成多个文件
// - main.js（主文件）
// - home.js（Home 组件）
// - about.js（About 组件）
// - products.js（Products 组件）
```

浏览器只下载当前路由需要的文件，其他文件在导航到对应路由时才下载。

### 懒加载路由的最佳实践

```tsx
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

// 懒加载页面组件
const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Products = lazy(() => import('./pages/Products'));
const ProductDetail = lazy(() => import('./pages/ProductDetail'));

function LoadingFallback() {
  return (
    <div className="loading-container">
      <div className="spinner"></div>
      <p>Loading page...</p>
    </div>
  );
}

function App() {
  return (
    <BrowserRouter>
      <div className="app">
        <nav>
          <Link to="/">Home</Link>
          <Link to="/about">About</Link>
          <Link to="/products">Products</Link>
        </nav>

        <Suspense fallback={<LoadingFallback />}>
          <Routes>
            <Route path="/" element={<Home />} />
            <Route path="/about" element={<About />} />
            <Route path="/products" element={<Products />} />
            <Route path="/products/:id" element={<ProductDetail />} />
          </Routes>
        </Suspense>
      </div>
    </BrowserRouter>
  );
}
```

### 错误边界（Error Boundary）

懒加载组件加载失败时（如网络错误），需要错误边界处理：

```tsx
import { Component, ErrorInfo, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
}

class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Lazy load error:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || <div>Failed to load component.</div>;
    }

    return this.props.children;
  }
}

// 使用
function App() {
  return (
    <ErrorBoundary fallback={<div>Something went wrong.</div>}>
      <Suspense fallback={<div>Loading...</div>}>
        <LazyComponent />
      </Suspense>
    </ErrorBoundary>
  );
}
```

### 懒加载的适用场景

适合懒加载的场景：
- 路由级组件（页面组件）
- 大型组件（如复杂图表、编辑器）
- 低频使用的组件（如设置页、管理面板）

不适合懒加载的场景：
- 高频使用的小型组件（如按钮、输入框）
- 首屏必需的组件（会延迟首屏显示）

```tsx
// ✅ 适合懒加载
const AdminPanel = lazy(() => import('./pages/AdminPanel'));
const SettingsPage = lazy(() => import('./pages/Settings'));

// ❌ 不适合懒加载
// const Button = lazy(() => import('./components/Button')); // 太小
// const Input = lazy(() => import('./components/Input')); // 使用频繁
```

## 你需要记住的

- 懒加载（`React.lazy`）按需加载组件代码，减少初始加载时间
- 动态 `import()` 是懒加载的基础，返回 Promise
- 懒加载组件必须包裹在 `<Suspense>` 中
- `Suspense` 的 `fallback` 指定加载时显示的内容
- 懒加载实现代码分割，构建工具生成多个独立文件
- 错误边界处理懒加载失败的情况
- 适合懒加载：路由组件、大型组件、低频组件
- 不适合：高频小型组件、首屏必需组件

## AI 代码中的线索

以下模式表示正在使用懒加载和 Suspense：

```tsx
// 1. React.lazy 懒加载
const Component = lazy(() => import('./Component'));

// 2. Suspense 包裹懒加载组件
<Suspense fallback={<Loading />}>
  <LazyComponent />
</Suspense>

// 3. 动态 import
import('./pages/Home')

// 4. fallback 内容
<Suspense fallback={<div>Loading...</div>}>

// 5. 错误边界包裹
<ErrorBoundary>
  <Suspense fallback={...}>
    <LazyComponent />
  </Suspense>
</ErrorBoundary>
```

## 验证问题

- [ ] 为什么懒加载组件必须包裹在 `<Suspense>` 中？
- [ ] 懒加载是如何减少初始加载时间的？背后的原理是什么？
- [ ] 在什么情况下应该使用懒加载，什么情况下不应该？
