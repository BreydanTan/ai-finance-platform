# 部署配置分析 | Deployment Configuration Analysis
# AI Finance Platform (Welth)

**生成时间 | Generated**: 2025-11-16
**分析范围 | Scope**: Build Process, Deployment Configuration, Runtime Requirements
**平台 | Platform**: Next.js 15 Full-Stack Application

---

## 目录 | Table of Contents

1. [构建流程 | Build Process](#1-构建流程--build-process)
2. [部署配置 | Deployment Configuration](#2-部署配置--deployment-configuration)
3. [运行时要求 | Runtime Requirements](#3-运行时要求--runtime-requirements)
4. [环境变量 | Environment Variables](#4-环境变量--environment-variables)
5. [数据库设置 | Database Setup](#5-数据库设置--database-setup)
6. [部署最佳实践 | Deployment Best Practices](#部署最佳实践--deployment-best-practices)

---

## 1. 构建流程 | Build Process

### 构建脚本配置 | Build Scripts Configuration

**文件 | File**: `package.json`
**代码 | Code** (lines 5-12):

```json
"scripts": {
  "dev": "next dev --turbopack",
  "build": "next build",
  "start": "next start",
  "lint": "next lint",
  "email": "email dev",
  "postinstall": "prisma generate"
}
```

#### 关键脚本 | Key Scripts

| 脚本 | 命令 | 说明 | Description |
|------|------|------|-------------|
| **开发环境** | `npm run dev` | 使用 Turbopack 运行 Next.js，实现更快的热重载 | Runs Next.js with Turbopack for faster HMR |
| **生产构建** | `npm run build` | 创建优化的生产构建 | Creates optimized production build |
| **生产启动** | `npm start` | 启动生产服务器 | Starts production server |
| **代码检查** | `npm run lint` | 运行 ESLint 代码检查 | Runs ESLint |
| **邮件开发** | `npm run email` | 邮件模板开发模式 | Email template dev mode |
| **安装后钩子** | `postinstall` | 自动运行 `prisma generate` | Automatically runs Prisma client generation |

### 构建输出配置 | Build Output Configuration

**文件 | File**: `next.config.mjs`
**代码 | Code** (lines 1-19):

```javascript
const nextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: "https",
        hostname: "randomuser.me",
      },
    ],
  },
  experimental: {
    serverActions: {
      bodySizeLimit: "5mb",
    },
  },
};

export default nextConfig;
```

#### 配置详情 | Configuration Details

**图片优化 | Image Optimization**:
- 配置用于 randomuser.me 的远程图片 | Configured for remote images from randomuser.me
- Next.js 自动图片优化 | Automatic Next.js image optimization
- 支持 WebP/AVIF 格式 | Supports WebP/AVIF formats

**服务器操作 | Server Actions**:
- 请求体大小限制增加到 5MB | Body size limit increased to 5MB
- 用于支持收据图片上传 | For receipt image uploads
- 默认值为 1MB | Default is 1MB

**输出 | Output**:
- 标准 Next.js 输出到 `.next/` 目录 | Standard output to `.next/` directory
- 生产构建包含优化的代码分割 | Production build includes optimized code splitting

### 优化设置 | Optimization Settings

#### PostCSS 配置 | PostCSS Configuration

**文件 | File**: `postcss.config.mjs`
**代码 | Code** (lines 1-9):

```javascript
const config = {
  plugins: {
    tailwindcss: {},
  },
};

export default config;
```

#### Tailwind CSS 配置 | Tailwind Configuration

**文件 | File**: `tailwind.config.js`
**代码 | Code** (lines 1-62):

**内容清除配置 | Content Purging** (lines 6-14):
```javascript
content: [
  "./pages/**/*.{js,ts,jsx,tsx,mdx}",
  "./components/**/*.{js,ts,jsx,tsx,mdx}",
  "./app/**/*.{js,ts,jsx,tsx,mdx}",
],
```

**优化效果 | Optimization Benefits**:
- 自动移除未使用的 CSS | Automatically removes unused CSS
- 显著减小 CSS 包大小 | Significantly reduces CSS bundle size
- 仅包含实际使用的样式 | Only includes actually used styles

**主题定制 | Theme Customization**:
- 使用 CSS 变量实现主题 | Uses CSS variables for theming
- 支持暗色模式 | Dark mode support
- 自定义颜色方案 | Custom color schemes

#### ESLint 配置 | ESLint Configuration

**文件 | File**: `.eslintrc.json`
**代码 | Code** (lines 1-6):

```json
{
  "extends": "next/core-web-vitals",
  "rules": {
    "no-unused-vars": ["warn"]
  }
}
```

**规则 | Rules**:
- 继承 Next.js 核心 Web Vitals 规则 | Extends Next.js core web vitals
- 未使用变量警告而非错误 | Unused vars as warnings, not errors

### 环境特定构建 | Environment-Specific Builds

#### 开发环境 | Development

**特性 | Features**:
- **Turbopack** 启用，实现更快的 HMR | Enabled for faster HMR
- **Prisma 客户端缓存** 通过 globalThis | Cached via globalThis

**文件 | File**: `lib/prisma.js` (lines 5-7):
```javascript
const globalForPrisma = globalThis;
globalForPrisma.prisma = globalForPrisma.prisma || new PrismaClient();
```

#### 生产环境 | Production

**自动优化 | Automatic Optimizations**:
- ✅ 代码分割 | Code splitting
- ✅ Tree shaking
- ✅ 图片优化 | Image optimization
- ✅ 字体优化 | Font optimization
- ✅ 静态页面生成（可能的情况下）| Static optimization where possible
- ✅ 服务器组件优化 | Server component optimization

---

## 2. 部署配置 | Deployment Configuration

### 当前状态 | Current Status

**已检测 | Detected**:
- ✅ Next.js 15 应用 | Next.js 15 application
- ✅ `.gitignore` 中包含 Vercel 配置 | Vercel references in `.gitignore` (line 36)

**缺失的部署文件 | Missing Deployment Files**:
- ❌ 无 Dockerfile
- ❌ 无 docker-compose.yml
- ❌ 无 `.github/workflows` CI/CD
- ❌ 无 `vercel.json`（将使用默认值）| Will use defaults
- ❌ 无 `netlify.toml`

### 推荐平台 | Recommended Platform

**最佳选择 | Best Choice**: Vercel
- Next.js 原生支持 | Native Next.js support
- 自动部署 | Automatic deployments
- 边缘函数 | Edge functions
- 图片优化 | Image optimization
- 分析功能 | Analytics

### 推荐的 Vercel 配置 | Recommended Vercel Configuration

**创建文件 | Create**: `vercel.json`

```json
{
  "buildCommand": "npm run build",
  "devCommand": "npm run dev",
  "installCommand": "npm install",
  "framework": "nextjs",
  "rewrites": [
    {
      "source": "/api/inngest/:path*",
      "destination": "/api/inngest/:path*"
    }
  ],
  "env": {
    "DATABASE_URL": "@database-url",
    "DIRECT_URL": "@direct-url"
  }
}
```

**配置说明 | Configuration Notes**:
- `buildCommand`: 生产构建命令 | Production build command
- `framework`: 自动检测为 Next.js | Auto-detected as Next.js
- `rewrites`: Inngest webhook 路由 | Inngest webhook routing
- `env`: 环境变量引用 | Environment variable references

### 推荐的 Docker 配置 | Recommended Docker Configuration

#### Dockerfile（多阶段构建）| Multi-Stage Build

**创建文件 | Create**: `Dockerfile`

```dockerfile
# 依赖阶段 | Dependencies stage
FROM node:22-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

# 构建阶段 | Builder stage
FROM node:22-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# 生成 Prisma 客户端 | Generate Prisma client
RUN npx prisma generate

# 构建 Next.js | Build Next.js
RUN npm run build

# 运行阶段 | Runner stage
FROM node:22-alpine AS runner
WORKDIR /app

ENV NODE_ENV=production

# 复制必要文件 | Copy necessary files
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/prisma ./prisma

EXPOSE 3000
ENV PORT 3000

CMD ["node", "server.js"]
```

**优化说明 | Optimization Notes**:
- 多阶段构建减小最终镜像大小 | Multi-stage reduces final image size
- 仅复制必要文件到运行阶段 | Only copies necessary files to runner
- 使用 Alpine Linux 减小基础镜像 | Alpine Linux for smaller base image

#### Docker Compose 配置

**创建文件 | Create**: `docker-compose.yml`

```yaml
version: '3.8'

services:
  # 应用服务 | Application service
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      # 数据库 | Database
      - DATABASE_URL=${DATABASE_URL}
      - DIRECT_URL=${DIRECT_URL}

      # 认证 | Authentication
      - NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=${NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY}
      - CLERK_SECRET_KEY=${CLERK_SECRET_KEY}

      # AI 服务 | AI Service
      - GEMINI_API_KEY=${GEMINI_API_KEY}

      # 邮件服务 | Email Service
      - RESEND_API_KEY=${RESEND_API_KEY}

      # 安全 | Security
      - ARCJET_KEY=${ARCJET_KEY}
    depends_on:
      - db

  # PostgreSQL 数据库 | PostgreSQL database
  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=finance
      - POSTGRES_PASSWORD=finance_password
      - POSTGRES_DB=finance_platform
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  postgres_data:
```

**使用方法 | Usage**:
```bash
# 启动所有服务 | Start all services
docker-compose up -d

# 查看日志 | View logs
docker-compose logs -f

# 停止服务 | Stop services
docker-compose down
```

### 推荐的 GitHub Actions CI/CD

**创建文件 | Create**: `.github/workflows/deploy.yml`

```yaml
name: Deploy to Production

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  # 测试作业 | Test job
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '22'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Lint code
        run: npm run lint

      - name: Generate Prisma client
        run: npx prisma generate

      - name: Build application
        run: npm run build

  # 部署作业 | Deploy job
  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v20
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.ORG_ID }}
          vercel-project-id: ${{ secrets.PROJECT_ID }}
          vercel-args: '--prod'
```

**配置 GitHub Secrets | Configure Secrets**:
- `VERCEL_TOKEN`: Vercel API token
- `ORG_ID`: Vercel organization ID
- `PROJECT_ID`: Vercel project ID

---

## 3. 运行时要求 | Runtime Requirements

### Node.js 要求 | Node.js Requirements

**当前版本 | Current Version**: v22.21.1（系统检测）| Detected on system

**推荐版本 | Recommended**: Node.js 18.x 或更高 | Node.js 18.x or higher
- Next.js 15 要求 | Next.js 15 requirement

**包管理器 | Package Manager**: npm
- 存在 package-lock.json | package-lock.json present

**依赖 | Dependencies** (`package.json` lines 34-36):
```json
"next": "15.0.3",
"react": "^19.0.0-rc-66855b96-20241106",
"react-dom": "^19.0.0-rc-66855b96-20241106"
```

#### 版本管理 | Version Management

**推荐创建 | Recommended**: `.nvmrc` 文件

```bash
22.21.1
```

**使用方法 | Usage**:
```bash
# 使用项目指定的 Node 版本 | Use project-specific Node version
nvm use
```

### 数据库要求 | Database Requirements

**文件 | File**: `prisma/schema.prisma` (lines 5-9)

```prisma
datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")
  directUrl = env("DIRECT_URL")
}
```

**数据库 | Database**: PostgreSQL

**版本要求 | Version Requirements**:
- **推荐 | Recommended**: PostgreSQL 12+
- **兼容 | Compatible**: PostgreSQL 10+

**连接配置 | Connection Configuration**:
- `DATABASE_URL`: 池化连接，用于常规操作 | Pooled connection for regular operations
- `DIRECT_URL`: 直接连接，用于迁移 | Direct connection for migrations

**连接池 | Connection Pooling**:
- 推荐使用 Supabase 的 PgBouncer | Recommended: Supabase PgBouncer
- 或使用 PgPool-II

**锁定文件 | Lock File**: `prisma/migrations/migration_lock.toml` (line 3)
```toml
provider = "postgresql"
```

### 系统依赖 | System Dependencies

#### 必需 | Required

| 依赖 | 说明 | Description |
|------|------|-------------|
| Node.js 18+ | JavaScript 运行时 | JavaScript runtime |
| PostgreSQL 客户端库 | 数据库连接 | Database connection |
| OpenSSL | Prisma 加密 | For Prisma encryption |

#### 可选 | Optional

| 依赖 | 用途 | Purpose |
|------|------|---------|
| PM2 | 生产环境进程管理器 | Process manager for production |
| Nginx | 反向代理 | Reverse proxy |
| Redis | 未来的缓存/会话（当前未使用）| For future caching/sessions |

### 外部服务依赖 | External Service Dependencies

#### 1. Clerk 认证 | Authentication

**文件 | Files**:
- `middleware.js` (lines 1-2, 32)
- `lib/checkUser.js`

**用途 | Usage**: 用户认证和会话管理 | User authentication and session management

**必需环境变量 | Required Environment Variables**:
```bash
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_xxxxx
CLERK_SECRET_KEY=sk_test_xxxxx
```

**功能 | Features**:
- 用户注册/登录 | User sign-up/sign-in
- 会话管理 | Session management
- 用户资料 | User profiles
- 社交登录（可选）| Social logins (optional)

#### 2. Resend 邮件服务 | Email Service

**文件 | File**: `actions/send-email.js` (lines 3-21)

**用途 | Usage**: 交易邮件（月度报告、预算警报）| Transactional emails

**必需 | Required**: `RESEND_API_KEY=re_xxxxx`

**发送内容 | Sends**:
- 月度财务报告 | Monthly financial reports
- 预算超支警报 | Budget overage alerts
- 交易通知 | Transaction notifications

**默认发件人 | Default Sender**: `onboarding@resend.dev`

#### 3. Google Gemini AI 服务

**文件 | Files**:
- `lib/inngest/function.js` (lines 5, 130-131)
- `actions/transaction.js` (收据扫描 | Receipt scanning)

**用途 | Usage**:
- 财务洞察生成 | Financial insights generation
- 收据 OCR 和数据提取 | Receipt OCR and data extraction

**模型 | Model**: `gemini-1.5-flash`

**必需 | Required**: `GEMINI_API_KEY=xxxxx`

**功能 | Features**:
- 收据扫描和解析 | Receipt scanning and parsing
- AI 生成的财务建议 | AI-generated financial advice
- 月度报告洞察 | Monthly report insights

#### 4. ArcJet 安全服务 | Security Service

**文件 | Files**:
- `middleware.js` (lines 1, 12-13)
- `lib/arcjet.js` (lines 3-4)

**用途 | Usage**:
- 机器人检测 | Bot detection
- 防护盾保护 | Shield protection
- 速率限制 | Rate limiting

**必需 | Required**: `ARCJET_KEY=ajkey_xxxxx`

**保护内容 | Protects**:
- API 端点 | API endpoints
- 表单提交 | Form submissions
- 账户创建 | Account creation

**速率限制配置 | Rate Limit Configuration** (`actions/dashboard.js` lines 63-83):
- 创建账户: 10 次/小时 | Account creation: 10/hour

#### 5. Inngest 后台作业 | Background Jobs

**文件 | Files**:
- `lib/inngest/client.js` (lines 1-10)
- `app/api/inngest/route.js` (lines 1-19)
- `lib/inngest/function.js`

**用途 | Usage**: 计划任务和后台处理 | Scheduled tasks and background processing

**开发环境 | Development**: 无需 API 密钥 | No API key needed

**生产环境 | Production**: 配置 webhook 端点 | Configure webhook endpoint
```
https://yourdomain.com/api/inngest
```

**计划作业 | Scheduled Jobs**:

| 作业 | 计划 | 说明 | Description |
|------|------|------|-------------|
| 触发循环交易 | 每日午夜 | 创建循环交易 | Trigger recurring transactions |
| 生成月度报告 | 每月 1 日 | AI 生成的财务报告 | Generate monthly financial reports |
| 检查预算警报 | 每 6 小时 | 检查预算超支 | Check budget overage |

**重试策略 | Retry Strategy**:
- 指数退避 | Exponential backoff
- 最大重试次数: 3 | Max retries: 3

---

## 4. 环境变量 | Environment Variables

### 完整环境变量列表 | Complete Environment Variable List

**参考 | Reference**: `README.md` (lines 6-24)

#### 数据库配置 | Database Configuration

```bash
# 池化连接（用于应用查询）| Pooled connection (for app queries)
DATABASE_URL=postgresql://user:password@host:5432/database?pgbouncer=true

# 直接连接（用于 Prisma 迁移）| Direct connection (for Prisma migrations)
DIRECT_URL=postgresql://user:password@host:5432/database
```

**说明 | Notes**:
- `DATABASE_URL`: 使用连接池（PgBouncer）| Uses connection pooling
- `DIRECT_URL`: 直接数据库连接，用于迁移 | Direct DB connection for migrations
- 两者都指向同一数据库 | Both point to the same database

#### 认证配置 | Authentication Configuration (Clerk)

```bash
# 公开密钥（客户端使用）| Publishable key (client-side)
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_xxxxx

# 秘密密钥（服务器端使用）| Secret key (server-side)
CLERK_SECRET_KEY=sk_test_xxxxx

# 路由配置 | Route configuration
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/onboarding
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/onboarding
```

#### AI 服务配置 | AI Service Configuration (Google Gemini)

```bash
GEMINI_API_KEY=xxxxx
```

#### 邮件服务配置 | Email Service Configuration (Resend)

```bash
RESEND_API_KEY=re_xxxxx
```

#### 安全配置 | Security Configuration (ArcJet)

```bash
ARCJET_KEY=ajkey_xxxxx
```

### 环境特定配置 | Environment-Specific Configuration

#### 开发环境 | Development (`.env.local`)

```bash
# ==========================================
# 开发环境配置 | Development Configuration
# ==========================================

# 数据库 - 本地 PostgreSQL 或 Supabase | Database - Local or Supabase
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/finance_dev?pgbouncer=true
DIRECT_URL=postgresql://postgres:postgres@localhost:5432/finance_dev

# Clerk - 开发密钥 | Development keys
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_xxxxx
CLERK_SECRET_KEY=sk_test_xxxxx
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/onboarding
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/onboarding

# 开发 API 密钥 | Development API keys
GEMINI_API_KEY=your_dev_key
RESEND_API_KEY=re_dev_xxxxx
ARCJET_KEY=ajkey_dev_xxxxx

# Node 环境 | Node environment
NODE_ENV=development
```

#### 预发布环境 | Staging (`.env.staging`)

```bash
# ==========================================
# 预发布环境配置 | Staging Configuration
# ==========================================

# 数据库 - 预发布 Supabase | Database - Staging Supabase
DATABASE_URL=postgresql://user:pass@staging-db.supabase.co:5432/postgres?pgbouncer=true
DIRECT_URL=postgresql://user:pass@staging-db.supabase.co:5432/postgres

# Clerk - 预发布实例 | Staging instance
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_staging_xxxxx
CLERK_SECRET_KEY=sk_test_staging_xxxxx
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/onboarding
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/onboarding

# 预发布 API 密钥 | Staging API keys
GEMINI_API_KEY=your_staging_key
RESEND_API_KEY=re_staging_xxxxx
ARCJET_KEY=ajkey_staging_xxxxx

NODE_ENV=production
```

#### 生产环境 | Production (`.env.production`)

```bash
# ==========================================
# 生产环境配置 | Production Configuration
# ==========================================

# 数据库 - 生产 Supabase（带连接池）| Database - Production with pooling
DATABASE_URL=postgresql://user:pass@prod-db.supabase.co:5432/postgres?pgbouncer=true
DIRECT_URL=postgresql://user:pass@prod-db.supabase.co:5432/postgres

# Clerk - 生产密钥 | Production keys
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_live_xxxxx
CLERK_SECRET_KEY=sk_live_xxxxx
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/onboarding
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/onboarding

# 生产 API 密钥 | Production API keys
GEMINI_API_KEY=your_production_key
RESEND_API_KEY=re_prod_xxxxx
ARCJET_KEY=ajkey_prod_xxxxx

NODE_ENV=production
```

### 如何获取 API 密钥和密钥 | How to Obtain API Keys and Secrets

#### 1. 数据库（Supabase）| Database

**步骤 | Steps**:

1. 访问 | Visit: https://supabase.com
2. 创建新项目 | Create new project
3. 导航到 Settings > Database
4. 复制 "Connection string" → `DATABASE_URL`
5. 复制 "Direct connection string" → `DIRECT_URL`
6. 启用连接池（PgBouncer）| Enable connection pooling

**连接字符串格式 | Connection String Format**:
```
postgresql://postgres:[YOUR-PASSWORD]@db.[PROJECT-REF].supabase.co:5432/postgres?pgbouncer=true
```

#### 2. Clerk 认证 | Clerk Authentication

**步骤 | Steps**:

1. 访问 | Visit: https://clerk.com
2. 创建应用 | Create application
3. Dashboard > API Keys
4. 复制 "Publishable key" → `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`
5. 复制 "Secret key" → `CLERK_SECRET_KEY`
6. 在 Clerk 仪表板中配置重定向 URL | Configure redirect URLs in dashboard

**重定向 URL 配置 | Redirect URL Configuration**:
- Sign-in URL: `/sign-in`
- Sign-up URL: `/sign-up`
- After sign-in: `/onboarding`
- After sign-up: `/onboarding`

#### 3. Google Gemini API

**步骤 | Steps**:

1. 访问 | Visit: https://makersuite.google.com/app/apikey
2. 创建 API 密钥 | Create API key
3. 复制密钥 → `GEMINI_API_KEY`
4. 在 Google Cloud Console 中启用 Gemini API | Enable Gemini API in Google Cloud Console

**定价 | Pricing**:
- Gemini 1.5 Flash: 免费层可用 | Free tier available
- 查看最新定价 | Check latest pricing at: https://ai.google.dev/pricing

#### 4. Resend 邮件 | Resend Email

**步骤 | Steps**:

1. 访问 | Visit: https://resend.com
2. 注册并验证域名 | Sign up and verify domain
3. API Keys 部分 | API Keys section
4. 创建 API 密钥 → `RESEND_API_KEY`

**域名验证 | Domain Verification**:
- 添加 DNS 记录以验证您的域名 | Add DNS records to verify your domain
- 或使用默认发件人 | Or use default sender: `onboarding@resend.dev`

**定价 | Pricing**:
- 免费: 100 封邮件/天 | Free: 100 emails/day
- Pro: $20/月，50,000 封邮件 | Pro: $20/month for 50,000 emails

#### 5. ArcJet 安全 | ArcJet Security

**步骤 | Steps**:

1. 访问 | Visit: https://arcjet.com
2. 创建账户和站点 | Create account and site
3. 复制 API 密钥 → `ARCJET_KEY`
4. 在仪表板中配置机器人检测规则 | Configure bot detection rules in dashboard

**规则配置 | Rule Configuration**:
- Bot detection: Enabled
- Shield: Enabled
- Rate limiting: 10 requests/hour for account creation

#### 6. Inngest 后台作业 | Inngest Background Jobs

**步骤 | Steps**:

1. 访问 | Visit: https://inngest.com
2. 创建账户 | Create account
3. **开发环境**: 无需 API 密钥 | **Development**: No API key needed
4. **生产环境**: 配置 webhook 端点 | **Production**: Configure webhook endpoint
   ```
   https://yourdomain.com/api/inngest
   ```

**Webhook 设置 | Webhook Setup**:
- 在 Inngest 仪表板中添加 webhook URL | Add webhook URL in Inngest dashboard
- 验证连接 | Verify connection
- 查看计划作业 | View scheduled jobs

---

## 5. 数据库设置 | Database Setup

### 数据库架构概述 | Database Schema Overview

**文件 | File**: `prisma/schema.prisma`

#### 数据模型 | Data Models

| 模型 | 代码行 | 说明 | Description |
|------|--------|------|-------------|
| **User** | 11-24 | Clerk 集成、邮箱、个人资料 | Clerk integration, email, profile |
| **Account** | 26-40 | 财务账户（活期/储蓄）| Financial accounts (CURRENT/SAVINGS) |
| **Transaction** | 42-65 | 收入/支出交易，支持循环 | Income/Expense with recurring support |
| **Budget** | 68-79 | 月度预算跟踪，带警报 | Monthly budget tracking with alerts |

#### 枚举类型 | Enums

```prisma
enum TransactionType {
  INCOME
  EXPENSE
}

enum AccountType {
  CURRENT  // 活期 | Current account
  SAVINGS  // 储蓄 | Savings account
}

enum TransactionStatus {
  PENDING
  COMPLETED
  FAILED
}

enum RecurringInterval {
  DAILY
  WEEKLY
  MONTHLY
  YEARLY
}
```

### 迁移系统 | Migration System

**迁移目录 | Migration Directory**: `prisma/migrations/`

**总计 | Total**: 9 个迁移 | 9 migrations

#### 迁移历史 | Migration History

| 时间戳 | 名称 | 说明 | Description |
|--------|------|------|-------------|
| 20241204141034 | init | 初始架构 | Initial schema |
| 20241205074927 | remove_currency | 移除货币字段 | Removed currency field |
| 20241205094020 | remove_categories | 移除分类表 | Removed categories table |
| 20241205094352 | remove_categories | 分类清理 | Categories cleanup |
| 20241206121749 | budget | 预算模型更改 | Budget model changes |
| 20241208092553 | budget | 预算更新 | Budget updates |
| 20241208122341 | budget | 预算优化 | Budget refinements |
| 20241209133842 | remove | 移除备注字段 | Removed notes field |

### 数据库初始化流程 | Database Initialization Process

#### 步骤 1: 安装依赖 | Install Dependencies

```bash
cd /home/user/ai-finance-platform
npm install
```

**自动执行 | Auto-executes**: `postinstall` 钩子运行 `prisma generate`

#### 步骤 2: 配置环境 | Configure Environment

```bash
# 创建 .env 文件 | Create .env file
touch .env

# 添加数据库 URL | Add database URLs
echo 'DATABASE_URL="postgresql://user:password@localhost:5432/finance_platform?pgbouncer=true"' >> .env
echo 'DIRECT_URL="postgresql://user:password@localhost:5432/finance_platform"' >> .env
```

#### 步骤 3: 生成 Prisma 客户端 | Generate Prisma Client

```bash
npx prisma generate
```

**输出 | Output**:
```
✔ Generated Prisma Client (v6.0.1) to ./node_modules/@prisma/client
```

#### 步骤 4: 运行迁移 | Run Migrations

**生产环境 | Production** (应用所有迁移 | Apply all migrations):
```bash
npx prisma migrate deploy
```

**开发环境 | Development** (如果架构更改则创建迁移 | Creates migration if schema changed):
```bash
npx prisma migrate dev
```

**输出示例 | Example Output**:
```
Applying migration `20241204141034_init`
Applying migration `20241205074927_remove_currency`
...
Database synchronized with schema
```

#### 步骤 5: 验证数据库 | Verify Database

```bash
# 打开 Prisma Studio 查看数据库 | Open Prisma Studio
npx prisma studio
```

**访问 | Access**: http://localhost:5555

### 种子脚本 | Seeding Scripts

#### 种子函数 | Seed Function

**文件 | File**: `actions/seed.js` (lines 1-110)

**功能 | Features**:
- 生成 90 天的示例交易数据 | Generates 90 days of sample transaction data
- 每天 1-3 笔交易 | 1-3 transactions per day
- 40% 收入概率，60% 支出概率 | 40% income, 60% expense probability
- 使用预定义分类 | Uses predefined categories

**分类数据 | Categories**: `data/categories.js` (lines 1-168)
- 6 个收入分类 | 6 income categories
  - 工资 | Salary
  - 自由职业 | Freelance
  - 投资 | Investments
  - 副业 | Side Hustle
  - 礼物 | Gifts
  - 其他 | Other
- 15 个支出分类 | 15 expense categories
  - 住房、交通、食品杂货、等等 | Housing, Transportation, Groceries, etc.

#### 种子端点 | Seed Endpoint

**文件 | File**: `app/api/seed/route.js`

**使用方法 | Usage**:
```bash
# 通过 HTTP 触发种子 | Trigger seeding via HTTP
curl http://localhost:3000/api/seed
```

#### 手动种子 | Manual Seeding

**创建 | Create**: `prisma/seed.js`

```javascript
const { PrismaClient } = require('@prisma/client');
const prisma = new PrismaClient();

async function main() {
  // 在此处添加初始数据 | Seed initial data here

  // 示例：创建测试用户 | Example: Create test user
  const user = await prisma.user.create({
    data: {
      clerkUserId: 'test_user_123',
      email: 'test@example.com',
      name: 'Test User',
    },
  });

  console.log('Seeding complete:', user);
}

main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

**添加到 package.json | Add to package.json**:
```json
"prisma": {
  "seed": "node prisma/seed.js"
}
```

**运行 | Run**:
```bash
npx prisma db seed
```

### 数据库备份流程 | Database Backup Procedures

#### 1. PostgreSQL 原生备份 | PostgreSQL Native Backup

**完整数据库备份 | Full Database Backup**:
```bash
pg_dump -h localhost -U postgres -d finance_platform \
  -F c -b -v -f backup_$(date +%Y%m%d).dump
```

**参数说明 | Parameters**:
- `-F c`: 自定义格式（压缩）| Custom format (compressed)
- `-b`: 包含大对象 | Include large objects
- `-v`: 详细输出 | Verbose output
- `-f`: 输出文件 | Output file

**恢复备份 | Restore from Backup**:
```bash
pg_restore -h localhost -U postgres -d finance_platform \
  -v backup_20241116.dump
```

#### 2. Supabase 备份 | Supabase Backups

**自动备份 | Automatic Backups**:
- 每日自动备份（付费计划）| Daily automatic backups (on paid plans)
- 通过 Supabase Dashboard > Database > Backups 访问 | Access via dashboard
- 时间点恢复可用 | Point-in-time recovery available

**手动备份 | Manual Backup**:
```bash
# 使用 Supabase CLI | Using Supabase CLI
supabase db dump -f backup.sql
```

#### 3. 基于迁移的备份 | Migration-Based Backup

**导出架构 | Export Schema**:
```bash
npx prisma db pull
```

**保存迁移目录 | Save Migrations Directory**:
```bash
tar -czf migrations_backup_$(date +%Y%m%d).tar.gz prisma/migrations/
```

#### 4. 数据导出 | Data Export

**使用 Prisma Studio | Using Prisma Studio**:
1. 运行 `npx prisma studio`
2. 选择表 | Select table
3. 导出为 CSV | Export to CSV

**使用 SQL | Using SQL**:
```bash
# 导出特定表 | Export specific table
pg_dump -h localhost -U postgres -d finance_platform \
  -t users --data-only -f users_backup.sql
```

### 数据库恢复流程 | Database Restore Procedures

#### 1. 全新安装 | Fresh Installation

```bash
# 重置数据库（删除所有数据）| Reset database (deletes all data)
npx prisma migrate reset

# 应用所有迁移 | Apply all migrations
npx prisma migrate deploy

# 种子数据（可选）| Seed data (optional)
curl http://localhost:3000/api/seed
```

#### 2. 从 PostgreSQL 备份恢复 | Restore from PostgreSQL Backup

```bash
# 删除现有数据库 | Drop existing database
dropdb finance_platform

# 创建新数据库 | Create new database
createdb finance_platform

# 从备份恢复 | Restore from backup
pg_restore -d finance_platform backup_20241116.dump

# 重新生成 Prisma 客户端 | Regenerate Prisma client
npx prisma generate
```

#### 3. 灾难恢复 | Disaster Recovery

```bash
# 1. 从备份恢复数据库 | Restore database from backup
pg_restore -h production-host -U postgres \
  -d finance_platform latest_backup.dump

# 2. 验证迁移是否同步 | Verify migrations are in sync
npx prisma migrate status

# 3. 应用任何待处理的迁移 | Apply any pending migrations
npx prisma migrate deploy

# 4. 验证数据完整性 | Verify data integrity
npx prisma studio
```

---

## 部署最佳实践 | Deployment Best Practices

### 部署前检查清单 | Pre-Deployment Checklist

#### 环境配置 | Environment Configuration

- [ ] 所有环境变量已配置 | All environment variables configured
- [ ] 数据库连接已测试 | Database connection tested
- [ ] API 密钥已验证 | API keys validated
- [ ] Clerk 认证已配置 | Clerk authentication configured
- [ ] Inngest webhook 端点已注册 | Inngest webhook endpoint registered

#### 安全性 | Security

- [ ] 环境变量未提交到 git | Environment variables not committed to git
- [ ] ArcJet 规则适当配置 | ArcJet rules configured appropriately
- [ ] 速率限制已测试 | Rate limiting tested
- [ ] CORS 配置（如需要）| CORS configured if needed
- [ ] 数据库连接使用 SSL | Database connection uses SSL

#### 性能 | Performance

- [ ] 数据库索引已验证 | Database indexes verified (check schema.prisma)
- [ ] 图片优化已配置 | Image optimization configured
- [ ] 构建大小已分析 | Build size analyzed (`npm run build`)
- [ ] Lighthouse 审计已通过 | Lighthouse audit passed

#### 数据库 | Database

- [ ] 迁移已测试 | Migrations tested
- [ ] 备份策略已就位 | Backup strategy in place
- [ ] 连接池已启用 | Connection pooling enabled
- [ ] 直接 URL 已配置用于迁移 | Direct URL configured for migrations

### 推荐的部署步骤 | Recommended Deployment Steps

#### 1. 初始设置 | Initial Setup

```bash
# 克隆仓库 | Clone repository
git clone <repository-url>
cd ai-finance-platform

# 安装依赖 | Install dependencies
npm install

# 设置环境 | Setup environment
cp .env.example .env
# 编辑 .env 填入您的值 | Edit .env with your values

# 生成 Prisma 客户端 | Generate Prisma client
npx prisma generate

# 运行迁移 | Run migrations
npx prisma migrate deploy
```

#### 2. 构建应用 | Build Application

```bash
# 生产构建 | Build for production
npm run build

# 本地测试生产构建 | Test production build locally
npm start
```

**验证构建输出 | Verify Build Output**:
```
Route (app)                              Size     First Load JS
┌ ○ /                                    5.89 kB         142 kB
├ ○ /_not-found                          871 B          87.2 kB
├ ƒ /account/[id]                        137 B           141 kB
├ ○ /dashboard                           12.1 kB         156 kB
└ ƒ /transaction/create                  137 B           141 kB
```

#### 3. 部署到 Vercel | Deploy to Vercel (推荐 | Recommended)

```bash
# 安装 Vercel CLI | Install Vercel CLI
npm i -g vercel

# 登录 | Login
vercel login

# 部署 | Deploy
vercel --prod
```

**在 Vercel 仪表板中配置 | Configure in Vercel Dashboard**:
1. 环境变量 | Environment Variables
2. 自定义域名 | Custom domains
3. Inngest webhook: `https://your-domain.com/api/inngest`

#### 4. 部署后验证 | Post-Deployment Verification

```bash
# 测试端点 | Test endpoints
curl https://your-domain.com/

# 验证 API | Verify API
curl https://your-domain.com/api/health

# 测试认证流程 | Test authentication flow
# 访问 | Visit: https://your-domain.com/sign-in

# 验证 Inngest 集成 | Verify Inngest integration
# 访问 Inngest 仪表板确认 webhook | Visit Inngest dashboard to confirm webhook
```

**功能测试 | Functional Tests**:
- [ ] 用户注册/登录 | User sign-up/sign-in
- [ ] 创建账户 | Create account
- [ ] 添加交易 | Add transaction
- [ ] 上传收据（AI 扫描）| Upload receipt (AI scanning)
- [ ] 设置预算 | Set budget
- [ ] 接收邮件通知 | Receive email notification

### 监控建议 | Monitoring Recommendations

#### 应用监控 | Application Monitoring

**Vercel Analytics**:
- 页面访问量 | Page views
- 性能指标 | Performance metrics
- 错误率 | Error rates

**错误跟踪 | Error Tracking** (推荐 Sentry):
```bash
npm install @sentry/nextjs
```

**API 响应时间 | API Response Times**:
- 监控服务器操作 | Monitor server actions
- 跟踪慢查询 | Track slow queries

#### 数据库监控 | Database Monitoring

**Supabase 仪表板指标 | Supabase Dashboard Metrics**:
- 连接数 | Connection count
- 查询性能 | Query performance
- 存储使用 | Storage usage

**查询性能监控 | Query Performance Monitoring**:
```sql
-- 慢查询日志 | Slow query log
SELECT * FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;
```

**连接池利用率 | Connection Pool Utilization**:
- 监控活动连接 | Monitor active connections
- 设置警报 | Set up alerts

#### 后台作业监控 | Background Jobs Monitoring

**Inngest 仪表板 | Inngest Dashboard**:
- 作业执行状态 | Job execution status
- 失败和重试 | Failures and retries
- 执行时长 | Execution duration

**邮件发送跟踪 | Email Delivery Tracking**:
- Resend 仪表板 | Resend dashboard
- 发送成功率 | Delivery success rate
- 退信和投诉 | Bounces and complaints

**预算警报验证 | Budget Alert Verification**:
- 测试警报触发 | Test alert triggers
- 验证邮件发送 | Verify email sending

#### 安全监控 | Security Monitoring

**ArcJet 仪表板 | ArcJet Dashboard**:
- 机器人检测事件 | Bot detection events
- 被阻止的请求 | Blocked requests
- 速率限制命中 | Rate limit hits

**定期安全审计 | Regular Security Audits**:
```bash
# 依赖项漏洞检查 | Check for dependency vulnerabilities
npm audit

# 更新依赖项 | Update dependencies
npm update

# 检查过时的包 | Check for outdated packages
npm outdated
```

---

## 总结 | Summary

### 技术栈概述 | Tech Stack Overview

**运行时 | Runtime**:
- Node.js 22+
- PostgreSQL 12+

**外部服务 | External Services**:
- **Clerk**: 认证 | Authentication
- **Supabase**: 数据库 | Database
- **Resend**: 邮件 | Email
- **Google Gemini**: AI
- **ArcJet**: 安全 | Security
- **Inngest**: 后台作业 | Background Jobs

**部署平台 | Deployment Platform**:
- 优化为 Vercel | Optimized for Vercel
- Docker-ready（提供配置）| Docker-ready with provided configurations

**数据库 | Database**:
- Prisma ORM
- 9 个迁移 | 9 migrations
- 自动种子可用 | Automated seeding available

### 关键文件 | Key Files

| 文件 | 说明 | Description |
|------|------|-------------|
| `package.json` | 依赖项和脚本 | Dependencies & scripts |
| `next.config.mjs` | Next.js 配置 | Next.js configuration |
| `prisma/schema.prisma` | 数据库架构 | Database schema |
| `middleware.js` | 认证和安全 | Auth & security |
| `lib/inngest/function.js` | 后台作业 | Background jobs |

### 缺失的配置（推荐创建）| Missing Configurations (Recommended)

- [ ] `Dockerfile`
- [ ] `docker-compose.yml`
- [ ] `.github/workflows/deploy.yml`
- [ ] `vercel.json`
- [ ] `.nvmrc`

### 快速开始命令 | Quick Start Commands

```bash
# 开发 | Development
npm install
npx prisma generate
npx prisma migrate deploy
npm run dev

# 生产 | Production
npm run build
npm start

# 部署 | Deployment
vercel --prod
```

---

**分析完成 | Analysis Complete** ✅

此文档提供了部署和运行 AI Finance Platform 所需的完整配置信息。

This comprehensive analysis provides all necessary information to deploy and run the AI Finance Platform.