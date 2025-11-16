# 02 - Backend Architecture Analysis | 后端架构分析

## Table of Contents | 目录

1. [Server Actions 完整清单](#1-server-actions-complete-inventory)
2. [API Routes 分析](#2-api-routes-analysis)
3. [后端逻辑流程图](#3-backend-logic-flow-diagrams)
4. [外部服务集成](#4-external-services-integration)
5. [Server Actions vs API Routes 对比](#5-server-actions-vs-api-routes-comparison)
6. [错误处理模式](#6-error-handling-patterns)
7. [数据序列化处理](#7-data-serialization-handling)

---

## 1. Server Actions Complete Inventory | Server Actions 完整清单

### 1.1 Account Actions (`actions/account.js`)

#### **getAccountWithTransactions**
- **文件位置**: `/home/user/ai-finance-platform/actions/account.js:18-49`
- **参数**:
  - `accountId` (string): 账户 ID
- **返回值**:
  ```javascript
  {
    ...account,
    balance: number,           // 序列化后的余额
    transactions: Array<{      // 交易记录数组
      ...transaction,
      amount: number           // 序列化后的金额
    }>,
    _count: {
      transactions: number     // 交易总数
    }
  }
  ```
- **业务逻辑**:
  1. 验证用户认证状态 (Clerk auth)
  2. 从数据库查询用户信息
  3. 获取账户及其关联的所有交易记录（按日期降序）
  4. 序列化 Decimal 类型数据为 Number
  5. 返回包含交易记录的完整账户信息
- **认证**: 需要登录用户
- **数据序列化**: 使用 `serializeDecimal()` 转换 balance 和 amount

#### **bulkDeleteTransactions**
- **文件位置**: `/home/user/ai-finance-platform/actions/account.js:51-112`
- **参数**:
  - `transactionIds` (string[]): 要删除的交易 ID 数组
- **返回值**:
  ```javascript
  { success: boolean, error?: string }
  ```
- **业务逻辑**:
  1. 验证用户认证和授权
  2. 查询要删除的所有交易记录
  3. 计算每个账户的余额变化
  4. 使用数据库事务执行：
     - 批量删除交易记录
     - 更新相关账户余额
  5. 重新验证相关页面缓存
- **认证**: 需要登录用户
- **数据库事务**: 使用 Prisma `$transaction` 确保原子性
- **缓存失效**: `/dashboard`, `/account/[id]`

#### **updateDefaultAccount**
- **文件位置**: `/home/user/ai-finance-platform/actions/account.js:114-150`
- **参数**:
  - `accountId` (string): 要设置为默认的账户 ID
- **返回值**:
  ```javascript
  { success: boolean, data?: Account, error?: string }
  ```
- **业务逻辑**:
  1. 验证用户认证
  2. 取消所有现有默认账户的标记
  3. 设置新的默认账户
  4. 序列化并返回更新后的账户
- **认证**: 需要登录用户
- **缓存失效**: `/dashboard`

---

### 1.2 Transaction Actions (`actions/transaction.js`)

#### **createTransaction**
- **文件位置**: `/home/user/ai-finance-platform/actions/transaction.js:18-100`
- **参数**:
  ```javascript
  {
    accountId: string,
    type: 'INCOME' | 'EXPENSE',
    amount: number,
    description?: string,
    date: Date,
    category: string,
    isRecurring?: boolean,
    recurringInterval?: 'DAILY' | 'WEEKLY' | 'MONTHLY' | 'YEARLY'
  }
  ```
- **返回值**:
  ```javascript
  { success: boolean, data: Transaction }
  ```
- **业务逻辑**:
  1. **认证验证**: 使用 Clerk 验证用户身份
  2. **速率限制**: 使用 ArcJet 检查速率限制（10次/小时）
  3. **账户验证**: 确认账户存在且属于当前用户
  4. **余额计算**:
     - EXPENSE: `newBalance = oldBalance - amount`
     - INCOME: `newBalance = oldBalance + amount`
  5. **数据库事务**:
     - 创建新交易记录
     - 计算周期性交易的下次执行日期
     - 更新账户余额
  6. **缓存失效**: 重新验证仪表板和账户页面
- **认证**: 需要登录用户
- **速率限制**: ArcJet Token Bucket (10/hour)
- **外部服务**: ArcJet 安全防护
- **数据库事务**: 确保交易创建和余额更新的原子性

#### **getTransaction**
- **文件位置**: `/home/user/ai-finance-platform/actions/transaction.js:102-122`
- **参数**:
  - `id` (string): 交易 ID
- **返回值**:
  ```javascript
  Transaction (序列化后的交易对象)
  ```
- **业务逻辑**:
  1. 验证用户认证
  2. 查询交易记录
  3. 验证交易属于当前用户
  4. 序列化并返回
- **认证**: 需要登录用户

#### **updateTransaction**
- **文件位置**: `/home/user/ai-finance-platform/actions/transaction.js:124-195`
- **参数**:
  - `id` (string): 交易 ID
  - `data` (Transaction): 更新的交易数据
- **返回值**:
  ```javascript
  { success: boolean, data: Transaction }
  ```
- **业务逻辑**:
  1. 验证用户认证和授权
  2. 获取原始交易记录
  3. 计算余额变化差额:
     ```javascript
     oldChange = type === 'EXPENSE' ? -amount : +amount
     newChange = type === 'EXPENSE' ? -amount : +amount
     netChange = newChange - oldChange
     ```
  4. 使用数据库事务:
     - 更新交易记录
     - 更新账户余额（增量更新）
  5. 重新验证缓存
- **认证**: 需要登录用户
- **数据库事务**: 确保更新的原子性

#### **getUserTransactions**
- **文件位置**: `/home/user/ai-finance-platform/actions/transaction.js:198-228`
- **参数**:
  - `query` (object): 可选的查询条件
- **返回值**:
  ```javascript
  { success: boolean, data: Transaction[] }
  ```
- **业务逻辑**:
  1. 验证用户认证
  2. 查询用户的所有交易记录
  3. 包含关联的账户信息
  4. 按日期降序排序
- **认证**: 需要登录用户

#### **scanReceipt**
- **文件位置**: `/home/user/ai-finance-platform/actions/transaction.js:231-291`
- **参数**:
  - `file` (File): 收据图片文件
- **返回值**:
  ```javascript
  {
    amount: number,
    date: Date,
    description: string,
    category: string,
    merchantName: string
  }
  ```
- **业务逻辑**:
  1. 将文件转换为 Base64 编码
  2. 使用 Gemini AI (gemini-1.5-flash) 分析收据图片
  3. 提取交易信息：
     - 总金额
     - 交易日期
     - 商品描述
     - 商家名称
     - 建议的分类
  4. 解析 AI 返回的 JSON 响应
  5. 进行错误处理和数据验证
- **外部服务**: Google Gemini AI
- **AI Prompt**: 详细的 JSON 格式提示，要求提取特定字段
- **错误处理**: JSON 解析错误处理

---

### 1.3 Budget Actions (`actions/budget.js`)

#### **getCurrentBudget**
- **文件位置**: `/home/user/ai-finance-platform/actions/budget.js:7-64`
- **参数**:
  - `accountId` (string): 账户 ID
- **返回值**:
  ```javascript
  {
    budget: { amount: number } | null,
    currentExpenses: number
  }
  ```
- **业务逻辑**:
  1. 验证用户认证
  2. 查询用户的预算设置
  3. 计算当前月份的日期范围
  4. 聚合查询当月指定账户的所有支出
  5. 序列化并返回预算和实际支出
- **认证**: 需要登录用户
- **数据聚合**: 使用 Prisma `aggregate` 计算总支出

#### **updateBudget**
- **文件位置**: `/home/user/ai-finance-platform/actions/budget.js:66-100`
- **参数**:
  - `amount` (number): 预算金额
- **返回值**:
  ```javascript
  { success: boolean, data?: Budget, error?: string }
  ```
- **业务逻辑**:
  1. 验证用户认证
  2. 使用 `upsert` 创建或更新预算:
     - 如果存在则更新金额
     - 如果不存在则创建新预算
  3. 序列化并返回结果
- **认证**: 需要登录用户
- **数据库操作**: Upsert (Update or Insert)
- **缓存失效**: `/dashboard`

---

### 1.4 Dashboard Actions (`actions/dashboard.js`)

#### **getUserAccounts**
- **文件位置**: `/home/user/ai-finance-platform/actions/dashboard.js:20-52`
- **参数**: 无
- **返回值**:
  ```javascript
  Account[] (序列化后的账户数组)
  ```
- **业务逻辑**:
  1. 验证用户认证
  2. 查询用户的所有账户
  3. 包含交易计数统计
  4. 按创建时间降序排序
  5. 序列化所有账户数据
- **认证**: 需要登录用户
- **关联查询**: 包含 `_count` 统计交易数量

#### **createAccount**
- **文件位置**: `/home/user/ai-finance-platform/actions/dashboard.js:54-135`
- **参数**:
  ```javascript
  {
    name: string,
    type: 'CURRENT' | 'SAVINGS',
    balance: string,
    isDefault?: boolean
  }
  ```
- **返回值**:
  ```javascript
  { success: boolean, data: Account }
  ```
- **业务逻辑**:
  1. **认证验证**: Clerk 用户认证
  2. **速率限制**: ArcJet Token Bucket (10次/小时)
  3. **数据验证**:
     - 将 balance 字符串转换为浮点数
     - 验证数字有效性
  4. **默认账户逻辑**:
     - 如果是用户第一个账户，自动设为默认
     - 如果设置为默认，取消其他账户的默认标记
  5. **创建账户**: 保存到数据库
  6. **缓存失效**: 重新验证仪表板
- **认证**: 需要登录用户
- **速率限制**: ArcJet (10/hour)
- **智能逻辑**: 自动处理第一个账户为默认账户

#### **getDashboardData**
- **文件位置**: `/home/user/ai-finance-platform/actions/dashboard.js:137-156`
- **参数**: 无
- **返回值**:
  ```javascript
  Transaction[] (序列化后的交易数组)
  ```
- **业务逻辑**:
  1. 验证用户认证
  2. 查询用户的所有交易记录
  3. 按日期降序排序
  4. 序列化所有交易数据
- **认证**: 需要登录用户

---

### 1.5 Email Actions (`actions/send-email.js`)

#### **sendEmail**
- **文件位置**: `/home/user/ai-finance-platform/actions/send-email.js:5-21`
- **参数**:
  ```javascript
  {
    to: string | string[],
    subject: string,
    react: ReactElement
  }
  ```
- **返回值**:
  ```javascript
  { success: boolean, data?: any, error?: any }
  ```
- **业务逻辑**:
  1. 初始化 Resend 客户端
  2. 发送 React 邮件模板
  3. 使用固定发件人地址
  4. 返回成功或错误结果
- **外部服务**: Resend Email API
- **邮件模板**: React Email 组件
- **发件人**: `Finance App <onboarding@resend.dev>`

---

### 1.6 Seed Actions (`actions/seed.js`)

#### **seedTransactions**
- **文件位置**: `/home/user/ai-finance-platform/actions/seed.js:44-109`
- **参数**: 无
- **返回值**:
  ```javascript
  { success: boolean, message?: string, error?: string }
  ```
- **业务逻辑**:
  1. 生成 90 天的模拟交易数据
  2. 每天随机生成 1-3 笔交易
  3. 收入/支出比例 40%/60%
  4. 根据分类生成随机金额
  5. 使用数据库事务:
     - 清空现有交易记录
     - 批量插入新交易
     - 更新账户余额
- **用途**: 开发和测试数据播种
- **数据量**: 约 90-270 笔交易
- **分类**: 收入 4 类，支出 10 类

---

## 2. API Routes Analysis | API Routes 分析

### 2.1 Inngest Webhook Route

**路径**: `/api/inngest`
**文件**: `/home/user/ai-finance-platform/app/api/inngest/route.js`

#### 功能说明
这是 Inngest 后台作业系统的 webhook 端点，用于接收和处理异步任务。

#### 支持的 HTTP 方法
- **GET**: 健康检查和配置验证
- **POST**: 接收 Inngest 事件
- **PUT**: 更新 Inngest 函数配置

#### 注册的后台函数
1. **processRecurringTransaction** (处理周期性交易)
2. **triggerRecurringTransactions** (触发周期性交易)
3. **generateMonthlyReports** (生成月度报告)
4. **checkBudgetAlerts** (检查预算警报)

#### 配置详情
```javascript
serve({
  client: inngest,           // Inngest 客户端实例
  functions: [               // 注册的函数列表
    processRecurringTransaction,
    triggerRecurringTransactions,
    generateMonthlyReports,
    checkBudgetAlerts,
  ],
})
```

---

### 2.2 Seed Data Route

**路径**: `/api/seed`
**文件**: `/home/user/ai-finance-platform/app/api/seed/route.js`

#### 功能说明
提供 HTTP 接口用于数据播种，方便开发和测试。

#### HTTP 方法
- **GET**: 触发数据播种

#### 实现
```javascript
export async function GET() {
  const result = await seedTransactions();
  return Response.json(result);
}
```

#### 使用场景
- 开发环境初始化数据
- 测试数据重置
- Demo 演示准备

---

## 3. Backend Logic Flow Diagrams | 后端逻辑流程图

### 3.1 User Authentication Flow | 用户认证流程

```
┌─────────────────────────────────────────────────────────────────┐
│                     User Request (HTTP)                         │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Middleware Chain                              │
│  ┌──────────────┐      ┌──────────────┐                        │
│  │   ArcJet     │ ───► │    Clerk     │                        │
│  │  (Security)  │      │   (Auth)     │                        │
│  └──────────────┘      └──────────────┘                        │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                ┌──────────┴──────────┐
                │                     │
                ▼                     ▼
    ┌──────────────────┐   ┌──────────────────┐
    │  Shield Check    │   │   Bot Detection  │
    │  (Content/XSS)   │   │   (Search Bots)  │
    └─────────┬────────┘   └─────────┬────────┘
              │                      │
              │    ┌─────────────────┘
              │    │
              ▼    ▼
         ┌─────────────┐
         │   Allowed?  │
         └──────┬──────┘
                │
        ┌───────┴────────┐
        │                │
        ▼ YES            ▼ NO
  ┌──────────┐    ┌──────────────┐
  │  Check   │    │  Block/       │
  │  userId  │    │  Redirect     │
  └────┬─────┘    └──────────────┘
       │
       ▼
┌──────────────┐
│ Protected    │
│ Route?       │
└──────┬───────┘
       │
   ┌───┴────┐
   │        │
   ▼ YES    ▼ NO
┌────────┐ ┌──────────────┐
│userId? │ │   Continue   │
└───┬────┘ └──────────────┘
    │
┌───┴────┐
│        │
▼ NO     ▼ YES
┌──────────────┐  ┌──────────────┐
│ Redirect to  │  │   Continue   │
│   Sign In    │  │              │
└──────────────┘  └──────────────┘
                  │
                  ▼
          ┌──────────────┐
          │ Route Handler│
          │  or Server   │
          │   Action     │
          └──────────────┘
```

### 3.2 Transaction CRUD Flow | 交易数据流程

```
┌─────────────────────────────────────────────────────────────────┐
│                   Create Transaction Request                     │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │  Clerk Auth     │
                  │  Validation     │
                  └────────┬────────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │  ArcJet Rate    │
                  │  Limit Check    │
                  │  (10/hour)      │
                  └────────┬────────┘
                           │
                    ┌──────┴─────┐
                    │            │
                    ▼ PASS       ▼ DENY
            ┌───────────┐  ┌──────────────┐
            │ Continue  │  │ Throw Error  │
            │           │  │ "Too many    │
            └─────┬─────┘  │  requests"   │
                  │        └──────────────┘
                  ▼
         ┌─────────────────┐
         │ Validate User   │
         │ & Account       │
         │ Ownership       │
         └────────┬────────┘
                  │
                  ▼
         ┌─────────────────┐
         │ Calculate       │
         │ Balance Change  │
         │ EXPENSE: -amt   │
         │ INCOME:  +amt   │
         └────────┬────────┘
                  │
                  ▼
         ┌─────────────────────────────────┐
         │   Start Database Transaction    │
         │                                 │
         │  ┌────────────────────────┐    │
         │  │ 1. Create Transaction  │    │
         │  └───────────┬────────────┘    │
         │              │                  │
         │              ▼                  │
         │  ┌────────────────────────┐    │
         │  │ 2. Update Account      │    │
         │  │    Balance             │    │
         │  └───────────┬────────────┘    │
         │              │                  │
         │              ▼                  │
         │  ┌────────────────────────┐    │
         │  │ 3. Set Recurring Date  │    │
         │  │    (if applicable)     │    │
         │  └────────────────────────┘    │
         │                                 │
         └────────┬────────────────────────┘
                  │
                  ▼
         ┌─────────────────┐
         │ Revalidate      │
         │ Cache Paths:    │
         │ - /dashboard    │
         │ - /account/[id] │
         └────────┬────────┘
                  │
                  ▼
         ┌─────────────────┐
         │ Return Success  │
         │ with Serialized │
         │ Transaction     │
         └─────────────────┘


┌─────────────────────────────────────────────────────────────────┐
│                   Update Transaction Request                     │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │  Fetch Original │
                  │  Transaction    │
                  └────────┬────────┘
                           │
                           ▼
         ┌──────────────────────────────┐
         │ Calculate Balance Differences│
         │                              │
         │ oldChange = original impact  │
         │ newChange = updated impact   │
         │ netChange = new - old        │
         └──────────┬───────────────────┘
                    │
                    ▼
         ┌─────────────────────────────────┐
         │   Start Database Transaction    │
         │                                 │
         │  ┌────────────────────────┐    │
         │  │ 1. Update Transaction  │    │
         │  └───────────┬────────────┘    │
         │              │                  │
         │              ▼                  │
         │  ┌────────────────────────┐    │
         │  │ 2. Increment Account   │    │
         │  │    Balance by netChange│    │
         │  └────────────────────────┘    │
         └─────────────────────────────────┘


┌─────────────────────────────────────────────────────────────────┐
│                Bulk Delete Transactions Request                  │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │ Fetch All       │
                  │ Transactions    │
                  │ to Delete       │
                  └────────┬────────┘
                           │
                           ▼
         ┌─────────────────────────────┐
         │ Group by Account &          │
         │ Calculate Balance Changes   │
         │ per Account                 │
         └──────────┬──────────────────┘
                    │
                    ▼
         ┌─────────────────────────────────┐
         │   Start Database Transaction    │
         │                                 │
         │  ┌────────────────────────┐    │
         │  │ 1. Delete All          │    │
         │  │    Transactions        │    │
         │  └───────────┬────────────┘    │
         │              │                  │
         │              ▼                  │
         │  ┌────────────────────────┐    │
         │  │ 2. Loop Each Account:  │    │
         │  │    Increment Balance   │    │
         │  └────────────────────────┘    │
         └─────────────────────────────────┘
```

### 3.3 Error Handling Pattern | 错误处理模式

```
┌─────────────────────────────────────────────────────────────────┐
│                      Server Action Call                          │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │  try { }        │
                  └────────┬────────┘
                           │
            ┌──────────────┴──────────────┐
            │                             │
            ▼                             ▼
    ┌────────────────┐          ┌────────────────┐
    │ Auth Check     │          │ Rate Limit     │
    │ if (!userId)   │          │ Check          │
    │ throw Error    │          └────────┬───────┘
    └────────┬───────┘                   │
             │                            │
             │         ┌──────────────────┘
             │         │
             ▼         ▼
      ┌──────────────────────┐
      │  Business Logic      │
      │  - DB operations     │
      │  - Calculations      │
      │  - Validations       │
      └──────────┬───────────┘
                 │
          ┌──────┴──────┐
          │             │
          ▼ SUCCESS     ▼ ERROR
  ┌───────────────┐   ┌─────────────┐
  │ return {      │   │ catch (e)   │
  │   success:true│   └──────┬──────┘
  │   data: ...   │          │
  │ }             │          ▼
  └───────────────┘   ┌─────────────────────┐
                      │ Error Type Check    │
                      └──────┬──────────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
      ┌──────────────┐ ┌──────────┐ ┌─────────────┐
      │ Rate Limit   │ │ Auth     │ │ Other Errors│
      │ Error        │ │ Error    │ │             │
      └──────┬───────┘ └────┬─────┘ └──────┬──────┘
             │              │               │
             ▼              ▼               ▼
      ┌─────────────────────────────────────────┐
      │  return {                               │
      │    success: false,                      │
      │    error: error.message                 │
      │  }                                      │
      │                                         │
      │  OR throw new Error(error.message)     │
      └─────────────────────────────────────────┘
```

---

## 4. External Services Integration | 外部服务集成

### 4.1 Clerk (Authentication) | 认证服务

**集成位置**:
- Middleware: `/home/user/ai-finance-platform/middleware.js`
- All Server Actions: `await auth()` 调用

**功能**:
- 用户认证和会话管理
- 保护路由和 API
- 用户信息获取

**配置**:
```javascript
// middleware.js:32-41
const clerk = clerkMiddleware(async (auth, req) => {
  const { userId } = await auth();

  if (!userId && isProtectedRoute(req)) {
    const { redirectToSignIn } = await auth();
    return redirectToSignIn();
  }

  return NextResponse.next();
});
```

**保护的路由**:
```javascript
// middleware.js:5-9
const isProtectedRoute = createRouteMatcher([
  "/dashboard(.*)",
  "/account(.*)",
  "/transaction(.*)",
]);
```

**用户自动创建流程**:
```javascript
// lib/checkUser.js:4-37
1. 从 Clerk 获取当前用户
2. 在数据库中查找 clerkUserId
3. 如果不存在:
   - 创建新用户记录
   - 同步 Clerk 用户信息 (name, email, imageUrl)
4. 返回用户对象
```

---

### 4.2 Google Gemini AI (Receipt Scanning) | 收据扫描

**集成位置**:
- `/home/user/ai-finance-platform/actions/transaction.js:231-291`
- `/home/user/ai-finance-platform/lib/inngest/function.js:129-165`

**用途**:
1. **收据扫描 (scanReceipt)**:
   - 提取交易金额、日期、描述、商家、分类
   - 模型: `gemini-1.5-flash`

2. **财务洞察生成 (generateFinancialInsights)**:
   - 分析月度财务数据
   - 生成 3 条可行的建议
   - 模型: `gemini-1.5-flash`

**API 配置**:
```javascript
const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY);
const model = genAI.getGenerativeModel({ model: "gemini-1.5-flash" });
```

**Prompt Engineering 示例**:
```javascript
// Receipt Scanning Prompt
const prompt = `
  Analyze this receipt image and extract the following information in JSON format:
  - Total amount (just the number)
  - Date (in ISO format)
  - Description or items purchased (brief summary)
  - Merchant/store name
  - Suggested category (one of: housing, transportation, groceries, ...)

  Only respond with valid JSON in this exact format:
  {
    "amount": number,
    "date": "ISO date string",
    "description": "string",
    "merchantName": "string",
    "category": "string"
  }

  If its not a recipt, return an empty object
`;
```

**错误处理**:
- JSON 解析失败时抛出 "Invalid response format"
- 提供降级的默认财务洞察

---

### 4.3 Resend (Email Service) | 邮件服务

**集成位置**: `/home/user/ai-finance-platform/actions/send-email.js`

**功能**:
- 发送事务性邮件
- 使用 React Email 模板
- 支持单个或多个收件人

**配置**:
```javascript
const resend = new Resend(process.env.RESEND_API_KEY);
```

**使用场景**:
1. **月度财务报告** (Inngest: generateMonthlyReports)
   - 主题: "Your Monthly Financial Report - {Month}"
   - 内容: 收入、支出统计 + AI 洞察

2. **预算警报** (Inngest: checkBudgetAlerts)
   - 主题: "Budget Alert for {AccountName}"
   - 内容: 预算使用百分比、剩余金额

**邮件模板**:
```javascript
EmailTemplate({
  userName: user.name,
  type: 'monthly-report' | 'budget-alert',
  data: {
    // type-specific data
  }
})
```

---

### 4.4 Inngest (Background Jobs) | 后台作业

**集成位置**:
- Client: `/home/user/ai-finance-platform/lib/inngest/client.js`
- Functions: `/home/user/ai-finance-platform/lib/inngest/function.js`
- Route: `/home/user/ai-finance-platform/app/api/inngest/route.js`

**配置**:
```javascript
// lib/inngest/client.js
export const inngest = new Inngest({
  id: "finance-platform",
  name: "Finance Platform",
  retryFunction: async (attempt) => ({
    delay: Math.pow(2, attempt) * 1000, // 指数退避
    maxAttempts: 2,
  }),
});
```

#### 后台函数列表

**1. processRecurringTransaction** (处理单个周期性交易)
- **触发**: Event `transaction.recurring.process`
- **节流**: 10 transactions/min per user
- **逻辑**:
  1. 验证交易是否到期
  2. 创建新交易记录
  3. 更新账户余额
  4. 更新下次执行时间

**2. triggerRecurringTransactions** (批量触发周期性交易)
- **触发**: Cron `0 0 * * *` (每天午夜)
- **逻辑**:
  1. 查询所有到期的周期性交易
  2. 批量发送 `transaction.recurring.process` 事件
  3. 返回触发数量

**3. generateMonthlyReports** (生成月度报告)
- **触发**: Cron `0 0 1 * *` (每月 1 日)
- **逻辑**:
  1. 获取所有用户
  2. 对每个用户:
     - 计算上月财务统计
     - 使用 Gemini AI 生成洞察
     - 发送邮件报告
  3. 返回处理用户数

**4. checkBudgetAlerts** (检查预算警报)
- **触发**: Cron `0 */6 * * *` (每 6 小时)
- **逻辑**:
  1. 获取所有预算设置
  2. 对每个预算:
     - 计算默认账户的本月支出
     - 检查是否超过 80% 阈值
     - 检查本月是否已发送警报
     - 发送警报邮件
     - 更新最后警报时间

---

### 4.5 ArcJet (Security & Rate Limiting) | 安全防护

**集成位置**:
- Middleware: `/home/user/ai-finance-platform/middleware.js`
- Server Actions Rate Limit: `/home/user/ai-finance-platform/lib/arcjet.js`

#### Middleware 配置 (全局安全)

```javascript
// middleware.js:12-29
const aj = arcjet({
  key: process.env.ARCJET_KEY,
  rules: [
    shield({
      mode: "LIVE",  // 实时防护
    }),
    detectBot({
      mode: "LIVE",  // 实时检测
      allow: [
        "CATEGORY:SEARCH_ENGINE", // Google, Bing
        "GO_HTTP",                // Inngest webhooks
      ],
    }),
  ],
});
```

**功能**:
1. **Shield 防护**:
   - 防止内容注入攻击
   - XSS 防护
   - SQL 注入防护

2. **Bot 检测**:
   - 阻止恶意 Bot
   - 允许搜索引擎爬虫
   - 允许 Inngest HTTP 客户端

#### Server Actions 速率限制

```javascript
// lib/arcjet.js:3-15
const aj = arcjet({
  key: process.env.ARCJET_KEY,
  characteristics: ["userId"],  // 基于用户 ID 限流
  rules: [
    tokenBucket({
      mode: "LIVE",
      refillRate: 10,      // 每小时补充 10 个令牌
      interval: 3600,      // 1 小时
      capacity: 10,        // 最大容量 10
    }),
  ],
});
```

**应用位置**:
- `createTransaction` (actions/transaction.js:27-47)
- `createAccount` (actions/dashboard.js:63-83)

**使用模式**:
```javascript
// 1. 获取请求对象
const req = await request();

// 2. 检查速率限制
const decision = await aj.protect(req, {
  userId,
  requested: 1,  // 消耗 1 个令牌
});

// 3. 处理限制结果
if (decision.isDenied()) {
  if (decision.reason.isRateLimit()) {
    const { remaining, reset } = decision.reason;
    throw new Error("Too many requests. Please try again later.");
  }
  throw new Error("Request blocked");
}
```

---

## 5. Server Actions vs API Routes Comparison | 对比

| 特性 | Server Actions | API Routes |
|------|----------------|------------|
| **定义位置** | 文件顶部 `"use server"` | `app/api/*/route.js` |
| **调用方式** | 直接导入函数调用 | HTTP 请求 (fetch) |
| **类型安全** | ✅ 完全类型安全 (TypeScript) | ⚠️ 需要手动定义类型 |
| **表单集成** | ✅ 原生支持 (action prop) | ❌ 需要手动处理 |
| **进度状态** | ✅ useFormStatus, useFormState | ❌ 需要自定义状态 |
| **缓存失效** | ✅ revalidatePath/Tag | ❌ 需要手动触发 |
| **错误处理** | try/catch + return | try/catch + Response |
| **认证** | `await auth()` | `await auth()` |
| **速率限制** | 需要手动集成 ArcJet | 中间件自动应用 |
| **适用场景** | 表单提交、数据变更 | Webhooks、第三方集成 |

### 项目中的使用策略

**Server Actions 用于**:
- ✅ 所有数据 CRUD 操作 (账户、交易、预算)
- ✅ 表单提交 (创建账户、创建交易)
- ✅ 用户触发的操作 (删除、更新)
- ✅ 需要缓存失效的操作

**API Routes 用于**:
- ✅ Inngest webhook 端点 (`/api/inngest`)
- ✅ 开发工具端点 (`/api/seed`)
- ✅ 第三方服务回调
- ✅ 需要公开访问的端点

---

## 6. Error Handling Patterns | 错误处理模式

### 6.1 Pattern A: Return Object with Success Flag

**使用场景**: 需要在客户端处理错误 UI 的操作

**示例**:
```javascript
// actions/budget.js:66-100
export async function updateBudget(amount) {
  try {
    // ... business logic
    return {
      success: true,
      data: { ...budget, amount: budget.amount.toNumber() },
    };
  } catch (error) {
    console.error("Error updating budget:", error);
    return { success: false, error: error.message };
  }
}
```

**客户端处理**:
```javascript
const result = await updateBudget(500);
if (result.success) {
  // Show success toast
} else {
  // Show error message: result.error
}
```

**优点**:
- ✅ 不会中断客户端执行
- ✅ 可以显示详细错误信息
- ✅ 适合表单验证错误

---

### 6.2 Pattern B: Throw Error

**使用场景**: 严重错误或认证失败

**示例**:
```javascript
// actions/transaction.js:18-100
export async function createTransaction(data) {
  try {
    const { userId } = await auth();
    if (!userId) throw new Error("Unauthorized");

    // Rate limiting
    if (decision.isDenied()) {
      if (decision.reason.isRateLimit()) {
        throw new Error("Too many requests. Please try again later.");
      }
      throw new Error("Request blocked");
    }

    // ... success logic
    return { success: true, data: serializeAmount(transaction) };
  } catch (error) {
    throw new Error(error.message);  // Re-throw
  }
}
```

**客户端处理**:
```javascript
try {
  const result = await createTransaction(data);
  // Handle success
} catch (error) {
  // Show error: error.message
}
```

**优点**:
- ✅ 阻止无效操作继续执行
- ✅ 清晰的错误边界
- ✅ 适合安全和认证错误

---

### 6.3 Pattern C: Mixed Approach

**使用场景**: 区分不同错误类型

**示例**:
```javascript
// actions/account.js:51-112
export async function bulkDeleteTransactions(transactionIds) {
  try {
    const { userId } = await auth();
    if (!userId) throw new Error("Unauthorized");  // 严重错误

    // ... delete logic
    return { success: true };
  } catch (error) {
    return { success: false, error: error.message };  // 包装返回
  }
}
```

---

### 6.4 ArcJet Rate Limit Error Handling

**特殊处理**: 详细日志记录

```javascript
// actions/transaction.js:32-44
if (decision.isDenied()) {
  if (decision.reason.isRateLimit()) {
    const { remaining, reset } = decision.reason;
    console.error({
      code: "RATE_LIMIT_EXCEEDED",
      details: {
        remaining,           // 剩余令牌数
        resetInSeconds: reset,  // 重置时间
      },
    });

    throw new Error("Too many requests. Please try again later.");
  }

  throw new Error("Request blocked");
}
```

**提供的信息**:
- `remaining`: 剩余可用请求数
- `reset`: 限制重置时间（秒）
- 有助于用户了解何时可以重试

---

## 7. Data Serialization Handling | 数据序列化处理

### 7.1 问题背景

**Prisma Decimal Type**:
- Prisma 使用 `Decimal` 类型存储精确的货币金额
- `Decimal` 对象无法直接序列化为 JSON
- Next.js Server Actions 需要可序列化的返回值

**错误示例**:
```javascript
// ❌ This will cause serialization error
return account;  // account.balance is Decimal
```

---

### 7.2 解决方案: Serialization Functions

#### Pattern A: serializeDecimal

**文件**: `/home/user/ai-finance-platform/actions/account.js:7-16`

```javascript
const serializeDecimal = (obj) => {
  const serialized = { ...obj };
  if (obj.balance) {
    serialized.balance = obj.balance.toNumber();
  }
  if (obj.amount) {
    serialized.amount = obj.amount.toNumber();
  }
  return serialized;
};
```

**使用场景**: 账户对象（包含 balance）

**使用示例**:
```javascript
const account = await db.account.findUnique({ ... });
return {
  ...serializeDecimal(account),
  transactions: account.transactions.map(serializeDecimal),
};
```

---

#### Pattern B: serializeAmount

**文件**: `/home/user/ai-finance-platform/actions/transaction.js:12-15`

```javascript
const serializeAmount = (obj) => ({
  ...obj,
  amount: obj.amount.toNumber(),
});
```

**使用场景**: 交易对象（仅包含 amount）

**使用示例**:
```javascript
const transaction = await db.transaction.create({ ... });
return { success: true, data: serializeAmount(transaction) };
```

---

#### Pattern C: serializeTransaction

**文件**: `/home/user/ai-finance-platform/actions/dashboard.js:9-18`

```javascript
const serializeTransaction = (obj) => {
  const serialized = { ...obj };
  if (obj.balance) {
    serialized.balance = obj.balance.toNumber();
  }
  if (obj.amount) {
    serialized.amount = obj.amount.toNumber();
  }
  return serialized;
};
```

**使用场景**: 通用对象（可能包含 balance 或 amount）

---

### 7.3 Serialization Best Practices | 最佳实践

**1. 在返回前始终序列化**
```javascript
✅ return serializeAmount(transaction);
❌ return transaction;
```

**2. 对数组使用 map 序列化**
```javascript
✅ return transactions.map(serializeAmount);
❌ return transactions;
```

**3. 嵌套对象的序列化**
```javascript
return {
  ...serializeDecimal(account),
  transactions: account.transactions.map(serializeDecimal),
};
```

**4. 聚合结果的手动转换**
```javascript
// actions/budget.js:54-59
return {
  budget: budget ? { ...budget, amount: budget.amount.toNumber() } : null,
  currentExpenses: expenses._sum.amount
    ? expenses._sum.amount.toNumber()
    : 0,
};
```

---

### 7.4 完整序列化流程

```
Database Query Result
         │
         │ Prisma returns Decimal objects
         │
         ▼
┌─────────────────┐
│ Prisma Object   │
│ {               │
│   amount: Decimal│  ← Cannot be serialized
│   balance: Decimal│ ← Cannot be serialized
│ }               │
└────────┬────────┘
         │
         │ Apply serialization function
         │
         ▼
┌─────────────────┐
│ serializeAmount │
│ or              │
│ serializeDecimal│
└────────┬────────┘
         │
         │ Decimal.toNumber()
         │
         ▼
┌─────────────────┐
│ Plain Object    │
│ {               │
│   amount: 100.50│  ← JavaScript number
│   balance: 5000 │  ← JavaScript number
│ }               │
└────────┬────────┘
         │
         │ Return from Server Action
         │
         ▼
┌─────────────────┐
│ JSON Serialized │
│ to Client       │
└─────────────────┘
```

---

## Summary | 总结

### Backend Architecture Highlights | 后端架构亮点

1. **Server Actions 为主导**:
   - 所有业务逻辑使用 Server Actions
   - 类型安全、易于维护
   - 与 React 19 深度集成

2. **多层安全防护**:
   - Clerk 用户认证
   - ArcJet Shield 内容防护
   - ArcJet Rate Limiting 速率限制
   - Bot 检测和过滤

3. **后台作业系统**:
   - Inngest 处理周期性任务
   - 自动财务报告生成
   - 预算警报监控

4. **AI 增强**:
   - Gemini AI 收据扫描
   - 自动财务洞察生成
   - 智能分类建议

5. **数据完整性**:
   - Prisma 事务确保原子性
   - 余额自动计算和同步
   - Decimal 精确货币处理

6. **性能优化**:
   - Next.js 缓存策略 (revalidatePath)
   - 批量操作支持
   - 数据序列化优化

---

**文档版本**: 1.0
**最后更新**: 2025-11-15
**相关文档**:
- `00-project-overview.md`
- `01-database-analysis.md`
- `04-security-analysis.md`
