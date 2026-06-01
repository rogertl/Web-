# 1.8 React 渲染机制

## 一句话
React 用虚拟 DOM 和 diff 算法高效更新页面，协调过程决定哪些组件需要重新渲染。

## 为什么需要它

不理解 React 渲染机制，当看到组件"不更新"或"更新太慢"时，你会不知道问题出在哪里。为什么改了数据页面没反应？为什么一个简单的状态更新让整个页面卡顿？这些问题都源于对渲染机制的不理解。

React 渲染机制是理解性能优化、React.memo、useMemo 的基础。不掌握它，就无法写出高性能的 React 应用。

## 类比

- **虚拟 DOM** 就像餐厅的菜单：菜单上列出菜品（UI 结构），顾客点了菜（状态变化），厨房根据新菜单准备菜品（更新 DOM）
- **diff 算法** 就像厨师对比新旧菜单：只准备变动的菜品（只更新变化的 DOM 节点），而不是重新做所有菜
- **协调（Reconciliation）** 就像餐厅经理安排工作：决定哪些厨师（组件）需要重新工作，哪些可以休息
- **Fragment** 就像餐盘分隔器：一个餐盘可以放多个菜，但不需要额外的餐盘包装

| 概念 | 类比 |
|------|------|
| 虚拟 DOM | 餐厅菜单（UI 结构的轻量级表示） |
| 真实 DOM | 实际菜品（浏览器中的页面元素） |
| diff 算法 | 厨师对比新旧菜单（找出变化部分） |
| 协调 | 餐厅经理安排工作（决定更新策略） |
| 渲染周期 | 从点菜到上菜的全过程（render → diff → commit → 更新 DOM） |
| Fragment | 餐盘分隔器（包装多个元素而不增加额外 DOM） |

## 核心内容

### 虚拟 DOM（Virtual DOM）

虚拟 DOM 是真实 DOM 的轻量级 JavaScript 表示：

```tsx
// 真实 DOM（浏览器中的元素）
<div id="app">
  <h1>Hello</h1>
  <p>World</p>
</div>

// 虚拟 DOM（React 内部的对象表示）
{
  type: 'div',
  props: {
    id: 'app',
    children: [
      { type: 'h1', props: { children: 'Hello' } },
      { type: 'p', props: { children: 'World' } }
    ]
  }
}
```

虚拟 DOM 的优势：
- **轻量**：只是普通 JavaScript 对象，创建和修改成本低
- **批量更新**：多个状态变化可以合并为一次 DOM 更新
- **跨浏览器**：屏蔽浏览器差异，统一处理

### diff 算法的基本原理

React 用 diff 算法对比新旧虚拟 DOM，找出最小变化集：

```tsx
// 旧虚拟 DOM
const oldVNode = {
  type: 'div',
  props: {
    children: [
      { type: 'p', props: { children: 'A' } },
      { type: 'p', props: { children: 'B' } },
      { type: 'p', props: { children: 'C' } }
    ]
  }
};

// 新虚拟 DOM
const newVNode = {
  type: 'div',
  props: {
    children: [
      { type: 'p', props: { children: 'A' } },
      { type: 'p', props: { children: 'B' } }, // 没变
      { type: 'p', props: { children: 'X' } }  // 变了
    ]
  }
};

// diff 结果：只更新第三个子节点
```

diff 算法的核心假设（优化策略）：
1. **不同类型元素**：直接替换整个子树
   ```tsx
   // 旧
   <div>...</div>

   // 新（类型不同，整个替换）
   <span>...</span>
   ```

2. **相同类型元素**：只更新属性
   ```tsx
   // 旧
   <div className="old">Content</div>

   // 新（只更新 className）
   <div className="new">Content</div>
   ```

3. **子元素对比**：用 key 识别移动
   ```tsx
   // 没有 key：按顺序对比，效率低
   [<li>A</li>, <li>B</li>, <li>C</li>]

   // 有 key：通过 key 追踪元素，效率高
   [<li key="a">A</li>, <li key="b">B</li>, <li key="c">C</li>]
   ```

### 协调（Reconciliation）

协调是 React 决定哪些组件需要更新的过程：

```tsx
function App() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <Header />
      <Counter count={count} />
      <Footer />
    </div>
  );
}

// 状态更新时：
// 1. React 触发协调
// 2. 对比新旧虚拟 DOM 树
// 3. 决定只更新 Counter 组件（Header 和 Footer 不变）
// 4. 只调用 Counter 的 render 方法
```

协调的关键点：
- **自上而下**：从根组件开始递归对比
- **可跳过**：未变化组件的 render 被跳过
- **批量处理**：多个状态变化合并为一次协调

### 渲染周期（Render → Diff → Commit → 更新 DOM）

完整渲染周期分为四个阶段：

```tsx
function Counter() {
  const [count, setCount] = useState(0);

  console.log('1. Render 阶段：生成虚拟 DOM');
  const handleClick = () => {
    setCount(count + 1); // 触发状态更新
  };

  console.log('2. Diff 阶段：对比新旧虚拟 DOM');
  // React 内部执行 diff 算法

  console.log('3. Commit 阶段：确定最终变更');
  // React 计算出最小变更集

  console.log('4. 更新 DOM 阶段：应用到真实 DOM');
  // React 更新真实 DOM

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClick}>Increment</button>
    </div>
  );
}
```

各阶段详解：

1. **Render 阶段**：
   - 计算组件新的虚拟 DOM
   - 纯计算，不修改真实 DOM
   - 可被中断（时间切片）

2. **Diff 阶段**：
   - 对比新旧虚拟 DOM
   - 计算最小变更集

3. **Commit 阶段**：
   - 确定最终的 DOM 操作
   - 不可中断，必须一次性完成

4. **更新 DOM 阶段**：
   - 执行真实 DOM 操作
   - 用户看到页面变化

### Fragment 的原理和使用场景

Fragment 让组件返回多个元素而不包装额外 DOM：

```tsx
// ❌ 不好：额外包装 div
function Table() {
  return (
    <div>
      <tr><td>A</td></tr>
      <tr><td>B</td></tr>
      <tr><td>C</td></tr>
    </div>
  );
}

// ✅ 好：Fragment 不产生额外 DOM
function Table() {
  return (
    <>
      <tr><td>A</td></tr>
      <tr><td>B</td></tr>
      <tr><td>C</td></tr>
    </>
  );
}

// 等价写法
import { Fragment } from 'react';

function Table() {
  return (
    <Fragment>
      <tr><td>A</td></tr>
      <tr><td>B</td></tr>
      <tr><td>C</td></tr>
    </Fragment>
  );
}
```

Fragment 的优势：
- **语义化**：保持 HTML 结构正确性（如 table、tr 必须直接嵌套）
- **减少 DOM 层级**：避免不必要的包装元素
- **样式隔离**：包装元素可能干扰布局

带 key 的 Fragment：

```tsx
function List({ items }) {
  return (
    <>
      {items.map(item => (
        <Fragment key={item.id}>
          <dt>{item.title}</dt>
          <dd>{item.description}</dd>
        </Fragment>
      ))}
    </>
  );
}
```

### 渲染性能优化建议

基于渲染机制的性能优化：

```tsx
// ❌ 不好：整个列表重新渲染
function List({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          <ExpensiveComponent data={item} />
        </li>
      ))}
    </ul>
  );
}

// ✅ 好：用 React.memo 跳过未变化组件
const ExpensiveComponent = React.memo(function ({ data }) {
  // 复杂计算
  return <div>{/* 渲染逻辑 */}</div>;
});
```

## 你需要记住的

- 虚拟 DOM 是真实 DOM 的轻量级 JavaScript 表示，修改成本低
- diff 算法对比新旧虚拟 DOM，只更新变化的部分（关键假设：同类型元素只更新属性，子元素用 key 追踪）
- 协调是 React 决定哪些组件需要更新的过程，自上而下递归对比
- 渲染周期：render（生成虚拟 DOM）→ diff（对比变化）→ commit（确定变更）→ 更新 DOM（应用变更）
- Fragment 让组件返回多个元素而不产生额外 DOM 节点，用于保持语义化和减少层级
- 性能优化核心：减少不必要的协调和渲染（React.memo、useMemo、正确的 key）

## AI 代码中的线索

以下模式表示正在使用 React 渲染机制相关代码：

```tsx
// 1. 虚拟 DOM 结构（JSX 返回的对象）
function Component() {
  return (
    <div className="container">
      {/* JSX 会被编译成虚拟 DOM 对象 */}
    </div>
  );
}

// 2. key 的使用（diff 算法的优化）
{items.map(item => (
  <Item key={item.id} data={item} /> // key 帮助 diff 算法追踪元素
))}

// 3. Fragment 的使用
<>
  <Header />
  <Main />
  <Footer />
</>

// 4. React.memo（跳过不必要的渲染）
export const MemoizedComponent = React.memo(function Component({ data }) {
  return <div>{/* 组件内容 */}</div>;
});

// 5. 渲染副作用（在 commit 阶段后执行）
useEffect(() => {
  // DOM 更新后执行
  console.log('DOM 已更新');
}, [dependency]);
```

## 验证问题

- [ ] 虚拟 DOM 是什么？为什么需要它？它与真实 DOM 有什么区别？
- [ ] diff 算法的基本原理是什么？为什么需要给列表元素添加 key？
- [ ] React 的完整渲染周期包含哪几个阶段？每个阶段做了什么？
- [ ] Fragment 是什么？什么情况下应该使用 Fragment 而不是 div 包装？
