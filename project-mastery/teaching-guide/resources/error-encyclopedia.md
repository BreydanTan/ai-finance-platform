# 错误百科全书 📕

项目中最常见的20个错误及解决方案。

---

## 1. Cannot find module 'xxx'

**错误示例**:
```
Error: Cannot find module 'next'
```

**什么意思**:
这个包没有安装或安装不完整。

**解决步骤**:

1. **检查 package.json**:
```bash
cat package.json | grep "next"
```

2. **重新安装**:
```bash
npm install
# 或
npm install next
```

3. **清除缓存重装**:
```bash
rm -rf node_modules package-lock.json
npm install
```

**预防措施**:
- 克隆项目后立即运行 `npm install`
- 不要手动删除 `node_modules` 中的文件

---

## 2. Port 3000 is already in use

**错误示例**:
```
Error: listen EADDRINUSE: address already in use :::3000
```

**什么意思**:
3000端口被其他程序占用。

**解决方案1 - 关闭占用进程**:

**macOS/Linux**:
```bash
lsof -ti:3000 | xargs kill
```

**Windows**:
```bash
netstat -ano | findstr :3000
taskkill /PID [PID号] /F
```

**解决方案2 - 使用其他端口**:
```bash
PORT=3001 npm run dev
```

---

## 3. Prisma Client did not initialize yet

**错误示例**:
```
PrismaClient is unable to be run in the browser
```

**原因**:
在客户端组件中使用了 Prisma。

**解决**:
Prisma 只能在服务器端使用：
- Server Components
- Server Actions
- API Routes

**正确示例**:
```javascript
// ✅ 服务器组件
export default async function Page() {
  const data = await db.user.findMany();
  return <div>{data.length}</div>;
}

// ❌ 客户端组件
"use client";
export default function Page() {
  const data = await db.user.findMany(); // 错误！
}
```

---

## 4. Hydration failed

**错误示例**:
```
Error: Hydration failed because the initial UI does not match what was rendered on the server
```

**原因**:
服务器和客户端渲染的 HTML 不一致。

**常见原因**:
1. 在服务器组件中使用 `Date.now()` 或 `Math.random()`
2. 嵌套错误（如 `<p>` 里套 `<div>`）
3. 浏览器扩展修改了 HTML

**解决**:
```javascript
// ❌ 错误
function Component() {
  return <div>{Date.now()}</div>;
}

// ✅ 正确
"use client";
function Component() {
  const [time, setTime] = useState(null);

  useEffect(() => {
    setTime(Date.now());
  }, []);

  return <div>{time}</div>;
}
```

---

## 5. Invalid `prisma.xxx.create()` invocation

**错误示例**:
```
Invalid `prisma.user.create()` invocation:
  Argument `data` is missing
```

**原因**:
Prisma 调用缺少必需参数。

**解决**:
检查 schema 中的必填字段：
```prisma
model User {
  id    String @id
  email String  // 必填
  name  String? // 可选（有?）
}
```

```javascript
// ❌ 错误 - 缺少 email
await db.user.create({
  data: { name: "张三" }
});

// ✅ 正确
await db.user.create({
  data: {
    id: cuid(),
    email: "test@example.com",
    name: "张三"
  }
});
```

---

## 6. CLERK_SECRET_KEY is not set

**错误示例**:
```
Error: Clerk: CLERK_SECRET_KEY is missing
```

**原因**:
环境变量未配置。

**解决**:

1. 检查 `.env` 文件是否存在
2. 检查变量名拼写
3. 重启开发服务器

```bash
# .env
CLERK_SECRET_KEY=sk_test_xxxxx
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_xxxxx
```

---

## 7. Unexpected token '<'

**错误示例**:
```
SyntaxError: Unexpected token '<'
```

**原因**:
通常是服务器返回了 HTML 而不是 JSON。

**常见场景**:
- API 路由返回错误页面
- 404 响应

**解决**:
检查网络请求：
1. 打开浏览器开发者工具
2. Network 标签
3. 查看失败请求的响应
4. 修复 API 端点

---

## 8. Too many re-renders

**错误示例**:
```
Error: Too many re-renders. React limits the number of renders to prevent an infinite loop.
```

**原因**:
在渲染时修改状态，导致无限循环。

**常见错误**:
```javascript
// ❌ 错误
function Component() {
  const [count, setCount] = useState(0);

  setCount(count + 1); // 每次渲染都执行！

  return <div>{count}</div>;
}

// ✅ 正确
function Component() {
  const [count, setCount] = useState(0);

  return (
    <div>
      {count}
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}
```

---

## 9. Cannot read property 'xxx' of undefined

**错误示例**:
```
TypeError: Cannot read property 'name' of undefined
```

**原因**:
访问了不存在的对象属性。

**解决**:

使用可选链：
```javascript
// ❌ 可能出错
const name = user.profile.name;

// ✅ 安全
const name = user?.profile?.name;

// ✅ 带默认值
const name = user?.profile?.name ?? "未知";
```

---

## 10. Failed to fetch

**错误示例**:
```
TypeError: Failed to fetch
```

**原因**:
网络请求失败。

**检查清单**:
- [ ] 开发服务器是否运行
- [ ] URL 拼写正确
- [ ] CORS 配置（如果跨域）
- [ ] 网络连接正常

**调试**:
```javascript
try {
  const res = await fetch('/api/data');
  if (!res.ok) {
    console.error('状态码:', res.status);
  }
  const data = await res.json();
} catch (error) {
  console.error('请求失败:', error);
}
```

---

## 11-20：更多错误...

（为节省篇幅，列出标题，详细内容类似上面）

11. Middleware must export a default function
12. React Hook useEffect has a missing dependency
13. Objects are not valid as a React child
14. Module not found: Can't resolve '@/...'
15. Invalid DateTime string
16. Prisma migration failed
17. Authentication redirect loop
18. Module parse failed: Unexpected token
19. Memory leak detected
20. Vercel deployment failed

---

## AI 协作提示词模板

```
我遇到以下错误：
[粘贴完整错误堆栈]

发生在：
- 文件: [文件路径]
- 操作: [描述在做什么]

相关代码：
[粘贴代码]

环境：
- Next.js 版本: [版本]
- Node 版本: [版本]

请帮我：
1. 解释错误原因
2. 提供解决方案
3. 如何预防
```

---

**使用建议**: 遇到错误先在本文档搜索关键词，90%的常见错误都在这里。
