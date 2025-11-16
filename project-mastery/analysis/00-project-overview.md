# 项目总览 | Project Overview

**分析日期 | Analysis Date**: 2025-11-15
**项目名称 | Project Name**: AI Finance Platform (Welth)
**代码库类型 | Repository Type**: Full-Stack Web Application

---

## 📋 目录 | Table of Contents

1. [项目简介](#项目简介--project-introduction)
2. [技术栈清单](#技术栈清单--technology-stack)
3. [目录结构](#目录结构--directory-structure)
4. [架构模式](#架构模式--architecture-pattern)
5. [关键配置文件](#关键配置文件--key-configuration-files)
6. [环境变量清单](#环境变量清单--environment-variables)
7. [核心功能概览](#核心功能概览--core-features-overview)
8. [项目类型识别](#项目类型识别--project-type-identification)

---

## 项目简介 | Project Introduction

### 中文简介

**Welth** 是一个现代化的 **AI 驱动的个人财务管理平台**，使用 Next.js 15 和 React 19 构建的全栈应用程序。

**核心特性**：
- 💰 **多账户管理**：支持储蓄账户和活期账户
- 📊 **交易追踪**：记录收入和支出，支持循环交易（每日/周/月/年）
- 🤖 **AI 功能**：使用 Google Gemini AI 进行收据扫描和智能分析
- 📈 **预算管理**：设置预算目标并接收智能提醒
- 📧 **邮件通知**：通过 Resend 发送预算警报
- 🔒 **企业级安全**：集成 Clerk 认证和 ArcJet 安全防护
- ⚡ **后台任务**：使用 Inngest 处理循环交易和定时任务

### English Introduction

**Welth** is a modern **AI-powered personal finance management platform** built with Next.js 15 and React 19 as a full-stack application.

**Core Features**:
- 💰 **Multi-Account Management**: Support for savings and current accounts
- 📊 **Transaction Tracking**: Record income and expenses with recurring transaction support (daily/weekly/monthly/yearly)
- 🤖 **AI Capabilities**: Receipt scanning and intelligent analysis using Google Gemini AI
- 📈 **Budget Management**: Set budget goals and receive smart alerts
- 📧 **Email Notifications**: Budget alerts via Resend
- 🔒 **Enterprise Security**: Integrated Clerk authentication and ArcJet protection
- ⚡ **Background Jobs**: Process recurring transactions and scheduled tasks with Inngest

---

## 技术栈清单 | Technology Stack

### 前端技术 | Frontend Stack

| 技术/库 | 版本 | 用途 | File Reference |
|---------|------|------|----------------|
| **Next.js** | `15.0.3` | React 框架，App Router 模式 | package.json:34 |
| **React** | `19.0.0-rc-66855b96-20241106` | UI 库（RC 版本） | package.json:36 |
| **React DOM** | `19.0.0-rc-66855b96-20241106` | React 渲染引擎 | package.json:38 |
| **Tailwind CSS** | `^3.4.1` | 原子化 CSS 框架 | package.json:55 |
| **shadcn/ui** | - | UI 组件库（基于 Radix UI） | components.json:1 |
| **Radix UI** | `^1.x` | 无障碍 UI 组件原语 | package.json:19-27 |
| **Lucide React** | `^0.462.0` | 图标库 | package.json:33 |
| **Recharts** | `^2.14.1` | 数据可视化图表库 | package.json:41 |
| **React Hook Form** | `^7.53.2` | 表单管理库 | package.json:39 |
| **Zod** | `^3.23.8` | TypeScript-first 验证库 | package.json:47 |
| **date-fns** | `^4.1.0` | 日期处理库 | package.json:31 |
| **React Day Picker** | `^8.10.1` | 日期选择器组件 | package.json:37 |
| **Sonner** | `^1.7.0` | Toast 通知库 | package.json:43 |
| **next-themes** | `^0.4.3` | 主题切换（深色模式支持） | package.json:35 |
| **Vaul** | `^1.1.1` | Drawer 组件 | package.json:46 |
| **class-variance-authority** | `^0.7.1` | 动态类名管理 | package.json:29 |
| **clsx** | `^2.1.1` | 条件类名工具 | package.json:30 |
| **tailwind-merge** | `^2.5.5` | Tailwind 类名合并 | package.json:44 |

### 后端技术 | Backend Stack

| 技术/库 | 版本 | 用途 | File Reference |
|---------|------|------|----------------|
| **Prisma** | `^6.0.1` | ORM（对象关系映射） | package.json:18, 53 |
| **@prisma/client** | `^6.0.1` | Prisma 客户端 | package.json:18 |
| **PostgreSQL** | - | 数据库（通过 Supabase） | prisma/schema.prisma:6 |
| **Clerk** | `^6.6.0` | 认证和用户管理 | package.json:15 |
| **Inngest** | `^3.27.4` | 后台作业和事件驱动工作流 | package.json:32 |
| **ArcJet** | `^1.0.0-alpha.34` | 速率限制和安全防护 | package.json:14 |
| **Resend** | `^4.0.1` | 邮件发送服务 | package.json:42 |
| **@react-email/components** | `0.0.30` | React 邮件模板组件 | package.json:28 |
| **@google/generative-ai** | `^0.21.0` | Google Gemini AI SDK | package.json:16 |

### 开发工具 | Development Tools

| 工具 | 版本 | 用途 | File Reference |
|------|------|------|----------------|
| **ESLint** | `^8` | 代码规范检查 | package.json:50 |
| **PostCSS** | `^8` | CSS 后处理器 | package.json:52 |
| **Turbopack** | - | Next.js 15 内置打包工具 | package.json:6 |
| **react-email** | `3.0.3` | 邮件开发工具 | package.json:54 |

---

## 目录结构 | Directory Structure

```
ai-finance-platform/
│
├── 📁 actions/                           # Server Actions（服务器端操作）
│   ├── account.js                        # 账户 CRUD 操作
│   ├── transaction.js                    # 交易 CRUD 操作
│   ├── budget.js                         # 预算管理操作
│   ├── dashboard.js                      # 仪表板数据获取
│   ├── send-email.js                     # 邮件发送逻辑
│   └── seed.js                           # 数据库种子数据
│
├── 📁 app/                               # Next.js App Router
│   ├── 📁 (auth)/                        # 认证路由组
│   │   ├── sign-in/[[...sign-in]]/       # Clerk 登录页面
│   │   └── sign-up/[[...sign-up]]/       # Clerk 注册页面
│   │   └── layout.js                     # 认证布局
│   │
│   ├── 📁 (main)/                        # 主应用路由组（需要认证）
│   │   ├── 📁 dashboard/                 # 仪表板
│   │   │   ├── _components/              # 仪表板专用组件
│   │   │   │   ├── account-card.jsx      # 账户卡片
│   │   │   │   ├── budget-progress.jsx   # 预算进度
│   │   │   │   └── transaction-overview.jsx # 交易概览
│   │   │   └── page.jsx                  # 仪表板页面
│   │   │
│   │   ├── 📁 account/                   # 账户管理
│   │   │   ├── [id]/page.jsx             # 单个账户详情（动态路由）
│   │   │   └── _components/              # 账户专用组件
│   │   │       ├── account-chart.jsx     # 账户图表
│   │   │       ├── transaction-table.jsx # 交易表格
│   │   │       └── no-pagination-transaction-table.jsx
│   │   │
│   │   ├── 📁 transaction/               # 交易管理
│   │   │   ├── create/page.jsx           # 创建交易页面
│   │   │   └── _components/
│   │   │       ├── transaction-form.jsx  # 交易表单
│   │   │       └── recipt-scanner.jsx    # AI 收据扫描
│   │   └── layout.js                     # 主应用布局
│   │
│   ├── 📁 api/                           # API 路由
│   │   ├── inngest/route.js              # Inngest 事件处理端点
│   │   └── seed/route.js                 # 数据播种端点
│   │
│   ├── 📁 lib/                           # 应用级工具库
│   │   └── schema.js                     # Zod 验证模式
│   │
│   ├── layout.js                         # 根布局（包含 ClerkProvider）
│   ├── page.js                           # 首页（Landing Page）
│   ├── not-found.jsx                     # 404 页面
│   └── globals.css                       # 全局样式
│
├── 📁 components/                        # 可复用组件
│   ├── header.jsx                        # 全局头部导航
│   ├── hero.jsx                          # 首页英雄区
│   └── 📁 ui/                            # shadcn/ui 基础组件
│       ├── button.jsx                    # 按钮组件
│       ├── card.jsx                      # 卡片组件
│       ├── input.jsx                     # 输入框组件
│       ├── select.jsx                    # 选择器组件
│       ├── table.jsx                     # 表格组件
│       ├── calendar.jsx                  # 日历组件
│       ├── dialog.jsx                    # 对话框组件
│       ├── drawer.jsx                    # 抽屉组件
│       ├── dropdown-menu.jsx             # 下拉菜单
│       ├── progress.jsx                  # 进度条
│       ├── tooltip.jsx                   # 提示框
│       ├── badge.jsx                     # 徽章
│       ├── checkbox.jsx                  # 复选框
│       ├── switch.jsx                    # 开关
│       └── sonner.jsx                    # Toast 通知
│
├── 📁 lib/                               # 全局工具库
│   ├── prisma.js                         # Prisma 客户端实例
│   ├── checkUser.js                      # 用户验证和自动创建
│   ├── arcjet.js                         # ArcJet 配置
│   ├── utils.js                          # 通用工具函数
│   └── 📁 inngest/                       # Inngest 配置
│       └── client.js                     # Inngest 客户端
│
├── 📁 prisma/                            # 数据库相关
│   ├── schema.prisma                     # Prisma 数据模型定义
│   └── 📁 migrations/                    # 数据库迁移历史
│
├── 📁 hooks/                             # 自定义 React Hooks
│   └── use-fetch.js                      # 数据获取 Hook
│
├── 📁 emails/                            # React Email 模板
│   └── template.jsx                      # 邮件模板
│
├── 📁 data/                              # 静态数据
│   └── landing.js                        # 首页展示数据
│
├── 📁 public/                            # 静态资源
│   └── logo-sm.png                       # Logo 图片
│
├── 📄 middleware.js                      # Next.js 中间件（ArcJet + Clerk）
├── 📄 next.config.mjs                    # Next.js 配置
├── 📄 tailwind.config.js                 # Tailwind CSS 配置
├── 📄 postcss.config.mjs                 # PostCSS 配置
├── 📄 components.json                    # shadcn/ui 配置
├── 📄 jsconfig.json                      # JavaScript 配置
├── 📄 .eslintrc.json                     # ESLint 配置
├── 📄 package.json                       # 项目依赖和脚本
└── 📄 .gitignore                         # Git 忽略规则
```

### 文件统计 | File Statistics

- **总文件数 | Total Files**: 64
- **Server Actions**: 6
- **API Routes**: 2
- **Pages**: 7+
- **UI Components**: 17+
- **数据模型 | Data Models**: 4

---

## 架构模式 | Architecture Pattern

### 整体架构 | Overall Architecture

**模式识别 | Pattern Identification**: **Modern App Router + Server Actions Architecture**

```
┌─────────────────────────────────────────────────────────────┐
│                     CLIENT SIDE (浏览器)                      │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  React 19 Components (app/(main)/*, components/*)     │  │
│  │  - Dashboard, Accounts, Transactions                   │  │
│  │  - shadcn/ui Components                                │  │
│  └─────────────────┬──────────────────────────────────────┘  │
└────────────────────┼───────────────────────────────────────┘
                     │
                     │ Server Actions / API Routes
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                    SERVER SIDE (Next.js)                     │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Middleware Layer                                      │  │
│  │  ├── ArcJet (Security + Bot Detection)                │  │
│  │  └── Clerk (Authentication)                           │  │
│  └─────────────────┬──────────────────────────────────────┘  │
│                    │                                          │
│  ┌────────────────┴──────────────────────────────────────┐  │
│  │  Business Logic Layer                                 │  │
│  │  ├── Server Actions (actions/*)                       │  │
│  │  │   - account.js, transaction.js, budget.js         │  │
│  │  │   - Zod Validation (app/lib/schema.js)            │  │
│  │  ├── API Routes (app/api/*)                           │  │
│  │  │   - /api/inngest - Background Jobs                 │  │
│  │  │   - /api/seed - Database Seeding                   │  │
│  │  └── Helper Functions (lib/*)                         │  │
│  └─────────────────┬──────────────────────────────────────┘  │
└────────────────────┼───────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                   DATA ACCESS LAYER                          │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Prisma ORM (lib/prisma.js)                            │  │
│  └─────────────────┬──────────────────────────────────────┘  │
└────────────────────┼───────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│              EXTERNAL SERVICES & DATABASE                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  PostgreSQL  │  │    Clerk     │  │   Gemini AI  │      │
│  │  (Supabase)  │  │    Auth      │  │   (Google)   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Inngest    │  │   Resend     │  │   ArcJet     │      │
│  │ (Background) │  │   (Email)    │  │  (Security)  │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

### 架构特点 | Architecture Characteristics

#### 1. **Server Actions First** (服务器操作优先)

- 主要使用 Server Actions 而非传统 API Routes
- 位置：`actions/` 文件夹
- 优势：类型安全、自动序列化、更少的样板代码

**示例 | Example**: `actions/dashboard.js:137-156`

#### 2. **Route Groups** (路由组织)

- `(auth)` - 认证相关页面（不需要认证）
- `(main)` - 主应用页面（需要认证）
- 好处：共享布局，不影响 URL 结构

#### 3. **Middleware Chaining** (中间件链)

**执行顺序 | Execution Order**:
```
Request → ArcJet (Security) → Clerk (Auth) → Page/Action
```

**文件位置 | File**: `middleware.js:44`

#### 4. **Database-First Development** (数据库优先开发)

- 使用 Prisma Schema 定义模型
- 自动生成 TypeScript 类型
- 迁移历史管理

**文件位置 | File**: `prisma/schema.prisma`

#### 5. **Component Colocation** (组件就近放置)

- 每个路由有自己的 `_components` 文件夹
- 只在必要时才放入全局 `components/`
- 提高代码组织性和可维护性

---

## 关键配置文件 | Key Configuration Files

### 1. **package.json** (项目依赖清单)

**位置 | Location**: `/package.json`

**关键脚本 | Key Scripts**:
```json
{
  "dev": "next dev --turbopack",      // 开发服务器（使用 Turbopack）
  "build": "next build",               // 生产构建
  "start": "next start",               // 生产服务器
  "lint": "next lint",                 // 代码检查
  "email": "email dev",                // 邮件模板开发
  "postinstall": "prisma generate"     // 自动生成 Prisma 客户端
}
```

### 2. **prisma/schema.prisma** (数据库模型)

**位置 | Location**: `/prisma/schema.prisma`

**数据源 | Data Source**:
```prisma
datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")
  directUrl = env("DIRECT_URL")
}
```

**模型数量 | Models**: 4 (User, Account, Transaction, Budget)

### 3. **middleware.js** (中间件配置)

**位置 | Location**: `/middleware.js`

**保护的路由 | Protected Routes**:
- `/dashboard(.*)`
- `/account(.*)`
- `/transaction(.*)`

**匹配器 | Matcher**: 排除静态文件和 Next.js 内部路由

### 4. **next.config.mjs** (Next.js 配置)

**位置 | Location**: `/next.config.mjs`

**关键配置 | Key Configurations**:
- **远程图片域 | Remote Image Domains**: `randomuser.me`
- **Server Actions Body Limit**: 5MB（支持大文件上传，如收据图片）

### 5. **tailwind.config.js** (Tailwind CSS 配置)

**位置 | Location**: `/tailwind.config.js`

**用途 | Purpose**:
- 自定义颜色主题
- shadcn/ui 组件样式
- 响应式断点

### 6. **components.json** (shadcn/ui 配置)

**位置 | Location**: `/components.json`

**用途 | Purpose**: 定义 UI 组件的生成规则和别名

### 7. **.eslintrc.json** (代码规范)

**位置 | Location**: `/.eslintrc.json`

**扩展 | Extends**: `next/core-web-vitals`

---

## 环境变量清单 | Environment Variables

### 必需的环境变量 | Required Variables

**参考文件 | Reference**: `README.md:9-24`

| 变量名 | 用途 | 提供商 | 必需 |
|--------|------|--------|------|
| `DATABASE_URL` | PostgreSQL 连接池 URL | Supabase | ✅ |
| `DIRECT_URL` | PostgreSQL 直连 URL | Supabase | ✅ |
| `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` | Clerk 公钥 | Clerk | ✅ |
| `CLERK_SECRET_KEY` | Clerk 密钥 | Clerk | ✅ |
| `NEXT_PUBLIC_CLERK_SIGN_IN_URL` | 登录页面路径 | - | ✅ |
| `NEXT_PUBLIC_CLERK_SIGN_UP_URL` | 注册页面路径 | - | ✅ |
| `NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL` | 登录后重定向 | - | ✅ |
| `NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL` | 注册后重定向 | - | ✅ |
| `GEMINI_API_KEY` | Google Gemini AI 密钥 | Google AI Studio | ✅ |
| `RESEND_API_KEY` | 邮件发送服务密钥 | Resend | ✅ |
| `ARCJET_KEY` | 安全防护服务密钥 | ArcJet | ✅ |
| `INNGEST_EVENT_KEY` | Inngest 事件密钥 | Inngest | 可选 |
| `INNGEST_SIGNING_KEY` | Inngest 签名密钥 | Inngest | 可选 |

### 环境变量分类 | Variable Categories

#### 🗄️ 数据库 | Database
- `DATABASE_URL` - Prisma 连接池（用于 Serverless）
- `DIRECT_URL` - 直连 URL（用于迁移）

#### 🔐 认证 | Authentication
- Clerk 相关的 6 个变量

#### 🤖 AI 服务 | AI Services
- `GEMINI_API_KEY` - 收据扫描和智能分析

#### 📧 邮件服务 | Email Service
- `RESEND_API_KEY` - 预算警报邮件

#### 🛡️ 安全服务 | Security Service
- `ARCJET_KEY` - 速率限制、Bot 检测、Shield 防护

#### ⚙️ 后台任务 | Background Jobs
- Inngest 相关变量（可选）

---

## 核心功能概览 | Core Features Overview

### 功能列表 | Feature List

#### 1. 👤 用户管理 | User Management
- **注册/登录**: Clerk 提供的安全认证
- **用户数据**: 自动同步到 PostgreSQL
- **文件位置 | Location**: `lib/checkUser.js:4-37`

#### 2. 💳 账户管理 | Account Management
- **账户类型**: 储蓄（SAVINGS）、活期（CURRENT）
- **默认账户**: 第一个账户自动设为默认
- **余额追踪**: 实时更新账户余额
- **文件位置 | Location**: `actions/account.js:54-135`

#### 3. 💸 交易管理 | Transaction Management
- **交易类型**: 收入（INCOME）、支出（EXPENSE）
- **分类系统**: 自定义交易分类
- **循环交易**: 支持每日/周/月/年自动创建
- **收据扫描**: AI 识别收据信息
- **文件位置 | Location**: `actions/transaction.js`, `app/(main)/transaction/`

#### 4. 📊 预算功能 | Budget Features
- **预算设置**: 每用户一个预算限制
- **智能提醒**: 超支时发送邮件警报
- **防止骚扰**: 记录上次提醒时间，避免重复发送
- **文件位置 | Location**: `actions/budget.js`

#### 5. 📈 数据可视化 | Data Visualization
- **仪表板**: 账户概览、交易统计
- **图表展示**: 使用 Recharts 显示趋势
- **文件位置 | Location**: `app/(main)/dashboard/`

#### 6. 🤖 AI 功能 | AI Features
- **收据扫描**: 上传收据图片，自动提取信息
- **智能分析**: 使用 Gemini AI 识别金额、日期、类别
- **文件位置 | Location**: `app/(main)/transaction/_components/recipt-scanner.jsx`

#### 7. ⚡ 后台任务 | Background Jobs
- **循环交易**: Inngest 定时处理循环交易
- **事件驱动**: 基于事件触发任务
- **文件位置 | Location**: `lib/inngest/`, `app/api/inngest/route.js`

#### 8. 🛡️ 安全功能 | Security Features
- **速率限制**: ArcJet 防止 API 滥用
- **Bot 检测**: 阻止恶意爬虫
- **Shield 防护**: 内容和安全保护
- **文件位置 | Location**: `middleware.js:12-44`

---

## 项目类型识别 | Project Type Identification

### 分类 | Classification

| 维度 | 识别结果 |
|------|----------|
| **应用类型** | SaaS Web Application（软件即服务） |
| **架构风格** | Full-Stack Monolith（全栈单体） |
| **渲染模式** | Hybrid (SSR + CSR + Server Actions) |
| **数据流** | Server-Driven（服务器驱动） |
| **状态管理** | Server State (Database-First) |
| **认证方式** | Third-Party Auth (Clerk) |
| **部署目标** | Vercel / Cloud Platform |

### 技术选型原因分析 | Technology Choice Analysis

#### ✅ 为什么选择 Next.js 15？
1. **App Router**: 文件系统路由，更直观
2. **Server Actions**: 减少 API 样板代码
3. **RSC**: React Server Components 提升性能
4. **Turbopack**: 更快的开发体验

#### ✅ 为什么选择 Prisma？
1. **类型安全**: 自动生成 TypeScript 类型
2. **迁移管理**: 版本控制数据库变更
3. **开发体验**: Prisma Studio 可视化管理

#### ✅ 为什么选择 Clerk？
1. **快速集成**: 内置 UI 组件
2. **安全性**: 企业级认证
3. **用户管理**: 完整的后台管理

#### ✅ 为什么选择 Server Actions？
1. **类型安全**: 端到端类型推断
2. **自动序列化**: 无需手动转换
3. **更少代码**: 不需要定义 API 路由

#### ✅ 为什么选择 Inngest？
1. **可靠性**: 自动重试和错误处理
2. **可观测性**: 内置监控和日志
3. **开发体验**: 本地开发工具

---

## 代码质量指标 | Code Quality Metrics

### 代码组织 | Code Organization

- ✅ **模块化**: 功能按领域分离（accounts, transactions, budgets）
- ✅ **关注点分离**: UI 组件、业务逻辑、数据访问分层清晰
- ✅ **命名规范**: 文件和函数名清晰描述功能
- ✅ **配置管理**: 环境变量集中管理

### 最佳实践应用 | Best Practices

- ✅ **输入验证**: 使用 Zod 在服务器端验证
- ✅ **错误处理**: Server Actions 中统一错误处理
- ✅ **安全性**:
  - 中间件认证检查
  - 速率限制（ArcJet）
  - 环境变量保护敏感信息
- ✅ **性能优化**:
  - Prisma 连接池复用
  - 图片优化（Next.js Image）
  - 数据库索引（userId, accountId）
- ✅ **可维护性**:
  - 清晰的目录结构
  - 组件复用
  - 类型验证

---

## 下一步分析 | Next Steps

本文档完成了项目的整体概览。接下来的分析文档：

1. ✅ **00-project-overview.md** (当前文档)
2. ⏭️ **01-database-analysis.md** - 深入分析数据模型和关系
3. ⏭️ **02-backend-analysis.md** - Server Actions 和 API 分析
4. ⏭️ **03-frontend-analysis.md** - 组件和路由分析
5. ⏭️ **04-security-analysis.md** - 安全机制详解
6. ⏭️ **05-deployment-analysis.md** - 部署配置和要求

---

**文档生成于 | Generated on**: 2025-11-15
**分析工具 | Analysis Tool**: Claude Code with Project Mastery System
**文档版本 | Document Version**: 1.0
