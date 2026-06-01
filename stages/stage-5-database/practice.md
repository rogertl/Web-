# 阶段 5 综合练习：用户管理系统

## 练习目标

应用阶段 5 学到的知识，阅读并理解一个包含完整数据层的用户管理系统代码。

## 练习内容

让 AI 生成一个用户管理系统，包含以下功能：

### 必需功能（覆盖 100% 的概念）

1. **数据库层**
   - 使用 Prisma 定义用户模型
   - 数据库连接配置（使用 `server-only` 保护）
   - 环境变量配置（数据库连接字符串）

2. **数据验证**
   - 使用 zod 定义用户创建和更新的验证 Schema
   - 验证邮箱格式、姓名长度、年龄范围

3. **CRUD 操作**
   - `lib/queries/user.ts` - 查询函数（获取用户列表、获取单个用户）
   - `app/actions/user.ts` - Server Actions（创建、更新、删除用户）
   - `app/api/users/route.ts` - API 路由（获取列表、创建用户）

4. **错误处理与安全**
   - 自定义错误类（NotFoundError、ValidationError）
   - 统一的错误处理逻辑
   - SQL 注入防护（使用 Prisma）
   - XSS 防护（React 自动转义）

### 代码要求

请让 AI 生成以下文件结构：

```
your-app/
├── prisma/
│   └── schema.prisma              # 用户模型定义
├── lib/
│   ├── db.ts                      # Prisma Client（带 server-only）
│   ├── env.ts                     # 环境变量验证
│   ├── queries/
│   │   └── user.ts                # 查询函数
│   └── errors.ts                  # 自定义错误类
├── app/
│   ├── actions/
│   │   └── user.ts                # Server Actions
│   ├── api/
│   │   └── users/
│   │       ├── route.ts           # GET /api/users, POST /api/users
│   │       └── [id]/
│   │           └── route.ts       # GET/PUT/DELETE /api/users/:id
│   └── users/
│       ├── page.tsx               # 用户列表页面（使用查询函数）
│       └── [id]/
│           └── page.tsx           # 用户详情页面
└── .env.local                     # 环境变量配置
```

## 数据流讲解任务

生成代码后，用你自己的话讲解以下数据流：

### 1. 用户列表页面
```
浏览器请求 → Server Component 执行 → 调用 lib/queries/user.ts →
Prisma Client 查询数据库 → 返回数据 → 渲染页面 → 返回 HTML
```

### 2. 创建用户表单
```
用户填写表单 → 提交到 Server Action → zod 验证数据 →
验证通过 → Prisma Client 创建记录 → 重新验证缓存 → 重定向到用户列表
```

### 3. API 创建用户
```
第三方应用 POST /api/users → Route Handler 接收请求 →
zod 验证 → Prisma Client 创建记录 → 返回 JSON 响应
```

## 验证问题

- [ ] 数据库连接代码为什么要放在 `lib/db.ts` 中，并用 `server-only` 保护？
- [ ] 环境变量 `DATABASE_URL` 为什么不应该暴露到客户端？
- [ ] Prisma 如何防止 SQL 注入？如果不用 Prisma 会怎样？
- [ ] zod 验证在什么时候执行？验证失败会发生什么？
- [ ] `lib/queries/`、Server Actions、Route Handlers 这三种位置分别适用于什么场景？
- [ ] 如果用户提交了无效的邮箱格式，错误信息如何传递到前端？
- [ ] React 如何防止 XSS 攻击？在哪些情况下需要手动清理 HTML？
- [ ] 为什么要使用自定义错误类而不是直接抛出普通 Error？

## 达标标准

完成这个练习后，你应该能够：

1. 看懂 AI 生成的任意 Next.js 应用的数据层代码
2. 知道如何查找数据库操作的代码位置
3. 理解数据验证、错误处理、安全防护的作用和实现方式
4. 能够解释数据从浏览器到数据库再返回的完整流程
5. 能够识别不安全的代码模式（如 SQL 注入风险、信息泄露）

## 下一步

完成这个练习后，你已准备好学习阶段 6：架构进阶与生产考虑。在阶段 6 中，我们将学习认证系统、缓存策略、部署方式等更高级的主题。
