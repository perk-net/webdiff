# 自定义 Webhook

将网页侦探的通知接入 **自建系统**（内部告警、低代码平台、二次转发等）。在 [通知渠道](./notify-channel.md) 中与其他渠道一并管理，创建任务时多选即可。客户端入口：**「我的」→ 通知渠道」→ 添加渠道 → 选择「自定义 Webhook」**（需高级会员）。

## 配置字段

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| 渠道名称 | ✅ | 自定义 |
| 请求 URL | ✅ | 完整的接收端 URL（http/https） |
| 请求方法 | ✅ | `POST` / `PUT` / `GET` |
| Content-Type | - | 默认 `application/json` |
| 自定义请求头 | - | JSON 格式，如 `{"Authorization":"Bearer xxx"}` |
| 请求体模板 | - | 留空则发送默认结构（见下文） |
| 超时 (秒) | - | 默认 10 秒 |

## 默认请求体

如果不配置「请求体模板」，网页侦探会按以下 JSON 结构发送：

```json
{
  "taskId": 12,
  "taskName": "京东 PS5 价格监控",
  "url": "https://item.jd.com/100012043978.html",
  "triggerTime": "2026-05-11T10:30:00+08:00",
  "summary": "价格变化：3899 → 3499",
  "changes": [
    {
      "selector": "//span[@class='price']",
      "oldValue": "3899",
      "newValue": "3499"
    }
  ],
  "executionId": 8821
}
```

## 自定义请求体模板

可以用变量模板自定义请求体，例如把它发往一个企业 IM 的接口：

```json
{
  "msg_type": "text",
  "content": {
    "text": "【网页侦探】{{taskName}} 已变化\n网址：{{url}}\n摘要：{{summary}}"
  }
}
```

### 可用变量

| 变量 | 类型 | 说明 |
| --- | --- | --- |
| <code v-pre>{{taskId}}</code> | number | 任务 ID |
| <code v-pre>{{taskName}}</code> | string | 任务名称 |
| <code v-pre>{{url}}</code> | string | 任务网址 |
| <code v-pre>{{triggerTime}}</code> | string | 触发时间 (ISO8601) |
| <code v-pre>{{summary}}</code> | string | 变化摘要 |
| <code v-pre>{{oldValue}}</code> | string | 旧值（仅单元素时） |
| <code v-pre>{{newValue}}</code> | string | 新值（仅单元素时） |
| <code v-pre>{{executionId}}</code> | number | 执行记录 ID |
| <code v-pre>{{changesJson}}</code> | string | 完整变化 JSON 字符串 |

## 自定义请求头

例如带签名 / Bearer Token：

```json
{
  "Authorization": "Bearer eyJxxxxxxxx",
  "X-Source": "WebDiff"
}
```

## 响应判断

- HTTP 状态码 `2xx` 视为发送成功
- 其他状态码视为失败，触发重试（默认 2 次）
- 响应体内容不会被检查，请确保接收端返回 2xx 即可

## 安全建议

- 请使用 **HTTPS**，避免请求中的敏感字段被窃听
- 接收端建议校验请求头中的密钥 / 签名，防止伪造请求
- 不要把数据库密码 / 长效 Token 等敏感字段放进 URL 查询参数

## 示例：接入 Slack

```text
URL：     https://hooks.slack.com/services/T0/B0/XX
方法：    POST
请求体：  {"text": "*{{taskName}}* changed → {{summary}} <{{url}}>"}
```

## 示例：接入飞书自定义机器人（非内置类型）

```text
URL：     https://open.feishu.cn/open-apis/bot/v2/hook/xxxxxxxxxx
方法：    POST
请求体：
{
  "msg_type": "text",
  "content": {"text": "[网页侦探] {{taskName}} 变化\n{{summary}}\n{{url}}"}
}
```

## 示例：接入企业内部告警系统

```text
URL：     https://alert.example.com/api/notify
方法：    POST
请求头：  {"X-API-Key": "your-key"}
请求体：
{
  "source": "webdiff",
  "level": "warning",
  "title": "{{taskName}}",
  "body": "{{summary}}",
  "url": "{{url}}"
}
```

## 调试技巧

- 用 [https://webhook.site](https://webhook.site) 拿一个临时 URL 看实际发送内容
- 检查「消息记录」里每次发送的完整请求体与响应体
