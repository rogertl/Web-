# 6.2 认证系统（NextAuth / Clerk）

## 一句话

认证系统验证用户身份，NextAuth（现 Auth.js）是开源方案，Clerk 是托管服务，都提供 Session 管理、OAuth 登录、权限控制。

## 为什么需要它

没有认证系统，每个页面都要自己验证用户、管理 Session、处理登录流程。认证库统一处理这些，你只需配置即可。

## 类比

把认证系统想象成"门禁卡系统"：

| 概念 | 类比 |
|------|------|
| 用户 | 居民（需要进入小区） |
| Session/Token | 门禁卡（刷卡验证身份） |
| 认证系统 | 门禁系统（验证卡片的真实性） |
| OAuth | 联合门禁（用微信、支付宝等第三方身份） |
| 权限控制 | 权限等级（业主卡 vs 访客卡） |

有了门禁系统，每栋楼不需要单独验证居民身份。

## 核心内容

### 认证系统的核心功能

| 功能 | 说明 |
|------|------|
| 用户注册 | 创建用户账户（邮箱/密码或 OAuth） |
| 用户登录 | 验证凭证（邮箱/密码或 OAuth） |
| Session 管理 | 维持登录状态（Cookie 或 JWT） |
| 权限验证 | 检查用户角色和权限 |
| OAuth 集成 | 支持 Google、GitHub 等第三方登录 |

### NextAuth（Auth.js）基本配置

```typescript
// app/api/auth/[...nextauth]/route.ts
import NextAuth from 'next-auth';
import GoogleProvider from 'next-auth/providers/google';

const handler = NextAuth({
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    }),
  ],
  pages: {
    signIn: '/login', // 自定义登录页
  },
});

export { handler as GET, handler as POST };
```

### 使用 Session

```typescript
// app/dashboard/page.tsx
import { getServerSession } from 'next-auth';
import { authOptions } from '@/app/api/auth/[...nextauth]/route';

export default async function DashboardPage() {
  const session = await getServerSession(authOptions);

  if (!session) {
    return <div>请先登录</div>;
  }

  return <div>欢迎，{session.user?.name}</div>;
}
```

## 你需要记住的

1. 认证系统管理用户身份和登录状态。
2. NextAuth 是开源方案，Clerk 是托管服务。
3. Session 通过 Cookie 或 JWT 维持。
4. OAuth 支持第三方登录（Google、GitHub 等）。
5. 权限控制检查用户角色和访问权限。
