# AI Finance Platform (Welth) - Technical Specification

**Version**: 1.0
**Generated**: 2025-11-16
**Target Audience**: Developers, Technical Managers, Contributors

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Technical Architecture](#2-technical-architecture)
3. [Database Design](#3-database-design)
4. [Core Business Flows](#4-core-business-flows)
5. [Frontend-Backend Interaction](#5-frontend-backend-interaction)
6. [API Documentation](#6-api-documentation)
7. [Environment Configuration](#7-environment-configuration)
8. [Development Guide](#8-development-guide)
9. [Deployment Guide](#9-deployment-guide)
10. [Appendix](#10-appendix)

---

## 1. Project Overview

### 1.1 Introduction

**Welth** is a modern personal finance management platform that helps users track income/expenses, manage budgets, and analyze financial trends.

**Core Principles**:
- 📊 **Data Visualization** - Intuitive charts for financial insights
- 🤖 **AI-Powered** - Smart receipt scanning and financial insights
- 📱 **Mobile-First** - Responsive design for all devices
- 🔒 **Security-First** - Multi-layer protection for user data

### 1.2 Key Features

| Feature Module | Description | Technical Highlights |
|----------------|-------------|---------------------|
| **Account Management** | Multi-account support (Current/Savings) | Prisma ORM + PostgreSQL |
| **Transactions** | Income/expense tracking with recurring support | Server Actions + Real-time updates |
| **Receipt Scanning** | AI-driven OCR data extraction | Google Gemini AI |
| **Budget Management** | Monthly budget setting and tracking | Real-time budget monitoring |
| **Data Visualization** | Charts and trend analysis | Recharts library |
| **Email Notifications** | Budget alerts and monthly reports | Resend + Inngest |
| **Background Jobs** | Automated task processing | Inngest scheduled jobs |

### 1.3 Technical Highlights

✅ **Next.js 15** - Latest App Router architecture
✅ **React 19 RC** - Server Components first
✅ **Prisma ORM** - Type-safe database access
✅ **Clerk** - Enterprise-grade authentication
✅ **ArcJet** - Smart security and rate limiting
✅ **Tailwind CSS** - Utility-first styling
✅ **TypeScript** - Full-stack type safety (via JSDoc)

### 1.4 Use Cases

- 👤 **Personal Finance** - Track daily expenses and income
- 💼 **Freelancers** - Manage multiple income sources
- 🏠 **Family Budgeting** - Collaborative household finance management
- 📈 **Financial Analysis** - Understand spending patterns and trends

---

## 2. Technical Architecture

### 2.1 System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      User Browser                            │
│                 (Chrome, Safari, Firefox)                    │
└────────────────┬────────────────────────────────────────────┘
                 │ HTTPS
                 ▼
┌─────────────────────────────────────────────────────────────┐
│                   Vercel Edge Network                        │
│                 (CDN + Edge Functions)                       │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│                  Next.js 15 Application                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │  Frontend    │  │   API Layer  │  │  Background  │     │
│  │  React 19    │  │   Server     │  │   Inngest    │     │
│  │  Components  │  │   Actions    │  │   Functions  │     │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘     │
│         │                  │                  │              │
└─────────┼──────────────────┼──────────────────┼──────────────┘
          │                  │                  │
          │                  ▼                  │
          │         ┌────────────────┐         │
          │         │   Middleware   │         │
          │         │  - ArcJet      │         │
          │         │  - Clerk Auth  │         │
          │         └────────┬───────┘         │
          │                  │                  │
          ▼                  ▼                  ▼
┌─────────────────┐ ┌────────────────┐ ┌──────────────┐
│   Clerk Auth    │ │   PostgreSQL   │ │   Inngest    │
│   (Sessions)    │ │   (Supabase)   │ │   (Cron)     │
└─────────────────┘ └────────────────┘ └──────────────┘
          │                  │                  │
          │                  ▼                  ▼
          │         ┌────────────────┐ ┌──────────────┐
          │         │   Prisma ORM   │ │  Email Jobs  │
          │         └────────────────┘ └──────────────┘
          │
          ▼
┌─────────────────────────────────────────────────────────────┐
│                   External Services                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │  Clerk   │  │  Resend  │  │  Gemini  │  │  ArcJet  │   │
│  │  (Auth)  │  │  (Email) │  │   (AI)   │  │(Security)│   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 Tech Stack

#### Frontend Stack

| Technology | Version | Purpose |
|-----------|---------|---------|
| **Next.js** | 15.0.3 | React framework, App Router |
| **React** | 19 RC | UI library, Server + Client Components |
| **Tailwind CSS** | 3.4.1 | Utility-first CSS framework |
| **Radix UI** | Multiple | Accessible UI component library |
| **Recharts** | 2.14.1 | Data visualization charts |
| **React Hook Form** | 7.53.2 | Form state management |
| **Zod** | 3.23.8 | Schema validation |
| **date-fns** | 4.1.0 | Date manipulation |

#### Backend Stack

| Technology | Version | Purpose |
|-----------|---------|---------|
| **Prisma** | 6.0.1 | Type-safe ORM |
| **PostgreSQL** | 15+ | Relational database |
| **Clerk** | 6.6.0 | Authentication & user management |
| **ArcJet** | 1.0.0-alpha.34 | Security & rate limiting |
| **Inngest** | 3.27.4 | Background jobs & scheduling |

#### AI & Communication Services

| Service | Purpose |
|---------|---------|
| **Google Gemini** | Receipt OCR, financial insights |
| **Resend** | Transactional email |

### 2.3 Frontend-Backend Interaction Pattern

#### Data Flow

```
┌──────────────────────────────────────────────────────────────┐
│                    User Action                                │
│         (Click button, Submit form, Upload file)              │
└────────────────────┬─────────────────────────────────────────┘
                     │
                     ▼
            ┌─────────────────┐
            │ Client Component│
            │  - useState     │
            │  - useForm      │
            │  - useFetch     │
            └────────┬────────┘
                     │ Call
                     ▼
            ┌─────────────────┐
            │  Server Action  │
            │  "use server"   │
            └────────┬────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
        ▼                         ▼
┌───────────────┐         ┌───────────────┐
│  Auth Check   │         │ Rate Limit    │
│  await auth() │         │  ArcJet       │
└───────┬───────┘         └───────┬───────┘
        │                         │
        └────────────┬────────────┘
                     ▼
            ┌─────────────────┐
            │ Business Logic  │
            │  - Validate     │
            │  - Transform    │
            └────────┬────────┘
                     │
                     ▼
            ┌─────────────────┐
            │  Prisma Query   │
            │  - findUnique   │
            │  - create       │
            │  - update       │
            └────────┬────────┘
                     │
                     ▼
            ┌─────────────────┐
            │   PostgreSQL    │
            │    Database     │
            └────────┬────────┘
                     │
                     ▼
            ┌─────────────────┐
            │ revalidatePath  │
            │  Clear cache    │
            └────────┬────────┘
                     │
                     ▼
            ┌─────────────────┐
            │  Return Result  │
            │  { success,     │
            │    data }       │
            └────────┬────────┘
                     │
                     ▼
            ┌─────────────────┐
            │  Client Update  │
            │  - Toast alert  │
            │  - Page refresh │
            └─────────────────┘
```

---

## 3. Database Design

### 3.1 ER Diagram

```
┌─────────────────┐
│      User       │
│─────────────────│
│ id (PK)         │
│ clerkUserId *   │──┐
│ email           │  │
│ name            │  │
│ imageUrl        │  │
│ createdAt       │  │
│ updatedAt       │  │
└─────────────────┘  │
                     │ 1:N
                     │
        ┌────────────┴───────────┬──────────────┐
        │                        │              │
        ▼                        ▼              ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│    Account      │    │   Transaction   │    │     Budget      │
│─────────────────│    │─────────────────│    │─────────────────│
│ id (PK)         │    │ id (PK)         │    │ id (PK)         │
│ name            │◄─┐ │ type            │    │ amount          │
│ type            │  │ │ amount          │    │ userId (FK)     │
│ balance         │  │ │ description     │    │ accountId (FK)  │
│ isDefault       │  │ │ date            │    │ month           │
│ userId (FK)     │  │ │ category        │    │ year            │
│ createdAt       │  │ │ receipt         │    │ lastAlertSent   │
└─────────────────┘  │ │ isRecurring     │    │ createdAt       │
         │           │ │ recurringInterval   │ updatedAt       │
         │ 1:N       │ │ nextRecurringDate   └─────────────────┘
         └───────────┼─│ accountId (FK)  │
                     │ │ userId (FK)     │
                     │ │ status          │
                     │ │ createdAt       │
                     │ │ updatedAt       │
                     │ └─────────────────┘
                     │
                     └─── 1:N
```

### 3.2 Table Details

#### User Table

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | String | PK, UUID | Primary key |
| clerkUserId | String | UNIQUE, NOT NULL | Clerk user ID |
| email | String | UNIQUE, NOT NULL | Email address |
| name | String | - | User name |
| imageUrl | String | - | Avatar URL |
| createdAt | DateTime | DEFAULT NOW | Creation timestamp |
| updatedAt | DateTime | AUTO UPDATE | Update timestamp |

**Relations**:
- `accounts` - One-to-Many → Account
- `transactions` - One-to-Many → Transaction
- `budgets` - One-to-Many → Budget

#### Account Table

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | String | PK, UUID | Primary key |
| name | String | NOT NULL | Account name |
| type | AccountType | NOT NULL | CURRENT / SAVINGS |
| balance | Decimal | DEFAULT 0 | Current balance |
| isDefault | Boolean | DEFAULT FALSE | Is default account |
| userId | String | FK, NOT NULL | Related user |
| createdAt | DateTime | DEFAULT NOW | Creation timestamp |
| updatedAt | DateTime | AUTO UPDATE | Update timestamp |

**Indexes**:
- `userId` - Speed up user account queries
- `userId + isDefault` - Find default account

#### Transaction Table

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | String | PK, UUID | Primary key |
| type | TransactionType | NOT NULL | INCOME / EXPENSE |
| amount | Decimal | NOT NULL | Transaction amount |
| description | String | - | Transaction description |
| date | DateTime | NOT NULL | Transaction date |
| category | String | NOT NULL | Category (from preset list) |
| receipt | String | - | Receipt URL |
| isRecurring | Boolean | DEFAULT FALSE | Is recurring |
| recurringInterval | RecurringInterval | - | Recurring interval |
| nextRecurringDate | DateTime | - | Next execution date |
| accountId | String | FK, NOT NULL | Related account |
| userId | String | FK, NOT NULL | Related user |
| status | TransactionStatus | DEFAULT COMPLETED | Transaction status |
| createdAt | DateTime | DEFAULT NOW | Creation timestamp |
| updatedAt | DateTime | AUTO UPDATE | Update timestamp |

**Indexes**:
- `userId` - User transaction queries
- `accountId` - Account transaction queries
- `date` - Date range queries
- `isRecurring + nextRecurringDate` - Recurring transaction processing

#### Budget Table

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | String | PK, UUID | Primary key |
| amount | Decimal | NOT NULL | Budget amount |
| userId | String | FK, NOT NULL | Related user |
| accountId | String | FK, NOT NULL | Related account |
| month | Int | NOT NULL | Month (1-12) |
| year | Int | NOT NULL | Year |
| lastAlertSent | DateTime | - | Last alert timestamp |
| createdAt | DateTime | DEFAULT NOW | Creation timestamp |
| updatedAt | DateTime | AUTO UPDATE | Update timestamp |

**Indexes**:
- `userId + accountId + year + month` - Unique budget

### 3.3 Data Flow Diagrams

#### Create Transaction Data Flow

```
1. User submits form
   ↓
2. createTransaction(data)
   ↓
3. Prisma transaction begins
   ├─ Create Transaction record
   │  └─ INSERT INTO "Transaction" (...)
   ├─ Calculate new balance
   │  └─ newBalance = oldBalance ± amount
   └─ Update Account balance
      └─ UPDATE "Account" SET balance = newBalance
   ↓
4. Transaction commit
   ↓
5. revalidatePath("/dashboard")
   ↓
6. Return new transaction data
```

---

## 4. Core Business Flows

### 4.1 User Registration/Login Flow

```
┌─────────────────────────────────────────────────────────────┐
│              User visits application homepage                 │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
            ┌─────────────────┐
            │ Middleware check │
            │  Authenticated?  │
            └────────┬────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
        ▼ NO                      ▼ YES
┌───────────────┐         ┌───────────────┐
│ Redirect to   │         │ Allow access  │
│ /sign-in      │         │ to protected  │
└───────┬───────┘         │ routes        │
        │                 └───────────────┘
        ▼
┌───────────────┐
│ Clerk sign-in │
│ - Email/Pass  │
│ - Google OAuth│
│ - GitHub OAuth│
└───────┬───────┘
        │
        ▼
┌───────────────┐
│ Clerk verify  │
│ Create session│
└───────┬───────┘
        │
        ▼
┌───────────────┐
│  checkUser()  │
│ Database check│
└───────┬───────┘
        │
    ┌───┴────┐
    ▼        ▼
 Exists   Not exists
    │        │
    │        ▼
    │   ┌────────────┐
    │   │ Create user│
    │   │User.create │
    │   └────┬───────┘
    │        │
    └────────┴─────┐
                   ▼
            ┌───────────┐
            │ Redirect  │
            │/dashboard │
            └───────────┘
```

**Code Location**:
- Middleware: `middleware.js:32-44`
- User check: `lib/checkUser.js:4-37`

### 4.2 Create Transaction Flow

```
┌─────────────────────────────────────────────────────────────┐
│          User visits /transaction/create                      │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
            ┌─────────────────┐
            │  Load form page │
            │  - Get accounts │
            │  - Init form    │
            └────────┬────────┘
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
    ┌──────────┐         ┌──────────┐
    │ Manual   │         │ Scan     │
    │ entry    │         │ receipt  │
    └────┬─────┘         └────┬─────┘
         │                    │
         │                    ▼
         │          ┌──────────────────┐
         │          │ scanReceipt()    │
         │          │ - Gemini AI OCR  │
         │          │ - Extract data   │
         │          └────┬─────────────┘
         │               │
         │               ▼
         │          ┌──────────────────┐
         │          │ Auto-fill form   │
         │          │ setValue(...)    │
         │          └────┬─────────────┘
         │               │
         └───────────────┘
                     │
                     ▼
            ┌─────────────────┐
            │ User submits    │
            │ handleSubmit()  │
            └────────┬────────┘
                     │
                     ▼
            ┌─────────────────┐
            │ Zod validation  │
            │transactionSchema│
            └────────┬────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
        ▼ FAIL                    ▼ SUCCESS
┌───────────────┐         ┌───────────────┐
│ Show error    │         │ Call Server   │
│ toast.error() │         │ Action        │
└───────────────┘         └───────┬───────┘
                                  │
                                  ▼
                         ┌───────────────┐
                         │createTransaction│
                         │ - Auth check  │
                         │ - Rate limit  │
                         └───────┬───────┘
                                 │
                                 ▼
                         ┌───────────────┐
                         │Prisma txn     │
                         │1. Create txn  │
                         │2. Update bal  │
                         └───────┬───────┘
                                 │
                                 ▼
                         ┌───────────────┐
                         │ revalidatePath│
                         │ /dashboard    │
                         │ /account/[id] │
                         └───────┬───────┘
                                 │
                                 ▼
                         ┌───────────────┐
                         │ Return success│
                         │Redirect account│
                         └───────────────┘
```

**Code Location**:
- Form: `app/(main)/transaction/_components/transaction-form.jsx:34-332`
- Server Action: `actions/transaction.js:18-100`
- Receipt scan: `actions/transaction.js:231-291`

### 4.3 Budget Alert Flow

```
┌─────────────────────────────────────────────────────────────┐
│           Inngest scheduled task (every 6 hours)              │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
            ┌─────────────────┐
            │checkBudgetAlerts│
            │   Job starts    │
            └────────┬────────┘
                     │
                     ▼
            ┌─────────────────┐
            │ Get all budgets │
            │ Budget.findMany │
            └────────┬────────┘
                     │
                     ▼
            ┌─────────────────┐
            │ Loop each budget│
            └────────┬────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
        ▼                         ▼
┌───────────────┐         ┌───────────────┐
│ Calculate     │         │ Check last    │
│ month spend   │         │ alert sent    │
│ sum(amount)   │         │lastAlertSent  │
└───────┬───────┘         └───────┬───────┘
        │                         │
        └────────────┬────────────┘
                     │
                     ▼
            ┌─────────────────┐
            │ Spend > 90%?    │
            └────────┬────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
        ▼ NO                      ▼ YES
┌───────────────┐         ┌───────────────┐
│ Skip budget   │         │Sent in 24hrs? │
└───────────────┘         └───────┬───────┘
                                  │
                     ┌────────────┴────────────┐
                     │                         │
                     ▼ YES                     ▼ NO
             ┌───────────────┐         ┌───────────────┐
             │ Skip (avoid   │         │ Send alert    │
             │  duplicate)   │         │ sendEmail()   │
             └───────────────┘         └───────┬───────┘
                                              │
                                              ▼
                                      ┌───────────────┐
                                      │ Update        │
                                      │lastAlertSent  │
                                      └───────────────┘
```

**Code Location**:
- Inngest function: `lib/inngest/function.js:171-230`
- Email sending: `actions/send-email.js:3-21`

---

## 5. Frontend-Backend Interaction

### 5.1 Server Actions vs API Routes

This project **primarily uses Server Actions**, not traditional API Routes.

| Comparison | Server Actions | API Routes |
|-----------|---------------|-----------|
| **Definition** | `"use server"` directive | `app/api/*/route.js` |
| **Invocation** | Direct function import | `fetch("/api/...")` |
| **Type Safety** | ✅ Fully type-safe | ❌ Manual type definitions |
| **Code Sharing** | ✅ Shared frontend/backend types | ❌ Separate codebases |
| **Use Cases** | Form submission, data mutation | Webhooks, third-party integration |

**Server Actions in Project**:

| File | Actions | Description |
|------|---------|-------------|
| `actions/dashboard.js` | getUserAccounts, createAccount, getDashboardData | Dashboard data |
| `actions/transaction.js` | createTransaction, updateTransaction, scanReceipt | Transaction management |
| `actions/account.js` | getAccountWithTransactions, bulkDeleteTransactions | Account operations |
| `actions/budget.js` | getCurrentBudget, updateBudget | Budget management |

**API Routes in Project**:

| Route | Purpose |
|-------|---------|
| `/api/inngest` | Inngest webhook endpoint |
| `/api/seed` | Development data seeding |

### 5.2 Data Fetching Patterns

#### Server Component Data Fetching (Recommended)

```javascript
// app/(main)/dashboard/page.jsx
export default async function DashboardPage() {
  // Parallel data fetching
  const [accounts, transactions] = await Promise.all([
    getUserAccounts(),
    getDashboardData(),
  ]);

  // Direct render, no loading state needed
  return (
    <div>
      <AccountList accounts={accounts} />
      <TransactionList transactions={transactions} />
    </div>
  );
}
```

**Advantages**:
- ✅ Server-side rendering, SEO friendly
- ✅ No client state management needed
- ✅ Automatic code splitting

#### Client Data Mutation (Using useFetch)

```javascript
// components/create-account-drawer.jsx
const { fn: createAccountFn, loading, error } = useFetch(createAccount);

const onSubmit = async (data) => {
  await createAccountFn(data);
};
```

**useFetch Hook** (`hooks/use-fetch.js`):
- Encapsulates loading, error, data state
- Auto error toast notifications
- Supports any async function

### 5.3 Form Validation Flow

```
┌─────────────────────────────────────────────────────────────┐
│                   User inputs form fields                     │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
            ┌─────────────────┐
            │ onChange event  │
            │React Hook Form  │
            └────────┬────────┘
                     │
                     ▼
            ┌─────────────────┐
            │  Zod Resolver   │
            │ Real-time valid │
            └────────┬────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
        ▼                         ▼
┌───────────────┐         ┌───────────────┐
│ Validation    │         │ Validation    │
│ failed        │         │ passed        │
│ errors.field  │         │ Clear errors  │
└───────┬───────┘         └───────────────┘
        │
        ▼
┌───────────────┐
│ Show error    │
│<ErrorMessage> │
└───────────────┘


                  User submits
                        │
                        ▼
            ┌─────────────────┐
            │ handleSubmit()  │
            │ Final validation│
            └────────┬────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
        ▼                         ▼
┌───────────────┐         ┌───────────────┐
│ Has errors    │         │ No errors     │
│ Block submit  │         │ Call Server   │
└───────────────┘         │ Action        │
                          └───────┬───────┘
                                  │
                     (Backend validation recommended)
                                  │
                                  ▼
                          ┌───────────────┐
                          │ Database op   │
                          └───────────────┘
```

**Zod Schema Example** (`app/lib/schema.js`):

```javascript
export const transactionSchema = z
  .object({
    type: z.enum(["INCOME", "EXPENSE"]),
    amount: z.string().min(1, "Amount required"),
    date: z.date({ required_error: "Date required" }),
    accountId: z.string().min(1, "Account required"),
    category: z.string().min(1, "Category required"),
    isRecurring: z.boolean().default(false),
    recurringInterval: z.enum(["DAILY", "WEEKLY", "MONTHLY", "YEARLY"]).optional(),
  })
  .superRefine((data, ctx) => {
    // Custom validation: recurring must have interval
    if (data.isRecurring && !data.recurringInterval) {
      ctx.addIssue({
        code: z.ZodIssueCode.custom,
        message: "Recurring transactions need interval",
        path: ["recurringInterval"],
      });
    }
  });
```

---

## 6. API Documentation

While this project primarily uses Server Actions, here are all callable function interfaces.

### 6.1 Account Management APIs

#### getUserAccounts

**Purpose**: Get all accounts for current user

**File**: `actions/dashboard.js:21-49`

**Parameters**: None (userId from auth session)

**Returns**:
```typescript
Array<{
  id: string;
  name: string;
  type: "CURRENT" | "SAVINGS";
  balance: number;
  isDefault: boolean;
  _count: {
    transactions: number;
  };
}>
```

**Example**:
```javascript
const accounts = await getUserAccounts();
// [
//   {
//     id: "abc123",
//     name: "Main Account",
//     type: "CURRENT",
//     balance: 5000.50,
//     isDefault: true,
//     _count: { transactions: 42 }
//   }
// ]
```

**Error Handling**:
- `Unauthorized` - Not logged in
- `User not found` - User doesn't exist

---

#### createAccount

**Purpose**: Create new account

**File**: `actions/dashboard.js:54-135`

**Parameters**:
```typescript
{
  name: string;           // Account name
  type: "CURRENT" | "SAVINGS";
  balance: string;        // Initial balance (string format)
  isDefault?: boolean;    // Set as default (optional)
}
```

**Returns**:
```typescript
{
  success: true;
  data: {
    id: string;
    name: string;
    type: string;
    balance: number;
    isDefault: boolean;
  };
}
```

**Example**:
```javascript
const result = await createAccount({
  name: "Savings Account",
  type: "SAVINGS",
  balance: "10000",
  isDefault: false,
});
```

**Rate Limit**: 10 requests/hour

**Error Handling**:
- `Unauthorized` - Not logged in
- `Too many requests` - Rate limit exceeded
- `Invalid balance amount` - Invalid balance format

---

### 6.2 Transaction Management APIs

#### createTransaction

**Purpose**: Create new transaction

**File**: `actions/transaction.js:18-100`

**Parameters**:
```typescript
{
  type: "INCOME" | "EXPENSE";
  amount: string;
  description?: string;
  date: Date;
  accountId: string;
  category: string;
  isRecurring?: boolean;
  recurringInterval?: "DAILY" | "WEEKLY" | "MONTHLY" | "YEARLY";
}
```

**Returns**:
```typescript
{
  id: string;
  type: string;
  amount: number;
  description: string | null;
  date: string;
  category: string;
  accountId: string;
  // ...
}
```

**Business Logic**:
1. Verify user authentication
2. Check rate limiting
3. Verify account ownership
4. **Atomic operation** (Prisma transaction):
   - Create transaction record
   - Update account balance
5. Clear cache
6. Return result

**Example**:
```javascript
const transaction = await createTransaction({
  type: "EXPENSE",
  amount: "50.00",
  description: "Lunch",
  date: new Date(),
  accountId: "account_123",
  category: "food",
});
```

**Error Handling**:
- `Unauthorized` - Not logged in
- `Too many requests` - Rate limited
- `Account not found` - Account doesn't exist
- `Invalid amount` - Invalid amount

---

#### scanReceipt

**Purpose**: AI scan receipt to extract data

**File**: `actions/transaction.js:231-291`

**Parameters**:
```typescript
File  // Image file object
```

**Returns**:
```typescript
{
  amount: number;
  date: Date;
  description: string;
  category: string;
}
```

**AI Prompt** (sent to Gemini):
```
Analyze this receipt and extract the following information:
- Total amount (number)
- Date (YYYY-MM-DD format)
- Description (merchant name or brief description)
- Category (select one from preset list)

Return only valid JSON object with keys: amount, date, description, category
```

**Example**:
```javascript
const scannedData = await scanReceipt(fileObject);
// {
//   amount: 42.50,
//   date: new Date("2024-11-16"),
//   description: "Starbucks Coffee",
//   category: "food"
// }
```

**Limitations**:
- File size: Max 5MB
- Supported formats: JPG, PNG, WebP
- Processing time: 2-5 seconds

---

### 6.3 Budget Management APIs

#### getCurrentBudget

**Purpose**: Get current month budget for account

**File**: `actions/budget.js:9-63`

**Parameters**:
```typescript
accountId: string
```

**Returns**:
```typescript
{
  id: string;
  amount: number;
  month: number;
  year: number;
  currentSpent: number;      // Current month spending
  remainingAmount: number;   // Remaining budget
  percentageUsed: number;    // Usage percentage
}
```

**Example**:
```javascript
const budget = await getCurrentBudget("account_123");
// {
//   id: "budget_456",
//   amount: 3000,
//   month: 11,
//   year: 2024,
//   currentSpent: 2100,
//   remainingAmount: 900,
//   percentageUsed: 70
// }
```

---

#### updateBudget

**Purpose**: Update or create budget

**File**: `actions/budget.js:68-112`

**Parameters**:
```typescript
{
  accountId: string;
  amount: number;
}
```

**Returns**: Updated budget object

**Business Logic**:
- If current month budget exists → Update
- If doesn't exist → Create new budget

**Example**:
```javascript
const budget = await updateBudget({
  accountId: "account_123",
  amount: 3500,
});
```

---

### 6.4 Dashboard Data APIs

#### getDashboardData

**Purpose**: Get dashboard overview data

**File**: `actions/dashboard.js:138-173`

**Returns**:
```typescript
{
  accounts: Array<Account>;
  recentTransactions: Array<Transaction>;  // Last 10
  totalIncome: number;     // Total income
  totalExpense: number;    // Total expense
  netIncome: number;       // Net income
}
```

**Data Aggregation**:
- Get all accounts
- Get last 10 transactions (desc by date)
- Calculate total income/expense

---

## 7. Environment Configuration

### 7.1 Environment Variables

Create `.env` file (root directory):

```bash
# ==========================================
# Database Configuration
# ==========================================
# Pooled connection (for app queries)
DATABASE_URL="postgresql://user:password@host:5432/database?pgbouncer=true"
# Direct connection (for Prisma migrations)
DIRECT_URL="postgresql://user:password@host:5432/database"

# ==========================================
# Authentication (Clerk)
# ==========================================
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_xxxxx
CLERK_SECRET_KEY=sk_test_xxxxx
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/onboarding
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/onboarding

# ==========================================
# AI Service (Google Gemini)
# ==========================================
GEMINI_API_KEY=xxxxx

# ==========================================
# Email Service (Resend)
# ==========================================
RESEND_API_KEY=re_xxxxx

# ==========================================
# Security Service (ArcJet)
# ==========================================
ARCJET_KEY=ajkey_xxxxx

# ==========================================
# Background Jobs (Inngest) - Optional for production
# ==========================================
INNGEST_EVENT_KEY=xxxxx
INNGEST_SIGNING_KEY=xxxxx
```

### 7.2 Getting API Keys

#### 1. Supabase Database

1. Visit https://supabase.com
2. Create new project
3. **Settings** → **Database**
4. Copy connection strings:
   - **Connection String** → `DATABASE_URL`
   - **Direct Connection** → `DIRECT_URL`
5. Enable **Connection Pooling** (PgBouncer)

**Connection String Format**:
```
# Pooled connection
postgresql://postgres:[PASSWORD]@db.[PROJECT-REF].supabase.co:5432/postgres?pgbouncer=true

# Direct connection
postgresql://postgres:[PASSWORD]@db.[PROJECT-REF].supabase.co:5432/postgres
```

---

#### 2. Clerk Authentication

1. Visit https://clerk.com
2. Create application (select Next.js)
3. **Dashboard** → **API Keys**
4. Copy:
   - **Publishable Key** → `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`
   - **Secret Key** → `CLERK_SECRET_KEY`
5. Configure redirect URLs:
   - Sign-in URL: `/sign-in`
   - Sign-up URL: `/sign-up`
   - After sign-in: `/onboarding`

---

#### 3. Google Gemini AI

1. Visit https://makersuite.google.com/app/apikey
2. Click **Create API Key**
3. Select Google Cloud project
4. Copy key → `GEMINI_API_KEY`
5. Enable **Gemini API** in **Google Cloud Console**

**Pricing**: Free tier available (60 requests/month)

---

#### 4. Resend Email

1. Visit https://resend.com
2. Sign up
3. **API Keys** → **Create API Key**
4. Copy key → `RESEND_API_KEY`
5. (Optional) Verify custom domain

**Free Tier**: 100 emails/day

---

#### 5. ArcJet Security

1. Visit https://arcjet.com
2. Create site
3. Copy **API Key** → `ARCJET_KEY`
4. Configure rules:
   - Bot Detection: Enabled
   - Shield: Enabled
   - Rate Limiting: 10 req/hour

---

#### 6. Inngest Background Jobs

1. Visit https://inngest.com
2. Create account
3. **Development**: No configuration needed
4. **Production**:
   - Add webhook: `https://yourdomain.com/api/inngest`
   - Copy Event Key and Signing Key

---

### 7.3 Local Development Setup

```bash
# 1. Clone project
git clone <repository-url>
cd ai-finance-platform

# 2. Install dependencies
npm install

# 3. Configure environment variables
cp .env.example .env
# Edit .env and add your keys

# 4. Generate Prisma client
npx prisma generate

# 5. Run database migrations
npx prisma migrate deploy

# 6. (Optional) Seed sample data
curl http://localhost:3000/api/seed

# 7. Start development server
npm run dev
```

Visit: http://localhost:3000

---

## 8. Development Guide

### 8.1 Project Structure

```
ai-finance-platform/
├── app/                        # Next.js App Router
│   ├── (auth)/                 # Auth route group
│   │   ├── sign-in/           # Sign-in page
│   │   └── sign-up/           # Sign-up page
│   ├── (main)/                 # Main app route group
│   │   ├── dashboard/         # Dashboard
│   │   ├── transaction/       # Transaction management
│   │   └── account/           # Account details
│   ├── api/                    # API routes
│   │   ├── inngest/           # Inngest webhook
│   │   └── seed/              # Data seeding
│   ├── lib/                    # Shared libraries
│   │   └── schema.js          # Zod validation schemas
│   ├── layout.js              # Root layout
│   ├── page.js                # Homepage
│   └── globals.css            # Global styles
├── actions/                    # Server Actions
│   ├── dashboard.js           # Dashboard operations
│   ├── transaction.js         # Transaction operations
│   ├── account.js             # Account operations
│   └── budget.js              # Budget operations
├── components/                 # React components
│   ├── ui/                    # Radix UI base components
│   ├── header.jsx             # Global header
│   └── create-account-drawer.jsx
├── hooks/                      # Custom Hooks
│   └── use-fetch.js           # Async operation wrapper
├── lib/                        # Utility libraries
│   ├── prisma.js              # Prisma client
│   ├── arcjet.js              # ArcJet configuration
│   ├── checkUser.js           # User sync
│   └── inngest/               # Inngest configuration
│       ├── client.js          # Inngest client
│       └── function.js        # Background job functions
├── prisma/                     # Prisma configuration
│   ├── schema.prisma          # Database schema
│   └── migrations/            # Migration files
├── data/                       # Static data
│   ├── categories.js          # Transaction categories
│   └── landing.js             # Landing page content
├── middleware.js               # Next.js middleware
├── next.config.mjs            # Next.js config
├── tailwind.config.js         # Tailwind config
└── package.json               # Project dependencies
```

### 8.2 Common Development Tasks

#### Task 1: Add New Transaction Category

**Steps**:

1. Edit `data/categories.js`
2. Add new category to `incomeCategories` or `expenseCategories`

```javascript
// data/categories.js
export const expenseCategories = [
  // ... existing categories
  {
    value: "pets",
    label: "Pets",
    icon: "🐕",
  },
];
```

3. No server restart needed, just refresh page

---

#### Task 2: Create New Server Action

**Steps**:

1. Create or edit file in `actions/` directory
2. Add `"use server"` directive
3. Implement function logic

```javascript
// actions/custom.js
"use server";

import { auth } from "@clerk/nextjs/server";
import { db } from "@/lib/prisma";

export async function myNewAction(data) {
  // 1. Auth check
  const { userId } = await auth();
  if (!userId) throw new Error("Unauthorized");

  // 2. Get user
  const user = await db.user.findUnique({
    where: { clerkUserId: userId },
  });

  // 3. Business logic
  const result = await db.someModel.create({
    data: {
      ...data,
      userId: user.id,
    },
  });

  // 4. Clear cache
  revalidatePath("/dashboard");

  // 5. Return result
  return result;
}
```

4. Call in component:

```javascript
import { myNewAction } from "@/actions/custom";

const { fn } = useFetch(myNewAction);
await fn({ /* data */ });
```

---

#### Task 3: Add New Page

**Steps**:

1. Create new folder in `app/(main)/`
2. Add `page.jsx`

```javascript
// app/(main)/reports/page.jsx
import { getUserReports } from "@/actions/reports";

export default async function ReportsPage() {
  const reports = await getUserReports();

  return (
    <div>
      <h1>Financial Reports</h1>
      {reports.map(report => (
        <div key={report.id}>{report.name}</div>
      ))}
    </div>
  );
}
```

3. Add navigation link (`components/header.jsx`):

```javascript
<Link href="/reports">Reports</Link>
```

4. Update middleware to protect route (`middleware.js`):

```javascript
const isProtectedRoute = createRouteMatcher([
  "/dashboard(.*)",
  "/account(.*)",
  "/transaction(.*)",
  "/reports(.*)",  // Add this line
]);
```

---

#### Task 4: Modify Database Schema

**Steps**:

1. Edit `prisma/schema.prisma`

```prisma
model Transaction {
  // ... existing fields
  tags String[]  // New field: tag array
}
```

2. Create migration:

```bash
npx prisma migrate dev --name add_tags_to_transaction
```

3. Regenerate Prisma client:

```bash
npx prisma generate
```

4. Update related code to use new field

---

#### Task 5: Add New Inngest Background Job

**Steps**:

1. Edit `lib/inngest/function.js`
2. Add new function

```javascript
export const myNewJob = inngest.createFunction(
  { id: "my-new-job" },
  { cron: "0 0 * * *" },  // Every midnight
  async ({ step }) => {
    await step.run("execute-task", async () => {
      // Your logic
      const users = await db.user.findMany();

      for (const user of users) {
        // Process each user
      }
    });
  }
);
```

3. Register function (`app/api/inngest/route.js`):

```javascript
import { myNewJob } from "@/lib/inngest/function";

export const { GET, POST, PUT } = serve({
  client: inngest,
  functions: [
    processRecurringTransaction,
    triggerRecurringTransactions,
    generateMonthlyReports,
    checkBudgetAlerts,
    myNewJob,  // Add this line
  ],
});
```

---

### 8.3 Debugging Tips

#### 1. Use Console Logging

```javascript
// Server Action
export async function myAction(data) {
  console.log("Received data:", data);

  const result = await db.transaction.create({ data });
  console.log("Created transaction:", result);

  return result;
}
```

**View Logs**:
- Development: Terminal output
- Production: Vercel Dashboard → Logs

---

#### 2. Use Prisma Studio

```bash
npx prisma studio
```

Visit http://localhost:5555 to view and edit database data

---

#### 3. Error Boundaries

Add `error.js` file to catch errors:

```javascript
// app/(main)/dashboard/error.js
"use client";

export default function Error({ error, reset }) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <p>{error.message}</p>
      <button onClick={() => reset()}>Retry</button>
    </div>
  );
}
```

---

#### 4. React DevTools

Install React DevTools browser extension to inspect component tree and props

---

### 8.4 Performance Optimization

#### 1. Use Suspense Boundaries

```javascript
<Suspense fallback={<Loading />}>
  <SlowComponent />
</Suspense>
```

#### 2. Optimize Images

Use Next.js `<Image>` component:

```javascript
import Image from "next/image";

<Image
  src="/logo.png"
  width={200}
  height={100}
  alt="Logo"
  priority  // For above-the-fold images
/>
```

#### 3. Database Query Optimization

```javascript
// ❌ Bad: N+1 queries
const accounts = await db.account.findMany();
for (const account of accounts) {
  const transactions = await db.transaction.findMany({
    where: { accountId: account.id }
  });
}

// ✅ Good: Use include
const accounts = await db.account.findMany({
  include: {
    transactions: true,
  },
});
```

#### 4. Use useMemo for Expensive Calculations

```javascript
const filteredTransactions = useMemo(() => {
  return transactions.filter(t => t.type === "EXPENSE");
}, [transactions]);
```

---

## 9. Deployment Guide

### 9.1 Vercel Deployment (Recommended)

**Steps**:

1. **Install Vercel CLI**:
```bash
npm i -g vercel
```

2. **Login**:
```bash
vercel login
```

3. **Deploy**:
```bash
vercel --prod
```

4. **Configure Environment Variables**:
   - Visit Vercel Dashboard
   - Project → Settings → Environment Variables
   - Add all variables from `.env`

5. **Configure Inngest Webhook**:
   - After deployment get domain: `https://your-app.vercel.app`
   - In Inngest Dashboard add: `https://your-app.vercel.app/api/inngest`

---

### 9.2 Docker Deployment

**Create Dockerfile**:

```dockerfile
FROM node:22-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:22-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npx prisma generate
RUN npm run build

FROM node:22-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

EXPOSE 3000
CMD ["node", "server.js"]
```

**Build and Run**:
```bash
docker build -t ai-finance-platform .
docker run -p 3000:3000 --env-file .env ai-finance-platform
```

---

### 9.3 Deployment Checklist

Pre-deployment:

- [ ] All environment variables configured
- [ ] Database migrations run
- [ ] API keys valid
- [ ] Clerk redirect URLs updated
- [ ] Inngest webhook registered
- [ ] Build successful (`npm run build`)
- [ ] Local production test passed (`npm start`)
- [ ] ArcJet rules configured
- [ ] Email service verified

Post-deployment:

- [ ] Homepage loads correctly
- [ ] User registration/login flow
- [ ] Create account
- [ ] Add transaction
- [ ] Receipt scanning works
- [ ] Budget setting
- [ ] Email notifications sent

---

## 10. Appendix

### 10.1 Technical Glossary

| Term | Explanation |
|------|-------------|
| **Server Component** | React components rendered on server, no JS sent to client |
| **Server Action** | Next.js 15 server-side functions callable from client |
| **Prisma ORM** | Type-safe database access layer, auto-generates query API |
| **Middleware** | Next.js middleware, executes before request reaches page |
| **revalidatePath** | Clear Next.js cache, force data refetch |
| **Zod** | TypeScript-first schema validation library |
| **Clerk** | Third-party auth service, handles login and sessions |
| **ArcJet** | Security service providing bot detection and rate limiting |
| **Inngest** | Background job platform for scheduled and async tasks |

### 10.2 Command Reference

```bash
# Development
npm run dev              # Start dev server
npm run build            # Production build
npm start                # Start production server
npm run lint             # Lint code

# Prisma
npx prisma generate      # Generate Prisma client
npx prisma migrate dev   # Create and apply migration (dev)
npx prisma migrate deploy # Apply migrations (production)
npx prisma studio        # Open database GUI
npx prisma db pull       # Generate schema from database
npx prisma db push       # Push schema to database (skip migrations)

# Database
npx prisma migrate reset # Reset database (dangerous!)
curl http://localhost:3000/api/seed # Seed data

# Deployment
vercel                   # Deploy to Vercel (preview)
vercel --prod            # Deploy to Vercel (production)
```

### 10.3 Learning Resources

#### Official Documentation

- [Next.js 15 Docs](https://nextjs.org/docs)
- [React 19 Docs](https://react.dev)
- [Prisma Docs](https://www.prisma.io/docs)
- [Tailwind CSS](https://tailwindcss.com/docs)
- [Clerk Docs](https://clerk.com/docs)

#### Video Tutorials

- YouTube: "Next.js App Router Tutorial"
- YouTube: "Prisma Complete Course"

### 10.4 FAQ

**Q: How to add new users?**

A: Users automatically created after Clerk registration. `checkUser()` function syncs Clerk users to database.

**Q: How to modify default categories?**

A: Edit `data/categories.js` file to add or remove categories.

**Q: Database migration failed?**

A: Check `DATABASE_URL` and `DIRECT_URL` configuration. Migrations must use `DIRECT_URL` (not through connection pooler).

**Q: Receipt scanning not working?**

A: Verify `GEMINI_API_KEY` is valid, ensure Gemini API enabled in Google Cloud project.

**Q: How to change rate limits?**

A: Edit `lib/arcjet.js`, modify `refillRate` and `capacity` parameters.

**Q: Email sending failed?**

A: Verify `RESEND_API_KEY`, check Resend dashboard send logs.

**Q: Inngest jobs not executing?**

A: Development: ensure Inngest Dev Server running. Production: check webhook configuration.

**Q: How to backup database?**

A: Use `pg_dump` or Supabase dashboard backup feature.

**Q: How to add dark mode?**

A: Use `next-themes` library with Tailwind's `dark:` prefix.

**Q: Performance slow?**

A: Check database queries (Prisma Studio), use `include` instead of multiple queries, add indexes.

---

## Conclusion

This specification document provides a complete technical overview of the AI Finance Platform. Use alongside:

1. **Project Analysis Docs** - `project-mastery/analysis/` directory
2. **Teaching Guide** - `project-mastery/teaching-guide/` directory
3. **Source Code Comments** - Read inline code comments

For questions, refer to:
- Official documentation links
- GitHub Issues
- Project README.md

Happy coding! 🚀

---

**Document Version**: 1.0
**Last Updated**: 2025-11-16
**Maintained By**: AI Finance Platform Team
