# 6.6 部署方式与扩展思路

## 一句话

Next.js 项目推荐部署到 Vercel（官方平台），一键部署、自动 HTTPS、全球 CDN；Docker 适合自建服务器或特殊环境。

## 为什么需要它

本地开发的项目需要部署到线上才能让用户访问。部署方式影响成本、性能、可扩展性，选择合适的部署方式很重要。

## 类比

把部署想象成"开分店"：

| 方式 | 类比 |
|------|------|
| Vercel | 连锁店（总部统一管理，快速复制） |
| Docker | 自营店（自己装修、自己管理） |
| 自建服务器 | 自己盖楼（从地基开始，完全掌控） |

## 核心内容

### Vercel 部署

```bash
# 一键部署
npx vercel
```

**优势**：
- ✅ 零配置部署（连接 GitHub 自动部署）
- ✅ 全球 CDN（用户访问最近节点）
- ✅ 自动 HTTPS
- ✅ Serverless Functions（按请求计费）
- ✅ 预览环境（每个 PR 自动生成预览链接）

### Docker 部署

```dockerfile
# Dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build
CMD ["npm", "start"]
```

**优势**：
- ✅ 完全掌控（可以安装任何依赖）
- ✅ 适合自建服务器
- ❌ 需要手动运维

## 你需要记住的

1. 推荐用 Vercel 部署，零配置且免费额度充足。
2. Docker 适合特殊环境或完全掌控需求。
3. Vercel 自动 HTTPS、CDN、Serverless Functions。
4. Docker 需要手动运维，但灵活性强。
