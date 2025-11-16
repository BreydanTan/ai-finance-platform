# 数据库深度分析 | Database Deep Analysis

**分析日期 | Analysis Date**: 2025-11-15
**数据库类型 | Database Type**: PostgreSQL (via Supabase)
**ORM**: Prisma 6.0.1

---

## 📋 目录 | Table of Contents

1. [数据库概览](#数据库概览--database-overview)
2. [数据模型详解](#数据模型详解--data-models-detailed)
3. [关系图谱](#关系图谱--relationship-diagrams)
4. [索引策略](#索引策略--indexing-strategy)
5. [数据流模式](#数据流模式--data-flow-patterns)
6. [迁移历史](#迁移历史--migration-history)
7. [数据验证规则](#数据验证规则--validation-rules)
8. [示例查询](#示例查询--example-queries)

---

## 数据库概览 | Database Overview

### 配置信息 | Configuration

**文件位置 | File**: `prisma/schema.prisma:6-9`

```prisma
datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")      // 连接池 URL (Supabase Pooler)
  directUrl = env("DIRECT_URL")        // 直连 URL (用于迁移)
}
```

### 数据模型统计 | Model Statistics

| 模型名称 | 表名 | 字段数 | 关系数 | 索引数 | 用途 |
|----------|------|--------|--------|--------|------|
| **User** | users | 7 | 3 (1:N) | 2 | 用户账户 |
| **Account** | accounts | 8 | 2 (N:1, 1:N) | 1 | 财务账户 |
| **Transaction** | transactions | 13 | 2 (N:1) | 2 | 交易记录 |
| **Budget** | budgets | 6 | 1 (N:1) | 1 | 预算限制 |

**总计 | Total**:
- **4 个数据表** (Models)
- **4 个枚举类型** (Enums)
- **8 个关系** (Relations)
- **6 个索引** (Indexes)
- **8 个外键约束** (Foreign Keys with CASCADE DELETE)

---

## 数据模型详解 | Data Models Detailed

### 1. User 模型（用户）

**表名 | Table**: `users`
**文件位置 | Location**: `prisma/schema.prisma:11-24`

#### 字段清单 | Fields

| 字段名 | 类型 | 约束 | 默认值 | 说明 |
|--------|------|------|--------|------|
| `id` | String | PRIMARY KEY | `uuid()` | 用户唯一标识符 |
| `clerkUserId` | String | UNIQUE, NOT NULL | - | Clerk 认证系统的用户 ID |
| `email` | String | UNIQUE, NOT NULL | - | 用户邮箱 |
| `name` | String? | NULLABLE | - | 用户全名 |
| `imageUrl` | String? | NULLABLE | - | 用户头像 URL |
| `createdAt` | DateTime | NOT NULL | `now()` | 创建时间 |
| `updatedAt` | DateTime | NOT NULL, AUTO | - | 更新时间（自动） |

#### 关系 | Relations

```
User (1) ──────→ (N) Account
User (1) ──────→ (N) Transaction
User (1) ──────→ (N) Budget
```

- **1 对多关系** (One-to-Many)：
  - `accounts: Account[]` - 一个用户可以有多个账户
  - `transactions: Transaction[]` - 一个用户可以有多个交易
  - `budgets: Budget[]` - 一个用户可以有多个预算

#### 索引 | Indexes

- `clerkUserId` - UNIQUE INDEX（唯一索引，用于快速查找）
- `email` - UNIQUE INDEX（唯一索引，防止重复邮箱）

#### 业务规则 | Business Rules

1. **自动创建**：当用户通过 Clerk 首次登录时自动创建
2. **数据同步**：从 Clerk 同步 `name`、`email`、`imageUrl`
3. **级联删除**：删除用户时自动删除所有关联的账户、交易、预算

**创建逻辑 | Creation Logic**: `lib/checkUser.js:4-37`

```javascript
// 检查用户是否存在，不存在则创建
const loggedInUser = await db.user.findUnique({
  where: { clerkUserId: user.id }
});

if (!loggedInUser) {
  const newUser = await db.user.create({
    data: {
      clerkUserId: user.id,
      name: `${user.firstName} ${user.lastName}`,
      imageUrl: user.imageUrl,
      email: user.emailAddresses[0].emailAddress,
    }
  });
}
```

---

### 2. Account 模型（账户）

**表名 | Table**: `accounts`
**文件位置 | Location**: `prisma/schema.prisma:26-40`

#### 字段清单 | Fields

| 字段名 | 类型 | 约束 | 默认值 | 说明 |
|--------|------|------|--------|------|
| `id` | String | PRIMARY KEY | `uuid()` | 账户唯一标识符 |
| `name` | String | NOT NULL | - | 账户名称（如"工资卡"） |
| `type` | AccountType | ENUM, NOT NULL | - | 账户类型（CURRENT/SAVINGS） |
| `balance` | Decimal | NOT NULL | `0` | 当前余额（自动计算） |
| `isDefault` | Boolean | NOT NULL | `false` | 是否为默认账户 |
| `userId` | String | FOREIGN KEY | - | 所属用户 ID |
| `createdAt` | DateTime | NOT NULL | `now()` | 创建时间 |
| `updatedAt` | DateTime | NOT NULL, AUTO | - | 更新时间 |

#### 枚举类型 | Enum

**AccountType** (prisma/schema.prisma:86-89):
```prisma
enum AccountType {
  CURRENT   // 活期账户（日常消费）
  SAVINGS   // 储蓄账户（长期存储）
}
```

#### 关系 | Relations

```
User (1) ←────── (N) Account
Account (1) ───→ (N) Transaction
```

- **多对一关系** (Many-to-One)：
  - `user: User` - 每个账户属于一个用户
  - 级联删除：用户删除时，账户也会被删除

- **一对多关系** (One-to-Many)：
  - `transactions: Transaction[]` - 一个账户可以有多个交易
  - 包含交易计数：`_count.transactions`

#### 索引 | Indexes

- `userId` - INDEX（用于快速查询用户的所有账户）

#### 业务规则 | Business Rules

**文件位置 | Location**: `actions/account.js:54-135`

1. **默认账户逻辑**：
   ```javascript
   // 如果是用户的第一个账户，自动设为默认
   const shouldBeDefault = existingAccounts.length === 0 ? true : data.isDefault;

   // 如果设置新的默认账户，取消其他账户的默认状态
   if (shouldBeDefault) {
     await db.account.updateMany({
       where: { userId: user.id, isDefault: true },
       data: { isDefault: false }
     });
   }
   ```

2. **余额管理**：
   - 初始余额：创建时设置
   - 自动更新：每次交易时更新（收入+，支出-）
   - 使用 Decimal 类型：精确的货币计算（避免浮点误差）

3. **数据验证**（Zod Schema）：
   ```javascript
   // app/lib/schema.js:3-8
   export const accountSchema = z.object({
     name: z.string().min(1, "Name is required"),
     type: z.enum(["CURRENT", "SAVINGS"]),
     balance: z.string().min(1, "Initial balance is required"),
     isDefault: z.boolean().default(false)
   });
   ```

---

### 3. Transaction 模型（交易）

**表名 | Table**: `transactions`
**文件位置 | Location**: `prisma/schema.prisma:42-65`

#### 字段清单 | Fields

| 字段名 | 类型 | 约束 | 默认值 | 说明 |
|--------|------|------|--------|------|
| `id` | String | PRIMARY KEY | `uuid()` | 交易唯一标识符 |
| `type` | TransactionType | ENUM, NOT NULL | - | 交易类型（INCOME/EXPENSE） |
| `amount` | Decimal | NOT NULL | - | 交易金额 |
| `description` | String? | NULLABLE | - | 交易描述 |
| `date` | DateTime | NOT NULL | - | 交易日期 |
| `category` | String | NOT NULL | - | 交易分类（字符串） |
| `receiptUrl` | String? | NULLABLE | - | 收据图片 URL |
| `isRecurring` | Boolean | NOT NULL | `false` | 是否为循环交易 |
| `recurringInterval` | RecurringInterval? | ENUM, NULLABLE | - | 循环间隔 |
| `nextRecurringDate` | DateTime? | NULLABLE | - | 下次自动创建日期 |
| `lastProcessed` | DateTime? | NULLABLE | - | 上次处理时间 |
| `status` | TransactionStatus | ENUM, NOT NULL | `COMPLETED` | 交易状态 |
| `userId` | String | FOREIGN KEY | - | 所属用户 ID |
| `accountId` | String | FOREIGN KEY | - | 所属账户 ID |
| `createdAt` | DateTime | NOT NULL | `now()` | 创建时间 |
| `updatedAt` | DateTime | NOT NULL, AUTO | - | 更新时间 |

#### 枚举类型 | Enums

**TransactionType** (prisma/schema.prisma:81-84):
```prisma
enum TransactionType {
  INCOME    // 收入
  EXPENSE   // 支出
}
```

**TransactionStatus** (prisma/schema.prisma:91-95):
```prisma
enum TransactionStatus {
  PENDING    // 待处理（循环交易等待执行）
  COMPLETED  // 已完成
  FAILED     // 失败
}
```

**RecurringInterval** (prisma/schema.prisma:97-102):
```prisma
enum RecurringInterval {
  DAILY     // 每日
  WEEKLY    // 每周
  MONTHLY   // 每月
  YEARLY    // 每年
}
```

#### 关系 | Relations

```
User (1) ←────── (N) Transaction
Account (1) ←──── (N) Transaction
```

- **多对一关系** (Many-to-One)：
  - `user: User` - 每笔交易属于一个用户
  - `account: Account` - 每笔交易属于一个账户
  - 级联删除：删除用户或账户时，交易也会被删除

#### 索引 | Indexes

- `userId` - INDEX（快速查询用户的所有交易）
- `accountId` - INDEX（快速查询账户的所有交易）

#### 业务规则 | Business Rules

**文件位置 | Location**: `actions/transaction.js`

##### 1. 创建交易（Create Transaction）

**流程 | Flow**:
```
1. 验证用户身份 (Clerk Auth)
2. ArcJet 速率限制检查
3. 验证账户归属权
4. 计算余额变化
5. 使用 Prisma Transaction 原子操作：
   ├─ 创建交易记录
   └─ 更新账户余额
6. 重新验证相关页面缓存
```

**代码示例 | Code**: `actions/transaction.js:18-100`

```javascript
// 计算余额变化
const balanceChange = data.type === "EXPENSE" ? -data.amount : data.amount;
const newBalance = account.balance.toNumber() + balanceChange;

// 原子操作（确保数据一致性）
const transaction = await db.$transaction(async (tx) => {
  const newTransaction = await tx.transaction.create({
    data: {
      ...data,
      userId: user.id,
      nextRecurringDate: data.isRecurring && data.recurringInterval
        ? calculateNextRecurringDate(data.date, data.recurringInterval)
        : null
    }
  });

  await tx.account.update({
    where: { id: data.accountId },
    data: { balance: newBalance }
  });

  return newTransaction;
});
```

##### 2. 更新交易（Update Transaction）

**特殊逻辑 | Special Logic**:
- 计算净余额变化（新交易 - 旧交易）
- 支持切换账户（会影响两个账户的余额）
- 使用 Prisma Transaction 确保一致性

**代码 | Code**: `actions/transaction.js:124-195`

```javascript
// 计算净变化
const oldBalanceChange = originalTransaction.type === "EXPENSE"
  ? -originalTransaction.amount.toNumber()
  : originalTransaction.amount.toNumber();

const newBalanceChange = data.type === "EXPENSE" ? -data.amount : data.amount;
const netBalanceChange = newBalanceChange - oldBalanceChange;

// 更新余额（使用 increment）
await tx.account.update({
  where: { id: data.accountId },
  data: { balance: { increment: netBalanceChange } }
});
```

##### 3. 循环交易（Recurring Transactions）

**日期计算 | Date Calculation**: `actions/transaction.js:294-313`

```javascript
function calculateNextRecurringDate(startDate, interval) {
  const date = new Date(startDate);

  switch (interval) {
    case "DAILY":   date.setDate(date.getDate() + 1); break;
    case "WEEKLY":  date.setDate(date.getDate() + 7); break;
    case "MONTHLY": date.setMonth(date.getMonth() + 1); break;
    case "YEARLY":  date.setFullYear(date.getFullYear() + 1); break;
  }

  return date;
}
```

**后台处理 | Background Processing**:
- 使用 Inngest 定时检查 `nextRecurringDate`
- 自动创建新交易并更新 `nextRecurringDate`
- 更新 `lastProcessed` 时间戳

##### 4. AI 收据扫描（Receipt Scanning）

**文件位置 | Location**: `actions/transaction.js:231-291`

**流程 | Flow**:
```
1. 上传图片文件
2. 转换为 Base64
3. 调用 Google Gemini AI (gemini-1.5-flash)
4. 提取信息：
   ├─ 金额 (amount)
   ├─ 日期 (date)
   ├─ 描述 (description)
   ├─ 商家名称 (merchantName)
   └─ 建议分类 (category)
5. 返回 JSON 数据
6. 前端自动填充表单
```

**提示词模板 | Prompt**:
```javascript
const prompt = `
  Analyze this receipt image and extract the following information in JSON format:
  - Total amount (just the number)
  - Date (in ISO format)
  - Description or items purchased (brief summary)
  - Merchant/store name
  - Suggested category (one of: housing,transportation,groceries,utilities,
    entertainment,food,shopping,healthcare,education,personal,travel,
    insurance,gifts,bills,other-expense)

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

#### 数据验证 | Validation

**Zod Schema** (`app/lib/schema.js:10-31`):

```javascript
export const transactionSchema = z.object({
  type: z.enum(["INCOME", "EXPENSE"]),
  amount: z.string().min(1, "Amount is required"),
  description: z.string().optional(),
  date: z.date({ required_error: "Date is required" }),
  accountId: z.string().min(1, "Account is required"),
  category: z.string().min(1, "Category is required"),
  isRecurring: z.boolean().default(false),
  recurringInterval: z.enum(["DAILY", "WEEKLY", "MONTHLY", "YEARLY"]).optional()
})
.superRefine((data, ctx) => {
  // 如果是循环交易，必须指定间隔
  if (data.isRecurring && !data.recurringInterval) {
    ctx.addIssue({
      code: z.ZodIssueCode.custom,
      message: "Recurring interval is required for recurring transactions",
      path: ["recurringInterval"]
    });
  }
});
```

---

### 4. Budget 模型（预算）

**表名 | Table**: `budgets`
**文件位置 | Location**: `prisma/schema.prisma:68-79`

#### 字段清单 | Fields

| 字段名 | 类型 | 约束 | 默认值 | 说明 |
|--------|------|------|--------|------|
| `id` | String | PRIMARY KEY | `uuid()` | 预算唯一标识符 |
| `amount` | Decimal | NOT NULL | - | 预算限额 |
| `lastAlertSent` | DateTime? | NULLABLE | - | 上次发送警报时间 |
| `userId` | String | UNIQUE, FOREIGN KEY | - | 所属用户 ID |
| `createdAt` | DateTime | NOT NULL | `now()` | 创建时间 |
| `updatedAt` | DateTime | NOT NULL, AUTO | - | 更新时间 |

#### 关系 | Relations

```
User (1) ←────── (1) Budget
```

- **一对一关系** (One-to-One)：
  - `user: User` - 每个用户只能有一个预算
  - `userId` 字段设置为 `@unique`
  - 级联删除：删除用户时，预算也会被删除

#### 索引 | Indexes

- `userId` - UNIQUE INDEX（确保一个用户只有一个预算）

#### 业务规则 | Business Rules

**文件位置 | Location**: `actions/budget.js`

##### 1. 预算检查逻辑

**流程 | Flow**:
```
1. 获取用户的预算限额
2. 计算当前月的支出总额（EXPENSE 类型）
3. 如果支出超过预算：
   ├─ 检查是否已在 24 小时内发送过警报
   ├─ 如果未发送，触发邮件通知
   └─ 更新 lastAlertSent 时间戳
```

##### 2. 防止警报骚扰

**逻辑 | Logic**:
```javascript
// 检查上次警报时间
const lastAlert = budget.lastAlertSent;
const now = new Date();
const hoursSinceLastAlert = lastAlert
  ? (now - lastAlert) / (1000 * 60 * 60)
  : Infinity;

// 只有超过 24 小时才发送新警报
if (hoursSinceLastAlert > 24) {
  await sendBudgetAlertEmail(user.email, {
    budgetAmount: budget.amount,
    currentSpending: totalExpenses,
    overspending: totalExpenses - budget.amount
  });

  await db.budget.update({
    where: { id: budget.id },
    data: { lastAlertSent: now }
  });
}
```

---

## 关系图谱 | Relationship Diagrams

### ER 图（实体关系图）| Entity-Relationship Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                          DATABASE SCHEMA                         │
└─────────────────────────────────────────────────────────────────┘

┌──────────────────────┐
│       User           │
│──────────────────────│
│ 🔑 id (PK)           │
│ 🔒 clerkUserId (UQ)  │
│ 🔒 email (UQ)        │
│    name              │
│    imageUrl          │
│    createdAt         │
│    updatedAt         │
└──────────┬───────────┘
           │
           │ 1:N
           ├──────────────────┐
           │                  │
           │                  │
           ▼                  ▼
┌────────────────────┐   ┌────────────────────┐
│     Account        │   │    Transaction     │
│────────────────────│   │────────────────────│
│ 🔑 id (PK)         │   │ 🔑 id (PK)         │
│    name            │   │    type (ENUM)     │
│    type (ENUM)     │   │    amount          │
│    balance         │   │    description     │
│    isDefault       │   │    date            │
│ 🔗 userId (FK) ────┼───┤ 🔗 userId (FK)     │
│    createdAt       │   │    category        │
│    updatedAt       │   │    receiptUrl      │
└──────────┬─────────┘   │    isRecurring     │
           │             │    recurringInterval│
           │ 1:N         │    nextRecurringDate│
           │             │    lastProcessed    │
           └─────────────┤    status (ENUM)    │
                         │ 🔗 accountId (FK)   │
                         │    createdAt        │
                         │    updatedAt        │
                         └────────────────────┘
           │
           │ 1:1
           ▼
┌────────────────────┐
│      Budget        │
│────────────────────│
│ 🔑 id (PK)         │
│    amount          │
│    lastAlertSent   │
│ 🔗 userId (FK, UQ) │
│    createdAt       │
│    updatedAt       │
└────────────────────┘

图例 (Legend):
🔑 = Primary Key (主键)
🔒 = Unique Constraint (唯一约束)
🔗 = Foreign Key (外键)
(UQ) = Unique (唯一)
(PK) = Primary Key (主键)
(FK) = Foreign Key (外键)
(ENUM) = Enumeration Type (枚举类型)
```

### 数据流向图 | Data Flow Diagram

```
┌────────────────────────────────────────────────────────────────┐
│                   TRANSACTION FLOW                             │
└────────────────────────────────────────────────────────────────┘

用户创建交易 (User Creates Transaction)
│
├─ 1. 前端表单提交 (Frontend Form)
│   ├─ 可选：AI 收据扫描 (Receipt Scan)
│   │   └─ Google Gemini AI 提取数据
│   └─ 验证：Zod Schema
│
├─ 2. Server Action: createTransaction()
│   ├─ 认证检查 (Clerk Auth)
│   ├─ 速率限制 (ArcJet)
│   ├─ 查找用户 (Find User in DB)
│   └─ 验证账户归属
│
├─ 3. Prisma Transaction (原子操作)
│   ├─ [A] 创建 Transaction 记录
│   │     └─ 如果 isRecurring，计算 nextRecurringDate
│   └─ [B] 更新 Account.balance
│         └─ INCOME: balance += amount
│         └─ EXPENSE: balance -= amount
│
├─ 4. 检查预算 (Budget Check)
│   ├─ 计算本月支出总额
│   ├─ 如果超过预算限额
│   │   └─ 发送邮件警报 (Resend)
│   └─ 更新 lastAlertSent
│
└─ 5. 缓存重新验证 (Cache Revalidation)
    ├─ revalidatePath('/dashboard')
    └─ revalidatePath('/account/:id')
```

---

## 索引策略 | Indexing Strategy

### 现有索引 | Existing Indexes

**文件位置 | Location**: `prisma/migrations/20241204141034_init/migration.sql:98-122`

| 表名 | 字段 | 类型 | 目的 | 代码行 |
|------|------|------|------|--------|
| users | `clerkUserId` | UNIQUE | 快速查找 + 防重复 | :98 |
| users | `email` | UNIQUE | 快速查找 + 防重复 | :101 |
| accounts | `userId` | INDEX | 查询用户的所有账户 | :104 |
| transactions | `userId` | INDEX | 查询用户的所有交易 | :110 |
| transactions | `accountId` | INDEX | 查询账户的所有交易 | :115 |
| budgets | `userId` | UNIQUE | 一对一关系 + 快速查找 | :119 |

### 索引使用场景 | Index Usage Scenarios

#### 1. 用户查询（高频）
```sql
-- 使用索引: users.clerkUserId
SELECT * FROM users WHERE "clerkUserId" = 'user_xyz';
```
**调用位置 | Called from**: `lib/checkUser.js:12`

#### 2. 账户列表查询（高频）
```sql
-- 使用索引: accounts.userId
SELECT * FROM accounts WHERE "userId" = 'abc-123' ORDER BY "createdAt" DESC;
```
**调用位置 | Called from**: `actions/dashboard.js:32`

#### 3. 交易历史查询（极高频）
```sql
-- 使用复合索引: transactions.userId + transactions.accountId
SELECT * FROM transactions
WHERE "userId" = 'abc-123'
  AND "accountId" = 'acc-456'
ORDER BY "date" DESC;
```
**调用位置 | Called from**: `actions/transaction.js:211`

### 性能优化建议 | Performance Optimization Suggestions

#### 建议添加的索引 | Recommended Indexes

1. **复合索引**（Transactions）：
```prisma
@@index([userId, date(sort: Desc)])  // 用户交易按日期排序
@@index([accountId, date(sort: Desc)])  // 账户交易按日期排序
```

2. **条件索引**（Recurring Transactions）：
```prisma
@@index([isRecurring, nextRecurringDate], where: { isRecurring: true })
// 仅索引循环交易，提高后台作业效率
```

---

## 数据流模式 | Data Flow Patterns

### 模式 1：用户注册流程

```
Clerk 注册成功
    │
    ├─ Webhook/Middleware 触发
    │
    ├─ checkUser() 函数调用
    │  └─ lib/checkUser.js:4-37
    │
    ├─ 查询数据库：findUnique(clerkUserId)
    │
    ├─ 如果不存在
    │  └─ 创建 User 记录
    │     └─ 同步 Clerk 数据（name, email, imageUrl）
    │
    └─ 返回 User 对象
```

### 模式 2：创建账户流程

```
用户提交表单
    │
    ├─ Zod 验证 (accountSchema)
    │  └─ app/lib/schema.js:3-8
    │
    ├─ Server Action: createAccount()
    │  └─ actions/account.js:54-135
    │
    ├─ ArcJet 速率限制检查
    │  └─ 防止 API 滥用
    │
    ├─ 查询现有账户数量
    │  ├─ 如果是第一个账户
    │  │  └─ 强制设为 isDefault = true
    │  └─ 如果不是第一个
    │     └─ 使用用户选择
    │
    ├─ 如果设为默认账户
    │  └─ 取消其他账户的 isDefault
    │     └─ updateMany({ isDefault: false })
    │
    ├─ 创建新账户
    │  └─ balance = 初始余额
    │
    └─ 重新验证缓存
       └─ revalidatePath('/dashboard')
```

### 模式 3：交易创建流程（原子操作）

```
用户创建交易
    │
    ├─ [可选] AI 收据扫描
    │  └─ actions/transaction.js:231-291
    │     ├─ 上传图片 → Base64
    │     ├─ Gemini AI 分析
    │     └─ 返回提取的数据
    │
    ├─ Zod 验证 (transactionSchema)
    │  └─ app/lib/schema.js:10-31
    │     └─ 检查循环交易配置
    │
    ├─ Server Action: createTransaction()
    │  └─ actions/transaction.js:18-100
    │
    ├─ ArcJet 速率限制
    │
    ├─ 验证账户归属权
    │  └─ account.userId === user.id
    │
    ├─ 计算余额变化
    │  ├─ EXPENSE: balance -= amount
    │  └─ INCOME:  balance += amount
    │
    ├─ **Prisma Transaction** (原子操作)
    │  ├─ 操作 A: 创建 Transaction
    │  │  └─ 计算 nextRecurringDate (如果循环)
    │  ├─ 操作 B: 更新 Account.balance
    │  └─ 全部成功或全部回滚
    │
    ├─ 检查预算超支
    │  └─ 发送邮件警报（如需要）
    │
    └─ 缓存重新验证
       ├─ /dashboard
       └─ /account/:id
```

### 模式 4：循环交易处理（后台作业）

```
Inngest 定时任务 (Cron)
    │
    ├─ 查询需要处理的交易
    │  └─ WHERE isRecurring = true
    │     AND nextRecurringDate <= NOW()
    │
    ├─ 对每笔交易：
    │  ├─ 创建新的 Transaction
    │  │  └─ 复制原交易数据
    │  │     └─ date = nextRecurringDate
    │  │
    │  ├─ 更新账户余额
    │  │
    │  ├─ 更新原交易
    │  │  ├─ lastProcessed = NOW()
    │  │  └─ nextRecurringDate = 计算下次日期
    │  │
    │  └─ 检查预算（如果是支出）
    │
    └─ 记录处理日志
```

### 模式 5：预算警报流程

```
交易创建/更新后
    │
    ├─ 如果是 EXPENSE 类型
    │
    ├─ 查询用户预算
    │  └─ Budget.findUnique({ where: { userId } })
    │
    ├─ 如果预算存在
    │  │
    │  ├─ 计算本月支出总额
    │  │  └─ SUM(amount) WHERE type = EXPENSE
    │  │     AND date >= 本月1号
    │  │
    │  ├─ 如果 总支出 > 预算限额
    │  │  │
    │  │  ├─ 检查上次警报时间
    │  │  │  └─ 距离上次 > 24 小时？
    │  │  │
    │  │  ├─ 如果是，发送邮件
    │  │  │  └─ Resend API
    │  │  │     ├─ 主题：预算超支警报
    │  │  │     ├─ 内容：预算额、当前支出、超支金额
    │  │  │     └─ 使用 React Email 模板
    │  │  │
    │  │  └─ 更新 lastAlertSent = NOW()
    │  │
    │  └─ 记录检查日志
    │
    └─ 结束
```

---

## 迁移历史 | Migration History

### 迁移时间线 | Migration Timeline

**位置 | Location**: `prisma/migrations/`

| 日期 | 迁移名称 | 变更内容 | 原因 |
|------|----------|----------|------|
| 2024-12-04 | `init` | 创建所有表 | 初始数据库结构 |
| 2024-12-05 | `remove_currency` | 移除 currency 字段 | 简化为单货币系统 |
| 2024-12-05 | `remove_categories` | 移除 categories 表 | 改用字符串分类 |
| 2024-12-06 | `budget` | 添加 Budget 模型 | 新增预算功能 |
| 2024-12-08 | `budget` (2次) | 调整 Budget 结构 | 简化预算模型 |
| 2024-12-09 | `remove` | 移除某些字段 | 功能精简 |

### 重要变更分析 | Important Changes Analysis

#### 变更 1：移除 Currency（货币）

**之前 | Before**:
```prisma
model Account {
  currency String NOT NULL  // 支持多货币
}

model Transaction {
  currency String NOT NULL
}

model Budget {
  currency String NOT NULL
}
```

**之后 | After**:
```prisma
// currency 字段已移除
// 默认假设为单一货币（如 USD 或 CNY）
```

**影响 | Impact**:
- ✅ 简化了数据模型
- ✅ 减少了存储空间
- ❌ 无法支持多货币账户（国际化需求）

#### 变更 2：移除 Categories 表

**之前 | Before**:
```prisma
model Category {
  id     String @id @default(uuid())
  name   String
  type   TransactionType
  color  String?
  icon   String?
  userId String
  user   User @relation(...)
  transactions Transaction[]
}

model Transaction {
  categoryId String
  category   Category @relation(...)
}
```

**之后 | After**:
```prisma
model Transaction {
  category String  // 直接存储分类名称（字符串）
}
```

**影响 | Impact**:
- ✅ 更灵活（无需预定义分类）
- ✅ 减少了一次 JOIN 查询
- ❌ 可能出现拼写不一致
- ❌ 无法为分类设置颜色/图标

**预定义分类 | Predefined Categories** (AI 扫描使用):
```
housing, transportation, groceries, utilities, entertainment,
food, shopping, healthcare, education, personal, travel,
insurance, gifts, bills, other-expense
```

#### 变更 3：简化 Budget 模型

**之前 | Before**:
```prisma
model Budget {
  period     BudgetPeriod  // DAILY, WEEKLY, MONTHLY, YEARLY
  startDate  DateTime
  endDate    DateTime?
  categoryId String
  category   Category @relation(...)
}
```

**之后 | After**:
```prisma
model Budget {
  amount        Decimal
  lastAlertSent DateTime?
  userId        String @unique  // 一对一关系
  // 移除了 period, dates, category
}
```

**影响 | Impact**:
- ✅ 简化为"每月总预算"模式
- ❌ 无法按分类设置预算
- ❌ 无法设置不同时间周期的预算

---

## 数据验证规则 | Validation Rules

### Zod Schema 定义

**文件位置 | Location**: `app/lib/schema.js`

### 1. Account Validation

```javascript
export const accountSchema = z.object({
  name: z.string().min(1, "Name is required"),
  type: z.enum(["CURRENT", "SAVINGS"]),
  balance: z.string().min(1, "Initial balance is required"),
  isDefault: z.boolean().default(false)
});
```

**验证规则 | Rules**:
- ✅ `name`: 必填，最少 1 个字符
- ✅ `type`: 只能是 CURRENT 或 SAVINGS
- ✅ `balance`: 必填（字符串格式，后端转 Decimal）
- ✅ `isDefault`: 可选，默认 false

### 2. Transaction Validation

```javascript
export const transactionSchema = z
  .object({
    type: z.enum(["INCOME", "EXPENSE"]),
    amount: z.string().min(1, "Amount is required"),
    description: z.string().optional(),
    date: z.date({ required_error: "Date is required" }),
    accountId: z.string().min(1, "Account is required"),
    category: z.string().min(1, "Category is required"),
    isRecurring: z.boolean().default(false),
    recurringInterval: z
      .enum(["DAILY", "WEEKLY", "MONTHLY", "YEARLY"])
      .optional()
  })
  .superRefine((data, ctx) => {
    if (data.isRecurring && !data.recurringInterval) {
      ctx.addIssue({
        code: z.ZodIssueCode.custom,
        message: "Recurring interval is required for recurring transactions",
        path: ["recurringInterval"]
      });
    }
  });
```

**验证规则 | Rules**:
- ✅ `type`: 必须是 INCOME 或 EXPENSE
- ✅ `amount`: 必填（字符串格式）
- ✅ `date`: 必填（Date 对象）
- ✅ `category`: 必填字符串
- ✅ **条件验证**: 如果 `isRecurring = true`，必须提供 `recurringInterval`

---

## 示例查询 | Example Queries

### 1. 获取用户的所有账户（含交易数）

```javascript
// actions/dashboard.js:32
const accounts = await db.account.findMany({
  where: { userId: user.id },
  orderBy: { createdAt: "desc" },
  include: {
    _count: {
      select: {
        transactions: true  // 计数，不加载所有交易
      }
    }
  }
});
```

**生成的 SQL**:
```sql
SELECT a.*, COUNT(t.id) as transaction_count
FROM accounts a
LEFT JOIN transactions t ON t."accountId" = a.id
WHERE a."userId" = $1
GROUP BY a.id
ORDER BY a."createdAt" DESC;
```

### 2. 获取账户的交易历史（含账户信息）

```javascript
// actions/transaction.js:211
const transactions = await db.transaction.findMany({
  where: { userId: user.id },
  include: { account: true },
  orderBy: { date: "desc" }
});
```

**生成的 SQL**:
```sql
SELECT t.*,
       a.id, a.name, a.type, a.balance, a."isDefault"
FROM transactions t
INNER JOIN accounts a ON t."accountId" = a.id
WHERE t."userId" = $1
ORDER BY t.date DESC;
```

### 3. 计算本月支出（用于预算检查）

```javascript
// 伪代码（实际在 actions/budget.js 中）
const startOfMonth = new Date(new Date().getFullYear(), new Date().getMonth(), 1);

const totalExpenses = await db.transaction.aggregate({
  where: {
    userId: user.id,
    type: "EXPENSE",
    date: { gte: startOfMonth }
  },
  _sum: { amount: true }
});
```

**生成的 SQL**:
```sql
SELECT SUM(amount) as total
FROM transactions
WHERE "userId" = $1
  AND type = 'EXPENSE'
  AND date >= $2;
```

### 4. 查找需要处理的循环交易

```javascript
// Inngest 作业中的查询
const recurringTransactions = await db.transaction.findMany({
  where: {
    isRecurring: true,
    nextRecurringDate: {
      lte: new Date()  // 小于等于当前时间
    }
  },
  include: { account: true, user: true }
});
```

**生成的 SQL**:
```sql
SELECT t.*, a.*, u.*
FROM transactions t
INNER JOIN accounts a ON t."accountId" = a.id
INNER JOIN users u ON t."userId" = u.id
WHERE t."isRecurring" = true
  AND t."nextRecurringDate" <= NOW();
```

### 5. 更新账户余额（原子操作）

```javascript
// actions/transaction.js:85-88
await tx.account.update({
  where: { id: data.accountId },
  data: { balance: newBalance }
});
```

**生成的 SQL**:
```sql
UPDATE accounts
SET balance = $1,
    "updatedAt" = NOW()
WHERE id = $2;
```

---

## 数据完整性保证 | Data Integrity Guarantees

### 1. 外键约束（CASCADE DELETE）

**文件位置 | Location**: `prisma/migrations/20241204141034_init/migration.sql:124-143`

```sql
-- 删除用户时，级联删除所有关联数据
ALTER TABLE accounts ADD CONSTRAINT accounts_userId_fkey
  FOREIGN KEY ("userId") REFERENCES users("id") ON DELETE CASCADE;

ALTER TABLE transactions ADD CONSTRAINT transactions_userId_fkey
  FOREIGN KEY ("userId") REFERENCES users("id") ON DELETE CASCADE;

ALTER TABLE transactions ADD CONSTRAINT transactions_accountId_fkey
  FOREIGN KEY ("accountId") REFERENCES accounts("id") ON DELETE CASCADE;

ALTER TABLE budgets ADD CONSTRAINT budgets_userId_fkey
  FOREIGN KEY ("userId") REFERENCES users("id") ON DELETE CASCADE;
```

**行为 | Behavior**:
```
删除 User
  ├─→ 自动删除所有 Accounts
  │   └─→ 自动删除所有关联的 Transactions
  ├─→ 自动删除所有 Transactions（直接关联）
  └─→ 自动删除 Budget
```

### 2. 唯一性约束（UNIQUE Constraints）

- ✅ `User.clerkUserId` - 防止重复创建用户
- ✅ `User.email` - 防止邮箱重复
- ✅ `Budget.userId` - 确保一个用户只有一个预算

### 3. Prisma Transaction（数据库事务）

**用途 | Usage**:
- 创建交易 + 更新余额（原子操作）
- 更新交易 + 调整余额（原子操作）
- 循环交易处理（原子操作）

**示例 | Example** (`actions/transaction.js:73-91`):
```javascript
const transaction = await db.$transaction(async (tx) => {
  // 操作 1: 创建交易
  const newTransaction = await tx.transaction.create({ ... });

  // 操作 2: 更新余额
  await tx.account.update({ ... });

  // 如果任何操作失败，全部回滚
  return newTransaction;
});
```

---

## 性能考虑 | Performance Considerations

### 1. Decimal 类型的精度

**为什么使用 Decimal 而非 Float？**

```prisma
balance Decimal @default(0)  // ✅ 精确
amount  Decimal              // ✅ 精确

// 而非
balance Float   // ❌ 可能有浮点误差（0.1 + 0.2 ≠ 0.3）
```

**JavaScript 处理 | Handling**:
```javascript
// Prisma 返回的是 Decimal 对象，需要转换
const serialized = {
  ...obj,
  balance: obj.balance.toNumber()  // 转为 JS Number
};
```

### 2. 索引使用

- ✅ 所有外键都有索引
- ✅ 查询字段（clerkUserId, email）有唯一索引
- ⚠️ 可考虑添加复合索引（userId + date）

### 3. 连接池管理

**文件位置 | Location**: `lib/prisma.js:1-12`

```javascript
// 开发环境：复用全局实例，避免热重载时创建多个连接
export const db = globalThis.prisma || new PrismaClient();

if (process.env.NODE_ENV !== "production") {
  globalThis.prisma = db;
}
```

---

## 下一步分析 | Next Steps

本文档完成了数据库的深度分析。接下来的分析文档：

1. ✅ **00-project-overview.md** - 项目总览
2. ✅ **01-database-analysis.md** (当前文档)
3. ⏭️ **02-backend-analysis.md** - Server Actions 和 API 深度分析
4. ⏭️ **03-frontend-analysis.md** - 前端组件和路由分析
5. ⏭️ **04-security-analysis.md** - 安全机制详解

---

**文档生成于 | Generated on**: 2025-11-15
**分析工具 | Analysis Tool**: Claude Code with Project Mastery System
**文档版本 | Document Version**: 1.0
