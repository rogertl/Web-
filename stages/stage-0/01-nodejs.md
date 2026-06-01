# 0.1 Node.js：什么是运行时，为什么需要它

## 一句话

**Node.js 让你的电脑能直接运行 JavaScript，而不需要浏览器。**

## 为什么需要它

你平时在浏览器里打开网页，网页里的 JavaScript 是浏览器负责运行的。但如果你想在电脑上运行一个 JS 文件（比如 `node server.js`），浏览器帮不了你——你需要一个**脱离浏览器的 JavaScript 运行环境**。

这就是 Node.js 干的事。

## 类比

| 概念 | 类比 |
|------|------|
| JavaScript | 一门语言（比如英语） |
| 浏览器 | 一个翻译官，能在网页上帮你翻译英语 |
| Node.js | 另一个翻译官，能在终端/服务器上帮你翻译英语 |

同一门语言，换个翻译官，就能在不同地方用了。

## 核心内容

### 最基本的用法

```bash
# 检查是否安装了 Node.js
node -v
# 输出类似：v20.11.0

# 直接运行一个 JS 文件
node hello.js
```

`node` 命令就是调用 Node.js 这个"翻译官"，让它把 `hello.js` 里的 JavaScript 代码翻译并执行。这一步不需要浏览器。

### Node.js 和浏览器有什么不同

虽然两者都能运行 JavaScript，但能做的事情不一样：

| 能力 | 浏览器 | Node.js |
|------|--------|---------|
| 操作网页（DOM） | ✅ | ❌ |
| 响应用户点击 | ✅ | ❌ |
| 读写本地文件 | ❌（安全限制） | ✅ |
| 创建服务器、监听端口 | ❌ | ✅ |
| 连接数据库 | ❌ | ✅ |

**为什么不同**：浏览器是给用户用的，出于安全考虑不允许直接操作用户电脑的文件系统。Node.js 是给开发者用的，运行在你自己的电脑或服务器上，没有这些限制。

### 你每天都在用 Node.js，只是不知道

当你执行这些日常命令时，底层都是 Node.js 在工作：

```bash
npm run dev
```

拆开看这行命令：
- `npm` — Node.js 自带的**包管理工具**（0.2 会详细讲）
- `run` — 告诉 npm "执行一个脚本"
- `dev` — 脚本名称，定义在 `package.json` 的 `scripts` 字段里（0.2 会详细讲）

所以 `npm run dev` 的本质是：**npm 读取 package.json 中 dev 脚本的定义，然后用 Node.js 执行对应的命令**。没有 Node.js，这行命令根本跑不起来。

```bash
pnpm install
```

`pnpm` 是另一种包管理工具（0.2 会详细讲），同样依赖 Node.js 运行。它帮你从网上下载项目需要的第三方库。

### Node.js 在 Next.js 项目中的角色

Next.js 是一个全栈框架（阶段 2 会详细讲），它的运行依赖 Node.js 完成以下工作：

| 环节 | Node.js 做什么 |
|------|----------------|
| 开发时 | 启动本地服务器、编译代码、热更新 |
| 构建时 | 把 TypeScript 编译成 JavaScript、打包优化 |
| 运行时 | 执行服务端代码（Server Components，阶段 3 会详细讲） |

## 你需要记住的

- `node` 命令 = 调用 Node.js 运行 JavaScript
- `npm`/`pnpm` 这些工具本身就是用 Node.js 运行的，没有 Node.js 它们无法工作
- `npm run dev` 的本质是：npm 读 package.json → 找到 dev 脚本 → 用 Node.js 执行
- Node.js ≠ 浏览器，浏览器操作网页，Node.js 操作服务器/文件系统

## AI 代码中的线索

当你看到 AI 生成的代码里有这些，说明这段代码运行在 Node.js 环境（不是浏览器）：

```js
import fs from 'fs'              // 文件系统，只有 Node.js 有
import path from 'path'          // 路径处理，只有 Node.js 有
export async function GET() {}   // Next.js 服务端函数，运行在 Node.js
```

反过来，如果看到 `document.getElementById` 或 `window.location`，说明代码运行在浏览器里。

## 验证问题

- [ ] Node.js 和浏览器都能运行 JavaScript，它们能做的事情一样吗？各能做什么对方做不了的？
- [ ] `npm run dev` 这行命令中，`dev` 是从哪里来的？为什么执行这行就能启动开发服务器？
- [ ] 看到 `import fs from 'fs'` 这行代码，你能判断它运行在哪个环境吗？为什么？
