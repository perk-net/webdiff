# WebDiff SDK

[自定义脚本](../features/task-script.md) 任务在客户端本地执行脚本时，会自动下发内置 **WebDiff SDK**（`webdiff` 包），用于读取运行环境、Cookie Plus 登录态、发送通知、编排其它任务等。

执行前客户端会在 **脚本工作目录**（`script-workspace`）生成 SDK 文件与本次任务专用的 `webdiff-context.json`，**无需**在 npm / pip 依赖中单独安装 `webdiff`。

## 引入方式

| 语言 | 引入 |
| --- | --- |
| JavaScript / TypeScript | `const webdiff = require('webdiff')` 或 `import webdiff from 'webdiff'` |
| Python 3 | `import webdiff` |
| Shell | `source "$WEBDIFF_HOME/webdiff.sh"`（脚本开头加载一次） |

## 进程环境变量

| 变量 | 说明 |
| --- | --- |
| `WEBDIFF_HOME` | 脚本工作目录绝对路径；内含 `webdiff.js`、`webdiff.sh`、`node_modules` 等 |
| `WEBDIFF_CONTEXT` | 本次执行的 `webdiff-context.json` 绝对路径 |
| `WEBDIFF_SHELL_SDK` | `webdiff.sh` 的绝对路径 |

任务「运行环境 → 环境变量」中的键值对会 **同时** 注入进程环境（Shell 可直接 `$VAR` 读取）并写入 `webdiff-context.json` 的 `env` 字段。

---

## JavaScript / TypeScript

### `webdiff.env(name)`

读取环境变量。**优先** 任务配置（`webdiff-context.json` → `env`），其次 `process.env`。空字符串视为 `undefined`。

```javascript
const apiUrl = webdiff.env('API_URL');
const retry = Number(webdiff.env('RETRY') || 3);
```

### `webdiff.envUrl(name, defaultValue?)`

在 `env` 基础上规范为 http(s) URL：已有 scheme 则原样返回；以 `//` 开头补 `https:`；域名或 IP 形式补 `https://`。

```javascript
const url = webdiff.envUrl('API_URL', 'https://httpbin.org/get');
```

### `webdiff.global(name)`

读取「任务列表 → 全局变量」中配置的变量（与任务级环境变量无关）。

```javascript
const token = webdiff.global('API_TOKEN') || '';
```

### `webdiff.cookiePlus(alias)`

按 **别名** 返回 Cookie Plus 绑定解析出的 Cookie 数组（Playwright Cookie 格式）。未绑定或解析失败时返回 `[]`。

```javascript
const cookies = webdiff.cookiePlus('my_site');
const cookieHeader = cookies.map((c) => `${c.name}=${c.value}`).join('; ');
```

### `webdiff.localStorage(alias)`

按别名返回 LocalStorage 键值对象。仅当绑定项勾选 **同步 LocalStorage** 时有数据；否则为 `{}`。

```javascript
const ls = webdiff.localStorage('my_site');
const token = ls['auth_token'] || ls.token;
```

### `webdiff.notify(title, content, options?)`

脚本内 **主动发送通知**，与任务「频率 → 启用通知 / 通知渠道」**无关**；也不受触发规则约束。返回 `Promise<boolean>`。

| 参数 | 说明 |
| --- | --- |
| `{ channel: '渠道名称' }` | 名称须与 [通知渠道](../features/notify-channel.md) 中 **完全一致** 且已启用 |
| `{ alias: '别名' }` | 别名须已在「运行环境 → 通知别名」中绑定到具体渠道 |

本地通知使用系统保留名 **`本地通知`**（`webdiff.LOCAL_NOTIFY_CHANNEL_NAME`）。

```javascript
await webdiff.notify('库存告警', 'SKU-001 库存 < 10', { channel: '我的钉钉' });
await webdiff.notify('库存告警', 'SKU-001 库存 < 10', { alias: 'alert' });
await webdiff.notify('脚本提醒', '执行完成', {
  channel: webdiff.LOCAL_NOTIFY_CHANNEL_NAME,
});
```

::: tip 通知标记不会进入快照
`notify` 会向 stdout 写入内部标记行，执行器会 **自动剥离** 后再写入执行记录，一般不影响「输出内容变化」类规则。
:::

### 任务编排

与 UI「启用 / 停用 / 立即执行」一致。`enableTask` / `disableTask` / `runTask` 在 **本脚本执行结束后** 处理；`isTaskEnabled` / `listTasks` 为 **同步查询**。

| API | 说明 |
| --- | --- |
| `webdiff.enableTask(nameOrId)` | 启用指定任务 |
| `webdiff.disableTask(nameOrId)` | 停用指定任务 |
| `webdiff.runTask(nameOrId)` | 立即执行指定任务（不等待完成；禁止触发自身） |
| `webdiff.isTaskEnabled(nameOrId)` | 查询任务是否已启用，返回 `boolean` |
| `webdiff.listTasks(options?)` | 分页查询任务列表，返回 `{ list, total, current, pageSize }` |

```javascript
webdiff.enableTask('每日签到');
webdiff.runTask('HTTP 健康检查');

const page = webdiff.listTasks({ page: 1, pageSize: 20, keyword: '监控' });
console.log('共', page.total, '条');
```

::: warning 试运行限制
「运行脚本」试运行 **不会** 真正执行 `enableTask` / `disableTask` / `runTask`；`isTaskEnabled` / `listTasks` 在试运行时会报错。
:::

### 完整示例

```javascript
const webdiff = require('webdiff');
const axios = require('axios'); // 需在任务「npm 依赖」中声明并安装

async function main() {
  const alias = webdiff.env('CP_ALIAS') || 'my_site';
  const url = webdiff.envUrl('API_URL', 'https://httpbin.org/cookies');
  const cookies = webdiff.cookiePlus(alias);

  const res = await axios.get(url, {
    headers: {
      Cookie: cookies.map((c) => `${c.name}=${c.value}`).join('; '),
    },
    timeout: 15000,
    validateStatus: () => true,
  });

  console.log('HTTP', res.status);
  console.log(JSON.stringify(res.data).slice(0, 500));

  if (res.status >= 400) {
    await webdiff.notify('接口异常', `GET ${url} → ${res.status}`, { channel: '我的钉钉' });
    process.exit(1);
  }
}

main().catch((err) => {
  console.error(err.message || err);
  process.exit(1);
});
```

---

## Python 3

文件头建议 `#!/usr/bin/env python3`，使用 `import webdiff` 引入 SDK。`stdout` 写入执行快照。

| API | 说明 |
| --- | --- |
| `webdiff.env(name)` | 读取任务环境变量；未配置则回退 `os.environ` |
| `webdiff.env_url(name, default=None)` | 规范为 http(s) URL |
| `webdiff.global_var(name)` | 读取全局变量（Python 中 `global` 为保留字） |
| `webdiff.cookie_plus(alias)` | 读取 Cookie Plus 绑定，返回 `list[dict]` |
| `webdiff.local_storage(alias)` | 读取 LocalStorage 字典 |
| `webdiff.notify(title, content, channel=None, alias=None)` | 发送通知，`channel` 与 `alias` 二选一 |
| `webdiff.enable_task(name_or_id)` | 启用任务 |
| `webdiff.disable_task(name_or_id)` | 停用任务 |
| `webdiff.run_task(name_or_id)` | 立即执行任务 |
| `webdiff.is_task_enabled(name_or_id)` | 查询是否已启用 |
| `webdiff.list_tasks(page=1, page_size=10, **options)` | 分页查询任务列表 |

```python
import webdiff

api_url = webdiff.env_url('API_URL', 'https://httpbin.org/get')
cookies = webdiff.cookie_plus('my_site')
webdiff.notify('告警', 'done', channel='我的钉钉')
```

pip 依赖（如 `requests`）需在「pip 依赖」步骤填写并安装。

---

## Shell

类 Unix 环境使用 bash 执行；Windows 使用 cmd。执行前须加载 SDK：

```bash
source "$WEBDIFF_HOME/webdiff.sh"
```

| 函数 | 说明 |
| --- | --- |
| `webdiff_env NAME` | 读取环境变量（优先 context，其次 shell 变量） |
| `webdiff_global NAME` | 读取全局变量 |
| `webdiff_cookie_file ALIAS` | 返回 Netscape cookies.txt 路径，供 `curl -b` 使用 |
| `webdiff_cookie_json ALIAS` | 返回含 cookies 与 localStorage 的 JSON 文件路径 |
| `webdiff_notify "标题" "内容" [--channel "渠道"] [--alias "别名"]` | 发送通知 |
| `webdiff_enable_task TARGET` | 启用任务 |
| `webdiff_disable_task TARGET` | 停用任务 |
| `webdiff_run_task TARGET` | 立即执行任务 |
| `webdiff_is_task_enabled TARGET` | 查询是否已启用（退出码 0=已启用） |
| `webdiff_list_tasks [--page N] [--page-size N] [--keyword KW]` | 分页查询，stdout 输出 JSON |

```bash
#!/bin/bash
set -euo pipefail
source "$WEBDIFF_HOME/webdiff.sh"

URL="$(webdiff_env CHECK_URL)"
URL="${URL:-https://httpbin.org/status/200}"

STATUS="$(curl -sS -o /dev/null -w '%{http_code}' --max-time 15 "$URL")"
echo "GET $URL => HTTP $STATUS"

if [[ "$STATUS" -ge 400 ]]; then
  webdiff_notify "HTTP 异常" "请求 $URL 返回 $STATUS" --channel "我的钉钉"
  exit 1
fi
```

::: warning 依赖 python3 或 node
`webdiff_notify` 及任务编排函数内部需 **python3** 或 **node** 与执行器通信；一般客户端环境已具备其一。
:::

---

## 脚本内通知 vs 任务级通知

| | 脚本内 `webdiff.notify` | 任务级通知（触发规则 + 频率步骤） |
| --- | --- | --- |
| 触发方式 | 脚本代码主动调用 | 规则匹配（如输出变化、执行异常） |
| 是否依赖「启用通知」 | 不依赖 | 须在频率步骤开启并选渠道 |
| 是否依赖触发规则 | 不依赖 | 须配置至少一条规则 |
| 渠道指定 | `channel` 或 `alias` | 频率步骤中勾选的通知渠道 |

两套机制 **可同时使用**：例如平时靠规则监控输出变化，脚本内在检测到严重错误时立即 `notify`。

## 与运行环境的对应关系

```
运行环境配置                    SDK 读取方式
─────────────────────────────────────────────────────
环境变量 NAME=值        →  webdiff.env('NAME') / webdiff_env NAME / $NAME
全局变量 TOKEN          →  webdiff.global('TOKEN') / webdiff_global TOKEN
Cookie Plus 别名 xxx    →  webdiff.cookiePlus('xxx') / webdiff_cookie_file xxx
  └ 同步 LocalStorage   →  webdiff.localStorage('xxx') / webdiff_cookie_json + jq
通知别名 alert → 渠道   →  webdiff.notify(..., { alias: 'alert' })
通知渠道「我的钉钉」    →  webdiff.notify(..., { channel: '我的钉钉' })
```

::: warning 错误处理
建议在 JS/TS 入口使用 `main().catch(...)` 并 `process.exit(1)`；Shell 使用 `set -e` 或显式判断退出码，以便「执行异常」规则正确识别失败。
:::

## 相关文档

- [自定义脚本](../features/task-script.md) — 创建步骤、依赖安装与触发规则
- [cookie plus 账号](../features/cookie-plus.md) — 登录态绑定
- [通知渠道](../features/notify-channel.md) — 渠道名称与别名配置
