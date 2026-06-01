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

## 实际使用场景

```
# 检查是否安装了 Node.js
node -v
# 输出类似：v20.11.0

# 直接运行一个 JS 文件
node hello.js
```

在 Next.js 项目中，Node.js 的作用：
1. **启动开发服务器**（`npm run dev` 底层就是 Node.js 在执行）
2. **安装依赖包**（npm/pnpm 本身就是用 Node.js 运行的）
3. **构建项目**（把你的代码编译成浏览器能运行的文件）
4. **运行服务端代码**（Next.js 的 Server Components 在 Node.js 环境中执行）

## 你需要记住的

- 看到 `node` 命令，就知道是在运行 JavaScript
- Next.js 项目的**开发、构建、运行**都依赖 Node.js
- Node.js ≠ 浏览器，它俩是不同的运行环境，能做的事情不同
  - 浏览器：操作网页（DOM）、响应用户点击
  - Node.js：读写文件、连接数据库、创建服务器

## AI 代码中的线索

当你看到 AI 生成的代码里有这些，说明这段代码运行在 Node.js 环境：

```js
import fs from 'fs'              // 文件系统，只有 Node.js 有
import path from 'path'          // 路径处理，只有 Node.js 有
export async function GET() {}   // Next.js 服务端函数，运行在 Node.js
```

## 验证问题

- [ ] Node.js 和浏览器都能运行 JavaScript，它们能做的事情一样吗？
- [ ] 看到 `import fs from 'fs'` 这行代码，你能判断它运行在哪个环境吗？
- [ ] 为什么 Next.js 项目必须安装 Node.js？
