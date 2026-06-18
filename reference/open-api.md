# 开放接口

网页侦探提供 **开放 API**，允许通过 HTTP 远程管理监控任务、客户端、通知渠道等资源。
 
## 服务端地址

所有接口统一使用以下基础地址：

```
https://webdiff.perk-net.com/api
```

下文路径均相对于该地址。示例：

```
https://webdiff.perk-net.com/api/open/task/stats
```

## 准备工作

1. 在客户端打开 **我的 → 开放接口**，开启「开放接口」开关。
2. 创建 **AccessKey / SecretKey** 密钥对；`SecretKey` 仅在创建时展示一次，请妥善保存。
3. 可选：开启 **IP 白名单**，仅允许名单内 IP 换取访问令牌（多个 IP 用英文分号 `;` 分隔）。

## 认证流程

开放 API 采用 **AccessKey + SecretKey 换取短期 Token** 的方式认证。

### 1. 获取访问令牌

**POST** `https://webdiff.perk-net.com/api/common/openApi/getAccessKey`

**请求 Header**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| Content-Type | string | 是 | 固定为 `application/json` |

**请求 Body（JSON）**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| accessKey | string | 是 | AccessKey |
| secretKey | string | 是 | SecretKey |

**请求示例**

```json
{
  "accessKey": "pk_xxxxxxxxxxxxxxxx",
  "secretKey": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}
```

**响应 `data` 字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| token | string | 后续请求的身份凭证，默认有效期 **7200 秒（2 小时）** |
| expiresIn | number | Token 有效时长（秒） |
| nickName | string | 用户昵称 |

**响应示例**

```json
{
  "code": 200,
  "msg": "",
  "data": {
    "token": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "expiresIn": 7200,
    "nickName": "用户昵称"
  }
}
```

若账号已有未过期的 Token 且剩余时间 ≥ 300 秒，会复用并续期。

### 2. 携带 Token 调用接口

在后续请求的 Header 中传入 Token（支持 Header 或 Cookie 读取）：

```http
GET https://webdiff.perk-net.com/api/open/user/userInfo
token: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

### 3. 统一响应格式

所有接口返回 JSON：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | number | `200` 表示成功，其它为错误码 |
| msg | string | 错误描述（成功时通常为空） |
| data | object / array / null | 业务数据；无数据时为 `null` |

### 4. 分页请求与响应

分页接口（任务列表、客户端列表等）使用统一的 `PageDto` 请求体与 `PageVo` 响应结构。

**分页请求 Body**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| current | number | 否 | 页码，从 1 开始，默认 `1` |
| pageSize | number | 否 | 每页条数，默认 `10` |
| params | object | 否 | 查询条件，各接口字段见下文 |

**分页响应 `data` 字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| pageNum | number | 当前页码 |
| pageSize | number | 每页条数 |
| total | number | 总记录数 |
| pages | number | 总页数 |
| list | array | 当前页数据列表 |

---

## 用户

### 获取当前用户信息

**GET** `https://webdiff.perk-net.com/api/open/user/userInfo`

**请求参数**：无

**响应 `data` 字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | number | 用户 ID |
| uid | string | 全局唯一用户标识，格式 `U` + 8 位大写字母数字 |
| userName | string | 用户昵称 |
| headImgUrl | string | 头像 URL |
| memberLevel | number | 当前会员等级：`0` 免费、`1` 会员、`2` 高级会员、`3` 企业会员；过期时为 `0` |
| memberLevelLabel | string | 会员等级名称 |
| memberExpireTime | string | 会员过期时间，格式 `yyyy-MM-dd HH:mm:ss`；未开通时为 `null` |
| taskLimit | number | 任务数量上限；`-1` 表示无限 |
| maxClients | number | 允许同时在线的客户端数量上限；`-1` 表示无限 |
| globalVarLimit | number | 全局变量数量上限；`-1` 表示无限 |

**响应示例**

```json
{
  "code": 200,
  "msg": "",
  "data": {
    "id": 10001,
    "uid": "UAB12CD34",
    "userName": "张三",
    "headImgUrl": "https://example.com/avatar.png",
    "memberLevel": 0,
    "memberLevelLabel": "免费用户",
    "memberExpireTime": null,
    "taskLimit": 10,
    "maxClients": 2,
    "globalVarLimit": 5
  }
}
```

---

## 监控任务

::: info 任务执行说明
「立即执行」仅向任务所属的 **在线客户端** 下发指令；任务实际运行在客户端本地，服务端不直接执行监控逻辑。
:::

### 任务列表（分页）

**POST** `https://webdiff.perk-net.com/api/open/task/list`

**请求 Body**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| current | number | 否 | 页码，默认 `1` |
| pageSize | number | 否 | 每页条数，默认 `10` |
| params | object | 否 | 筛选条件 |
| params.keyword | string | 否 | 关键字，按任务名称 / 网址模糊匹配 |
| params.status | number | 否 | 启用状态：`1` 运行中 / `0` 已停用；为空表示全部 |
| params.taskType | string | 否 | 任务类型：`WEBSITE` / `HTTP` / `RSS` / `DOMAIN_EXPIRY` / `SSL_CERT` / `PING` / `SCRIPT`；为空表示全部 |
| params.clientDeviceId | number | 否 | 运行客户端筛选：`null` 全部、`0` 未分配（主客户端执行）、`-1` 全节点、`>0` 指定客户端设备行 ID |

**响应 `data.list[]` 字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | number | 任务 ID |
| name | string | 任务名称 |
| clientDeviceId | number | 归属客户端设备 ID（兼容旧版，已废弃） |
| runClientMode | string | 运行方案：`SINGLE` 单节点 / `ALL` 全节点 / `SPECIFIED` 指定节点 |
| runClientDeviceIds | number[] | 指定节点时的客户端设备行 ID 列表 |
| runClientLabel | string | 运行客户端展示文案 |
| clientDeviceName | string | 客户端名称（已废弃，请使用 `runClientLabel`） |
| taskType | string | 任务类型 |
| url | string | 监控目标 URL |
| cronExpression | string | Cron 表达式（分 时 日 月 周） |
| enabled | number | 是否启用：`0` 停用 / `1` 启用 |
| screenshotEnabled | number | 是否启用截图 |
| notifyEnabled | number | 是否启用通知 |
| notifyChannelIds | string | 绑定的通知渠道 ID（逗号分隔字符串） |
| notifyChannelCount | number | 绑定的通知渠道数量 |
| elementCount | number | 监控元素数量 |
| createTime | string | 创建时间 |
| updateTime | string | 更新时间 |

### 任务统计

**GET** `https://webdiff.perk-net.com/api/open/task/stats`

**请求参数**：无

**响应 `data` 字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| totalCount | number | 当前用户全部任务数 |
| runningCount | number | 当前用户运行中任务数 |

### 任务详情

**GET** `https://webdiff.perk-net.com/api/open/task/detail/{id}`

**路径参数**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| id | number | 是 | 任务 ID |

**响应 `data` 字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | number | 任务 ID |
| userId | number | 任务所属用户 ID |
| clientDeviceId | number | 归属客户端设备 ID（兼容旧版） |
| runClientMode | string | 运行方案 |
| runClientDeviceIds | number[] | 指定节点时的设备行 ID 列表 |
| name | string | 任务名称 |
| taskType | string | 任务类型 |
| taskConfig | string | 任务类型专属配置（JSON 字符串） |
| url | string | 监控目标 URL |
| cookies | string | Cookie 配置（JSON 字符串） |
| cookiePlusAccountLabels | object | Cookie Plus 账号 `clientId` → 中文名称映射 |
| userAgent | string | 自定义 User-Agent |
| cronExpression | string | Cron 表达式 |
| randomDelayEnabled | number | 是否启用随机延迟：`0` 否 / `1` 是 |
| randomDelayMinSeconds | number | 随机延迟最小秒数 |
| randomDelayMaxSeconds | number | 随机延迟最大秒数 |
| enabled | number | 是否启用 |
| screenshotEnabled | number | 是否启用截图 |
| screenshotFullPage | number | 是否启用全页长截图 |
| screenshotMaxScroll | number | 长截图最大滚动次数 |
| extractScrollEnabled | number | 提取元素前是否滚动以触发懒加载 |
| extractMaxScroll | number | 元素提取前最大滚动次数 |
| notifyEnabled | number | 是否启用通知 |
| notifyChannelIds | number[] | 绑定的通知渠道 ID 列表 |
| notifyChannelNames | string[] | 绑定的通知渠道名称列表 |
| notifyChannelTypes | string[] | 绑定的通知渠道类型列表 |
| notifyTemplateId | number | 通知模板 ID；`null` 表示自动匹配内置默认 |
| notifyTemplateName | string | 通知模板名称 |
| aiRule | string | AI 生成的比对规则 |
| elements | array | 监控元素列表，见下表 |
| createTime | string | 创建时间 |
| updateTime | string | 更新时间 |

**`elements[]` 字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | number | 元素 ID |
| selector | string | CSS / XPath 选择器 |
| selectorType | string | 选择器类型 |
| extractType | string | 提取模式：`HTML` / `TEXT` |
| label | string | 元素标签 |
| sortOrder | number | 排序序号 |

::: tip taskConfig 说明
`taskConfig` 为 JSON 字符串，结构因任务类型而异。详见 [监控任务类型](../features/task-types.md) 各类型说明页。
:::

### 创建监控任务

**POST** `https://webdiff.perk-net.com/api/open/task/create`

**请求 Body**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| name | string | 是 | 任务名称 |
| url | string | 是 | 监控目标 URL |
| clientDeviceId | number | 否 | 归属客户端设备 ID（兼容旧版） |
| runClientMode | string | 否 | 运行方案：`SINGLE` / `ALL` / `SPECIFIED`，默认 `SINGLE` |
| runClientDeviceIds | number[] | 否 | 指定节点时的客户端设备行 ID 列表 |
| taskType | string | 否 | 任务类型，默认 `WEBSITE` |
| taskConfig | string | 否 | 任务类型专属配置（JSON 字符串） |
| cookies | string | 否 | Cookie 配置（JSON 字符串） |
| userAgent | string | 否 | 自定义 User-Agent，最长 500 字符 |
| cronExpression | string | 否 | Cron 表达式（5 字段：分 时 日 月 周），如 `0 * * * *` |
| randomDelayEnabled | number | 否 | 是否启用随机延迟 |
| randomDelayMinSeconds | number | 否 | 随机延迟最小秒数，≥ 0 |
| randomDelayMaxSeconds | number | 否 | 随机延迟最大秒数，≥ 0 |
| screenshotEnabled | number | 否 | 是否启用截图 |
| screenshotFullPage | number | 否 | 是否启用全页长截图 |
| screenshotMaxScroll | number | 否 | 长截图最大滚动次数，1–100 |
| extractScrollEnabled | number | 否 | 提取元素前是否滚动 |
| extractMaxScroll | number | 否 | 元素提取前最大滚动次数，1–100 |
| notifyEnabled | number | 否 | 是否启用通知 |
| notifyChannelIds | number[] | 否 | 绑定的通知渠道 ID 列表 |
| notifyTemplateId | number | 否 | 通知模板 ID |
| aiRule | string | 否 | AI 生成的比对规则 |
| enabled | number | 否 | 是否启用，默认 `1` |
| elements | array | 否 | 监控元素列表，最多 5 个，见下表 |

**`elements[]` 字段**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| selector | string | 是 | CSS / XPath 选择器 |
| selectorType | string | 是 | 选择器类型 |
| extractType | string | 否 | 提取模式：`HTML`（默认）/ `TEXT` |
| label | string | 否 | 元素标签 |
| sortOrder | number | 否 | 排序序号 |

**响应 `data`**：number，新创建的任务 ID

### 更新监控任务

**PUT** `https://webdiff.perk-net.com/api/open/task/update`

**请求 Body**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| id | number | 是 | 任务 ID |
| name | string | 否 | 任务名称 |
| url | string | 否 | 监控目标 URL |
| clientDeviceId | number | 否 | 归属客户端设备 ID（兼容旧版） |
| runClientMode | string | 否 | 运行方案 |
| runClientDeviceIds | number[] | 否 | 指定节点时的客户端设备行 ID 列表 |
| taskType | string | 否 | 任务类型 |
| taskConfig | string | 否 | 任务类型专属配置（JSON 字符串） |
| cookies | string | 否 | Cookie 配置（JSON 字符串） |
| userAgent | string | 否 | 自定义 User-Agent |
| cronExpression | string | 否 | Cron 表达式 |
| randomDelayEnabled | number | 否 | 是否启用随机延迟 |
| randomDelayMinSeconds | number | 否 | 随机延迟最小秒数 |
| randomDelayMaxSeconds | number | 否 | 随机延迟最大秒数 |
| screenshotEnabled | number | 否 | 是否启用截图 |
| screenshotFullPage | number | 否 | 是否启用全页长截图 |
| screenshotMaxScroll | number | 否 | 长截图最大滚动次数 |
| extractScrollEnabled | number | 否 | 提取元素前是否滚动 |
| extractMaxScroll | number | 否 | 元素提取前最大滚动次数 |
| notifyEnabled | number | 否 | 是否启用通知 |
| notifyChannelIds | number[] | 否 | 绑定的通知渠道 ID 列表 |
| notifyTemplateId | number | 否 | 通知模板 ID |
| aiRule | string | 否 | AI 生成的比对规则 |
| elements | array | 否 | 监控元素列表，最多 5 个 |

**响应 `data`**：`null`

### 删除监控任务

**DELETE** `https://webdiff.perk-net.com/api/open/task/delete/{id}`

**路径参数**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| id | number | 是 | 任务 ID |

**响应 `data`**：`null`

### 批量启用 / 停用任务

**PUT** `https://webdiff.perk-net.com/api/open/task/batch/enabled`

**请求 Body**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| ids | number[] | 是 | 任务 ID 列表，最多 100 个 |
| enabled | number | 是 | 目标状态：`1` 启用 / `0` 停用 |

**响应 `data` 字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| successCount | number | 成功数量 |
| failureCount | number | 失败数量 |
| successIds | number[] | 成功的任务 ID |
| failures | array | 失败明细 |
| failures[].id | number | 失败的任务 ID |
| failures[].message | string | 失败原因 |
| successDetails | array | 批量启用时返回的成功任务详情（供客户端注册调度）；停用时为 `[]` |

### 批量设置运行客户端

**PUT** `https://webdiff.perk-net.com/api/open/task/batch/client-device`

**请求 Body**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| ids | number[] | 是 | 任务 ID 列表，最多 100 个 |
| runClientMode | string | 否 | 运行方案：`SINGLE` / `ALL` / `SPECIFIED` |
| runClientDeviceIds | number[] | 否 | 指定节点时的客户端设备行 ID 列表 |
| clientDeviceId | number | 否 | 目标运行客户端设备行 ID（兼容旧版，等效于 `SPECIFIED` + 单设备） |

**响应 `data`**：同「批量启用 / 停用任务」

### 立即执行任务

**POST** `https://webdiff.perk-net.com/api/open/task/run-now/{id}`

**路径参数**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| id | number | 是 | 任务 ID |

**响应 `data` 字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| dispatched | boolean | 是否已成功下发到目标客户端 |
| targetOnline | boolean | 目标客户端当前是否在线 |
| targetDeviceId | string | 目标客户端 deviceId |
| targetDeviceName | string | 目标客户端展示名称 |
| message | string | 提示文案 |

---

## 客户端设备

### 注册 / 上报客户端实例

**POST** `https://webdiff.perk-net.com/api/open/client/device/register`

**请求 Body**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| deviceId | string | 是 | 客户端实例唯一标识（本地持久化生成的 UUID），最长 64 字符 |
| name | string | 否 | 客户端名称，最长 100 字符 |
| os | string | 否 | 客户端平台，如 `win32-x64` / `cli`，最长 50 字符 |
| version | string | 否 | 客户端版本，最长 50 字符 |

**响应 `data` 字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | number | 设备行 ID |
| deviceId | string | 客户端实例唯一标识 |
| name | string | 客户端名称 |
| clientOs | string | 客户端平台 |
| clientVersion | string | 客户端版本 |
| isPrimary | number | 是否主客户端：`0` 否 / `1` 是 |
| online | boolean | 是否在线 |
| taskCount | number | 归属该客户端的任务数量 |
| lastSeen | string | 最后活跃时间 |
| createTime | string | 创建时间 |

### 客户端列表（轻量）

**GET** `https://webdiff.perk-net.com/api/open/client/device/list`

**请求参数**：无

**响应 `data[]` 字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | number | 设备行 ID |
| deviceId | string | 客户端实例唯一标识 |
| name | string | 客户端名称 |
| online | boolean | 是否在线 |
| createTime | string | 创建时间 |
| sessionStartedAt | string | 本次会话启动时间（SSE 连接建立时刻；离线时为 `null`） |

### 客户端列表（分页）

**POST** `https://webdiff.perk-net.com/api/open/client/device/list/page`

**请求 Body**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| current | number | 否 | 页码，默认 `1` |
| pageSize | number | 否 | 每页条数，默认 `10` |
| params | object | 否 | 查询条件（预留扩展，当前为空对象） |

**响应 `data.list[]` 字段**：同「注册 / 上报客户端实例」响应

### 客户端统计

**GET** `https://webdiff.perk-net.com/api/open/client/device/stats`

**请求参数**：无

**响应 `data` 字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| totalCount | number | 在册客户端总数 |
| onlineCount | number | 当前在线客户端数 |

### 重命名客户端

**PUT** `https://webdiff.perk-net.com/api/open/client/device/rename`

**请求 Body**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| deviceId | string | 是 | 客户端实例唯一标识 |
| name | string | 是 | 新名称，最长 100 字符 |

**响应 `data`**：`null`

### 删除客户端

**DELETE** `https://webdiff.perk-net.com/api/open/client/device/delete/{deviceId}`

**路径参数**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| deviceId | string | 是 | 客户端实例唯一标识 |

**响应 `data`**：`null`

### 设为主客户端

**PUT** `https://webdiff.perk-net.com/api/open/client/device/primary/{deviceId}`

**路径参数**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| deviceId | string | 是 | 客户端实例唯一标识 |

**响应 `data`**：`null`

### 踢客户端下线

**POST** `https://webdiff.perk-net.com/api/open/client/device/kick/{deviceId}`

**路径参数**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| deviceId | string | 是 | 客户端实例唯一标识 |

**响应 `data`**：`null`

---

## 通知渠道

### 通知渠道列表

**GET** `https://webdiff.perk-net.com/api/open/notifyChannel/list`

**请求参数**：无

**响应 `data[]` 字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | number | 渠道 ID |
| name | string | 渠道名称 |
| type | string | 渠道类型：`local` / `pushplus` / `dingtalk` / `wechatwork` / `feishu` / `webhook` / `email` |
| config | string | 渠道配置（JSON 字符串） |
| enabled | number | 是否启用：`0` 停用 / `1` 启用 |
| createTime | string | 创建时间 |
| updateTime | string | 更新时间 |

::: tip config 配置说明
各渠道 `config` 字段结构详见 [通知渠道](../features/notify-channel.md) 及各渠道说明页。
:::

### 新增通知渠道

**POST** `https://webdiff.perk-net.com/api/open/notifyChannel/create`

**请求 Body**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| name | string | 是 | 渠道名称，最长 100 字符 |
| type | string | 是 | 渠道类型：`pushplus` / `dingtalk` / `wechatwork` / `feishu` / `webhook` / `email` |
| config | string | 否 | 渠道配置（JSON 字符串） |
| enabled | number | 否 | 是否启用 |

**响应 `data`**：number，新创建的渠道 ID

### 更新通知渠道

**PUT** `https://webdiff.perk-net.com/api/open/notifyChannel/update`

**请求 Body**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| id | number | 是 | 渠道 ID |
| name | string | 是 | 渠道名称 |
| type | string | 是 | 渠道类型 |
| config | string | 否 | 渠道配置（JSON 字符串） |
| enabled | number | 否 | 是否启用 |

**响应 `data`**：`null`

### 删除通知渠道

**DELETE** `https://webdiff.perk-net.com/api/open/notifyChannel/delete/{id}`

**路径参数**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| id | number | 是 | 渠道 ID |

**响应 `data`**：`null`

### 构造测试通知请求

**POST** `https://webdiff.perk-net.com/api/open/notifyChannel/test`

后端将通知消息渲染成「待发送的 HTTP 请求」返回，调用方根据返回内容自行发起请求。

**请求 Body**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| type | string | 是 | 渠道类型 |
| config | string | 否 | 渠道配置（JSON 字符串） |

**响应 `data` 字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| channelId | number | 渠道 ID（测试时可为 `null`） |
| channelType | string | 渠道类型 |
| channelName | string | 渠道展示名称 |
| method | string | 请求方法：`POST` / `GET` |
| url | string | 完整请求 URL |
| headers | object | 请求头（键值对） |
| body | string | 请求体（JSON 字符串）；GET 请求可能为空 |
| successCheck | string | 响应成功判断方式 |
| errorField | string | 识别错误消息时使用的 JSON 字段 |
| localTitle | string | 本地桌面通知标题（仅 `local` 类型） |
| localBody | string | 本地桌面通知正文（仅 `local` 类型） |

---

## 通知模板

### 通知模板列表

**GET** `https://webdiff.perk-net.com/api/open/notifyTemplate/list`

**Query 参数**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| taskType | string | 否 | 按任务类型过滤 |
| ruleType | string | 否 | 按规则类型过滤 |

**响应 `data[]` 字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | number | 模板 ID |
| userId | number | 用户 ID；`0` 表示系统内置 |
| code | string | 内置模板 code；用户自定义为空 |
| name | string | 模板名称 |
| taskType | string | 适用任务类型 |
| ruleType | string | 适用规则类型 |
| titleTpl | string | 标题模板 |
| bodyTpl | string | 正文模板（Markdown） |
| isBuiltin | number | 是否系统内置：`1` 是（只读） |
| isDefault | number | 是否为该 `(taskType, ruleType)` 组合的默认模板 |
| enabled | number | 是否启用 |
| createTime | string | 创建时间 |
| updateTime | string | 更新时间 |

### 通知模板详情

**GET** `https://webdiff.perk-net.com/api/open/notifyTemplate/detail/{id}`

**路径参数**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| id | number | 是 | 模板 ID |

**响应 `data`**：同列表单项字段

### 新增通知模板

**POST** `https://webdiff.perk-net.com/api/open/notifyTemplate/create`

**请求 Body**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| name | string | 是 | 模板名称，最长 120 字符 |
| titleTpl | string | 是 | 标题模板，最长 255 字符 |
| bodyTpl | string | 是 | 正文模板（Markdown） |
| taskType | string | 否 | 适用任务类型：`*` / `WEBSITE` / `HTTP` / `RSS` / `DOMAIN_EXPIRY` / `SSL_CERT` / `PING` |
| ruleType | string | 否 | 适用规则类型：`*` / `changed` / `days_before_expiry` 等 |
| enabled | number | 否 | 是否启用 |

**响应 `data`**：number，新创建的模板 ID

### 更新通知模板

**PUT** `https://webdiff.perk-net.com/api/open/notifyTemplate/update`

**请求 Body**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| id | number | 是 | 模板 ID |
| name | string | 是 | 模板名称 |
| titleTpl | string | 是 | 标题模板 |
| bodyTpl | string | 是 | 正文模板 |
| taskType | string | 否 | 适用任务类型 |
| ruleType | string | 否 | 适用规则类型 |
| enabled | number | 否 | 是否启用 |

**响应 `data`**：`null`

### 删除通知模板

**DELETE** `https://webdiff.perk-net.com/api/open/notifyTemplate/delete/{id}`

**路径参数**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| id | number | 是 | 模板 ID |

**响应 `data`**：`null`

### 预览模板渲染结果

**POST** `https://webdiff.perk-net.com/api/open/notifyTemplate/preview`

**请求 Body**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| templateId | number | 否 | 模板 ID（与 `titleTpl` / `bodyTpl` 二选一） |
| titleTpl | string | 否 | 标题模板（实时预览未保存内容时使用） |
| bodyTpl | string | 否 | 正文模板（实时预览未保存内容时使用） |
| taskType | string | 否 | 任务类型，决定 mock 变量集 |
| ruleType | string | 否 | 规则类型，决定 mock 变量集 |

**响应 `data` 字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| title | string | 渲染后的标题（纯文本） |
| bodyMarkdown | string | 渲染后的正文（Markdown 源） |
| bodyHtml | string | 渲染后的正文（HTML） |
| bodyText | string | 渲染后的正文（纯文本兜底） |

---

## Cookie Plus 账号

### Cookie Plus 账号列表

**GET** `https://webdiff.perk-net.com/api/open/cookiePlusAccount/list`

**请求参数**：无

**响应 `data[]` 字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| clientId | string | 客户端账号 ID（跨设备一致） |
| label | string | 账号名称 |
| accessKey | string | Access Key |
| secretKey | string | Secret Key |
| createTime | string | 创建时间 |
| updateTime | string | 更新时间 |

### 新增或更新 Cookie Plus 账号

**PUT** `https://webdiff.perk-net.com/api/open/cookiePlusAccount/upsert`

**请求 Body**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| clientId | string | 是 | 客户端账号 ID，最长 64 字符 |
| accessKey | string | 是 | Access Key，最长 200 字符 |
| secretKey | string | 是 | Secret Key，最长 500 字符 |
| label | string | 否 | 账号名称，最长 200 字符 |

**响应 `data`**：`null`

### 删除 Cookie Plus 账号

**DELETE** `https://webdiff.perk-net.com/api/open/cookiePlusAccount/delete/{clientId}`

**路径参数**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| clientId | string | 是 | 客户端账号 ID |

**响应 `data`**：`null`

---

## 全局变量

### 全局变量列表

**GET** `https://webdiff.perk-net.com/api/open/global-var/list`

**请求参数**：无

**响应 `data[]` 字段**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | number | 变量 ID |
| name | string | 变量名称 |
| value | string | 变量值 |
| remark | string | 备注 |
| sort | number | 排序序号 |
| createTime | string | 创建时间 |
| updateTime | string | 更新时间 |

### 保存全局变量

**PUT** `https://webdiff.perk-net.com/api/open/global-var/save`

整表替换保存，传入的列表即为保存后的完整变量集。

**请求 Body**

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| vars | array | 是 | 全局变量列表 |
| vars[].name | string | 是 | 变量名称，最长 128 字符 |
| vars[].value | string | 否 | 变量值 |
| vars[].remark | string | 否 | 备注，最长 255 字符 |

**响应 `data[]`**：保存后的全局变量列表，字段同「全局变量列表」

---

## 调用示例

```bash
# 1. 换取 Token
TOKEN=$(curl -sS -X POST 'https://webdiff.perk-net.com/api/common/openApi/getAccessKey' \
  -H 'Content-Type: application/json' \
  -d '{"accessKey":"YOUR_AK","secretKey":"YOUR_SK"}' \
  | jq -r '.data.token')

# 2. 查询任务统计
curl -sS 'https://webdiff.perk-net.com/api/open/task/stats' \
  -H "token: $TOKEN"

# 3. 分页查询任务列表
curl -sS -X POST 'https://webdiff.perk-net.com/api/open/task/list' \
  -H "token: $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{"current":1,"pageSize":10,"params":{}}'
```

## 错误码说明

| 场景 | 典型响应 |
| --- | --- |
| AccessKey / SecretKey 无效 | `code` 非 200，未授权 |
| 开放接口未启用 | 未授权 |
| 账号已禁用 | 未授权 |
| IP 不在白名单 | 禁止访问（含请求方 IP） |
| Token 过期或缺失 | 未授权，需重新换取 Token |
| 参数校验失败 | `code` 非 200，`msg` 含具体字段错误信息 |
