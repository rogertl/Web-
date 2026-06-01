# 0.5 项目启动流程（npm run dev 背后发生了什么）

## 一句话

**你输入 `npm run dev`，背后是一套链条：读配置 → 启动 Node.js → 编译代码 → 开启本地服务器 → 打开浏览器。**

## 完整链路

```
你在终端输入
    │
    ▼
npm run dev
    │
    ▼
读取 package.json 的 scripts.dev → "next dev"
    │
    ▼
执行 next dev（Node.js 进程启动）
    │
    ├── 1. 加载 next.config.js 配置
    ├── 2. 扫描 app/ 目录结构，建立路由表
    ├── 3. 启动开发服务器（默认 http://localhost:3000）
    ├── 4. 启用 HMR（文件变化时自动重新编译）
    └── 5. 编译 TypeScript → JavaScript
    │
    ▼
浏览器访问 localhost:3000
    │
    ▼
服务器收到请求
    │
    ├── 1. 匹配路由（URL → 对应的 page.tsx）
    ├── 2. 执行 Server Component 代码
    ├── 3. 获取数据（如果有数据库查询）
    ├── 4. 渲染 HTML
    └── 5. 发送给浏览器
    │
    ▼
浏览器显示页面
    │
    ├── 加载 HTML（页面骨架）
    ├── 加载 CSS（样式）
    ├── 加载 Client Component JS（交互逻辑）
    └── 页面可交互 ✅
```

## 每个环节的关键文件

| 环节 | 关键文件 | 作用 |
|------|----------|------|
| 配置 | `package.json` | 定义 scripts 命令 |
| 配置 | `next.config.js` | Next.js 自身配置 |
| 配置 | `tsconfig.json` | TypeScript 编译规则 |
| 路由 | `app/**/page.tsx` | URL 对应的页面 |
| 布局 | `app/layout.tsx` | 页面的公共外壳 |
| 入口 | `app/page.tsx` | 首页（访问 `/` 时显示） |

## 你改代码后发生了什么

```
你修改了 app/page.tsx 并保存
    │
    ▼
文件系统监听到变化（HMR）
    │
    ▼
Next.js 只重新编译被修改的模块
    │
    ▼
浏览器通过 WebSocket 收到更新通知
    │
    ▼
浏览器替换对应模块（不刷新整个页面）
    │
    ▼
你立刻看到更新后的效果 ✅
```

## 端口号

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

1. `npm run dev` → 启动开发服务器，支持热更新
2. `npm run build` → 编译生产版本（更快、更小）
3. `npm run start` → 运行编译后的生产版本
4. 开发时用 `dev`，部署时用 `build + start`

## 验证：你能回答这些问题吗

- [ ] `npm run dev` 实际上执行的是什么命令？
- [ ] 为什么改代码后浏览器会自动更新？
- [ ] `localhost:3000` 对应的是哪个文件？
- [ ] `package.json` 中的 `scripts` 字段有什么用？

如果都能回答，阶段 0 就通关了。
