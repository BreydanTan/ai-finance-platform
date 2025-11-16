# 第一阶段：Hello World - 环境搭建与首次成功 🚀

**学习时间**: 1-2 天（每天 2-3 小时）
**难度等级**: ⭐ 入门
**完成标志**: 浏览器中看到运行的项目

---

## 🎯 学习目标

完成这个阶段后，你将能够：
- ✅ 在本地成功运行项目
- ✅ 看到项目的登录页面和仪表板
- ✅ 理解项目的基本文件结构
- ✅ 知道如何修改简单的文本内容
- ✅ 建立使用 AI 辅助开发的信心

---

## 📚 前置知识

### 必须掌握
- ✅ 会使用电脑和浏览器
- ✅ 了解什么是文件夹和文件
- ✅ 会使用文本编辑器（如记事本）

### 不需要掌握（我们会一步步教）
- ❌ 不需要会编程
- ❌ 不需要懂 JavaScript
- ❌ 不需要了解数据库

### 需要准备的工具

1. **代码编辑器** - VS Code（免费）
   - 下载：https://code.visualstudio.com
   - 安装后打开，熟悉界面

2. **Node.js** - JavaScript 运行环境（免费）
   - 下载：https://nodejs.org（选择 LTS 版本）
   - 验证安装：打开终端输入 `node -v`，应显示版本号

3. **Git** - 代码版本管理工具（免费）
   - 下载：https://git-scm.com
   - 验证安装：终端输入 `git --version`

4. **GitHub 账号**（免费）
   - 注册：https://github.com

**参考概念**：
→ [Node.js](../02-concept-dictionary.md#nodejs)
→ [Git](../02-concept-dictionary.md#git)

---

## 🛠️ 实践任务

### 任务 1：克隆项目到本地

**目标**: 把项目代码下载到你的电脑

**步骤**:

#### 1.1 打开终端（命令行）

**Windows**:
- 按 `Win + R`
- 输入 `cmd`
- 按回车

**macOS**:
- 按 `Cmd + 空格`
- 输入 `Terminal`
- 按回车

**Linux**:
- 按 `Ctrl + Alt + T`

#### 1.2 选择存放位置

```bash
# Windows 用户
cd C:\Users\你的用户名\Documents

# macOS/Linux 用户
cd ~/Documents
```

**解释**: `cd` 是 "Change Directory" 的缩写，意思是"进入某个文件夹"。

#### 1.3 克隆项目

```bash
git clone https://github.com/[用户名]/ai-finance-platform.git
cd ai-finance-platform
```

**解释**:
- `git clone` - 下载项目
- `cd ai-finance-platform` - 进入项目文件夹

**验证**: 输入 `ls`（或 Windows 的 `dir`），应该看到很多文件和文件夹。

---

### 任务 2：安装项目依赖

**目标**: 下载项目需要的所有"材料"（代码库）

**步骤**:

#### 2.1 安装依赖包

```bash
npm install
```

**解释**: 这个命令会读取 `package.json` 文件，下载所有需要的包到 `node_modules` 文件夹。

**时间**: 可能需要 3-10 分钟，取决于网速。

**进度显示**: 你会看到很多下载信息滚动，这是正常的。

**完成标志**: 看到类似这样的提示：
```
added 500 packages in 5m
```

#### 2.2 验证安装

```bash
ls node_modules  # macOS/Linux
dir node_modules # Windows
```

应该看到很多文件夹（这些都是下载的包）。

**AI 协作提示词**:
```
如果安装过程中出现错误，可以这样问 AI：

"我在运行 npm install 时遇到以下错误：
[粘贴错误信息]

我的系统是 [Windows/macOS/Linux]
Node.js 版本是 [运行 node -v 的结果]

应该如何解决？"
```

---

### 任务 3：配置环境变量

**目标**: 设置项目运行需要的"密钥"

**步骤**:

#### 3.1 复制环境变量模板

```bash
# macOS/Linux
cp .env.example .env

# Windows (命令提示符)
copy .env.example .env

# Windows (PowerShell)
Copy-Item .env.example .env
```

**解释**: 创建一个 `.env` 文件，存放敏感配置。

#### 3.2 编辑 .env 文件

**使用 VS Code 打开**:
```bash
code .env
```

或者：
1. 打开 VS Code
2. 文件 → 打开文件夹 → 选择 `ai-finance-platform`
3. 在左侧文件树中找到 `.env`

#### 3.3 填写必需的配置

**暂时跳过外部服务**（我们先让项目运行起来）：

```bash
# 数据库配置 - 暂时使用本地 SQLite
DATABASE_URL="file:./dev.db"
DIRECT_URL="file:./dev.db"

# Clerk - 暂时留空，先不登录
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=
CLERK_SECRET_KEY=

# 其他服务 - 暂时留空
GEMINI_API_KEY=
RESEND_API_KEY=
ARCJET_KEY=
```

**参考概念**：
→ [Environment Variables](../02-concept-dictionary.md#environment-variables-环境变量)
→ [.env 文件](../02-concept-dictionary.md#env-文件)

#### 3.4 修改数据库配置使用 SQLite

为了快速启动，我们先使用本地 SQLite 数据库：

**编辑 `prisma/schema.prisma`**:

找到这部分：
```prisma
datasource db {
  provider = "postgresql"  // 改成下面这行
  provider = "sqlite"       // 使用本地数据库
  url      = env("DATABASE_URL")
}
```

**保存文件**: `Ctrl + S` (或 `Cmd + S`)

---

### 任务 4：初始化数据库

**目标**: 创建数据库和表结构

**步骤**:

#### 4.1 生成 Prisma 客户端

```bash
npx prisma generate
```

**解释**: 根据 `schema.prisma` 生成数据库访问代码。

#### 4.2 创建数据库

```bash
npx prisma db push
```

**解释**: 在本地创建数据库文件和所有表。

**完成标志**: 看到类似提示：
```
✔ Generated Prisma Client
✔ The database is now in sync with your Prisma schema.
```

#### 4.3 查看数据库（可选）

```bash
npx prisma studio
```

**解释**: 打开一个网页界面，可视化查看数据库。

- 会自动打开浏览器 http://localhost:5555
- 你能看到 User、Account、Transaction、Budget 等表
- 现在都是空的，没有数据

**关闭 Prisma Studio**: 回到终端，按 `Ctrl + C`

**参考概念**：
→ [Prisma](../02-concept-dictionary.md#prisma)
→ [Database](../02-concept-dictionary.md#database-数据库)

---

### 任务 5：启动开发服务器

**目标**: 运行项目，在浏览器中打开

**步骤**:

#### 5.1 启动服务器

```bash
npm run dev
```

**解释**: 启动 Next.js 开发服务器。

**成功标志**: 看到类似输出：
```
▲ Next.js 15.0.3
- Local:        http://localhost:3000
- Network:      http://192.168.1.x:3000

✓ Ready in 2.5s
```

#### 5.2 打开浏览器

1. 打开浏览器（Chrome、Firefox、Safari 等）
2. 访问：http://localhost:3000

**你应该看到**:
- 项目的首页（Landing Page）
- 有"Sign In"和"Get Started"按钮
- 页面样式正常显示

#### 5.3 初步探索

**尝试点击**:
- "Get Started" 按钮
- "Sign In" 按钮

**预期结果**: 可能会跳转到 Clerk 登录页，但因为我们还没配置 Clerk，可能会显示错误。**这是正常的！**

---

### 任务 6：修改首页文本（你的第一次代码改动）

**目标**: 修改页面上的文字，看到效果

**步骤**:

#### 6.1 找到首页文件

在 VS Code 中打开：
```
app/page.jsx
```

#### 6.2 定位要修改的文字

找到这一行（大约在第 23 行）：

```jsx
<h1 className="text-5xl font-bold tracking-tight">
  AI Finance Platform
</h1>
```

#### 6.3 修改文字

改成：
```jsx
<h1 className="text-5xl font-bold tracking-tight">
  我的第一个 AI 财务平台
</h1>
```

#### 6.4 保存并查看效果

1. **保存**: `Ctrl + S` (或 `Cmd + S`)
2. **回到浏览器** - 页面会自动刷新
3. **看到变化**: 标题变成了中文！

**🎉 恭喜！你完成了第一次代码修改！**

#### 6.5 继续尝试

修改其他文字：
```jsx
<p className="text-lg text-muted-foreground">
  管理你的财务，追踪收支，实现财务自由
</p>
```

**AI 协作提示词**:
```
"我想修改首页的这段文字：[粘贴原文]

改成更有吸引力的内容，并保持原来的 HTML 结构。

请提供修改后的代码。"
```

---

### 任务 7：理解项目结构

**目标**: 知道主要文件夹的用途

**打开 VS Code 左侧文件树**，你会看到：

```
ai-finance-platform/
├── 📁 app/                 ← 所有页面和路由
│   ├── 📁 (auth)/         ← 登录/注册页面
│   ├── 📁 (main)/         ← 主应用（仪表板等）
│   └── 📄 page.jsx        ← 首页（刚才改的）
├── 📁 components/         ← 可复用组件
│   └── 📁 ui/            ← 基础 UI 组件
├── 📁 actions/            ← 后端功能（Server Actions）
├── 📁 lib/                ← 工具库
├── 📁 prisma/             ← 数据库配置
│   └── schema.prisma     ← 数据库结构
├── 📁 public/             ← 静态文件（图片等）
├── 📄 package.json        ← 项目配置
└── 📄 .env                ← 环境变量（密钥）
```

**重点文件夹解释**:

| 文件夹 | 作用 | 类比 |
|-------|------|------|
| `app/` | 所有页面 | 网站的"房间" |
| `components/` | 可复用组件 | 乐高积木 |
| `actions/` | 后端功能 | 厨房（处理数据） |
| `prisma/` | 数据库 | 仓库（存储数据） |
| `public/` | 图片、字体 | 装饰品 |

**参考文档**：
→ [项目结构详解](../../specifications/PROJECT_SPEC_CN.md#81-项目结构说明)

---

## ✅ 验证清单

完成任务后，检查以下项目：

### 环境验证
- [ ] Node.js 已安装（`node -v` 显示版本）
- [ ] Git 已安装（`git --version` 显示版本）
- [ ] VS Code 已安装并能打开项目

### 项目验证
- [ ] 项目已克隆到本地
- [ ] `npm install` 成功完成
- [ ] `node_modules` 文件夹存在
- [ ] `.env` 文件已创建

### 运行验证
- [ ] 数据库已初始化（`dev.db` 文件存在）
- [ ] 开发服务器启动成功
- [ ] 浏览器能访问 http://localhost:3000
- [ ] 能看到首页内容

### 修改验证
- [ ] 成功修改了首页文字
- [ ] 保存后页面自动刷新
- [ ] 修改的内容正确显示

### 理解验证
- [ ] 知道项目主要文件夹的作用
- [ ] 能够在 VS Code 中找到文件
- [ ] 了解如何启动和停止开发服务器

---

## ❓ 常见问题

### Q1: npm install 很慢或失败

**原因**: 网络问题或 npm 源速度慢

**解决方案 1** - 使用国内镜像:
```bash
npm config set registry https://registry.npmmirror.com
npm install
```

**解决方案 2** - 使用 cnpm:
```bash
npm install -g cnpm --registry=https://registry.npmmirror.com
cnpm install
```

**AI 协作提示词**:
```
"我运行 npm install 时出现以下错误：
[粘贴错误信息]

网速较慢，有什么解决办法？"
```

---

### Q2: 页面空白或显示错误

**检查步骤**:

1. **查看终端输出**
   - 开发服务器是否还在运行？
   - 有没有红色错误信息？

2. **查看浏览器控制台**
   - 按 F12 打开开发者工具
   - 查看 Console 标签
   - 截图错误信息

3. **重启开发服务器**
   - 终端按 `Ctrl + C` 停止
   - 重新运行 `npm run dev`

**AI 协作提示词**:
```
"我访问 http://localhost:3000 时页面空白。

终端显示：
[粘贴终端输出]

浏览器控制台显示：
[粘贴控制台错误]

应该如何排查？"
```

---

### Q3: 找不到 .env 文件

**原因**: `.env` 是隐藏文件

**VS Code 解决**:
1. 文件已经在项目中，只是看不到
2. VS Code 会显示隐藏文件

**终端查看**:
```bash
# macOS/Linux
ls -la

# Windows
dir /a
```

**创建 .env**（如果真的不存在）:
```bash
# macOS/Linux
touch .env

# Windows
type nul > .env
```

---

### Q4: Prisma 命令报错

**常见错误**: `Command failed: prisma`

**原因**: Prisma 没有正确安装

**解决**:
```bash
# 重新安装 Prisma
npm install prisma --save-dev
npm install @prisma/client

# 重新生成
npx prisma generate
```

---

### Q5: 端口 3000 被占用

**错误信息**: `Port 3000 is already in use`

**解决方案 1** - 更改端口:
```bash
PORT=3001 npm run dev  # macOS/Linux
set PORT=3001 && npm run dev  # Windows CMD
$env:PORT=3001; npm run dev  # Windows PowerShell
```

**解决方案 2** - 找到并关闭占用的进程:

macOS/Linux:
```bash
lsof -ti:3000 | xargs kill
```

Windows:
```bash
netstat -ano | findstr :3000
taskkill /PID [PID号] /F
```

---

### Q6: 修改代码后页面没变化

**原因**: 浏览器缓存或热更新失败

**解决步骤**:

1. **硬刷新浏览器**
   - Windows: `Ctrl + Shift + R`
   - macOS: `Cmd + Shift + R`

2. **检查文件是否保存**
   - VS Code 文件标签有点 · 表示未保存
   - 按 `Ctrl + S` 保存

3. **重启开发服务器**
   - `Ctrl + C` 停止
   - `npm run dev` 重启

---

## 🚀 进阶挑战（可选）

如果你轻松完成了所有任务，可以尝试：

### 挑战 1: 添加自己的一段话

在首页添加你的个人介绍：

```jsx
<div className="mt-6 text-center">
  <p className="text-sm text-gray-500">
    这是我学习全栈开发的第一个项目 - [你的名字]
  </p>
</div>
```

### 挑战 2: 修改颜色主题

找到 `app/globals.css`，尝试修改主题颜色：

```css
/* 找到 :root 部分，修改这些值 */
--primary: 220 90% 56%;  /* 主色调 */
```

保存后看页面颜色变化。

### 挑战 3: 探索其他页面

停止开发服务器后，查看其他页面文件：
- `app/(main)/dashboard/page.jsx` - 仪表板
- `app/(main)/transaction/create/page.jsx` - 创建交易

**不需要运行**，只是阅读代码，看能理解多少。

### 挑战 4: 使用 Prisma Studio

```bash
npx prisma studio
```

尝试手动添加一条用户数据：
1. 打开 User 表
2. 点击 "Add record"
3. 填写 email 和 name
4. 保存

---

## 📊 阶段小结

### 你学到了什么

✅ **工具使用**
- Git 克隆项目
- npm 安装依赖
- VS Code 编辑代码
- 终端基本命令

✅ **项目认知**
- 理解项目文件结构
- 知道主要文件夹的作用
- 能定位首页文件

✅ **开发流程**
- 启动开发服务器
- 修改代码
- 浏览器查看效果
- 热更新机制

✅ **问题解决**
- 遇到错误时如何查看
- 如何使用 AI 寻求帮助
- 常见问题的解决方法

### 自我评估

回答以下问题：

1. **我能独立启动项目吗？**
   - [ ] 能，不看文档也能操作
   - [ ] 能，需要看文档
   - [ ] 还不太熟练

2. **我理解项目结构吗？**
   - [ ] 知道每个文件夹的作用
   - [ ] 大概知道，但不确定
   - [ ] 还不太清楚

3. **我能修改简单内容吗？**
   - [ ] 能，而且知道原理
   - [ ] 能，但不太理解
   - [ ] 还不会

**评分标准**:
- 8-9 分：优秀，可以进入下一阶段
- 5-7 分：良好，建议复习后再继续
- 0-4 分：需要重新学习这个阶段

---

## ➡️ 下一步

### 如果你完成得很好

🎉 恭喜！你已经掌握了基础，准备好进入：

**→ [第二阶段：理解结构](./04-stage2-skeleton.md)**

在下一阶段，你将：
- 深入理解文件组织方式
- 学习路由系统
- 理解组件的概念
- 创建你的第一个组件

### 如果遇到困难

💪 不要气馁！建议：

1. **重新按步骤操作一遍** - 慢慢来，不要跳步骤
2. **寻求帮助** - 使用 AI 助手（ChatGPT、Claude 等）
3. **观看视频** - B站搜索 "Next.js 环境搭建"
4. **休息一下** - 明天再试，说不定就成功了

### 学习资源

**视频教程**:
- B站："Next.js 15 入门教程"
- YouTube: "Next.js Tutorial for Beginners"

**文档**:
- [Next.js 官方文档（中文）](https://nextjs.org/docs)
- [Node.js 官方教程](https://nodejs.org/zh-cn/learn)

**社区**:
- V2EX Next.js 板块
- 掘金 Next.js 话题

---

## 💡 给自己的奖励

完成第一阶段后，你应该：

✅ 给自己一个赞 👍
✅ 截图保存你的成果
✅ 在社交媒体分享（可选）
✅ 告诉朋友你在学习编程

**记住**: 每个高手都是从这一步开始的！

---

**阶段完成日期**: ________________
**用时**: ______ 小时
**遇到的主要问题**:

**笔记**:
