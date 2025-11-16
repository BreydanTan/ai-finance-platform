# 调试决策树 🔍

遇到问题时的系统化排查流程。

---

## 主决策树

```
🤔 遇到问题了？
    │
    ├─❓ 页面显示问题？
    │   ├─ 页面空白
    │   │   ├─ 检查开发服务器是否运行
    │   │   ├─ 检查浏览器控制台 (F12 → Console)
    │   │   ├─ 检查路由配置是否正确
    │   │   └─ 检查组件是否导出 (export default)
    │   │
    │   ├─ 样式不显示
    │   │   ├─ 检查 Tailwind 类名拼写
    │   │   ├─ 检查文件在 content 配置中
    │   │   ├─ 重启开发服务器
    │   │   └─ 检查 CSS 文件导入
    │   │
    │   └─ 布局错乱
    │       ├─ 检查 HTML 结构
    │       ├─ 检查响应式类名
    │       └─ 使用浏览器检查工具
    │
    ├─❓ 看到错误消息？
    │   ├─ 红色错误（致命）
    │   │   ├─ 语法错误 → 检查代码拼写
    │   │   ├─ 导入错误 → 检查路径和包
    │   │   └─ 类型错误 → 检查数据类型
    │   │
    │   ├─ 黄色警告（警告）
    │   │   ├─ 通常可以忽略
    │   │   └─ 但最好修复
    │   │
    │   └─ 网络错误
    │       ├─ 检查 API 端点
    │       ├─ 检查服务器运行
    │       └─ 检查 CORS 配置
    │
    ├─❓ 功能不工作？
    │   ├─ 数据问题
    │   │   ├─ 检查数据库连接
    │   │   ├─ 检查 Prisma schema
    │   │   ├─ 使用 Prisma Studio 查看数据
    │   │   └─ 检查查询逻辑
    │   │
    │   ├─ 交互问题
    │   │   ├─ 检查事件绑定
    │   │   ├─ 检查状态更新
    │   │   ├─ 添加 console.log 调试
    │   │   └─ 检查条件渲染
    │   │
    │   └─ 认证问题
    │       ├─ 检查环境变量
    │       ├─ 检查 Clerk 配置
    │       ├─ 检查中间件设置
    │       └─ 清除浏览器缓存
    │
    └─❓ 性能问题？
        ├─ 加载慢
        │   ├─ 检查网络请求数量
        │   ├─ 检查图片大小
        │   ├─ 优化数据库查询
        │   └─ 使用代码分割
        │
        └─ 响应慢
            ├─ 检查是否有循环渲染
            ├─ 使用 React DevTools
            ├─ 优化组件性能
            └─ 添加缓存
```

---

## 详细排查步骤

### 1. 页面空白排查

```
开始
  ↓
检查开发服务器
  ├─ 未运行 → 运行 npm run dev
  └─ 正在运行 → 继续
  ↓
打开浏览器控制台 (F12)
  ├─ 有错误信息 → 复制错误,查询解决方案
  └─ 无错误 → 继续
  ↓
检查网络标签
  ├─ 请求失败 → 检查 API 端点
  └─ 请求成功 → 继续
  ↓
检查 React DevTools
  ├─ 组件未渲染 → 检查条件渲染逻辑
  └─ 组件已渲染 → 检查返回值
  ↓
检查代码
  ├─ 检查 export default
  ├─ 检查 return 语句
  └─ 检查 JSX 语法
```

### 2. 数据不显示排查

```
开始
  ↓
添加 console.log 调试
  console.log("数据:", data)
  ↓
检查控制台输出
  ├─ 无输出 → 函数未执行
  │   └─ 检查函数调用
  ├─ undefined → 数据未获取
  │   └─ 检查 API/Server Action
  └─ 有数据 → 继续
  ↓
检查数据结构
  console.log("类型:", typeof data)
  console.log("长度:", data?.length)
  ↓
检查渲染逻辑
  ├─ 条件渲染问题 → 修改条件
  ├─ 数组为空 → 检查数据源
  └─ 映射错误 → 检查 map 函数
```

### 3. 表单提交失败排查

```
开始
  ↓
检查表单事件
  <form onSubmit={handleSubmit}>
  ├─ 未绑定 → 添加 onSubmit
  └─ 已绑定 → 继续
  ↓
检查是否阻止默认行为
  e.preventDefault()
  ├─ 未阻止 → 添加 preventDefault
  └─ 已阻止 → 继续
  ↓
检查验证
  ├─ 表单验证失败 → 查看错误提示
  └─ 验证通过 → 继续
  ↓
检查 Server Action
  ├─ 添加 try/catch
  ├─ console.log 输入数据
  └─ 检查返回值
  ↓
检查数据库
  ├─ 使用 Prisma Studio 查看
  └─ 检查约束和关系
```

### 4. 样式不生效排查

```
开始
  ↓
检查类名拼写
  ├─ 拼写错误 → 修正
  └─ 拼写正确 → 继续
  ↓
检查 Tailwind 配置
  tailwind.config.js
  ├─ content 路径不包含文件 → 添加路径
  └─ 路径正确 → 继续
  ↓
重启开发服务器
  Ctrl+C → npm run dev
  ├─ 样式出现 → 完成
  └─ 仍不显示 → 继续
  ↓
检查浏览器
  ├─ 打开检查元素
  ├─ 查看应用的样式
  └─ 检查是否被覆盖
  ↓
清除缓存
  ├─ 硬刷新 (Ctrl+Shift+R)
  └─ 清除浏览器缓存
```

---

## 调试工具箱

### 浏览器工具

```
开发者工具 (F12)
  ├─ Console: 查看日志和错误
  ├─ Network: 查看网络请求
  ├─ Elements: 检查 HTML 和 CSS
  ├─ Sources: 调试 JavaScript
  └─ React DevTools: 检查组件
```

### 代码调试技巧

```javascript
// 1. console.log 大法
console.log("变量值:", variable);
console.log("函数执行", { param1, param2 });

// 2. debugger 断点
function myFunction() {
  debugger; // 代码会在这里暂停
  // ... 你的代码
}

// 3. 条件日志
if (condition) {
  console.error("不应该到这里");
}

// 4. 性能计时
console.time("操作");
// ... 你的代码
console.timeEnd("操作");

// 5. 表格输出
console.table(arrayOfObjects);
```

### Prisma 调试

```javascript
// 启用查询日志
// prisma/schema.prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
  previewFeatures = ["tracing"]  // 添加这行
}

// 代码中
const result = await db.user.findMany();
// 控制台会显示实际的 SQL 查询
```

---

## AI 协助调试提示词

```
我遇到以下问题:
[详细描述问题]

环境：
- 框架: Next.js 15
- 发生在: [文件路径]
- 操作: [描述在做什么时发生]

错误信息:
[粘贴完整错误堆栈]

相关代码:
[粘贴代码片段]

我已经尝试:
- [列出已尝试的解决方案]

浏览器控制台输出:
[粘贴控制台内容]

请帮我:
1. 分析可能的原因
2. 提供排查步骤
3. 给出解决方案
```

---

## 预防性措施

### 开发规范

- ✅ 代码提交前测试
- ✅ 使用 TypeScript/JSDoc
- ✅ 添加错误处理
- ✅ 编写有意义的注释
- ✅ 遵循命名规范

### 测试清单

每次修改后检查:
- [ ] 页面能正常访问
- [ ] 控制台无错误
- [ ] 功能按预期工作
- [ ] 样式正确显示
- [ ] 数据正确保存

---

**记住**: 90%的bug都能通过系统化排查找到原因！
