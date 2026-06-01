# 0.5 项目启动流程（npm run dev 背后发生了什么）

## 一句话

**输入一行命令，代码变成网页。**

## 为什么需要它

你写完了代码，但代码只是文本文件。要让它在浏览器里变成可交互的网页，中间需要一套"加工链"。不了解这条链，出了问题你不知道该从哪里排查。

## 类比

| 概念 | 类比 |
|------|------|
| 你写的代码 | 原材料（生牛肉、面粉） |
| `npm run dev` | 按下厨房电闸，开始加工 |
| 构建工具 | 厨师：把原材料加工成菜品 |
| `localhost:3000` | 出餐窗口（你电脑上的 3000 号窗口） |
| HMR | 菜里盐放多了，厨师直接换一份，不用重新点菜 |

## 核心内容

### npm run dev 的完整解析

```
你在终端输入 npm run dev
    │
    ▼
1. npm 读取 package.json，找到 scripts.dev 的值："next dev"
    │
    ▼
2. npm 用 Node.js 执行 next dev
   （相当于运行：node node_modules/next/cli.js dev）
    │
    ▼
3. Next.js CLI 解析命令，识别出子命令是 dev
    │
    ▼
4. 读取 next.config.js 配置（Next.js 框架配置文件）
    │
    ▼
5. 扫描 app/ 目录结构，建立路由表
   （路由表 = URL 路径和页面文件的对照表，阶段 2 会详细讲）
    │
    ▼
6. 启动开发服务器 ← 这里需要展开讲（见下一节）
    │
    ▼
7. 启用 HMR（文件变化时自动重新编译，0.3 讲过原理）
    │
    ▼
8. 输出启动完成信息
   ✓ Ready in 2.3s
   ○ Local: http://localhost:3000
```

**关键**：`npm run dev` 只是快捷方式，真正干活的是 `next dev`。你直接在终端输入 `next dev` 也能启动（前提是项目已安装 next）。

### 启动服务：Next.js 如何创建服务器

**核心问题**：为什么一个命令就能让你的电脑变成"服务器"？

本质：Next.js 用 Node.js 的 `http` 模块在你的电脑上开了一个"门"（端口），等待外部连接。

#### 底层原理（简化版）

```js
// Next.js 启动代码的本质逻辑

// 1. 引入 Node.js 内置模块
const http = require('http')        // Node.js 自带的 HTTP 服务器模块
const url = require('url')          // 解析 URL 的模块

// 2. 创建服务器
const server = http.createServer((req, res) => {
  // req = 请求（浏览器发来的）
  // res = 响应（要返回给浏览器的）

  const pathname = url.parse(req.url).pathname  // 解析 URL 路径

  // 根据路由表找到对应的页面文件
  const page = matchRoute(pathname)

  // 执行页面代码，生成 HTML
  const html = renderPage(page)

  // 返回给浏览器
  res.writeHead(200, { 'Content-Type': 'text/html' })
  res.end(html)
})

// 3. 绑定端口（开"门"）
server.listen(3000, 'localhost', () => {
  console.log('服务器已启动，访问 http://localhost:3000')
})
```

#### 操作系统层面发生了什么

```
你执行 next dev
    │
    ▼
Next.js 请求操作系统："给我 3000 号端口"
    │
    ▼
操作系统检查：3000 号端口空闲吗？
    │
    ├── 空闲 → 操作系统把 3000 分配给 Next.js
    │        │
    │        └── Next.js 开始监听这个端口
    │           （等待有数据发到 3000）
    │
    └── 被占用 → 报错：Error: Port 3000 is already in use
```

**端口监听的本质**：在操作系统层面"申请一个门牌号"，然后守在那里等待消息。发到这个门牌号的消息，都会被你收到。

#### 浏览器访问时发生了什么

```
浏览器访问 http://localhost:3000
    │
    ▼
浏览器发一个请求到 3000 号端口："给我主页（/）"
    │
    ▼
Next.js 收到请求 → 匹配路由 → 执行 page.tsx → 生成 HTML
    │
    ▼
Next.js 把 HTML 发回浏览器
    │
    ▼
浏览器渲染页面 → 你看到网页 ✅
```

#### 开发服务器 vs 生产服务器

| | 开发服务器（`next dev`） | 生产服务器（`next start`） |
|---|------------------------|------------------------|
| 启动方式 | 每次重新编译 | 启动已编译好的代码 |
| 热更新 | 有（文件变了自动重编译） | 无（需重启服务） |
| 速度 | 启动快，响应稍慢 | 启动慢，响应快 |
| 优化 | 未优化（调试友好） | 已优化（压缩、tree-shaking） |
| 什么时候用 | 写代码时 | 部署到服务器后 |

**关键**：开发时用 `dev`（有热更新，方便调试），部署时用 `build + start`（更快、更安全）。

### 完整链路（从代码到网页）

```
启动完成（服务器已在 3000 端口监听）
    │
    ▼
浏览器访问 localhost:3000
    │
    ▼
服务器收到请求
    │
    ├── 1. 根据路由表匹配 URL → 找到对应的 page.tsx
    ├── 2. 执行 Server Component 代码（阶段 3 会详细讲）
    ├── 3. 获取数据（如果有数据库查询）
    ├── 4. 渲染 HTML
    └── 5. 发送给浏览器
    │
    ▼
浏览器显示页面
    │
    ├── 加载 HTML（页面骨架）
    ├── 加载 CSS（样式）
    ├── 加载 Client Component 的 JS（交互逻辑，阶段 3 会详细讲）
    └── 页面可交互 ✅
```

### localhost 和端口是什么

- **localhost** = 你自己的电脑（不是互联网上的某台服务器）。浏览器访问 localhost，等于在跟自己电脑上的程序通信
- **端口（Port）** = 程序的"门牌号"。一台电脑可以运行很多程序，端口用来区分哪个请求发给哪个程序。3000 是 Next.js 的默认端口

**端口的本质**：操作系统给程序分配的一个"通信通道"。程序监听某个端口，就等于在那个"门牌号"等着收消息。

```
你的电脑
├── 端口 3000 → Next.js 开发服务器（监听这个端口）
├── 端口 3001 → 另一个项目
└── 端口 8080 → 数据库（如果有）
```

**端口冲突**：如果 3000 已被占用，Next.js 会报错。解决方法：换端口或关掉占用端口的程序。

### 你改代码后发生了什么

```
你修改了 app/page.tsx 并保存
    │
    ▼
Next.js 的文件监听检测到变化
    │
    ▼
只重新编译被修改的模块（不是整个项目）
    │
    ▼
通过 WebSocket（一种浏览器和服务器之间的实时通信通道）
通知浏览器
    │
    ▼
浏览器只替换变化的模块（不刷新整个页面，保留当前状态）
    │
    ▼
你立刻看到更新后的效果 ✅
```

### 三个关键命令的区别

| 命令 | 作用 | 什么时候用 |
|------|------|-----------|
| `npm run dev` | 启动开发服务器，支持热更新 | 写代码时 |
| `npm run build` | 编译优化，生成生产版本 | 准备部署时 |
| `npm run start` | 运行编译后的生产版本 | 部署到服务器后 |

开发时用 `dev`（有热更新，方便调试），部署时用 `build + start`（更快、更安全）。

### 配置文件和执行文件

启动过程涉及两类文件：

#### 1. 框架配置文件（你不用改，但需要知道存在）

**`next.config.js`** — Next.js 框架的全局配置
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,     // React 严格模式（检查不安全的代码模式）
  swcMinify: true,           // 用 SWC 压缩代码（比 Terser 快）
  experimental: {             // 实验性功能开关
    serverActions: true      // 启用 Server Actions（阶段 4 会详细讲）
  }
}
module.exports = nextConfig
```

这个文件影响整个框架的行为，不是单个页面的配置。大多数项目用默认值即可。

**`tsconfig.json`** — TypeScript 编译规则（0.4 讲过 TypeScript）
```json
{
  "compilerOptions": {
    "target": "ES2020",        // 编译目标（ES 版本）
    "module": "ESNext",        // 模块系统
    "jsx": "preserve",         // JSX 处理方式
    "strict": true             // 严格模式检查
  }
}
```

#### 2. 执行文件（启动器 vs 你的代码）

**Next.js 启动器**（框架代码，在 `node_modules/` 里）：
```
node_modules/
└── next/
    ├── cli.js          ← 执行 `next dev` 时实际运行这个
    ├── lib/
    │   └── dev.js      ← cli.js 调用这个，启动开发服务器
    └── ...
```

**你的代码入口**（项目根目录的 `app/` 文件夹）：

| 文件 | 作用 | 什么时候执行 |
|------|------|-------------|
| `app/layout.tsx` | 全局布局（页面的公共外壳，如导航栏、页脚） | 每个页面渲染前 |
| `app/page.tsx` | 首页（访问 `/` 时显示） | 访问根路径时 |
| `app/**/page.tsx` | 其他页面（如 `app/about/page.tsx`） | 访问对应路径时 |

**关键区别**：
- `cli.js` 是框架的启动器（Next.js 提供，你不用管）
- `app/page.tsx` 是你写的代码入口（你的业务逻辑）

#### 启动时的文件读取顺序

```
1. next 读取 next.config.js（框架配置）
2. next 读取 tsconfig.json（TypeScript 规则）
3. next 扫描 app/ 目录（建立路由表）
4. next 启动服务器，等待请求
5. 浏览器访问时，执行对应的 page.tsx
```

### 端口号操作

```bash
# 默认端口
npm run dev          # → http://localhost:3000

# 指定端口
npm run dev -- -p 4000   # → http://localhost:4000

# 端口被占用时的报错
# Error: Port 3000 is already in use
# 解决：换一个端口，或者关掉占用 3000 的进程
```

## 你需要记住的

1. `npm run dev` 完整流程：npm 读 package.json → 找 scripts.dev → 用 Node.js 执行 `next dev`
2. `next dev` 的本质：用 Node.js 的 `http` 模块创建服务器，绑定端口，监听请求
3. `localhost` = 你自己的电脑，`3000` = 门牌号（端口），端口是操作系统分配的通信通道
4. 端口监听 = 程序向操作系统申请端口，然后守在那里等待消息
5. HMR 的链路：保存文件 → 检测变化 → 编译改动的部分 → WebSocket 通知 → 浏览器替换
6. 开发用 `dev`（有热更新），部署用 `build + start`（优化后运行）
7. `next.config.js` 是框架配置文件，`cli.js` 是框架启动器，`app/page.tsx` 是你的代码入口

## AI 代码中的线索

```bash
# AI 生成项目后常给你这些命令
npm run dev           # 启动开发
npm run build         # 构建生产版本
npm run start         # 运行生产版本
npm run lint          # 代码检查
pnpm dev              # pnpm 版本（等价于 pnpm run dev，0.2 讲过）
```

如果启动报错，AI 通常会建议你：
```bash
# 删掉依赖重装（解决 90% 的启动问题）
rm -rf node_modules
pnpm install
```

## 验证问题

- [ ] `npm run dev` 实际上执行的是什么命令？它是怎么找到要执行的命令的？
- [ ] `localhost` 是什么？`3000` 是什么？为什么需要端口号？
- [ ] 端口监听的原理是什么？程序如何向操作系统申请端口？
- [ ] 你改了代码保存后，浏览器是怎么知道要更新的？（说出完整的链路）
- [ ] `npm run dev` 和 `npm run build` 有什么区别？各在什么场景下使用？
- [ ] `next.config.js`、`cli.js`、`app/page.tsx` 这三个文件各有什么作用？哪个是框架代码，哪个是你的代码？
