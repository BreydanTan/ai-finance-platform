# 前端重建提示词 🎨

完整的前端功能重建指令。

---

## 提示词 3.1: 创建基础 UI 组件库

```
基于 Radix UI 和 Tailwind CSS,创建以下基础组件:

组件列表:
1. Button - 按钮组件
   - 变体: default, destructive, outline, ghost
   - 尺寸: sm, default, lg

2. Input - 输入框组件
   - 支持类型: text, email, password, number
   - 支持错误状态显示

3. Card - 卡片组件
   - Header, Content, Footer 三部分
   - 支持阴影和边框

4. Dialog - 对话框组件
   - 标题和描述
   - 关闭按钮
   - 可自定义内容

5. Select - 下拉选择组件
   - 基于 Radix UI Select
   - 支持搜索（可选）

技术要求:
- TypeScript 类型（或 JSDoc）
- 支持所有 Tailwind 类名透传
- 可访问性（ARIA）
- 使用 forwardRef

目录结构:
components/ui/
  ├── button.jsx
  ├── input.jsx
  ├── card.jsx
  ├── dialog.jsx
  └── select.jsx
```

**参考代码**: `components/ui/`

---

## 提示词 3.2: 实现响应式布局

```
创建全局布局组件,支持响应式设计:

布局要求:
1. Header (头部)
   - Logo
   - 导航菜单
   - 用户头像菜单
   - 移动端汉堡菜单

2. Sidebar (侧边栏 - 可选)
   - 桌面端固定显示
   - 移动端抽屉式

3. Main Content (主内容区)
   - 自适应高度
   - 内容居中
   - 最大宽度限制

4. Footer (底部 - 可选)
   - 版权信息
   - 链接

响应式断点:
- mobile: < 640px
- tablet: 640px - 1024px
- desktop: > 1024px

实现位置: app/(main)/layout.jsx

使用技术:
- Tailwind 响应式类名 (sm:, md:, lg:)
- Radix UI Sheet (抽屉组件)
```

**参考**: `app/(main)/layout.jsx`

---

## 提示词 3.3: 创建仪表板页面

```
创建财务仪表板页面,包含以下部分:

1. 统计卡片区 (顶部)
   - 总余额卡片
   - 本月收入卡片
   - 本月支出卡片
   - 净收入卡片

   样式: 网格布局 (1列 → 2列 → 4列)

2. 图表区域
   - 收支趋势折线图 (Recharts)
   - 分类占比饼图

3. 最近交易列表
   - 显示最近10笔交易
   - 包含: 日期,描述,分类,金额
   - 支持点击查看详情

4. 快速操作按钮
   - 添加交易
   - 创建账户
   - 设置预算

数据来源: Server Component 直接调用 Server Action
  - getDashboardData()
  - getUserAccounts()

文件位置: app/(main)/dashboard/page.jsx

技术栈:
- Server Component (async function)
- Recharts 图表库
- Tailwind CSS Grid/Flex
```

**参考**: `app/(main)/dashboard/page.jsx`

---

## 提示词 3.4: 实现交易表单

```
创建完整的交易录入表单:

表单字段:
1. type (类型)
   - Radio 组: INCOME / EXPENSE
   - 默认选中 EXPENSE

2. amount (金额)
   - Input type="text"
   - 验证: 正数,最多2位小数
   - Placeholder: "0.00"

3. date (日期)
   - DatePicker 组件
   - 默认今天
   - 格式: YYYY-MM-DD

4. category (分类)
   - Select 组件
   - 选项根据 type 动态变化
   - INCOME: 工资,奖金,投资,其他
   - EXPENSE: 食品,交通,购物,娱乐,其他

5. accountId (账户)
   - Select 从用户账户列表
   - 显示: 账户名 + 当前余额

6. description (描述) - 可选
   - Textarea
   - 最多200字符

7. receipt (收据) - 可选
   - 文件上传
   - 支持 JPG, PNG
   - 或者 AI 扫描按钮

8. 循环交易设置 (可选)
   - isRecurring: Checkbox
   - recurringInterval: Select (DAILY, WEEKLY, MONTHLY, YEARLY)
   - 只有勾选时显示间隔选择

技术实现:
- React Hook Form 管理状态
- Zod schema 验证
- useFetch hook 提交
- 成功后 revalidate 并跳转

文件位置: app/(main)/transaction/_components/transaction-form.jsx

集成 Server Action: createTransaction()
```

**参考**: `app/(main)/transaction/_components/transaction-form.jsx`

---

## 提示词 3.5: 创建账户详情页

```
创建账户详情页面,显示单个账户的完整信息:

页面结构:
1. 账户信息卡片
   - 账户名称
   - 账户类型 (活期/储蓄)
   - 当前余额 (大号显示)
   - 编辑按钮

2. 预算进度条 (如果设置了预算)
   - 预算总额
   - 已使用金额
   - 剩余金额
   - 使用百分比进度条
   - 编辑预算按钮

3. 交易列表
   - 按日期倒序
   - 每条显示: 日期, 描述, 分类图标, 金额(红/绿)
   - 支持筛选: 收入/支出, 日期范围
   - 支持搜索: 描述关键词
   - 分页: 每页20条

4. 操作按钮
   - 添加交易
   - 导出数据
   - 删除账户 (二次确认)

数据获取: getAccountWithTransactions(accountId)

路由: /account/[id]
文件: app/(main)/account/[id]/page.jsx

交互:
- 点击交易 → 打开编辑对话框
- 批量选择 → 批量删除
- 下拉加载更多
```

**参考**: `app/(main)/account/[id]/page.jsx`

---

## 提示词 3.6: 添加收据扫描UI

```
在交易表单中集成 AI 收据扫描功能:

UI 流程:
1. 显示上传区域
   - 点击或拖拽上传
   - 显示预览图
   - 支持重新选择

2. 扫描按钮
   - 文字: "AI 扫描收据"
   - 点击后:
     a. 显示加载动画
     b. 调用 scanReceipt Server Action
     c. 解析返回的JSON
     d. 自动填充表单字段

3. 结果处理
   - 成功:
     * 填充 amount, date, description, category
     * 显示成功提示
     * 用户可以修改
   - 失败:
     * 显示错误信息
     * 允许重试
     * 可以手动输入

组件位置:
- app/(main)/transaction/_components/receipt-upload.jsx

集成到: transaction-form.jsx

Server Action: scanReceipt(file)

技术细节:
- 使用 FileReader 读取图片
- 转 base64 发送给后端
- 后端调用 Gemini AI API
- 返回结构化数据
```

**参考**: `actions/transaction.js:231-291`

---

## 提示词 3.7: 实现数据可视化

```
使用 Recharts 创建财务数据可视化图表:

图表1: 收支趋势折线图
- 数据: 最近30天的每日收入和支出
- X轴: 日期
- Y轴: 金额
- 两条线: 收入(绿色), 支出(红色)
- 鼠标悬停显示具体数值

图表2: 分类占比饼图
- 数据: 本月各分类的支出占比
- 显示百分比和金额
- 颜色区分不同分类
- 点击分类高亮相关数据

图表3: 月度对比柱状图
- 数据: 最近6个月的收支对比
- X轴: 月份
- Y轴: 金额
- 两组柱: 收入, 支出
- 显示净收入趋势线

组件位置:
- components/charts/income-expense-chart.jsx
- components/charts/category-pie-chart.jsx
- components/charts/monthly-bar-chart.jsx

集成到: app/(main)/dashboard/page.jsx

数据准备:
- Server Action 聚合查询
- 格式化为 Recharts 所需格式
```

**参考**: Recharts 文档 + 项目现有图表

---

## 提示词 3.8: 创建移动端优化

```
优化项目的移动端体验:

优化点:
1. 导航
   - 桌面: 顶部横向导航
   - 移动: 底部 Tab 导航或汉堡菜单

2. 表单
   - 字段垂直排列
   - 大号输入框和按钮
   - 日期选择器适配移动端

3. 列表
   - 卡片式布局代替表格
   - 左滑显示操作按钮

4. 图表
   - 自适应容器宽度
   - 简化 tooltip
   - 可横向滑动

5. 触摸优化
   - 按钮最小尺寸 44x44px
   - 适当间距防止误触
   - 支持手势操作

响应式断点:
- Mobile: < 640px
- Tablet: 640px - 1024px
- Desktop: > 1024px

Tailwind 类名:
- hidden md:block (桌面显示)
- block md:hidden (移动显示)
- grid-cols-1 md:grid-cols-2 lg:grid-cols-4

测试:
- Chrome DevTools 移动模拟器
- 真实设备测试
```

---

## 提示词 3.9: 添加加载和错误状态

```
为所有异步操作添加友好的状态提示:

1. 加载状态 (Loading)
   位置:
   - 页面级: loading.jsx
   - 组件级: useFetch hook

   实现:
   - Skeleton 骨架屏
   - Spinner 转圈动画
   - 进度条

2. 错误状态 (Error)
   位置:
   - 页面级: error.jsx
   - 全局: Error Boundary

   实现:
   - 错误图标 + 消息
   - 重试按钮
   - 返回首页链接

3. 空状态 (Empty)
   场景:
   - 无数据时显示
   - 搜索无结果

   实现:
   - 空状态插图
   - 引导文案
   - 创建按钮

4. 成功提示 (Success)
   实现: Toast 通知
   - 绿色背景
   - 对勾图标
   - 自动消失(3秒)

示例文件:
- app/(main)/dashboard/loading.jsx
- app/(main)/dashboard/error.jsx
- components/empty-state.jsx
```

---

## 提示词 3.10: 实现主题切换

```
添加暗色/亮色主题切换功能:

实现步骤:
1. 安装 next-themes
   npm install next-themes

2. 配置 ThemeProvider
   app/layout.jsx:
   - 包裹 <ThemeProvider>
   - 属性: attribute="class"

3. 更新 Tailwind 配置
   tailwind.config.js:
   - darkMode: 'class'
   - 定义暗色主题颜色

4. 创建主题切换器
   components/theme-toggle.jsx:
   - 按钮: 太阳/月亮图标
   - 使用 useTheme hook
   - 三种模式: light, dark, system

5. 更新所有组件
   - 添加 dark: 前缀类名
   - dark:bg-gray-800
   - dark:text-white

6. 持久化
   - localStorage 存储偏好
   - 页面加载时应用

集成位置:
- 头部右上角
- 用户菜单中
```

---

**总结**: 这些提示词覆盖了前端的所有核心功能，可以按顺序逐个实现。
