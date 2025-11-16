# 第五阶段：功能扩展 - 添加自定义功能 🚀

**学习时间**: 持续学习
**难度等级**: ⭐⭐⭐⭐⭐ 专家
**完成标志**: 成功添加自定义功能

---

## 🎯 学习目标

- ✅ 独立分析需求
- ✅ 设计数据库结构
- ✅ 实现完整功能
- ✅ 测试和优化
- ✅ 部署到生产环境

---

## 💡 扩展功能建议

### 1. 数据导出功能

**需求**: 导出交易记录为 Excel/CSV

**实现步骤**:

1. **安装依赖**:
```bash
npm install xlsx
```

2. **创建导出函数**:
```javascript
// actions/export.js
"use server";

import * as XLSX from "xlsx";

export async function exportTransactions(userId) {
  // 获取用户交易
  const transactions = await db.transaction.findMany({
    where: { userId },
    include: { account: true },
  });

  // 转换数据格式
  const data = transactions.map(t => ({
    日期: t.date.toLocaleDateString('zh-CN'),
    类型: t.type === "INCOME" ? "收入" : "支出",
    金额: t.amount,
    分类: t.category,
    账户: t.account.name,
    描述: t.description || "",
  }));

  // 生成 Excel
  const ws = XLSX.utils.json_to_sheet(data);
  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, ws, "交易记录");

  // 返回 buffer
  const buffer = XLSX.write(wb, { type: "buffer", bookType: "xlsx" });

  return buffer;
}
```

3. **添加下载按钮**:
```jsx
"use client";

export function ExportButton({ userId }) {
  const handleExport = async () => {
    const result = await exportTransactions(userId);

    // 创建下载链接
    const blob = new Blob([result], { type: "application/xlsx" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = `transactions-${new Date().toISOString()}.xlsx`;
    a.click();
  };

  return (
    <button onClick={handleExport}>
      导出 Excel
    </button>
  );
}
```

---

### 2. 循环交易管理界面

**需求**: 可视化管理循环交易

**功能点**:
- 查看所有循环交易
- 暂停/恢复循环
- 修改循环规则
- 查看执行历史

**数据模型** (已存在):
```prisma
model Transaction {
  isRecurring         Boolean @default(false)
  recurringInterval   RecurringInterval?
  nextRecurringDate   DateTime?
}
```

**创建页面**: `app/(main)/recurring/page.jsx`

---

### 3. 财务报告生成

**需求**: 生成月度/年度财务报告

**报告内容**:
- 总收入/支出
- 分类统计
- 环比增长
- 可视化图表

**实现**:
```javascript
// actions/reports.js
export async function generateMonthlyReport(userId, year, month) {
  // 1. 查询该月所有交易
  const transactions = await db.transaction.findMany({
    where: {
      userId,
      date: {
        gte: new Date(year, month - 1, 1),
        lt: new Date(year, month, 1),
      },
    },
  });

  // 2. 统计数据
  const stats = {
    totalIncome: transactions
      .filter(t => t.type === "INCOME")
      .reduce((sum, t) => sum + t.amount, 0),

    totalExpense: transactions
      .filter(t => t.type === "EXPENSE")
      .reduce((sum, t) => sum + t.amount, 0),

    byCategory: groupByCategory(transactions),
  };

  // 3. 生成报告
  return {
    period: `${year}年${month}月`,
    ...stats,
    netIncome: stats.totalIncome - stats.totalExpense,
  };
}
```

---

### 4. 多用户协作（家庭账本）

**需求**: 允许多人管理同一账户

**数据模型扩展**:
```prisma
model AccountMember {
  id          String   @id @default(cuid())
  accountId   String
  userId      String
  role        Role     @default(MEMBER)  // OWNER, ADMIN, MEMBER
  permissions Json     // 自定义权限
  account     Account  @relation(fields: [accountId], references: [id])
  user        User     @relation(fields: [userId], references: [id])
}

enum Role {
  OWNER
  ADMIN
  MEMBER
  VIEWER
}
```

**实现要点**:
- 邀请链接生成
- 权限检查
- 活动日志

---

### 5. 财务目标追踪

**需求**: 设置和追踪财务目标

**示例目标**:
- 6个月存够10万
- 每月投资5000元
- 年底资产达到50万

**数据模型**:
```prisma
model FinancialGoal {
  id            String   @id @default(cuid())
  userId        String
  title         String
  targetAmount  Decimal
  currentAmount Decimal  @default(0)
  deadline      DateTime
  status        GoalStatus @default(ACTIVE)
  createdAt     DateTime @default(now())
}

enum GoalStatus {
  ACTIVE
  COMPLETED
  PAUSED
  FAILED
}
```

---

### 6. 智能分类建议

**需求**: AI 自动推荐交易分类

**实现**:
```javascript
// 使用 Gemini AI
export async function suggestCategory(description) {
  const prompt = `
    基于交易描述,推荐最合适的分类:
    描述: "${description}"

    可选分类: ${categories.join(", ")}

    只返回分类名称,不要其他内容。
  `;

  const result = await gemini.generateText(prompt);
  return result.trim();
}
```

---

### 7. 定制化仪表板

**需求**: 用户自定义仪表板布局

**功能**:
- 拖拽排列组件
- 选择显示的图表
- 保存布局配置

**技术方案**:
- 使用 `react-grid-layout`
- 配置存储在数据库

---

### 8. 移动端优化

**需求**: PWA 支持，可安装到手机

**步骤**:

1. **创建 manifest.json**:
```json
{
  "name": "AI Finance Platform",
  "short_name": "Finance",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#3b82f6",
  "icons": [
    {
      "src": "/icon-192.png",
      "sizes": "192x192",
      "type": "image/png"
    }
  ]
}
```

2. **添加 Service Worker**:
```javascript
// public/sw.js
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open('v1').then((cache) => {
      return cache.addAll([
        '/',
        '/dashboard',
      ]);
    })
  );
});
```

3. **更新 Next.js 配置**:
```javascript
// next.config.mjs
const withPWA = require('next-pwa')({
  dest: 'public'
});

module.exports = withPWA({
  // 其他配置
});
```

---

## 🎯 实战项目建议

选择1-2个扩展功能独立完成：

### 小项目（3-5天）
- 数据导出
- 简单报告

### 中项目（1-2周）
- 循环交易界面
- 财务目标追踪

### 大项目（2-4周）
- 多用户协作
- 完整报告系统

---

## 💻 开发流程

### 1. 需求分析
- 明确功能需求
- 列出验收标准
- 画原型图

### 2. 数据库设计
- 设计新表结构
- 确定字段和关系
- 创建迁移

### 3. 后端开发
- 创建 Server Actions
- 实现业务逻辑
- 添加验证

### 4. 前端开发
- 设计 UI 界面
- 连接后端接口
- 处理用户交互

### 5. 测试
- 功能测试
- 边界情况
- 用户体验

### 6. 优化
- 性能优化
- 代码重构
- 文档编写

---

## 🚀 部署到生产环境

### 部署到 Vercel

**步骤**:

1. **推送到 GitHub**:
```bash
git add .
git commit -m "完成项目开发"
git push origin main
```

2. **连接 Vercel**:
- 访问 vercel.com
- Import Git Repository
- 选择你的仓库

3. **配置环境变量**:
- 在 Vercel 项目设置中
- 添加所有 `.env` 变量

4. **部署**:
- Vercel 自动部署
- 获得生产环境 URL

**参考**: `project-mastery/specifications/PROJECT_SPEC_CN.md` 第9章

---

## 📚 持续学习资源

### 进阶主题

1. **性能优化**
   - React Server Components
   - 缓存策略
   - 数据库索引

2. **安全加固**
   - CSRF 防护
   - XSS 防护
   - SQL 注入防护

3. **测试**
   - 单元测试 (Jest)
   - 集成测试
   - E2E 测试 (Playwright)

4. **监控和日志**
   - Sentry 错误追踪
   - Analytics 集成
   - 性能监控

### 推荐课程

- [Next.js 大师课程](https://nextjs.org/learn)
- [Prisma 完整教程](https://www.prisma.io/docs)
- [全栈开发路线图](https://roadmap.sh/full-stack)

---

## 🎓 毕业项目

### 要求

创建一个完整的个人项目，包含：

1. **核心功能** (必须)
   - 用户系统
   - CRUD 操作
   - 数据可视化

2. **扩展功能** (选3个)
   - 从上面的建议中选择
   - 或自己设计新功能

3. **部署** (必须)
   - 部署到 Vercel
   - 配置自定义域名(可选)

4. **文档** (必须)
   - README.md
   - API 文档
   - 用户手册

### 评估标准

| 项目 | 分值 |
|-----|------|
| 功能完整性 | 40分 |
| 代码质量 | 20分 |
| 用户体验 | 20分 |
| 文档质量 | 10分 |
| 创新性 | 10分 |

**总分**: 100分
**优秀**: 85分以上
**良好**: 70-84分
**及格**: 60-69分

---

## 🎉 恭喜完成学习！

你已经掌握：
- ✅ Next.js 15 全栈开发
- ✅ Prisma 数据库操作
- ✅ 现代 React 开发
- ✅ AI 集成应用
- ✅ 项目部署运维

### 下一步建议

1. **构建作品集**
   - 在 GitHub 展示项目
   - 写技术博客
   - 参与开源项目

2. **深入专精**
   - 选择一个方向深入
   - 前端/后端/全栈

3. **职业发展**
   - 准备技术面试
   - 关注招聘信息
   - 建立人脉网络

---

**记住**: 学习永无止境，keep coding! 🚀
