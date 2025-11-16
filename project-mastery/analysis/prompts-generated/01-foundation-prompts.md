# 基础设置提示词 | Foundation Setup Prompts
# AI Finance Platform 重建指南 | Rebuild Guide

**阶段 | Phase**: 1 - 项目基础 | Foundation
**预计时间 | Estimated Time**: 30-45 分钟 | 30-45 minutes

---

## 目录 | Table of Contents

1. [提示词 1.1: 初始化项目](#提示词-11-初始化项目--prompt-11-initialize-project)
2. [提示词 1.2: 配置数据库](#提示词-12-配置数据库--prompt-12-configure-database)
3. [提示词 1.3: 设置认证](#提示词-13-设置认证--prompt-13-setup-authentication)
4. [提示词 1.4: 配置安全和速率限制](#提示词-14-配置安全和速率限制--prompt-14-configure-security)
5. [提示词 1.5: 配置外部服务](#提示词-15-配置外部服务--prompt-15-configure-external-services)

---

## 提示词 1.1: 初始化项目 | Prompt 1.1: Initialize Project

### 中文提示词 | Chinese Prompt

```
创建一个名为 "ai-finance-platform" 的 Next.js 15 项目，使用以下精确版本和配置：

**技术栈**:
- Next.js 15.0.3
- React 19.0.0-rc-66855b96-20241106
- TypeScript（配置为可选）
- Tailwind CSS 3.4.1

**项目结构**:
```
ai-finance-platform/
├── app/
│   ├── (auth)/
│   │   ├── layout.js
│   │   ├── sign-in/[[...sign-in]]/page.jsx
│   │   └── sign-up/[[...sign-up]]/page.jsx
│   ├── (main)/
│   │   ├── layout.js
│   │   ├── dashboard/
│   │   │   ├── layout.js
│   │   │   ├── page.jsx
│   │   │   └── _components/
│   │   ├── transaction/
│   │   │   └── create/
│   │   │       └── page.jsx
│   │   └── account/
│   │       └── [id]/
│   │           └── page.jsx
│   ├── api/
│   │   ├── inngest/route.js
│   │   └── seed/route.js
│   ├── layout.js
│   ├── page.js
│   └── globals.css
├── components/
│   ├── ui/
│   └── (其他组件)
├── actions/
├── lib/
├── hooks/
├── data/
├── prisma/
│   └── schema.prisma
├── public/
├── .env.local
├── next.config.mjs
├── tailwind.config.js
├── postcss.config.mjs
└── package.json
```

**package.json scripts**:
```json
{
  "scripts": {
    "dev": "next dev --turbopack",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "postinstall": "prisma generate"
  }
}
```

**关键依赖项（精确版本）**:
```json
{
  "dependencies": {
    "next": "15.0.3",
    "react": "^19.0.0-rc-66855b96-20241106",
    "react-dom": "^19.0.0-rc-66855b96-20241106",
    "@prisma/client": "6.0.1",
    "@clerk/nextjs": "6.6.0",
    "@arcjet/next": "1.0.0-alpha.34",
    "@google/generative-ai": "0.21.0",
    "inngest": "3.27.4",
    "resend": "4.0.1",
    "react-hook-form": "7.53.2",
    "zod": "3.23.8",
    "@hookform/resolvers": "3.9.1",
    "tailwindcss": "3.4.1",
    "recharts": "2.14.1",
    "date-fns": "4.1.0",
    "react-day-picker": "8.10.1",
    "lucide-react": "^0.468.0",
    "sonner": "^1.7.1",
    "vaul": "^1.1.1",
    "class-variance-authority": "^0.7.1",
    "clsx": "2.1.1",
    "tailwind-merge": "2.5.5"
  },
  "devDependencies": {
    "prisma": "6.0.1",
    "eslint": "^9",
    "eslint-config-next": "15.0.3",
    "postcss": "^8"
  }
}
```

**next.config.mjs**:
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

**tailwind.config.js** (使用 CSS 变量主题系统):
- 扫描 pages, components, app 目录
- 使用 CSS 变量实现主题
- 包含暗色模式支持
- 添加自定义动画

请创建完整的项目结构并安装所有依赖项。
```

### English Prompt

```
Create a Next.js 15 project named "ai-finance-platform" with the following exact versions and configuration:

**Tech Stack**:
- Next.js 15.0.3
- React 19.0.0-rc-66855b96-20241106
- TypeScript (optional configuration)
- Tailwind CSS 3.4.1

**Project Structure**:
[Same structure as Chinese version]

**package.json scripts**:
[Same as Chinese version]

**Key Dependencies (exact versions)**:
[Same as Chinese version]

**next.config.mjs**:
[Same as Chinese version]

**tailwind.config.js** (with CSS variables theme system):
- Scan pages, components, app directories
- Use CSS variables for theming
- Include dark mode support
- Add custom animations

Please create the complete project structure and install all dependencies.
```

### 验证步骤 | Verification Steps

运行以下命令验证设置 | Run these commands to verify setup:

```bash
# 检查 Node 版本（应为 18+）| Check Node version (should be 18+)
node --version

# 安装依赖 | Install dependencies
npm install

# 验证 Next.js 启动 | Verify Next.js starts
npm run dev

# 检查 http://localhost:3000
```

---

## 提示词 1.2: 配置数据库 | Prompt 1.2: Configure Database

### 中文提示词

```
配置 Prisma ORM 和 PostgreSQL 数据库，包含完整的数据模型。

**创建 prisma/schema.prisma**:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")
  directUrl = env("DIRECT_URL")
}

// 用户模型（与 Clerk 集成）
model User {
  id           String   @id @default(uuid())
  clerkUserId  String   @unique
  email        String   @unique
  name         String?
  imageUrl     String?
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt

  accounts     Account[]
  transactions Transaction[]

  @@index([clerkUserId])
  @@index([email])
}

// 账户模型（活期/储蓄）
model Account {
  id          String      @id @default(uuid())
  name        String
  type        AccountType
  balance     Float       @default(0)
  isDefault   Boolean     @default(false)
  createdAt   DateTime    @default(now())
  updatedAt   DateTime    @updatedAt

  userId      String
  user        User        @relation(fields: [userId], references: [id], onDelete: Cascade)

  transactions Transaction[]
  budgets      Budget[]

  @@index([userId])
}

// 交易模型（收入/支出，支持循环）
model Transaction {
  id                 String              @id @default(uuid())
  type               TransactionType
  amount             Float
  description        String?
  date               DateTime
  category           String
  status             TransactionStatus   @default(COMPLETED)
  isRecurring        Boolean             @default(false)
  recurringInterval  RecurringInterval?
  lastProcessed      DateTime?
  nextRecurring      DateTime?
  receiptUrl         String?
  createdAt          DateTime            @default(now())
  updatedAt          DateTime            @updatedAt

  accountId          String
  account            Account             @relation(fields: [accountId], references: [id], onDelete: Cascade)

  userId             String
  user               User                @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
  @@index([accountId])
  @@index([date])
  @@index([type])
  @@index([isRecurring])
}

// 预算模型
model Budget {
  id          String   @id @default(uuid())
  amount      Float
  month       DateTime
  lastAlertSent DateTime?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  accountId   String   @unique
  account     Account  @relation(fields: [accountId], references: [id], onDelete: Cascade)

  @@index([month])
}

// 枚举类型
enum TransactionType {
  INCOME
  EXPENSE
}

enum AccountType {
  CURRENT
  SAVINGS
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

**创建 lib/prisma.js**:

```javascript
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis;

const prisma = globalForPrisma.prisma || new PrismaClient();

if (process.env.NODE_ENV !== "production") {
  globalForPrisma.prisma = prisma;
}

export default prisma;
```

**环境变量（.env.local）**:

```bash
# 数据库 URL（Supabase 或本地 PostgreSQL）
DATABASE_URL="postgresql://user:password@host:5432/database?pgbouncer=true"
DIRECT_URL="postgresql://user:password@host:5432/database"
```

**初始化数据库**:

```bash
# 生成 Prisma 客户端
npx prisma generate

# 创建初始迁移
npx prisma migrate dev --name init

# 打开 Prisma Studio 查看数据库
npx prisma studio
```

请实现以上数据库配置，确保所有关系和索引正确设置。
```

### English Prompt

```
Configure Prisma ORM and PostgreSQL database with complete data models.

[Same content as Chinese version with schema, lib/prisma.js, environment variables, and initialization commands]

Please implement the database configuration above, ensuring all relationships and indexes are correctly set up.
```

### 验证步骤 | Verification Steps

```bash
# 验证 Prisma 客户端生成 | Verify Prisma client generation
npx prisma generate

# 检查迁移状态 | Check migration status
npx prisma migrate status

# 打开 Prisma Studio | Open Prisma Studio
npx prisma studio
# 访问 | Access: http://localhost:5555
```

---

## 提示词 1.3: 设置认证 | Prompt 1.3: Setup Authentication

### 中文提示词

```
使用 Clerk 实现完整的用户认证系统。

**安装 Clerk**:
已包含在 package.json 中：@clerk/nextjs@6.6.0

**配置环境变量（.env.local）**:

```bash
# Clerk 认证
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_xxxxx
CLERK_SECRET_KEY=sk_test_xxxxx
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/onboarding
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/onboarding
```

**创建 middleware.js**（根目录）:

```javascript
import { clerkMiddleware, createRouteMatcher } from "@clerk/nextjs/server";
import { NextResponse } from "next/server";

// 定义受保护的路由
const isProtectedRoute = createRouteMatcher([
  "/dashboard(.*)",
  "/account(.*)",
  "/transaction(.*)",
]);

// Clerk 中间件
const clerk = clerkMiddleware(async (auth, req) => {
  const { userId } = await auth();

  // 如果用户未登录且访问受保护路由，重定向到登录
  if (!userId && isProtectedRoute(req)) {
    const { redirectToSignIn } = await auth();
    return redirectToSignIn();
  }

  return NextResponse.next();
});

export default clerk;

export const config = {
  matcher: [
    "/((?!_next|[^?]*\\.(?:html?|css|js(?!on)|jpe?g|webp|png|gif|svg|ttf|woff2?|ico|csv|docx?|xlsx?|zip|webmanifest)).*)",
    "/(api|trpc)(.*)",
  ],
};
```

**创建 lib/checkUser.js**（用户同步工具）:

```javascript
import { currentUser } from "@clerk/nextjs/server";
import prisma from "./prisma";

export async function checkUser() {
  const user = await currentUser();

  if (!user) {
    return null;
  }

  try {
    // 检查用户是否已存在于数据库
    const loggedInUser = await prisma.user.findUnique({
      where: {
        clerkUserId: user.id,
      },
    });

    // 如果用户存在，返回
    if (loggedInUser) {
      return loggedInUser;
    }

    // 如果不存在，创建新用户
    const newUser = await prisma.user.create({
      data: {
        clerkUserId: user.id,
        email: user.emailAddresses[0].emailAddress,
        name: `${user.firstName || ""} ${user.lastName || ""}`.trim(),
        imageUrl: user.imageUrl,
      },
    });

    return newUser;
  } catch (error) {
    console.error("Error in checkUser:", error);
    return null;
  }
}
```

**更新 app/layout.js**（包装 ClerkProvider）:

```javascript
import { ClerkProvider } from "@clerk/nextjs";
import { Inter } from "next/font/google";
import "./globals.css";

const inter = Inter({ subsets: ["latin"] });

export const metadata = {
  title: "AI Finance Platform - Welth",
  description: "AI-powered personal finance management",
};

export default function RootLayout({ children }) {
  return (
    <ClerkProvider>
      <html lang="en">
        <body className={inter.className}>
          {children}
        </body>
      </html>
    </ClerkProvider>
  );
}
```

**创建认证页面**:

**app/(auth)/layout.js**:
```javascript
export default function AuthLayout({ children }) {
  return (
    <div className="flex items-center justify-center min-h-screen">
      {children}
    </div>
  );
}
```

**app/(auth)/sign-in/[[...sign-in]]/page.jsx**:
```javascript
import { SignIn } from "@clerk/nextjs";

export default function SignInPage() {
  return <SignIn />;
}
```

**app/(auth)/sign-up/[[...sign-up]]/page.jsx**:
```javascript
import { SignUp } from "@clerk/nextjs";

export default function SignUpPage() {
  return <SignUp />;
}
```

请实现以上认证配置。注意：需要先在 https://clerk.com 创建应用并获取 API 密钥。
```

### English Prompt

```
Implement complete user authentication system using Clerk.

[Same content as Chinese version with installation, environment variables, middleware, checkUser utility, layout update, and auth pages]

Please implement the authentication configuration above. Note: You need to create an app at https://clerk.com and obtain API keys first.
```

### 验证步骤 | Verification Steps

```bash
# 启动开发服务器 | Start dev server
npm run dev

# 测试认证流程 | Test auth flow:
# 1. 访问 | Visit: http://localhost:3000/sign-in
# 2. 注册新账户 | Sign up for new account
# 3. 验证重定向到 /onboarding | Verify redirect to /onboarding
# 4. 检查数据库中是否创建了用户 | Check if user was created in database
```

---

## 提示词 1.4: 配置安全和速率限制 | Prompt 1.4: Configure Security

### 中文提示词

```
使用 ArcJet 配置安全防护和速率限制。

**环境变量（.env.local）**:

```bash
ARCJET_KEY=ajkey_xxxxx
```

**创建 lib/arcjet.js**:

```javascript
import arcjet, { shield, detectBot } from "@arcjet/next";

const aj = arcjet({
  key: process.env.ARCJET_KEY,
  characteristics: ["userId"],
  rules: [
    // 防护盾 - 阻止可疑流量
    shield({
      mode: "LIVE",
    }),
    // 机器人检测
    detectBot({
      mode: "LIVE",
      allow: [],
    }),
  ],
});

export default aj;
```

**更新 middleware.js**（添加 ArcJet）:

```javascript
import { clerkMiddleware, createRouteMatcher } from "@clerk/nextjs/server";
import { NextResponse } from "next/server";
import aj from "./lib/arcjet";

const isProtectedRoute = createRouteMatcher([
  "/dashboard(.*)",
  "/account(.*)",
  "/transaction(.*)",
]);

export default clerkMiddleware(async (auth, req) => {
  const { userId } = await auth();

  // ArcJet 防护
  const decision = await aj.protect(req, {
    userId: userId || "anonymous",
    requested: 1,
  });

  // 如果请求被拒绝
  if (decision.isDenied()) {
    return NextResponse.json(
      { error: "Forbidden", reason: decision.reason },
      { status: 403 }
    );
  }

  // Clerk 认证检查
  if (!userId && isProtectedRoute(req)) {
    const { redirectToSignIn } = await auth();
    return redirectToSignIn();
  }

  return NextResponse.next();
});

export const config = {
  matcher: [
    "/((?!_next|[^?]*\\.(?:html?|css|js(?!on)|jpe?g|webp|png|gif|svg|ttf|woff2?|ico|csv|docx?|xlsx?|zip|webmanifest)).*)",
    "/(api|trpc)(.*)",
  ],
};
```

**在服务器操作中添加速率限制**（示例）:

```javascript
import { auth } from "@clerk/nextjs/server";
import aj from "@/lib/arcjet";
import { request as ArcJetRequest } from "@arcjet/next";

export async function createAccount(data) {
  const { userId } = await auth();
  if (!userId) throw new Error("Unauthorized");

  // 速率限制：每小时 10 次
  const req = await ArcJetRequest();
  const decision = await aj.protect(req, {
    userId,
    requested: 1,
  });

  if (decision.isDenied()) {
    if (decision.reason.isRateLimit()) {
      throw new Error("Too many requests. Please try again later.");
    }
    throw new Error("Request blocked");
  }

  // 继续处理...
}
```

请实现以上安全配置。注意：需要在 https://arcjet.com 获取 API 密钥。
```

### English Prompt

```
Configure security protection and rate limiting using ArcJet.

[Same content as Chinese version with environment variables, lib/arcjet.js, middleware update, and rate limiting example]

Please implement the security configuration above. Note: You need to obtain an API key from https://arcjet.com.
```

### 验证步骤 | Verification Steps

```bash
# 测试机器人检测 | Test bot detection
curl -A "BadBot" http://localhost:3000/dashboard
# 应该返回 403 | Should return 403

# 测试速率限制（快速连续请求）| Test rate limiting (rapid requests)
for i in {1..15}; do curl http://localhost:3000/api/some-endpoint; done
# 第 11 次请求应被阻止 | 11th request should be blocked
```

---

## 提示词 1.5: 配置外部服务 | Prompt 1.5: Configure External Services

### 中文提示词

```
配置所有外部服务：Resend（邮件）、Google Gemini（AI）、Inngest（后台作业）。

**环境变量（.env.local）**:

```bash
# Resend 邮件
RESEND_API_KEY=re_xxxxx

# Google Gemini AI
GEMINI_API_KEY=xxxxx
```

**1. 配置 Resend 邮件服务**:

创建 **lib/resend.js**:

```javascript
import { Resend } from "resend";

const resend = new Resend(process.env.RESEND_API_KEY);

export default resend;
```

创建 **actions/send-email.js**:

```javascript
"use server";

import { auth } from "@clerk/nextjs/server";
import resend from "@/lib/resend";

export async function sendEmail({ to, subject, html }) {
  const { userId } = await auth();
  if (!userId) throw new Error("Unauthorized");

  try {
    const { data, error } = await resend.emails.send({
      from: "onboarding@resend.dev",
      to,
      subject,
      html,
    });

    if (error) {
      throw new Error(error.message);
    }

    return { success: true, data };
  } catch (error) {
    throw new Error(error.message);
  }
}
```

**2. 配置 Google Gemini AI**:

创建 **lib/gemini.js**:

```javascript
import { GoogleGenerativeAI } from "@google/generative-ai";

const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY);

export default genAI;
```

**3. 配置 Inngest 后台作业**:

创建 **lib/inngest/client.js**:

```javascript
import { Inngest } from "inngest";

export const inngest = new Inngest({
  id: "ai-finance-platform",
  name: "AI Finance Platform",
});
```

创建 **app/api/inngest/route.js**:

```javascript
import { serve } from "inngest/next";
import { inngest } from "@/lib/inngest/client";
// 导入所有后台函数（稍后创建）
// import { recurringTransactions, monthlyReports, budgetAlerts } from "@/lib/inngest/function";

export const { GET, POST, PUT } = serve({
  client: inngest,
  functions: [
    // recurringTransactions,
    // monthlyReports,
    // budgetAlerts,
  ],
});
```

**验证配置**:

1. **Resend**: 访问 https://resend.com 获取 API 密钥
2. **Gemini**: 访问 https://makersuite.google.com/app/apikey 获取密钥
3. **Inngest**: 访问 https://inngest.com 创建账户（生产环境需要配置 webhook）

请实现以上外部服务配置。
```

### English Prompt

```
Configure all external services: Resend (Email), Google Gemini (AI), Inngest (Background Jobs).

[Same content as Chinese version with environment variables, Resend config, Gemini config, and Inngest config]

Please implement the external service configuration above.
```

### 验证步骤 | Verification Steps

```bash
# 测试邮件发送 | Test email sending
# 创建一个测试服务器操作并调用 sendEmail()

# 测试 AI 集成 | Test AI integration
# 在控制台测试 Gemini API 连接

# 验证 Inngest | Verify Inngest
# 访问 | Visit: http://localhost:3000/api/inngest
# 应该看到 Inngest 端点响应 | Should see Inngest endpoint response
```

---

## 基础设置完成检查清单 | Foundation Setup Checklist

完成以上所有提示词后，验证以下内容 | After completing all prompts above, verify:

- [ ] Next.js 15 项目正常运行 | Next.js 15 project runs
- [ ] 所有依赖项已安装（精确版本）| All dependencies installed (exact versions)
- [ ] Prisma 数据库已配置并迁移 | Prisma database configured and migrated
- [ ] Clerk 认证正常工作 | Clerk authentication works
- [ ] ArcJet 安全防护已激活 | ArcJet security active
- [ ] Resend 邮件服务已配置 | Resend email service configured
- [ ] Google Gemini AI 已配置 | Google Gemini AI configured
- [ ] Inngest 端点可访问 | Inngest endpoint accessible

**下一步 | Next Step**: 继续 `02-backend-prompts.md` 实现后端功能 | Continue to `02-backend-prompts.md` for backend implementation

---

**文档版本 | Document Version**: 1.0
**最后更新 | Last Updated**: 2025-11-16