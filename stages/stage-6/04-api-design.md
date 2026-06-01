# 6.4 API 设计与文档

## 一句话

RESTful API 用标准 HTTP 方法（GET/POST/PUT/DELETE）操作资源，返回统一的响应格式，Swagger 自动生成文档。

## 为什么需要它

没有统一规范，API 各写各的，调用方无法预测行为。RESTful 规范让 API 一致、可预测，Swagger 文档让调用方快速理解接口。

## 类比

把 API 想象成"餐厅菜单"：

| 概念 | 类比 |
|------|------|
| 端点（Endpoint） | 菜品编号（如 #001 = 宫保鸡丁） |
| HTTP 方法 | 操作类型（GET = 查看，POST = 点菜） |
| 请求参数 | 菜品备注（少辣、不要香菜） |
| 响应格式 | 菜品规格（分量、价格、辣度） |
| Swagger 文档 | 菜单说明（图文并茂的介绍） |

## 核心内容

### RESTful 设计原则

| 原则 | 说明 |
|------|------|
| 资源导向 | URL 表示资源（`/users/123`） |
| HTTP 方法语义 | GET 查、POST 增、PUT 改、DELETE 删 |
| 统一响应格式 | 所有接口返回 `{ data, error, meta }` |
| 状态码规范 | 200 成功、400 客户端错误、500 服务端错误 |

### 响应格式示例

```typescript
// 成功响应
{
  "data": { "id": 1, "name": "Alice" },
  "error": null,
  "meta": { "timestamp": "2025-01-01T00:00:00Z" }
}

// 错误响应
{
  "data": null,
  "error": { "code": "USER_NOT_FOUND", "message": "用户不存在" },
  "meta": { "timestamp": "2025-01-01T00:00:00Z" }
}
```

## 你需要记住的

1. RESTful API 用 HTTP 方法表示操作类型。
2. 统一响应格式：`{ data, error, meta }`。
3. 状态码规范：200 成功、400 客户端错误、500 服务端错误。
4. Swagger 自动生成文档，用注解配置。
