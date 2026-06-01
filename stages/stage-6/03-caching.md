# 6.3 缓存与渲染策略（Static / Dynamic / ISR）

## 一句话

缓存决定页面何时重新生成，Static 永不更新、Dynamic 每次请求都更新、ISR 按时间间隔更新，用 `revalidate` 配置。

## 为什么需要它

没有缓存，每次请求都重新生成页面，服务器压力大、响应慢。缓存让页面预先生成并存储，用户请求时直接返回，速度快且成本低。

## 类比

把缓存想象成"预制菜"：

| 策略 | 类比 | 说明 |
|------|------|------|
| Static | 罐头（永不改） | 构建时生成一次，永久不变 |
| Dynamic | 现炒（每次现做） | 每次请求都重新生成 |
| ISR（增量静态再生成） | 冷藏预制（定期更新） | 按时间间隔重新生成 |

## 核心内容

| 策略 | 配置 | 适用场景 |
|------|------|---------|
| Static | `export const dynamic = 'force-static'` | 静态内容（如文档、首页） |
| Dynamic | `export const dynamic = 'force-dynamic'` 或默认 | 动态内容（如用户数据、实时数据） |
| ISR | `export const revalidate = 60`（秒） | 定期更新的内容（如博客、新闻） |

## 你需要记住的

1. Static 永不更新，Dynamic 每次更新，ISR 按时间更新。
2. 用 `dynamic = 'force-static'` 或 `revalidate` 配置。
3. Static 适合不常变的内容，Dynamic 适合实时数据。
4. ISR 是 Static 和 Dynamic 的折中方案。
