# 0.2 包管理器与 package.json

## 一句话

**package.json 是项目的"采购清单"，npm/pnpm 是帮你去"商店"采购的工具。**

## 为什么需要它

你的项目不可能所有代码都自己写。React 框架、UI 组件库、数据库工具——这些第三方代码（叫做"包"或"依赖"）需要从网上下载、管理版本、保持一致。

手动下载？不现实。一个 Next.js 项目通常有几百个依赖包，手动管理会崩溃。包管理器就是自动化这个过程。

## 类比

| 概念 | 类比 |
|------|------|
| package.json | 采购清单（写明要买什么、什么版本） |
| npm / pnpm | 采购员（拿着清单去商店采购） |
| node_modules/ | 仓库（采购回来的东西存放的地方） |
| npm registry | 商店（全世界的包都在这里上架） |
| lock 文件 | 购物小票（记录实际买到的精确版本） |

## 核心内容

### package.json — 项目的配置中心

每个项目根目录都有一个 `package.json`，它定义了项目的所有关键信息：

```json
{
  "name": "my-app",              // 项目名称
  "version": "1.0.0",            // 版本号
  "private": true,               // 是否私有（不发布到 npm）
  "scripts": {
    "dev": "next dev",           // npm run dev → 实际执行 next dev
    "build": "next build",       // npm run build → 实际执行 next build
    "start": "next start",       // npm run start → 实际执行 next start
    "lint": "next lint"          // npm run lint → 实际执行 next lint
  },
  "dependencies": {
    "next": "14.2.0",            // 项目运行必须的包
    "react": "18.3.0",           // React 框架（阶段 1 会详细讲）
    "react-dom": "18.3.0"        // React 的浏览器渲染适配层
  },
  "devDependencies": {
    "typescript": "5.4.0",       // 只在开发时需要的包（0.4 会详细讲）
    "@types/react": "18.3.0"     // React 的类型定义（0.4 会详细讲）
  }
}
```

### scripts 字段的工作原理

`scripts` 是快捷命令的映射表。原理：

```
你在终端输入          npm 读取 scripts        实际执行
npm run dev    →    找到 "dev": "next dev"  →  next dev（用 Node.js 执行）
npm run build  →    找到 "build": "next build" → next build
```

- `npm run` 后面的词（dev、build）不是固定命令，而是你在 scripts 中**自己定义**的名称
- 冒号右边的值才是真正要执行的命令
- `next` 是 Next.js 框架提供的命令行工具，安装 `next` 包后才有

**pnpm 的简写**：`pnpm dev` 等同于 `pnpm run dev`（pnpm 允许省略 `run`）。npm 不支持这个简写。

### dependencies vs devDependencies

| 字段 | 含义 | 类比 | 举例 |
|------|------|------|------|
| `dependencies` | 项目运行必须的包 | 汽车零件（没它车跑不了） | React、Next.js、数据库驱动 |
| `devDependencies` | 只在开发时用的包 | 组装工具（车造好后不需要） | TypeScript、测试工具、代码检查器 |

区分的意义：部署到生产环境时，可以只安装 `dependencies`，跳过 `devDependencies`，减少安装量和攻击面。

### npm vs pnpm — 两个采购员

| | npm | pnpm |
|---|---|---|
| 安装命令 | `npm install` | `pnpm install` |
| 速度 | 较慢 | 更快 |
| 磁盘占用 | 每个项目各存一份 | 全局共享，节省空间 |
| 识别标志 | 生成 `package-lock.json` | 生成 `pnpm-lock.yaml` |

AI 生成的项目两种都可能用。**看根目录下的 lock 文件就知道用的哪个**。

### pnpm install 做了什么

```bash
pnpm install
```

这行命令的完整流程：
1. 读取 `package.json`，知道要安装哪些包
2. 检查 `pnpm-lock.yaml`，确定每个包的精确版本
3. 从 npm registry（全球包仓库）下载所有包
4. 存入 `node_modules/` 目录
5. 如果是 pnpm，还会通过硬链接指向全局存储（所以省空间）

### node_modules — 不要碰的仓库

```
my-app/
├── package.json        ← 采购清单
├── pnpm-lock.yaml      ← 精确版本记录
├── node_modules/       ← 采购回来的所有东西（不要手动改）
│   ├── next/
│   ├── react/
│   └── ...几百个包
└── src/
```

`node_modules/` 完全由包管理器自动管理。**手动修改里面的文件没有意义**，下次 `pnpm install` 会被覆盖。这个目录已经通过 `.gitignore` 排除在 git 之外（git 是版本控制工具，用于追踪代码变更历史）。

## 你需要记住的

1. `package.json` 的 `scripts` 定义了快捷命令——`npm run dev` 中的 `dev` 就来自这里
2. `pnpm dev` 是 `pnpm run dev` 的简写（npm 不能省略 `run`）
3. `dependencies` = 运行必须，`devDependencies` = 开发专用
4. `node_modules/` 不要手动改、不要提交到 git
5. 看 lock 文件判断项目用的 npm 还是 pnpm

## AI 代码中的线索

```bash
# AI 生成项目后常给你这些命令
pnpm install                        # 安装所有依赖
pnpm add prisma @prisma/client      # 安装数据库工具（加到 dependencies）
pnpm add -D @types/lodash           # 安装类型定义（-D = 加到 devDependencies）
pnpm dev                            # 启动开发
pnpm build                          # 构建生产版本
```

`pnpm add` 做了两件事：下载包到 `node_modules/`，同时在 `package.json` 中添加记录。

## 验证问题

- [ ] `npm run dev` 中的 `dev` 是 npm 的内置命令吗？如果不是，它从哪来的？
- [ ] `dependencies` 和 `devDependencies` 有什么区别？为什么部署时只装 dependencies？
- [ ] `pnpm dev` 和 `pnpm run dev` 是一样的吗？npm 也能这样简写吗？
- [ ] 克隆别人的项目后，第一步应该运行什么命令？为什么？
