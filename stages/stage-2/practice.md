# 阶段 2 综合练习

## 练习目标

让 AI 生成一个 Next.js 项目结构，你能快速定位：
1. 每个路由对应的文件位置
2. 数据在哪个环节加载
3. 布局如何嵌套

## 练习内容

让 AI 生成以下代码（直接复制提示词）：

```
生成一个 Next.js 14 App Router 项目结构，要求：
1. 有一个首页 (/)
2. 有一个仪表盘模块 (/dashboard)，包含：
   - 仪表盘首页 (/dashboard)
   - 设置页面 (/dashboard/settings)
   - 用户列表页面 (/dashboard/users)
   - 用户详情页面 (/dashboard/users/[userId])
3. 仪表盘模块有一个共享布局，包含导航栏
4. 用户详情页面从服务端获取数据（用一个模拟函数 getUserById）
5. 项目包含以下文件夹：app/, components/, lib/, public/

按实际的项目结构生成代码，包括：
- 每个 page.tsx 文件
- layout.tsx 文件（根布局 + 仪表盘布局）
- lib/users.ts（模拟数据查询）
- 完整的文件夹结构
```

## 你的任务

AI 生成代码后，不看任何文档，回答以下问题：

### 任务 1：路由定位

1. 用户访问 `/dashboard/users/123`，Next.js 会加载哪个文件？
2. 如果我想在所有页面都显示一个页脚，应该修改哪个文件？
3. `/dashboard` 和 `/dashboard/settings` 页面会显示同一个导航栏吗？为什么？

### 任务 2：数据流追踪

4. `getUserById(123)` 在哪个环节执行？浏览器还是服务端？
5. 如果 `getUserById` 返回 `null`，用户会看到什么？
6. 模拟一个数据加载出错的情况，应该添加哪个文件来优化用户体验？

### 任务 3：结构判断

7. 以下代码应该放在哪个文件夹？
   ```typescript
   export function formatEmail(email: string): string {
     return email.toLowerCase().trim();
   }
   ```
8. 如果我想创建一个可复用的"用户卡片"组件，应该在哪个文件夹创建文件？
9. 图片 `logo.png` 放在哪个文件夹，才能通过 `/images/logo.png` 访问？

### 任务 4：加载流程

10. 从用户输入 URL 到看到页面，列出 5 个关键步骤。
11. 客户端导航（Link 组件）和浏览器刷新的加载流程有什么不同？
12. 为什么 Next.js 首屏加载比纯 React SPA 快？

## 达标标准

- ✅ 能在 2 分钟内找到任何路由对应的文件
- ✅ 能解释数据在哪个环节加载，出错时如何处理
- ✅ 能判断代码应该放在 `components/` 还是 `lib/`
- ✅ 能说出服务端渲染和客户端渲染的加载差异

## 下一步

完成这个练习后，进入阶段 3：Server Components 与现代渲染模型（Next.js 灵魂）。
