# 第二阶段：理解骨架 - 路由与组件结构 🗺️

**学习时间**: 3-5 天
**难度等级**: ⭐⭐ 初级
**完成标志**: 理解项目文件组织，能找到任何功能的代码

---

## 🎯 学习目标

- ✅ 理解 Next.js App Router 工作原理
- ✅ 掌握文件夹路由映射关系
- ✅ 理解组件的层次结构
- ✅ 能快速定位功能对应的代码文件
- ✅ 创建第一个简单页面

---

## 📚 前置知识

**必须完成**: [第一阶段](./03-stage1-foundation.md)

**需要理解的概念**:
→ [App Router](../02-concept-dictionary.md#app-router)
→ [Component](../02-concept-dictionary.md#component-组件)
→ [Route](../02-concept-dictionary.md#route-路由)

---

## 🛠️ 核心任务

### 任务 1: 理解路由映射

**文件夹结构 = URL 结构**

```
app/
├── page.jsx                    → http://localhost:3000/
├── (auth)/
│   ├── sign-in/
│   │   └── page.jsx           → /sign-in
│   └── sign-up/
│       └── page.jsx           → /sign-up
└── (main)/
    ├── dashboard/
    │   └── page.jsx           → /dashboard
    ├── account/
    │   └── [id]/
    │       └── page.jsx       → /account/123  (动态路由)
    └── transaction/
        ├── create/
        │   └── page.jsx       → /transaction/create
        └── [id]/
            └── page.jsx       → /transaction/456
```

**练习**: 访问以上每个 URL，观察对应的页面

---

### 任务 2: 创建你的第一个页面

**目标**: 创建 `/about` 页面

**步骤**:

1. **创建文件夹和文件**
```bash
mkdir -p app/about
```

2. **创建 `app/about/page.jsx`**:
```jsx
export default function AboutPage() {
  return (
    <div className="container mx-auto p-8">
      <h1 className="text-4xl font-bold mb-4">关于本项目</h1>
      <p className="text-lg">
        这是我学习 Next.js 15 + Prisma 的练习项目。
      </p>
      <div className="mt-6 space-y-2">
        <p>📅 开始日期：{new Date().toLocaleDateString('zh-CN')}</p>
        <p>⚡ 技术栈：Next.js, Prisma, Tailwind CSS</p>
        <p>🎯 目标：掌握全栈开发</p>
      </div>
    </div>
  );
}
```

3. **访问页面**: http://localhost:3000/about

**AI 协作提示词**:
```
"我想在项目中添加一个 /about 页面，显示：
- 项目介绍
- 开始日期
- 使用的技术栈
- 学习目标

请帮我创建这个页面，使用 Tailwind CSS 样式。"
```

---

### 任务 3: 理解组件复用

**查看现有组件**:

打开 `components/header.jsx`，这是全局头部组件：

```jsx
// 在多个页面中复用
<Header />
```

**创建你的第一个组件**:

1. **创建 `components/my-card.jsx`**:
```jsx
export default function MyCard({ title, content }) {
  return (
    <div className="border rounded-lg p-4 shadow-sm">
      <h3 className="font-bold text-lg">{title}</h3>
      <p className="text-gray-600">{content}</p>
    </div>
  );
}
```

2. **在 about 页面使用**:
```jsx
import MyCard from "@/components/my-card";

export default function AboutPage() {
  return (
    <div className="container mx-auto p-8">
      <h1 className="text-4xl font-bold mb-4">关于本项目</h1>

      <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
        <MyCard
          title="学习进度"
          content="已完成第1-2阶段"
        />
        <MyCard
          title="下一步"
          content="学习数据库操作"
        />
      </div>
    </div>
  );
}
```

---

### 任务 4: 探索布局系统

**理解 Layout**:

每个文件夹可以有自己的 `layout.jsx`，包裹所有子页面。

**查看主布局**: `app/(main)/layout.jsx`

```jsx
export default function MainLayout({ children }) {
  return (
    <div>
      <Header />  {/* 所有主页面都有头部 */}
      <main>{children}</main>  {/* 页面内容 */}
    </div>
  );
}
```

**效果**: `/dashboard`, `/account/[id]`, `/transaction/create` 都会显示 Header。

---

## ✅ 验证清单

- [ ] 理解文件夹 → URL 的映射关系
- [ ] 成功创建了 /about 页面
- [ ] 理解动态路由 `[id]` 的含义
- [ ] 创建并使用了自定义组件
- [ ] 理解 Layout 的作用
- [ ] 能快速找到任意功能的代码文件

---

## 📊 自我评估

**问题**:
1. 如果我想创建 `/blog/post/123` 页面，文件结构应该是？
2. Layout 和 Page 的区别是什么？
3. 如何在多个页面复用一个组件？

**参考答案**: 查看 `project-mastery/specifications/PROJECT_SPEC_CN.md` 第8章

---

## ➡️ 下一步

准备好了吗？进入 **[第三阶段：数据库连接](./05-stage3-core-features.md)**
