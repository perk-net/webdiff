# 前置操作脚本

**前置操作** 是 [网站内容监控](../features/task-website) 任务中的可选步骤：在页面加载完成后、提取监控元素之前，自动执行一段 JavaScript 代码，用于登录、点击展开、搜索筛选、滚动加载等页面交互。

## 运行方式

你填写的代码会作为一个 **async 函数体** 执行，等价于：

```javascript
async (page, context, vars) => {
  // 你的代码
}
```

**执行时机**：在 `page.goto(网址)` 与页面加载完成之后、「提取前滚动」与提取监控元素之前运行一次。任务每次执行都会重新运行。

## 可用对象

| 对象 | 说明 |
| --- | --- |
| `page` | 当前页面对象，支持点击、输入、等待、跳转等全部 Playwright page API |
| `context` | 浏览器上下文 BrowserContext（任务配置的 Cookie 已注入，打开即登录态） |
| `vars` | 「变量」表的键值对象，用 `vars.变量名` 读取（如 `vars.PASSWORD`） |

## 常用写法

```javascript
// 点击元素
await page.click('#login-btn');

// 输入文本（敏感信息建议用变量）
await page.fill('#username', 'myname');
await page.fill('#password', vars.PASSWORD);

// 下拉选择
await page.selectOption('#city', '北京');

// 按键
await page.press('#search', 'Enter');

// 等待元素出现 / 固定等待
await page.waitForSelector('.list-item');
await page.waitForTimeout(1000);

// 跳转到另一个地址
await page.goto('https://example.com/list', { waitUntil: 'domcontentloaded' });

// 滚动页面
await page.evaluate(() => window.scrollTo(0, document.body.scrollHeight));
```

## 逻辑控制

代码按完整的 async 函数体执行，可使用 `if/else`、`for`、`while`、`try/catch`、变量声明、函数等全部 JavaScript 语法。循环和判断里调用页面操作时记得加 `await`。

```javascript
// 条件判断：元素存在才点击
if (await page.$('#cookie-accept')) {
  await page.click('#cookie-accept');
}

// if-else：根据是否已登录决定流程
const loggedIn = await page.$('.user-avatar');
if (loggedIn) {
  await page.click('.user-avatar');
} else {
  await page.fill('#username', vars.USERNAME);
  await page.fill('#password', vars.PASSWORD);
  await page.click('#login-btn');
}

// for 循环：点击「加载更多」3 次
for (let i = 0; i < 3; i++) {
  const moreBtn = await page.$('text=加载更多');
  if (!moreBtn) break;
  await moreBtn.click();
  await page.waitForTimeout(800);
}

// while 循环：不断下拉直到没有新内容
let last = 0;
while (true) {
  await page.evaluate(() => window.scrollTo(0, document.body.scrollHeight));
  await page.waitForTimeout(600);
  const h = await page.evaluate(() => document.body.scrollHeight);
  if (h === last) break;
  last = h;
}

// try/catch：捕获异常避免中断
try {
  await page.click('#maybe-missing', { timeout: 2000 });
} catch (e) {
  // 忽略：按钮不存在时跳过
}

// 遍历多个元素
const items = await page.$$('.list-item a');
for (const item of items) {
  await item.hover();
}
```

## 选择器

支持 **CSS 选择器**（如 `#id`、`.class`、`div > a`）与 **XPath**（以 `//` 开头，系统会自动识别）。录制生成的选择器以 id、唯一属性优先，可手动改成更稳定的写法。

## 变量与密码

录制登录时，**密码框不会写入明文**，而是自动生成 `vars.PASSWORD` 这样的占位，并在「变量」表中新增对应条目，请在那里填写真实值。其他需要复用或保密的值（账号、Token 等）也可自行添加为变量后用 `vars.xxx` 引用。

## 注意事项

- 代码在 **本机客户端** 执行，单次执行超时约 **60 秒**。若代码 **语法错误、运行报错或超时**，本次任务会判定为 **执行失败** 并附上失败原因，不再继续提取元素——可在「立即执行」的提示或「执行记录」中查看。
- 建议在关键操作后加 `await page.waitForSelector(...)` 或适当等待，提升回放稳定性。
- 页面结构变化可能导致选择器失效，必要时回到预览窗口重新录制或手动调整。
- 关闭「启用前置操作」开关后，执行将跳过前置操作，直接提取元素。

## 相关文档

- [网站内容监控](../features/task-website) — 创建步骤与截图说明
- [cookie plus 账号](../features/cookie-plus) — 登录态注入
