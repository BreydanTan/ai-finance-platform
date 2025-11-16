# 完整重建提示词集 | Complete Rebuild Prompts
# AI Finance Platform (Welth)

**基于分析 | Based On**: 完整代码库分析 | Full codebase analysis
**版本 | Version**: 1.0
**日期 | Date**: 2025-11-16

---

## 使用说明 | Usage Instructions

### 提示词使用顺序 | Prompt Usage Order

1. **基础设置** (01-foundation-prompts.md) ✅ 已完成 | Already completed
2. **后端实现** (本文档 - 第 2-3 部分)
3. **前端实现** (本文档 - 第 4-5 部分)
4. **集成与部署** (本文档 - 第 6 部分)

### 如何使用这些提示词 | How to Use These Prompts

1. **按顺序执行** - 每个提示词依赖前面的提示词
2. **逐个验证** - 完成每个提示词后,运行验证步骤
3. **保存代码** - 每个提示词完成后,提交到 Git
4. **引用分析** - 遇到问题时,参考 analysis/ 目录中的详细分析

---

## 第 2 部分: 后端核心功能 | Part 2: Backend Core Features

### 提示词 2.1: 实现数据分类系统

```
创建交易分类数据结构和工具函数。

**创建 data/categories.js**:

定义两类分类:
1. **收入分类** (6 个):
   - Salary (工资) - #10b981 绿色
   - Freelance (自由职业) - #8b5cf6 紫色
   - Investments (投资) - #eab308 黄色
   - Side Hustle (副业) - #06b6d4 青色
   - Gifts (礼物) - #ec4899 粉色
   - Other Income (其他收入) - #6b7280 灰色

2. **支出分类** (15 个):
   - Housing (住房) - #ef4444 红色
   - Transportation (交通) - #f97316 橙色
   - Groceries (食品杂货) - #84cc16 lime
   - Utilities (水电费) - #0ea5e9 蓝色
   - Entertainment (娱乐) - #a855f7 紫色
   - Food & Dining (餐饮) - #f59e0b 琥珀色
   - Shopping (购物) - #ec4899 粉色
   - Healthcare (医疗) - #14b8a6 teal
   - Education (教育) - #6366f1 indigo
   - Personal (个人) - #8b5cf6 紫罗兰
   - Travel (旅行) - #06b6d4 青色
   - Insurance (保险) - #64748b slate
   - Gifts & Donations (礼物捐赠) - #d946ef 品红
   - Bills & EMI (账单和分期付款) - #f43f5e rose
   - Other Expense (其他支出) - #6b7280 灰色

每个分类包含: name, type (INCOME/EXPENSE), icon (Lucide 图标名), color

**导出格式**:
```javascript
export const categories = [
  { name: "Salary", type: "INCOME", icon: "Briefcase", color: "#10b981" },
  // ... 其他分类
];
```

参考文件: /home/user/ai-finance-platform/data/categories.js
```

### 提示词 2.2: 实现账户管理服务器操作

```
实现完整的账户 CRUD 操作（Create, Read, Update, Delete）。

**创建 actions/dashboard.js**:

实现以下服务器操作（"use server"）:

1. **getUserAccounts()** - 获取用户所有账户
   - 使用 auth() 获取 userId
   - 调用 checkUser() 同步用户
   - 使用 Prisma 查询账户
   - 按 createdAt 降序排序
   - 序列化日期字段

2. **createAccount(data)** - 创建新账户
   - 验证 userId
   - ArcJet 速率限制（10 次/小时）
   - 检查是否为第一个账户（自动设为默认）
   - 如果 isDefault=true,更新其他账户为非默认
   - 创建账户
   - revalidatePath("/dashboard")
   - 返回序列化后的账户

3. **updateDefaultAccount(accountId)** - 更新默认账户
   - 验证账户属于当前用户
   - 使用事务更新:
     - 将所有账户设为非默认
     - 将指定账户设为默认
   - revalidatePath("/dashboard")

4. **getDashboardData()** - 获取仪表板数据
   - 获取最近 30 天的交易
   - 计算总收入、总支出、净额
   - 按分类分组支出
   - 获取最近 5 笔交易
   - 返回序列化数据

**关键代码模式**:
- 所有函数都使用 try-catch
- 使用 auth() 验证用户
- 日期使用 toISOString() 序列化
- 使用 revalidatePath() 刷新缓存

参考: /home/user/ai-finance-platform/actions/dashboard.js (lines 6-186)
```

### 提示词 2.3: 实现交易管理服务器操作

```
实现交易的完整 CRUD 操作,包括 AI 收据扫描。

**创建 actions/transaction.js**:

实现以下服务器操作:

1. **createTransaction(data)** - 创建交易
   - 验证用户和账户所有权
   - 计算新余额（收入加,支出减）
   - 使用 Prisma 事务:
     - 创建交易记录
     - 更新账户余额
   - 如果是循环交易,计算 nextRecurring 日期
   - revalidatePath(["/dashboard", "/account/[id]"])

2. **updateTransaction(transactionId, data)** - 更新交易
   - 获取原交易数据
   - 计算余额差异
   - 使用事务更新交易和账户余额
   - 处理循环交易日期更新

3. **bulkDeleteTransactions(transactionIds)** - 批量删除
   - 验证所有交易属于当前用户
   - 计算余额调整
   - 使用事务批量删除并更新余额

4. **scanReceipt(file)** - AI 收据扫描
   - 验证文件大小（< 5MB）
   - 转换为 Base64
   - 调用 Google Gemini API:
     - 模型: gemini-1.5-flash
     - 提取: amount, date, description, category
   - 返回结构化数据

**Gemini 提示词模板**:
```
Analyze this receipt and extract:
- Total amount (number)
- Date (YYYY-MM-DD format)
- Description (merchant name)
- Category (from predefined list)

Return ONLY valid JSON: { amount, date, description, category }
```

参考: /home/user/ai-finance-platform/actions/transaction.js
```

---

## 第 3 部分: 后台作业系统 | Part 3: Background Jobs System

### 提示词 3.1: 实现 Inngest 后台函数

```
创建三个主要的后台作业：循环交易、月度报告、预算警报。

**创建 lib/inngest/function.js**:

**1. 循环交易触发器** - 每日午夜执行:
```javascript
export const recurringTransactions = inngest.createFunction(
  { id: "trigger-recurring-transactions" },
  { cron: "0 0 * * *" }, // 每日午夜
  async ({ step }) => {
    // 查找需要处理的循环交易（nextRecurring <= now）
    // 为每个循环交易:
    //   - 创建新交易记录
    //   - 更新账户余额
    //   - 计算下一次执行日期
    //   - 更新 lastProcessed
  }
);
```

**2. 月度报告生成器** - 每月 1 日执行:
```javascript
export const monthlyReports = inngest.createFunction(
  { id: "generate-monthly-reports" },
  { cron: "0 0 1 * *" }, // 每月 1 日
  async ({ step }) => {
    // 获取所有用户
    // 对每个用户:
    //   - 获取上月交易数据
    //   - 使用 Gemini AI 生成财务洞察
    //   - 通过 Resend 发送邮件报告
  }
);
```

**3. 预算警报检查器** - 每 6 小时执行:
```javascript
export const budgetAlerts = inngest.createFunction(
  { id: "check-budget-alerts" },
  { cron: "0 */6 * * *" }, // 每 6 小时
  async ({ step }) => {
    // 查找所有预算
    // 对每个预算:
    //   - 计算当月总支出
    //   - 如果超过预算 90%:
    //     - 检查是否已发送警报（lastAlertSent）
    //     - 发送警报邮件
    //     - 更新 lastAlertSent
  }
);
```

**更新 app/api/inngest/route.js**:
```javascript
import { recurringTransactions, monthlyReports, budgetAlerts } from "@/lib/inngest/function";

export const { GET, POST, PUT } = serve({
  client: inngest,
  functions: [
    recurringTransactions,
    monthlyReports,
    budgetAlerts,
  ],
});
```

参考: /home/user/ai-finance-platform/lib/inngest/function.js
```

---

## 第 4 部分: UI 组件库 | Part 4: UI Components Library

### 提示词 4.1: 创建 Radix UI 基础组件

```
使用 shadcn/ui 模式创建所有基础 UI 组件。

**创建 lib/utils.js** (cn 工具函数):
```javascript
import { clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs) {
  return twMerge(clsx(inputs));
}
```

**创建以下组件** (在 components/ui/ 目录):

1. **button.jsx** - 按钮组件
   - 使用 class-variance-authority (CVA)
   - 变体: default, destructive, outline, secondary, ghost, link
   - 尺寸: default, sm, lg, icon
   - 使用 forwardRef 模式

2. **input.jsx** - 输入框
3. **card.jsx** - 卡片（Card, CardHeader, CardTitle, CardDescription, CardContent, CardFooter）
4. **select.jsx** - 下拉选择（Radix UI Select）
5. **switch.jsx** - 切换开关（Radix UI Switch）
6. **popover.jsx** - 弹出框（Radix UI Popover）
7. **calendar.jsx** - 日历（react-day-picker）
8. **drawer.jsx** - 抽屉（Vaul）
9. **table.jsx** - 表格（Table, TableHeader, TableBody, TableRow, TableHead, TableCell）
10. **badge.jsx** - 徽章
11. **progress.jsx** - 进度条
12. **dropdown-menu.jsx** - 下拉菜单（Radix UI）
13. **tooltip.jsx** - 工具提示（Radix UI）
14. **checkbox.jsx** - 复选框（Radix UI）
15. **sonner.jsx** - Toast 通知（Sonner）

**组件模式**:
- 所有组件使用 forwardRef
- 使用 cn() 合并 className
- Radix UI 组件进行浅包装
- 保持简单和可组合

参考目录: /home/user/ai-finance-platform/components/ui/
```

### 提示词 4.2: 创建自定义 Hooks

```
实现 useFetch hook 用于异步操作。

**创建 hooks/use-fetch.js**:

```javascript
import { useState } from "react";
import { toast } from "sonner";

const useFetch = (cb) => {
  const [data, setData] = useState(undefined);
  const [loading, setLoading] = useState(null);
  const [error, setError] = useState(null);

  const fn = async (...args) => {
    setLoading(true);
    setError(null);

    try {
      const response = await cb(...args);
      setData(response);
      setError(null);
    } catch (error) {
      setError(error);
      toast.error(error.message);
    } finally {
      setLoading(false);
    }
  };

  return { data, loading, error, fn, setData };
};

export default useFetch;
```

**使用示例**:
```javascript
const { data, loading, error, fn: createAccountFn } = useFetch(createAccount);

const handleSubmit = async (formData) => {
  await createAccountFn(formData);
};
```

参考: /home/user/ai-finance-platform/hooks/use-fetch.js
```

---

## 第 5 部分: 前端页面和组件 | Part 5: Frontend Pages & Components

### 提示词 5.1: 实现仪表板页面

```
创建完整的仪表板页面,包含账户卡片、预算进度和交易概览。

**创建 app/(main)/layout.js**:
```javascript
export default function MainLayout({ children }) {
  return (
    <main className="container mx-auto px-4 py-8">
      {children}
    </main>
  );
}
```

**创建 app/(main)/dashboard/layout.js**:
```javascript
import { Suspense } from "react";
import { BarLoader } from "react-spinners";

export default function DashboardLayout({ children }) {
  return (
    <div className="px-5">
      <h1 className="text-6xl font-bold gradient-title mb-5">Dashboard</h1>
      <Suspense fallback={<BarLoader className="mt-4" width={"100%"} color="#9333ea" />}>
        {children}
      </Suspense>
    </div>
  );
}
```

**创建 app/(main)/dashboard/page.jsx**:
```javascript
import { getUserAccounts, getDashboardData } from "@/actions/dashboard";
import { getCurrentBudget } from "@/actions/budget";
import AccountCard from "./_components/account-card";
import BudgetProgress from "./_components/budget-progress";
import DashboardOverview from "./_components/transaction-overview";
import CreateAccountDrawer from "@/components/create-account-drawer";

export default async function DashboardPage() {
  // 并行获取数据
  const [accounts, transactions] = await Promise.all([
    getUserAccounts(),
    getDashboardData(),
  ]);

  const defaultAccount = accounts?.find((account) => account.isDefault);

  // 获取预算数据
  let budgetData = null;
  if (defaultAccount) {
    budgetData = await getCurrentBudget(defaultAccount.id);
  }

  return (
    <div className="space-y-8">
      {/* 账户卡片网格 */}
      <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-3">
        <CreateAccountDrawer />
        {accounts.map((account) => (
          <AccountCard key={account.id} account={account} />
        ))}
      </div>

      {/* 预算进度 */}
      {defaultAccount && (
        <BudgetProgress
          initialBudget={budgetData}
          accountId={defaultAccount.id}
        />
      )}

      {/* 交易概览 */}
      <DashboardOverview
        accounts={accounts}
        transactions={transactions}
      />
    </div>
  );
}
```

参考: /home/user/ai-finance-platform/app/(main)/dashboard/page.jsx
```

### 提示词 5.2: 实现交易表单和收据扫描

```
创建交易创建/编辑表单，集成 AI 收据扫描。

**创建 app/(main)/transaction/create/page.jsx**:
```javascript
import { getUserAccounts } from "@/actions/dashboard";
import { getTransaction } from "@/actions/transaction";
import AddTransactionForm from "../_components/transaction-form";

export default async function AddTransactionPage({ searchParams }) {
  const accounts = await getUserAccounts();
  const editId = searchParams?.edit;

  let initialData = null;
  if (editId) {
    initialData = await getTransaction(editId);
  }

  return (
    <div className="max-w-3xl mx-auto px-5">
      <h1 className="text-5xl font-bold gradient-title mb-8">
        {editId ? "Edit Transaction" : "Add Transaction"}
      </h1>
      <AddTransactionForm
        accounts={accounts}
        editMode={!!editId}
        initialData={initialData}
      />
    </div>
  );
}
```

**创建 app/(main)/transaction/_components/transaction-form.jsx**:

实现功能:
1. React Hook Form + Zod 验证
2. 条件字段（type, isRecurring, recurringInterval）
3. 收据扫描器集成
4. 日期选择器（Popover + Calendar）
5. 账户选择下拉菜单
6. 分类下拉菜单（根据 type 筛选）
7. 提交后重定向

**创建 app/(main)/transaction/_components/recipt-scanner.jsx**:
```javascript
"use client";

import { useState } from "react";
import { scanReceipt } from "@/actions/transaction";
import useFetch from "@/hooks/use-fetch";
import { Button } from "@/components/ui/button";
import { Loader2 } from "lucide-react";
import { toast } from "sonner";

export default function ReceiptScanner({ onScanComplete }) {
  const [file, setFile] = useState(null);
  const { loading, fn: scanReceiptFn, data } = useFetch(scanReceipt);

  const handleScan = async () => {
    if (!file) {
      toast.error("Please select a file");
      return;
    }

    if (file.size > 5 * 1024 * 1024) {
      toast.error("File size should be less than 5MB");
      return;
    }

    const scannedData = await scanReceiptFn(file);
    if (scannedData) {
      onScanComplete(scannedData);
    }
  };

  return (
    <div>
      <input
        type="file"
        accept="image/*"
        onChange={(e) => setFile(e.target.files[0])}
      />
      <Button onClick={handleScan} disabled={loading || !file}>
        {loading ? <Loader2 className="animate-spin" /> : "Scan Receipt"}
      </Button>
    </div>
  );
}
```

参考:
- /home/user/ai-finance-platform/app/(main)/transaction/_components/transaction-form.jsx
- /home/user/ai-finance-platform/app/(main)/transaction/_components/recipt-scanner.jsx
```

### 提示词 5.3: 实现账户详情页

```
创建账户详情页，包含图表和交易表格。

**创建 app/(main)/account/[id]/page.jsx**:
```javascript
import { getAccountWithTransactions } from "@/actions/account";
import { notFound } from "next/navigation";
import AccountChart from "../_components/account-chart";
import TransactionTable from "../_components/transaction-table";
import { Suspense } from "react";
import { BarLoader } from "react-spinners";

export default async function AccountPage({ params }) {
  const accountData = await getAccountWithTransactions(params.id);

  if (!accountData) {
    notFound();
  }

  const { transactions, ...account } = accountData;

  return (
    <div className="space-y-8 px-5">
      {/* 账户头部 */}
      <div className="flex justify-between items-center">
        <h1 className="text-5xl font-bold gradient-title">
          {account.name}
        </h1>
        <div className="text-4xl font-bold">
          ${account.balance.toFixed(2)}
        </div>
      </div>

      {/* 图表 */}
      <Suspense fallback={<BarLoader />}>
        <AccountChart transactions={transactions} />
      </Suspense>

      {/* 交易表格 */}
      <Suspense fallback={<BarLoader />}>
        <TransactionTable transactions={transactions} />
      </Suspense>
    </div>
  );
}
```

**创建 app/(main)/account/_components/transaction-table.jsx**:

实现功能:
1. 搜索筛选（description）
2. 类型筛选（INCOME/EXPENSE）
3. 循环筛选（isRecurring）
4. 排序（date, amount, type）
5. 分页（每页 10 条）
6. 批量删除（带确认）
7. 编辑/删除单个交易
8. 使用 useMemo 优化性能

**创建 app/(main)/account/_components/account-chart.jsx**:

实现功能:
1. 日期范围选择（7D, 1M, 3M, 6M, ALL）
2. 收入/支出柱状图（Recharts）
3. 按日期分组聚合数据
4. 使用 useMemo 优化数据处理

参考:
- /home/user/ai-finance-platform/app/(main)/account/[id]/page.jsx
- /home/user/ai-finance-platform/app/(main)/account/_components/transaction-table.jsx
- /home/user/ai-finance-platform/app/(main)/account/_components/account-chart.jsx
```

---

## 第 6 部分: 全局组件和集成 | Part 6: Global Components & Integration

### 提示词 6.1: 实现全局 Header 和 Footer

```
创建全局导航头部和底部。

**创建 components/header.jsx**:
```javascript
import Link from "next/link";
import { SignedIn, SignedOut, UserButton } from "@clerk/nextjs";
import { Button } from "./ui/button";
import CreateAccountDrawer from "./create-account-drawer";

export default function Header() {
  return (
    <header className="border-b">
      <nav className="container mx-auto px-4 py-4 flex items-center justify-between">
        <Link href="/" className="text-2xl font-bold">
          Welth
        </Link>

        <div className="flex items-center gap-4">
          <SignedIn>
            <Link href="/dashboard">
              <Button variant="outline">Dashboard</Button>
            </Link>
            <CreateAccountDrawer />
            <UserButton />
          </SignedIn>

          <SignedOut>
            <Link href="/sign-in">
              <Button variant="outline">Sign In</Button>
            </Link>
            <Link href="/sign-up">
              <Button>Get Started</Button>
            </Link>
          </SignedOut>
        </div>
      </nav>
    </header>
  );
}
```

**更新 app/layout.js**:
```javascript
import { ClerkProvider } from "@clerk/nextjs";
import { Inter } from "next/font/google";
import Header from "@/components/header";
import { Toaster } from "@/components/ui/sonner";
import "./globals.css";

const inter = Inter({ subsets: ["latin"] });

export default function RootLayout({ children }) {
  return (
    <ClerkProvider>
      <html lang="en">
        <body className={inter.className}>
          <Header />
          <main className="min-h-screen">{children}</main>
          <footer className="bg-gray-900 py-12 text-white text-center">
            <p>© 2024 Welth. All rights reserved.</p>
          </footer>
          <Toaster position="top-right" />
        </body>
      </html>
    </ClerkProvider>
  );
}
```

参考: /home/user/ai-finance-platform/components/header.jsx
```

### 提示词 6.2: 实现首页

```
创建营销首页，包含 Hero, Features, How It Works 等section。

**创建 app/page.js**:

实现以下sections:
1. **Hero Section**
   - 标题: "Manage Your Finances with Intelligence"
   - 副标题介绍 AI 功能
   - CTA 按钮: "Get Started"
   - Hero 图片（带滚动动画）

2. **Stats Section**
   - 显示统计数据（用户数、交易数等）

3. **Features Section**
   - AI-Powered Insights
   - Smart Budgeting
   - Recurring Transactions
   - Multi-Account Support

4. **How It Works**
   - 3步流程说明

5. **Testimonials**
   - 用户评价（可选）

6. **CTA Section**
   - 最终行动号召

**创建 components/hero.jsx**:
```javascript
"use client";

import { useEffect, useRef } from "react";
import Link from "next/link";
import { Button } from "./ui/button";
import Image from "next/image";

export default function HeroSection() {
  const imageRef = useRef(null);

  useEffect(() => {
    const handleScroll = () => {
      const scrollPosition = window.scrollY;
      const scrollThreshold = 100;

      if (scrollPosition > scrollThreshold) {
        imageRef.current?.classList.add("scrolled");
      } else {
        imageRef.current?.classList.remove("scrolled");
      }
    };

    window.addEventListener("scroll", handleScroll);
    return () => window.removeEventListener("scroll", handleScroll);
  }, []);

  return (
    <section className="py-20">
      <div className="container mx-auto px-4 text-center">
        <h1 className="text-6xl font-bold gradient-title mb-6">
          Manage Your Finances with Intelligence
        </h1>
        <p className="text-xl text-gray-600 mb-8 max-w-2xl mx-auto">
          AI-powered personal finance management. Track expenses, set budgets, and get intelligent insights.
        </p>
        <Link href="/sign-up">
          <Button size="lg">Get Started Free</Button>
        </Link>

        <div ref={imageRef} className="mt-12 hero-image">
          <Image
            src="/hero-dashboard.png"
            alt="Dashboard Preview"
            width={1200}
            height={600}
            className="rounded-lg shadow-2xl"
          />
        </div>
      </div>
    </section>
  );
}
```

参考: /home/user/ai-finance-platform/app/page.js
```

### 提示词 6.3: 配置全局样式

```
配置 Tailwind CSS 和全局样式。

**app/globals.css**:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --primary: 271 91% 65%;
    --primary-foreground: 210 40% 98%;
    /* ... 其他 CSS 变量 */
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    /* ... 暗色模式变量 */
  }
}

@layer components {
  .gradient-title {
    @apply bg-gradient-to-r from-purple-600 to-pink-600 bg-clip-text text-transparent;
  }

  .hero-image {
    @apply transform transition-all duration-300;
  }

  .hero-image.scrolled {
    @apply scale-95 opacity-80;
  }
}
```

**tailwind.config.js** (完整配置):
```javascript
module.exports = {
  darkMode: ["class"],
  content: [
    './pages/**/*.{js,jsx}',
    './components/**/*.{js,jsx}',
    './app/**/*.{js,jsx}',
  ],
  theme: {
    extend: {
      colors: {
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
        primary: {
          DEFAULT: "hsl(var(--primary))",
          foreground: "hsl(var(--primary-foreground))",
        },
        // ... 其他颜色
      },
      borderRadius: {
        lg: "var(--radius)",
        md: "calc(var(--radius) - 2px)",
        sm: "calc(var(--radius) - 4px)",
      },
      keyframes: {
        "accordion-down": {
          from: { height: 0 },
          to: { height: "var(--radix-accordion-content-height)" },
        },
        "accordion-up": {
          from: { height: "var(--radix-accordion-content-height)" },
          to: { height: 0 },
        },
      },
      animation: {
        "accordion-down": "accordion-down 0.2s ease-out",
        "accordion-up": "accordion-up 0.2s ease-out",
      },
    },
  },
  plugins: [require("tailwindcss-animate")],
};
```

参考:
- /home/user/ai-finance-platform/app/globals.css
- /home/user/ai-finance-platform/tailwind.config.js
```

---

## 第 7 部分: 测试与部署 | Part 7: Testing & Deployment

### 提示词 7.1: 创建数据库种子脚本

```
实现数据库种子功能，生成测试数据。

**创建 actions/seed.js**:
```javascript
"use server";

import { auth } from "@clerk/nextjs/server";
import prisma from "@/lib/prisma";
import { categories } from "@/data/categories";

export async function seedTransactions(accountId) {
  const { userId } = await auth();
  if (!userId) throw new Error("Unauthorized");

  // 验证账户所有权
  const account = await prisma.account.findFirst({
    where: { id: accountId, userId },
  });

  if (!account) {
    throw new Error("Account not found");
  }

  // 生成 90 天的交易数据
  const transactions = [];
  const today = new Date();

  for (let i = 90; i >= 0; i--) {
    const date = new Date(today);
    date.setDate(date.getDate() - i);

    // 每天 1-3 笔交易
    const numTransactions = Math.floor(Math.random() * 3) + 1;

    for (let j = 0; j < numTransactions; j++) {
      const isIncome = Math.random() > 0.6; // 40% 概率为收入
      const type = isIncome ? "INCOME" : "EXPENSE";

      const categoryList = categories.filter((cat) => cat.type === type);
      const category =
        categoryList[Math.floor(Math.random() * categoryList.length)];

      const amount = parseFloat(
        (Math.random() * (isIncome ? 500 : 200) + 10).toFixed(2)
      );

      transactions.push({
        userId,
        accountId,
        type,
        category: category.name,
        amount,
        description: `${category.name} transaction`,
        date,
        status: "COMPLETED",
      });
    }
  }

  // 批量创建
  await prisma.transaction.createMany({
    data: transactions,
  });

  // 重新计算账户余额
  const totalIncome = transactions
    .filter((t) => t.type === "INCOME")
    .reduce((sum, t) => sum + t.amount, 0);

  const totalExpense = transactions
    .filter((t) => t.type === "EXPENSE")
    .reduce((sum, t) => sum + t.amount, 0);

  const newBalance = account.balance + totalIncome - totalExpense;

  await prisma.account.update({
    where: { id: accountId },
    data: { balance: newBalance },
  });

  return { success: true, count: transactions.length };
}
```

**创建 app/api/seed/route.js**:
```javascript
import { NextResponse } from "next/server";
import { auth } from "@clerk/nextjs/server";
import { seedTransactions } from "@/actions/seed";

export async function GET(request) {
  try {
    const { userId } = await auth();
    if (!userId) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const { searchParams } = new URL(request.url);
    const accountId = searchParams.get("accountId");

    if (!accountId) {
      return NextResponse.json(
        { error: "accountId required" },
        { status: 400 }
      );
    }

    const result = await seedTransactions(accountId);
    return NextResponse.json(result);
  } catch (error) {
    return NextResponse.json({ error: error.message }, { status: 500 });
  }
}
```

使用方法:
```bash
curl "http://localhost:3000/api/seed?accountId=YOUR_ACCOUNT_ID"
```

参考:
- /home/user/ai-finance-platform/actions/seed.js
- /home/user/ai-finance-platform/app/api/seed/route.js
```

### 提示词 7.2: 配置部署

```
配置 Vercel 部署和生产环境变量。

**步骤 1: 准备生产环境变量**

在 Vercel 仪表板中配置以下环境变量:

```bash
# 数据库（Supabase）
DATABASE_URL=postgresql://...?pgbouncer=true
DIRECT_URL=postgresql://...

# Clerk（生产密钥）
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_live_...
CLERK_SECRET_KEY=sk_live_...
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/dashboard
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/dashboard

# 外部服务
GEMINI_API_KEY=...
RESEND_API_KEY=re_...
ARCJET_KEY=ajkey_...
```

**步骤 2: 部署到 Vercel**

```bash
# 安装 Vercel CLI
npm i -g vercel

# 登录
vercel login

# 部署
vercel --prod
```

**步骤 3: 配置 Inngest Webhook**

1. 访问 Inngest 仪表板
2. 添加 webhook URL: `https://your-domain.vercel.app/api/inngest`
3. 验证连接
4. 测试后台作业

**步骤 4: 运行生产迁移**

```bash
# 在本地连接生产数据库
DATABASE_URL="your-production-url" npx prisma migrate deploy

# 或在 Vercel 构建时自动运行（package.json）
"vercel-build": "prisma generate && prisma migrate deploy && next build"
```

**步骤 5: 验证部署**

测试清单:
- [ ] 首页加载正常
- [ ] 认证流程工作
- [ ] 创建账户
- [ ] 添加交易
- [ ] 收据扫描
- [ ] 仪表板数据显示
- [ ] 后台作业运行（检查 Inngest 仪表板）
```

---

## 完整重建检查清单 | Complete Rebuild Checklist

完成所有提示词后，验证以下内容:

### 基础设施 | Infrastructure
- [ ] Next.js 15 + React 19 RC 运行正常
- [ ] Prisma + PostgreSQL 配置完成
- [ ] Clerk 认证工作正常
- [ ] ArcJet 安全防护激活
- [ ] 所有外部服务集成（Resend, Gemini, Inngest）

### 后端功能 | Backend Features
- [ ] 账户 CRUD 操作
- [ ] 交易 CRUD 操作
- [ ] AI 收据扫描
- [ ] 预算管理
- [ ] 后台作业（循环交易、月度报告、预算警报）

### 前端功能 | Frontend Features
- [ ] 首页（营销页面）
- [ ] 认证页面（登录/注册）
- [ ] 仪表板（账户、预算、概览）
- [ ] 交易表单（创建/编辑）
- [ ] 账户详情（图表、表格）
- [ ] 全局导航和 UI 组件

### 数据和测试 | Data & Testing
- [ ] 数据库种子脚本可用
- [ ] 测试数据生成正常
- [ ] 所有页面可访问
- [ ] 无控制台错误

### 部署 | Deployment
- [ ] Vercel 部署成功
- [ ] 生产环境变量配置
- [ ] Inngest webhook 配置
- [ ] 数据库迁移已应用

---

## 故障排除 | Troubleshooting

### 常见问题

**1. Prisma 客户端未生成**
```bash
npx prisma generate
```

**2. 认证重定向循环**
- 检查 middleware.js 中的路由匹配器
- 验证 Clerk 环境变量

**3. 服务器操作失败**
- 检查 "use server" 指令
- 验证用户认证
- 检查数据库连接

**4. AI 收据扫描失败**
- 验证 GEMINI_API_KEY
- 检查文件大小限制（5MB）
- 确认 next.config.mjs 中的 bodySizeLimit

**5. 后台作业未触发**
- 检查 Inngest webhook 配置
- 验证 cron 表达式
- 查看 Inngest 仪表板日志

---

## 下一步 | Next Steps

完成重建后:

1. **自定义功能** - 添加您自己的功能
2. **性能优化** - 分析和优化加载时间
3. **SEO** - 添加元数据和 sitemap
4. **测试** - 编写单元和集成测试
5. **监控** - 设置错误跟踪（Sentry）
6. **分析** - 配置 Vercel Analytics

参考 `teaching-guide/` 目录获取详细的学习路径和扩展指南。

---

**文档完成 | Document Complete** ✅
**总提示词数 | Total Prompts**: 20+
**预计总时间 | Estimated Total Time**: 8-12 小时 | 8-12 hours

基于完整代码库分析生成 | Generated from complete codebase analysis
版本 | Version: 1.0