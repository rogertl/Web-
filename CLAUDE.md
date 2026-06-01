# AI 时代 Next.js 全栈代码理解 - 学习项目

## 项目结构

```
stages/                   # 7 个阶段的教学内容
  stage-0-basics/          # 环境准备与最基础认知
  stage-1-react/           # React 基础与组件思维
  stage-2-nextjs-structure/# Next.js 项目结构与加载逻辑
  stage-3-server-components/# Server Components 与现代渲染模型
  stage-4-server-actions/  # 交互与写操作（Server Actions）
  stage-5-database/        # 数据层与后端逻辑
  stage-6-architecture/    # 架构进阶与生产考虑
practice/                 # 实践项目（后续创建）
notes/                    # 学习笔记
```

## 教学流水线 Skill

使用 `tutorial-pipeline` skill 自动生成教学内容：
- **单阶段**：说"开始阶段 X"
- **全量生成**：说"生成所有阶段"
- **质量检查**：说"检查质量"

Skill 路径：`.claude/skills/tutorial-pipeline/`

## 教学节奏

- 每周重点攻克 1 个阶段
- 每阶段结束：AI 生成小功能代码 → 用自己的话讲解数据流
- 达标标准：能看懂 AI 生成的中型项目代码（Server Action + 数据库 + 权限）

## 进度

- [ ] 阶段 0：环境准备与最基础认知
- [ ] 阶段 1：React 基础与组件思维
- [ ] 阶段 2：Next.js 项目结构与加载逻辑
- [ ] 阶段 3：Server Components 与现代渲染模型
- [ ] 阶段 4：交互与写操作（Server Actions）
- [ ] 阶段 5：数据层与后端逻辑
- [ ] 阶段 6：架构进阶与生产考虑
