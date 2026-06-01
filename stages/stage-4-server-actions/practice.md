# 阶段 4 综合练习

## 练习目标

让 AI 生成一个包含 Server Actions 和 Route Handlers 的完整交互页面，你能识别：
1. 什么时候用 Server Actions，什么时候用 Route Handlers
2. 表单提交和数据修改的完整流程
3. API 路由的创建和调用

## 练习内容

让 AI 生成以下代码（直接复制提示词）：

```
生成一个 Next.js 14 交互功能演示，包含 Server Actions 和 Route Handlers：

1. 创建一个待办事项（Todo）应用：
   - 页面：/todos
   - 功能：添加、删除、标记完成

2. Server Actions 部分：
   - 添加待办的表单（用 Server Action）
   - 删除待办的按钮（用 Server Action）
   - 切换完成状态的按钮（用 Server Action）
   - 所有 Server Actions 放在 app/actions.ts

3. Route Handlers 部分：
   - GET /api/todos - 获取所有待办
   - POST /api/todos - 创建待办（作为备用，供第三方调用）
   - PUT /api/todos/[id] - 更新待办状态
   - DELETE /api/todos/[id] - 删除待办

4. 项目包含：
   - app/todos/page.tsx（Server Component，获取数据）
   - components/TodoList.tsx（Client Component，显示列表）
   - components/AddTodoForm.tsx（Client Component，添加表单）
   - app/actions.ts（Server Actions）
   - app/api/todos/route.ts（Route Handlers）
   - lib/todos.ts（数据查询函数）

请按实际的项目结构生成完整代码。
```

## 达标标准

- ✅ 能区分 Server Actions 和 Route Handlers 的使用场景
- ✅ 能解释表单提交的完整流程
- ✅ 能创建和调用 REST API
- ✅ 理解服务端交互的安全性考虑

## 下一步

完成这个练习后，进入阶段 5：数据层与后端逻辑。
