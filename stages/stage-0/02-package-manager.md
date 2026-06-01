# 0.2 包管理器与 package.json

## 一句话

**package.json 是项目的"采购清单"，npm/pnpm 是帮你去"商店"采购的工具。**

## 核心概念

### package.json — 项目的身份证 + 采购清单

每个项目根目录都有一个 `package.json`，它告诉世界：

```json
{
  "name": "my-app",              // 项目名称
  "version": "1.0.0",            // 版本号
  "private": true,               // 是否私有（不发布到 npm）
  "scripts": {
    "dev": "next dev",           // 快捷命令：npm run dev = 执行 next dev
    "build": "next build",       // 构建生产版本
    "start": "next start",       // 启动生产服务器
    "lint": "next lint"          // 代码检查
  },
  "dependencies": {
    "next": "14.2.0",            // 项目运行必须的包
    "react": "18.3.0",
    "react-dom": "18.3.0"
  },
  "devDependencies": {
    "typescript": "5.4.0",       // 只在开发时需要的包
    "@types/react": "18.3.0"     // 类型定义文件
  }
}
```

### 两个关键区域

| 字段 | 含义 | 举例 |
|------|------|------|
| `dependencies` | 项目运行必须的包 | React、Next.js、数据库驱动 |
| `devDependencies` | 只在开发时用的包 | TypeScript、测试工具、代码检查器 |

**类比**：`dependencies` 是汽车零件（没它车跑不了），`devDependencies` 是组装工具（车造好后不需要）。

### npm vs pnpm — 两个采购员

| | npm | pnpm |
|---|---|---|
| 安装命令 | `npm install` | `pnpm install` |
| 速度 | 较慢 | 更快 |
| 磁盘占用 | 每个项目各存一份 | 全局共享，节省空间 |
| 识别标志 | 生成 `package-lock.json` | 生成 `pnpm-lock.yaml` |

AI 生成的项目两种都可能用，**看到 lock 文件就知道用的哪个**。

### node_modules — 采购回来的仓库

`npm install` 或 `pnpm install` 执行后，所有包会下载到 `node_modules/` 目录。

```
my-app/
├── package.json        ← 采购清单
├── pnpm-lock.yaml      ← 实际采购记录（精确版本号）
├── node_modules/       ← 采购回来的所有东西（不要手动改）
│   ├── next/
│   ├── react/
│   └── ...几百个包
└── src/
```

## 实际操作

```bash
# 1. 克隆别人的项目后，第一步永远是安装依赖
pnpm install

# 2. 运行项目
pnpm dev

# 3. 添加新依赖
pnpm add lodash          # 加到 dependencies
pnpm add -D @types/lodash # 加到 devDependencies
```

## 你需要记住的

1. 看到 `package.json` → 这是项目的配置中心
2. 看到 `npm run xxx` 或 `pnpm xxx` → 在执行 scripts 里定义的命令
3. **不要手动编辑 `node_modules/`**，它由包管理器自动管理
4. **不要把 `node_modules/` 提交到 git**（已经在 `.gitignore` 中排除）

## AI 代码中的线索

```bash
# AI 常给你这些命令
npm install prisma @prisma/client   # 安装数据库工具
pnpm add framer-motion              # 安装动画库
npm run dev                         # 启动开发
npm run build                       # 构建生产版本
```

看到这些，你就知道：AI 在帮你往 `package.json` 里加依赖，然后运行对应的脚本。
