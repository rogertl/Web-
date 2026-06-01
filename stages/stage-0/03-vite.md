# 0.3 Vite：开发时的构建工具（热更新 HMR）

## 一句话

**Vite 让你改代码后浏览器立刻更新，不用手动刷新页面。**

## 它解决什么问题

没有构建工具的世界：
1. 你写了一个 React 组件
2. 需要手动编译成浏览器能理解的 JS/CSS
3. 刷新浏览器看效果
4. 改一行代码 → 再编译 → 再刷新 → 再看

有了 Vite：
1. 你改了代码
2. **保存的瞬间**，浏览器自动更新对应部分
3. 不用刷新，状态保留，即时看到效果

这就是 **HMR（Hot Module Replacement，热模块替换）**。

## 类比

| 概念 | 类比 |
|------|------|
| 没有 Vite | 改设计稿 → 重新印刷整本杂志 → 再翻到那一页 |
| 有 Vite | 改设计稿 → 那一页自动更新，其他页不动 |

## Vite vs Next.js 的构建工具

| | Vite | Next.js（内置 Turbopack/Webpack） |
|---|---|---|
| 定位 | 纯前端开发构建工具 | 全栈框架，自带构建能力 |
| 适用场景 | React SPA、Vue 等纯前端项目 | 需要服务端渲染的全栈项目 |
| AI 项目中 | 较少出现在 Next.js 项目中 | Next.js 项目默认使用 |

**重要**：Next.js 项目**不需要 Vite**，它有自己的构建工具。但很多 AI 生成的纯前端项目（比如一个单独的 React 页面）会用 Vite。两个都认识即可。

## 你需要记住的

1. 看到 `vite.config.ts` → 这是一个 Vite 项目
2. 看到 `next.config.js` → 这是一个 Next.js 项目（不用 Vite）
3. HMR = 改代码后自动更新，开发时的核心体验
4. **构建工具只在开发时重要**，最终用户看到的是编译后的产物

## AI 代码中的线索

```js
// vite.config.ts — Vite 项目配置文件
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
})
```

```js
// next.config.js — Next.js 项目配置文件
/** @type {import('next').NextConfig} */
const nextConfig = {}
module.exports = nextConfig
```

看到哪个配置文件，就知道用的什么构建体系。

## 验证问题

- [ ] HMR 解决了什么问题？没有它开发体验会怎样？
- [ ] 为什么 Next.js 项目不需要 Vite？
- [ ] 你打开一个项目看到了 `vite.config.ts`，这说明什么？
