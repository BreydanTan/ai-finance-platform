# 集成与部署提示词 🔗

外部服务集成和项目部署的完整指令。

---

## 提示词 4.1: 配置 Clerk 认证

```
集成 Clerk 实现用户认证系统:

步骤1: 注册和配置
1. 访问 https://clerk.com 注册账号
2. 创建新应用,选择 "Next.js"
3. 复制 API 密钥到 .env:
   NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_xxxxx
   CLERK_SECRET_KEY=sk_test_xxxxx

4. 配置重定向URL(.env):
   NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
   NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
   NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/dashboard
   NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/dashboard

步骤2: 安装 Clerk SDK
npm install @clerk/nextjs

步骤3: 配置中间件
创建 middleware.js:
- 导入 clerkMiddleware
- 保护受保护路由
- 配置公开路由列表

步骤4: 创建认证页面
- app/(auth)/sign-in/[[...sign-in]]/page.jsx
- app/(auth)/sign-up/[[...sign-up]]/page.jsx

步骤5: 用户数据同步
创建 lib/checkUser.js:
- 在用户登录后触发
- 检查数据库是否有该用户
- 如果没有,创建新用户记录
- 同步: id, email, name, imageUrl

步骤6: 获取当前用户
在 Server Components/Actions:
  const { userId } = await auth();
  if (!userId) throw new Error("Unauthorized");

步骤7: 用户头像菜单
- 显示用户信息
- 退出登录按钮
```

**验证**:
1. 访问 /sign-in 能看到登录页
2. 注册新用户成功
3. 登录后跳转到 /dashboard
4. 数据库有用户记录

**参考代码**:
- `middleware.js`
- `lib/checkUser.js`
- `app/(auth)/`

---

## 提示词 4.2: 集成 Gemini AI

```
集成 Google Gemini AI 实现收据扫描:

步骤1: 获取 API 密钥
1. 访问 https://makersuite.google.com/app/apikey
2. 创建 API Key
3. 添加到 .env:
   GEMINI_API_KEY=xxxxx

步骤2: 安装 Google AI SDK
npm install @google/generative-ai

步骤3: 创建 AI 客户端
lib/gemini.js:
  import { GoogleGenerativeAI } from "@google/generative-ai";
  const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY);
  export const model = genAI.getGenerativeModel({ model: "gemini-pro-vision" });

步骤4: 实现收据扫描 Server Action
actions/transaction.js - scanReceipt():

1. 接收图片文件
2. 转换为 base64
3. 构建 prompt:
   "分析这张收据,提取:
    - amount: 总金额(数字)
    - date: 日期(YYYY-MM-DD)
    - description: 商家名称
    - category: 分类(food/transport/shopping等)
    仅返回JSON格式"

4. 调用 Gemini API:
   const result = await model.generateContent([prompt, imageData]);
   const text = result.response.text();
   const data = JSON.parse(text);

5. 返回解析的数据

步骤5: 前端集成
- 文件上传组件
- 调用 scanReceipt
- 解析响应填充表单
- 错误处理

限制:
- 文件大小: 最大 5MB
- 格式: JPG, PNG, WebP
- 免费配额: 60次/月
```

**测试**:
1. 准备测试收据图片
2. 上传并扫描
3. 验证提取的数据准确性

**参考**: `actions/transaction.js:231-291`

---

## 提示词 4.3: 配置 Resend 邮件服务

```
集成 Resend 实现邮件通知:

步骤1: 注册 Resend
1. 访问 https://resend.com
2. 创建账户
3. 获取 API Key
4. 添加到 .env:
   RESEND_API_KEY=re_xxxxx

步骤2: 安装 Resend SDK
npm install resend

步骤3: 创建邮件发送函数
actions/send-email.js:
  import { Resend } from "resend";
  const resend = new Resend(process.env.RESEND_API_KEY);

  export async function sendBudgetAlert(userEmail, data) {
    await resend.emails.send({
      from: "noreply@yourdomain.com",
      to: userEmail,
      subject: "预算警报",
      html: `
        <h1>预算使用警告</h1>
        <p>您的账户 ${data.accountName} 预算已使用 ${data.percentage}%</p>
      `
    });
  }

步骤4: 触发邮件
在合适的时机调用:
- 预算超过90%时
- 大额支出时
- 月度报告

步骤5: 邮件模板
创建 emails/ 文件夹:
- budget-alert.jsx (React Email)
- monthly-report.jsx

使用 React Email:
npm install react-email @react-email/components

免费配额: 100封/天
```

**测试**:
1. 发送测试邮件
2. 检查收件箱
3. 验证格式正确

**参考**: `actions/send-email.js`

---

## 提示词 4.4: 配置 Inngest 后台作业

```
集成 Inngest 实现定时任务和后台作业:

步骤1: 注册 Inngest
1. 访问 https://inngest.com
2. 创建账户
3. 开发环境无需配置
4. 生产环境获取密钥:
   INNGEST_EVENT_KEY=xxxxx
   INNGEST_SIGNING_KEY=xxxxx

步骤2: 安装 Inngest SDK
npm install inngest

步骤3: 创建 Inngest 客户端
lib/inngest/client.js:
  import { Inngest } from "inngest";
  export const inngest = new Inngest({
    id: "ai-finance-platform"
  });

步骤4: 定义后台函数
lib/inngest/function.js:

1. 循环交易处理(每天执行)
2. 预算警报检查(每6小时)
3. 月度报告生成(每月1号)

示例 - 预算警报:
  export const checkBudgetAlerts = inngest.createFunction(
    { id: "check-budget-alerts" },
    { cron: "0 */6 * * *" }, // 每6小时
    async ({ step }) => {
      const budgets = await step.run("get-budgets", async () => {
        return db.budget.findMany();
      });

      for (const budget of budgets) {
        // 检查并发送警报
      }
    }
  );

步骤5: 创建 API 端点
app/api/inngest/route.js:
  import { serve } from "inngest/next";
  import { inngest } from "@/lib/inngest/client";
  import * as functions from "@/lib/inngest/function";

  export const { GET, POST, PUT } = serve({
    client: inngest,
    functions: Object.values(functions),
  });

步骤6: 开发环境测试
1. 启动 Inngest Dev Server:
   npx inngest-cli dev

2. 访问 http://localhost:8288
3. 查看和测试函数

步骤7: 生产部署
1. 在 Inngest 后台添加 webhook:
   https://yourdomain.com/api/inngest
2. 配置环境变量
```

**测试**:
1. 手动触发函数
2. 检查日志
3. 验证作业执行

**参考**:
- `lib/inngest/`
- `app/api/inngest/route.js`

---

## 提示词 4.5: 配置 Supabase 数据库

```
使用 Supabase 托管 PostgreSQL 数据库:

步骤1: 创建 Supabase 项目
1. 访问 https://supabase.com
2. 创建新项目
3. 等待数据库初始化(~2分钟)

步骤2: 获取连接字符串
1. Settings → Database
2. 复制 Connection String:
   - Connection pooling (推荐)
   - Direct connection

3. 添加到 .env:
   DATABASE_URL="postgresql://postgres:[PASSWORD]@[HOST]:5432/postgres?pgbouncer=true"
   DIRECT_URL="postgresql://postgres:[PASSWORD]@[HOST]:5432/postgres"

步骤3: 运行 Prisma 迁移
npx prisma migrate deploy

步骤4: 查看数据
- Supabase Table Editor
- 或 npx prisma studio

配置建议:
- 启用 Row Level Security (可选)
- 设置备份计划
- 监控查询性能

免费额度:
- 500MB 数据库
- 无限 API 请求
- 50,000 月活用户
```

**验证**:
1. 连接成功
2. 表已创建
3. 可以插入数据

---

## 提示词 4.6: 部署到 Vercel

```
将项目部署到 Vercel 生产环境:

步骤1: 准备代码
1. 确保代码已推送到 GitHub:
   git add .
   git commit -m "准备部署"
   git push origin main

2. 本地测试构建:
   npm run build
   npm start

步骤2: 连接 Vercel
1. 访问 https://vercel.com
2. 注册/登录
3. Import Git Repository
4. 选择你的 GitHub 仓库

步骤3: 配置项目
1. Framework Preset: Next.js
2. Root Directory: ./
3. Build Command: npm run build
4. Output Directory: .next
5. Install Command: npm install

步骤4: 配置环境变量
在 Vercel 项目设置中添加所有环境变量:
- DATABASE_URL
- DIRECT_URL
- CLERK_SECRET_KEY
- NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY
- GEMINI_API_KEY
- RESEND_API_KEY
- ARCJET_KEY
- INNGEST_EVENT_KEY
- INNGEST_SIGNING_KEY

⚠️ 不要提交 .env 到 Git!

步骤5: 部署
1. 点击 Deploy
2. 等待构建完成(~2-5分钟)
3. 获得生产 URL

步骤6: 配置自定义域名(可选)
1. Vercel Project → Settings → Domains
2. 添加域名
3. 配置 DNS 记录

步骤7: 配置 Clerk 生产环境
1. Clerk Dashboard
2. 添加生产域名
3. 更新重定向 URL

步骤8: 配置 Inngest 生产环境
1. Inngest Dashboard
2. 添加 webhook: https://yourdomain.com/api/inngest
3. 验证连接

步骤9: 运行数据库迁移
Vercel Dashboard → Deployments → 点击最新部署 → Terminal
  npx prisma migrate deploy
```

**部署检查清单**:
- [ ] 构建成功
- [ ] 所有环境变量已配置
- [ ] 数据库迁移已运行
- [ ] 认证系统工作正常
- [ ] API 调用成功
- [ ] 邮件发送正常
- [ ] 定时任务触发

**自动部署**:
每次推送到 main 分支,Vercel 自动重新部署。

---

## 提示词 4.7: 配置域名和SSL

```
为生产环境配置自定义域名:

步骤1: 购买域名
推荐域名注册商:
- Namecheap
- GoDaddy
- 阿里云(中国)
- Cloudflare

步骤2: Vercel 添加域名
1. Project → Settings → Domains
2. 输入域名: yourdomain.com
3. 复制 DNS 记录

步骤3: 配置 DNS
在域名注册商处添加记录:

类型: A
名称: @
值: 76.76.21.21

类型: CNAME
名称: www
值: cname.vercel-dns.com

步骤4: 等待 DNS 传播(1-48小时)

步骤5: 验证
1. 访问 https://yourdomain.com
2. 检查 SSL 证书(Vercel 自动配置)

步骤6: 配置邮件域名(Resend)
如果使用自定义邮箱发送:
1. Resend → Domains
2. 添加域名
3. 配置 SPF/DKIM 记录
```

**检查**:
- [ ] HTTP 自动跳转 HTTPS
- [ ] www 和无 www 都可访问
- [ ] SSL 证书有效

---

## 总结

完成所有集成后,你的项目将拥有:
- ✅ 用户认证 (Clerk)
- ✅ AI 功能 (Gemini)
- ✅ 邮件通知 (Resend)
- ✅ 定时任务 (Inngest)
- ✅ 云数据库 (Supabase)
- ✅ 生产部署 (Vercel)
- ✅ 自定义域名 + SSL

**下一步**: 监控、优化、持续迭代！
