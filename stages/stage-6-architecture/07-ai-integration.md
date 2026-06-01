# 6.7 AI 集成基础（Tool Calling）

## 一句话

Tool Calling 让 AI 模型（如 GPT-4）能调用外部函数，实现数据库查询、API 调用等功能，让 AI 应用能访问实时数据。

## 为什么需要它

没有 Tool Calling，AI 只能训练时的知识，无法访问实时数据（如数据库、API）。Tool Calling 让 AI 能执行代码，真正变成"智能助手"而不只是"聊天机器人"。

## 类比

把 Tool Calling 想象成"电话客服"：

| 概念 | 类比 |
|------|------|
| AI 模型 | 客服人员（理解用户需求） |
| Tools | 查询系统（客服可以查订单、查库存） |
| Tool Calling | 客服操作系统（查到结果后回复用户） |
| 函数定义 | 操作手册（告诉客服有哪些系统可用） |

客服不能凭空回答，需要查系统才能给出准确答案。

## 核心内容

### Tool Calling 流程

1. 定义工具函数（如 `getUser`、`createOrder`）
2. 把工具定义传给 AI 模型
3. 用户提问
4. AI 决定是否调用工具
5. 执行工具函数，获取结果
6. 把结果传回 AI
7. AI 生成最终回复

### 基本示例

```typescript
import OpenAI from 'openai';

const openai = new OpenAI();

// 定义工具
const tools = [
  {
    type: 'function',
    function: {
      name: 'getUser',
      description: '获取用户信息',
      parameters: {
        type: 'object',
        properties: {
          userId: { type: 'string', description: '用户 ID' },
        },
        required: ['userId'],
      },
    },
  },
];

// 工具实现
async function getUser(userId: string) {
  const user = await db.user.findUnique({ where: { id: userId } });
  return user;
}

// AI 调用
const response = await openai.chat.completions.create({
  model: 'gpt-4',
  messages: [{ role: 'user', content: '查一下用户 123 的信息' }],
  tools,
});

// 执行工具调用
const toolCall = response.choices[0].message.tool_calls?.[0];
if (toolCall) {
  const result = await getUser(JSON.parse(toolCall.function.arguments).userId);
  // 把结果传回 AI，生成最终回复
}
```

## 你需要记住的

1. Tool Calling 让 AI 能调用外部函数。
2. 需要定义工具描述、参数、实现。
3. AI 决定何时调用工具，调用后获取结果并生成回复。
4. 适合需要实时数据的 AI 应用。
