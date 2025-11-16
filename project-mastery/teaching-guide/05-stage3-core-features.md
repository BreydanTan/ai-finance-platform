# 第三阶段：核心功能 - 数据库与 CRUD 操作 💾

**学习时间**: 6-10 天
**难度等级**: ⭐⭐⭐ 中级
**完成标志**: 能执行完整的增删改查操作

---

## 🎯 学习目标

- ✅ 理解数据库表结构和关系
- ✅ 掌握 Prisma 基本操作
- ✅ 实现完整的 CRUD 功能
- ✅ 理解 Server Actions
- ✅ 处理表单提交和数据验证

---

## 📚 关键概念

→ [Prisma](../02-concept-dictionary.md#prisma)
→ [Server Actions](../02-concept-dictionary.md#server-actions)
→ [CRUD](../02-concept-dictionary.md#crud)
→ [Validation](../02-concept-dictionary.md#validation-验证)

---

## 🛠️ 核心任务

### 任务 1: 理解数据库结构

**打开 `prisma/schema.prisma`**，理解4个主要模型：

```prisma
model User {
  id       String @id @default(cuid())
  email    String @unique
  accounts Account[]       // 一个用户有多个账户
}

model Account {
  id      String @id
  name    String
  balance Decimal
  userId  String
  user    User @relation(fields: [userId], references: [id])
}

model Transaction {
  amount      Decimal
  date        DateTime
  accountId   String
  userId      String
}

model Budget {
  amount      Decimal
  month       Int
  year        Int
  accountId   String
}
```

**关系图**: 查看 `project-mastery/analysis/01-database-analysis.md`

---

### 任务 2: 第一次数据查询

**创建测试页面**: `app/test-db/page.jsx`

```jsx
import { db } from "@/lib/prisma";

export default async function TestDBPage() {
  // 服务器组件可以直接查询数据库
  const users = await db.user.findMany();

  return (
    <div className="p-8">
      <h1 className="text-2xl font-bold mb-4">数据库测试</h1>
      <p>用户数量: {users.length}</p>
      <ul>
        {users.map(user => (
          <li key={user.id}>{user.email}</li>
        ))}
      </ul>
    </div>
  );
}
```

**访问**: http://localhost:3000/test-db

---

### 任务 3: 创建 Server Action

**创建 `actions/my-actions.js`**:

```javascript
"use server";

import { db } from "@/lib/prisma";
import { revalidatePath } from "next/cache";

export async function createTestAccount(data) {
  try {
    // 创建账户
    const account = await db.account.create({
      data: {
        name: data.name,
        type: "CURRENT",
        balance: 0,
        userId: data.userId,  // 需要真实用户ID
      },
    });

    // 清除缓存
    revalidatePath("/test-db");

    return { success: true, data: account };
  } catch (error) {
    console.error("Error:", error);
    return { success: false, error: error.message };
  }
}
```

---

### 任务 4: 实现表单提交

**创建表单组件**: `app/test-db/_components/account-form.jsx`

```jsx
"use client";

import { useState } from "react";
import { createTestAccount } from "@/actions/my-actions";

export default function AccountForm({ userId }) {
  const [name, setName] = useState("");
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setLoading(true);

    const result = await createTestAccount({ name, userId });

    if (result.success) {
      alert("账户创建成功！");
      setName("");  // 清空表单
      window.location.reload();  // 刷新页面
    } else {
      alert("创建失败：" + result.error);
    }

    setLoading(false);
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div>
        <label className="block mb-2">账户名称</label>
        <input
          type="text"
          value={name}
          onChange={(e) => setName(e.target.value)}
          className="border rounded px-3 py-2 w-full"
          required
        />
      </div>
      <button
        type="submit"
        disabled={loading}
        className="bg-blue-500 text-white px-4 py-2 rounded"
      >
        {loading ? "创建中..." : "创建账户"}
      </button>
    </form>
  );
}
```

---

### 任务 5: 添加数据验证

**安装 Zod** (如果需要):
```bash
npm install zod
```

**创建验证规则**: `lib/my-schemas.js`

```javascript
import { z } from "zod";

export const accountSchema = z.object({
  name: z.string().min(1, "账户名称不能为空"),
  type: z.enum(["CURRENT", "SAVINGS"]),
  balance: z.string()
    .regex(/^\d+(\.\d{1,2})?$/, "金额格式错误"),
});
```

**在 Server Action 中使用**:

```javascript
import { accountSchema } from "@/lib/my-schemas";

export async function createTestAccount(data) {
  // 验证数据
  const validated = accountSchema.safeParse(data);

  if (!validated.success) {
    return {
      success: false,
      error: validated.error.errors[0].message
    };
  }

  // 继续创建...
}
```

---

### 任务 6: 完整 CRUD 示例

**参考现有代码**:

| 操作 | 文件 | 函数 |
|-----|------|------|
| Create | `actions/transaction.js` | `createTransaction` |
| Read | `actions/account.js` | `getAccountWithTransactions` |
| Update | `actions/transaction.js` | `updateTransaction` |
| Delete | `actions/transaction.js` | `deleteTransaction` |

**学习方法**:
1. 打开上述文件
2. 阅读代码和注释
3. 理解数据流程
4. 模仿创建自己的 CRUD

**AI 协作提示词**:
```
"基于项目中的 createTransaction 函数 (actions/transaction.js:18-100)，
帮我创建一个类似的函数用于创建预算 (Budget)。

需要：
1. 验证用户认证
2. 验证输入数据（金额、月份、年份）
3. 创建预算记录
4. 清除缓存
5. 返回结果

请提供完整代码和注释。"
```

---

## ✅ 验证清单

- [ ] 理解 Prisma schema 定义
- [ ] 能使用 Prisma Studio 查看数据
- [ ] 成功创建 Server Action
- [ ] 实现了表单提交功能
- [ ] 添加了数据验证
- [ ] 理解 revalidatePath 的作用
- [ ] 完成了至少一个完整的 CRUD 操作

---

## 🚀 实战挑战

### 挑战 1: 创建简单的待办事项

1. 添加 `Todo` 模型到 `schema.prisma`
2. 创建迁移
3. 创建 CRUD Server Actions
4. 创建展示和添加页面

### 挑战 2: 添加数据统计

创建页面显示：
- 总账户数
- 总交易数
- 本月支出总额

---

## 📊 学习资源

**Prisma 学习**:
- [Prisma 官方文档](https://www.prisma.io/docs)
- [Prisma 中文教程](https://prisma.yoga)

**视频教程**:
- B站: "Prisma 完整教程"
- YouTube: "Prisma Crash Course"

---

## ➡️ 下一步

掌握了数据库操作后，进入 **[第四阶段：完整功能复制](./06-stage4-full-replication.md)**
