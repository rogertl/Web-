# 阶段 3 综合练习

## 练习目标

让 AI 生成一个混合 Server Components 和 Client Components 的页面，你能快速识别：
1. 哪些组件是 Server，哪些是 Client
2. 数据在哪个环节获取
3. 交互在哪里处理
4. 边界划分是否合理

## 练习内容

让 AI 生成以下代码（直接复制提示词）：

```
生成一个 Next.js 14 App Router 页面，要求混合使用 Server Components 和 Client Components：

1. 创建一个博客文章页面 /blog/[slug]
2. 页面结构：
   - 外层 Server Component 获取文章数据和评论数据
   - 内层 Client Component 1：显示文章内容（支持Markdown渲染）
   - 内层 Client Component 2：评论列表（支持展开/收起）
   - 内层 Client Component 3：评论输入框（支持提交）
3. 数据获取：
   - 文章数据用 getPostBySlug(slug) 函数（模拟）
   - 评论数据用 getCommentsByPostId(postId) 函数（模拟）
4. 交互功能：
   - 评论列表支持展开/收起
   - 评论输入框支持提交（提交后重新获取评论）
5. 项目包含：
   - app/blog/[slug]/page.tsx（Server Component）
   - components/PostContent.tsx（Client Component）
   - components/CommentList.tsx（Client Component）
   - components/CommentForm.tsx（Client Component）
   - lib/posts.ts 和 lib/comments.ts（数据获取函数）

请按实际的项目结构生成完整代码。
```

## 你的任务

AI 生成代码后，不看任何文档，回答以下问题：

### 任务 1：组件类型识别

1. `app/blog/[slug]/page.tsx` 是 Server Component 还是 Client Component？为什么？
2. `PostContent.tsx` 为什么要标记 `'use client'`？如果不标记会怎样？
3. `CommentList.tsx` 可以改成 Server Component 吗？为什么？

### 任务 2：数据流追踪

4. 文章数据在哪里获取？服务端还是浏览器？
5. 评论数据在哪里获取？如果评论很多（比如 1000 条），这种方式有什么问题？如何优化？
6. 用户提交评论后，新评论如何显示？需要刷新页面吗？

### 任务 3：边界判断

7. 以下代码应该放在 Server Component 还是 Client Component？为什么？
   ```typescript
   export function formatDate(date: Date): string {
     return date.toLocaleDateString('zh-CN');
   }
   ```
8. 如果你想给文章添加"点赞"按钮（点击后点赞数 +1），应该把按钮放在哪里？
9. `getPostBySlug` 和 `getCommentsByPostId` 函数应该放在哪个文件夹？为什么？

### 任务 4：协作规则

10. Server Component 可以把函数作为 props 传给 Client Component 吗？为什么？
11. 如果 `CommentForm.tsx` 需要提交评论到服务器，应该用什么方式？（提示：阶段 4 会详细讲 Server Actions）
12. 为什么说"外层 Server，内层 Client"是最佳实践？

### 任务 5：性能优化

13. 如果文章页面访问量很大（比如每秒 1000 次请求），数据获取有什么优化方法？
14. 评论列表可以缓存吗？如果可以，应该用什么缓存策略？（静态/动态/ISR）
15. 如何减少客户端 JS 体积？（提示：边界划分）

## 达标标准

- ✅ 能一眼识别 Server Component 和 Client Component
- ✅ 能解释数据在哪里获取，如何流动
- ✅ 能判断交互应该在哪里处理
- ✅ 能理解边界划分对性能的影响
- ✅ 能说出 Server Components 数据获取的优势

## 下一步

完成这个练习后，进入阶段 4：交互与写操作（Server Actions）。
