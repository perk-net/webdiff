# 全局变量

在多任务之间复用同一套配置（如 API 地址、Token、域名、Cookie 片段），避免每个任务重复填写；修改全局变量后，所有引用该变量的任务在 **下次执行** 时自动使用新值。

## 与环境变量的区别

| | 全局变量 | 任务级环境变量（自定义脚本） |
| --- | --- | --- |
| 作用范围 | 账号下所有任务 | 仅当前脚本任务 |
| 配置入口 | 任务列表 → **全局变量** | 创建任务 → 运行环境 → 环境变量 |
| 任务配置引用 | <code v-pre>{{变量名}}</code> 占位符 | 不支持 <code v-pre>{{}}</code>；脚本内用 `webdiff.env()` |
| 脚本内读取 | `webdiff.global()` 等 | `webdiff.env()` / `process.env` |

::: tip
自定义脚本的 **运行环境 → 环境变量** 只注入当前任务进程；若要在脚本里读全局变量，请调用 `webdiff.global('NAME')`（见下文「在自定义脚本中使用」），不要与环境变量混用。
:::

## 管理全局变量

<ol class="feature-step-list">

<li>

### 打开全局变量对话框

在客户端 **任务列表** 页顶部，点击 **「全局变量」** 按钮。

</li>

<li>

### 添加并保存

每条变量包含 **变量名**、**默认值** 与 **备注**（备注仅用于界面辨识，不参与替换）。

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| 变量名 | ✅ | 如 `API_HOST`、`API_TOKEN` |
| 默认值 | 建议填写 | 执行时替换 <code v-pre>{{变量名}}</code> 的内容 |
| 备注 | 可选 | 如「生产环境 Token」 |

点击 **保存** 后，变量会同步到服务端，本机执行器也会刷新缓存。

</li>

</ol>

### 命名规则

- 仅允许 **字母、数字、下划线**
- 须以 **字母或下划线** 开头，不能以数字开头
- 变量名 **不可重复**

### 数量限制

按会员等级限制可保存的全局变量条数（企业会员不限）。对话框顶部会显示当前用量，达到上限时可升级会员或删除部分变量。

## 在任务配置中引用

占位符语法：<code v-pre>{{变量名}}</code>。任务 **执行时** 将占位符替换为对应默认值；若变量不存在或未配置，占位符 **保持原样** 不替换。

在支持全局变量的输入框右侧，可点击 **「全局变量」** 按钮，从列表中快速插入占位符。

### 支持的配置项

| 任务类型 | 可引用全局变量的字段 |
| --- | --- |
| 网站内容监控 | User-Agent；手动 Cookie 的 **值**；监控元素的 **选择器**、**备注** |
| HTTP 请求 | 手动 Cookie 的 **值**；请求 Header 的 **值**；**请求体** |
| RSS 订阅 | 手动 Cookie 的 **值** |
| 域名到期时间 | **域名** |

示例：将 API 根地址存为全局变量 `API_HOST`，HTTP 任务 Header 值填写：

```
Bearer {{API_TOKEN}}
```

网站监控元素选择器可写：

```
//div[@data-id='{{PRODUCT_ID}}']
```

::: warning Cookie Plus 同步模式
使用 **cookie plus 同步** 时，Cookie 由扩展拉取，手动 Cookie 列表不适用全局变量占位符。
:::

## 在自定义脚本中使用

任务配置里的 <code v-pre>{{变量名}}</code> **不会** 自动注入脚本进程。脚本内需通过 [WebDiff SDK](../reference/webdiff-sdk.md) 读取：

| 语言 | 读取方式 |
| --- | --- |
| JavaScript / TypeScript | `webdiff.global('NAME')` |
| Python 3 | `webdiff.global_var('NAME')`（`global` 为保留字） |
| Shell | `webdiff_global NAME` |

```javascript
const apiHost = webdiff.global('API_HOST') || '(未设置 API_HOST)';
const token = webdiff.global('API_TOKEN') || '';
```

```python
api_host = webdiff.global_var('API_HOST') or '(未设置 API_HOST)'
token = webdiff.global_var('API_TOKEN') or ''
```

无值或空字符串时 API 返回「空」，可用 `||` / `or` 提供默认文案。

**URL 类变量**（如 `API_URL`）须为完整地址（含 `https://`），或在脚本中使用 `webdiff.env_url('API_URL', '默认地址')` 配合 **任务级环境变量**；全局变量侧请直接存完整 URL。

::: info 远程脚本
使用 **远程文件** 作为脚本来源时，执行器无法在保存时扫描脚本内容；若脚本内调用了 `webdiff.global()`，请确保已在「全局变量」中配置对应变量名。
:::

## 常见用法

- **多环境切换**：维护 `API_HOST`、`API_TOKEN`，各任务 Header / 请求体引用 <code v-pre>{{API_TOKEN}}</code>，切换环境时只改变量默认值
- **共用域名**：域名到期、证书监控等任务引用 <code v-pre>{{MONITOR_DOMAIN}}</code>
- **动态选择器**：网站监控 XPath/CSS 中含 <code v-pre>{{PRODUCT_ID}}</code> 等可变片段
- **脚本 + 配置分离**：敏感 Token 放在全局变量，脚本用 `webdiff.global()` 读取，避免硬编码在代码里

## 排查

- **占位符未替换**：检查变量名是否与 <code v-pre>{{}}</code> 内完全一致（区分大小写）；确认已在任务列表保存全局变量
- **脚本读不到值**：确认使用的是 `webdiff.global()` / `webdiff.global_var()` / `webdiff_global`，而非 `webdiff.env()`
- **保存失败**：检查变量名格式、是否重名、是否超出会员数量上限
