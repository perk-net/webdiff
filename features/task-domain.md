# 域名到期时间

通过 WHOIS 查询域名注册与到期信息，在距到期不足 N 天或到期信息变化时提醒续费。

## 适用场景

- 公司主域名续费提醒
- 多域名资产到期统一巡检
- WHOIS 信息异常变更告警

## 创建步骤

<ol class="feature-step-list">

<li>

### 选择域名到期类型

新建任务时选择「域名到期时间」。

<img src="/features/domain/step1.png" alt="选择域名到期类型" class="doc-screenshot" />

</li>

<li>

### 填写域名

输入要查询的根域名（如 `example.com`），无需带协议头。

<img src="/features/domain/step2.png" alt="填写域名" class="doc-screenshot" />

</li>

<li>

### 设置到期规则

常见配置：距到期不足 30/7 天提醒，或 WHOIS 到期时间发生变化时通知。

<img src="/features/domain/step3.png" alt="设置到期规则" class="doc-screenshot" />

</li>

<li>

### 配置检查频率

域名信息变化较慢，通常按天或周检查即可。

<img src="/features/domain/step4.png" alt="配置检查频率" class="doc-screenshot" />

</li>

<li>

### 通知与保存

绑定通知渠道后保存。通知可包含剩余天数、到期日期、注册商等变量。

<img src="/features/domain/step5.png" alt="通知与保存" class="doc-screenshot" />

</li>

</ol>

## 触发规则

距到期不足 N 天、到期时间变化、信息变化
