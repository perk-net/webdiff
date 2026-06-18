# 更新日志

按时间倒序记录 **网页侦探** 的版本更新。如需了解未来计划，欢迎在 [联系我们](contact.md) 中反馈建议。

<div class="changelog">

## <span><span class="version-badge latest">v1.2.3</span></span> <span class="version-date">· 2026-06-18</span>

<div class="changelog-section-title"><span class="tag new">NEW</span>网页内容监控</div>

- 新增 **前置操作**，可执行录制好的网页操作步骤，在监控前自动完成登录、导航等准备

<div class="changelog-section-title"><span class="tag new">NEW</span>自定义脚本</div>

- **[自定义脚本](../features/task-script.md)** 任务新增 **Python 3** 语言支持
- 脚本中可 **编排、操作其他任务** 的运行，实现任务间联动

<div class="changelog-section-title"><span class="tag new">NEW</span>全局变量</div>

- 新增 **全局变量**，可在创建任务参数中引用，统一管理共用配置

<div class="changelog-section-title"><span class="tag new">NEW</span>触发规则</div>

- 新增 **规则组** 功能，支持 AND / OR 等复杂规则逻辑组合

<div class="changelog-section-title"><span class="tag new">NEW</span>开放接口</div>

- 新增 **开放 API**，方便第三方程序对接与操控任务

<div class="changelog-section-title">体验优化</div>

- 任务执行频率支持 **随机延迟**，降低固定周期执行的可预测性

## <span><span class="version-badge">v1.2.1</span></span> <span class="version-date">· 2026-06-15</span>

<div class="changelog-section-title">体验优化</div>

- **HTTP 请求**与 **RSS 订阅**任务地址支持输入 `http` 协议
- 增加 **登录界面 Logo**

## <span><span class="version-badge">v1.2.0</span></span> <span class="version-date">· 2026-06-13</span>

<div class="changelog-section-title"><span class="tag new">NEW</span>运行客户端</div>

- 任务可 **指定运行客户端**，支持单节点、全节点与指定节点三种方案，灵活分配执行环境

<div class="changelog-section-title"><span class="tag new">NEW</span>自定义脚本任务</div>

- 新增 **[自定义脚本](../features/task-script.md)** 任务类型，支持 JavaScript / TypeScript / Shell 定时执行与触发规则

<div class="changelog-section-title">体验优化</div>

- 优化任务创建、列表与客户端管理等 **整体交互体验**

## <span><span class="version-badge">v1.1.2</span></span> <span class="version-date">· 2026-06-07</span>

<div class="changelog-section-title"><span class="tag new">NEW</span>Web 部署</div>

- 新增 **Web 部署方案**，支持 Docker 一键部署，功能与桌面客户端一致

<div class="changelog-section-title"><span class="tag new">NEW</span>桌面客户端</div>

- 支持 **Linux 桌面环境**，Windows / macOS / Linux 全平台覆盖

<div class="changelog-section-title"><span class="tag new">NEW</span>HTTP 请求任务</div>

- 支持 **cURL 导入**，快速从命令行请求创建监控任务

<div class="changelog-section-title"><span class="tag new">NEW</span>RSS 订阅任务</div>

- 新增 **Cookie 配置**，支持需登录的 RSS 源
- 优化标签选择交互

<div class="changelog-section-title"><span class="tag new">NEW</span>触发规则</div>

- 增加更多条件选项，告警判断更灵活

## <span><span class="version-badge">v1.1.1</span></span> <span class="version-date">· 2026-06-05</span>

<div class="changelog-section-title"><span class="tag new">NEW</span>桌面客户端</div>

- 新增 **开机自启动** 功能

<div class="changelog-section-title"><span class="tag fix">FIX</span>通知</div>

- 修复通知模板中 **链接跳转** 问题
- 补全 **pushplus** 通知渠道配置项

## <span><span class="version-badge">v1.1.0</span>正式版发布</span> <span class="version-date">· 2026-06-05</span>

🎉 **网页侦探正式版上线**，桌面端零代码本地监控，7×24 自动巡检，覆盖网页、接口、RSS、域名证书、Ping 等场景。

<div class="changelog-section-title"><span class="tag new">NEW</span>桌面客户端</div>

- **Windows / macOS** 双平台安装包
- 本地运行，任务、Cookie、执行快照默认保存在本机
- 微信扫码登录，会员权益云端同步

<div class="changelog-section-title"><span class="tag new">NEW</span>六大监控任务类型</div>

- [网站内容监控](../features/task-website.md)：真实浏览器渲染 + 可视化元素挑选
- [HTTP 请求](../features/task-http.md)：按响应码 / 响应体 / 内容变化告警
- [RSS 订阅](../features/task-rss.md)：跟踪博客、播客等源的新条目
- [域名到期（WHOIS）](../features/task-domain.md)：临期提醒
- [SSL 证书](../features/task-ssl.md)：到期 / 异常预警
- [Ping 检测](../features/task-ping.md)：主机连通性与平均延迟

<div class="changelog-section-title"><span class="tag new">NEW</span>网站任务能力</div>

- 内嵌 Chromium 浏览器，[点选元素](../features/element-picker.md) 自动生成 XPath / CSS
- 单任务最多并行监控 **20 个元素**
- 变更对比含 **HTML Diff**，可在执行记录中回溯

<div class="changelog-section-title"><span class="tag new">NEW</span>调度与通知</div>

- 灵活定时调度：常用频率选项 + [Cron 表达式](../features/cron.md)
- [多渠道通知](../features/notify-channel.md)：pushplus、本地桌面、钉钉 / 企业微信 / 飞书机器人、[自定义 Webhook](../features/notify-webhook.md)、邮件（高级会员）
- 通知支持 [自定义模板](../features/notify-template.md)，变量按任务类型动态展开

<div class="changelog-section-title"><span class="tag new">NEW</span>账号与登录态</div>

- 微信扫码登录，无需注册
- [cookie plus](../features/cookie-plus.md) 同步浏览器登录态，支持需登录页面监控

<div class="changelog-section-title"><span class="tag new">NEW</span>会员与配额</div>

- 4 档会员：**免费 / 会员 / 高级会员 / 企业会员**
- 免费版 10 个任务起步，详见 [会员与配额](membership.md)

<div class="changelog-section-title"><span class="tag new">NEW</span>记录与文档</div>

- 执行记录与消息记录可追溯（网站任务含 HTML Diff）
- 上线 **产品官网** 与 **帮助文档站**

</div>
 