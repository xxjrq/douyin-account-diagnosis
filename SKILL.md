---
name: douyin-account-diagnosis
description: 读取指定抖音账号的公开可见资料和作品样本，生成有证据、有时间范围、可解释的账号诊断与选题建议。用户要求分析抖音账号、对比近期作品、整理公开数据或制定今日选题时使用；不用于推断后台权重、限流原因或保证增长。
---

# 抖音账号诊断

只分析用户指定账号在已授权浏览器中公开可见的资料。输出样本范围、证据链接、可复算指标、问题优先级和选题建议。默认只读，不点赞、评论、私信、发布或修改账号。

## 输入

接受一个 JSON 对象：

```json
{
  "accountUrl": "https://www.douyin.com/user/...",
  "browserId": "已授权的 EasyBR 环境，可选",
  "sampleLimit": 10,
  "mode": "real",
  "capturedAt": "2026-09-08T10:00:00+08:00"
}
```

`sampleLimit` 默认 10，范围 3-30。没有 `browserId` 时可以读取用户已打开的普通浏览器；需要 EasyBR 时必须使用用户指定的环境，不得猜账号。

## 执行步骤

1. 打开账号公开页，记录页面 URL、采集时间和页面是否登录可见。
2. 只读取个人简介、公开作品列表和页面显示的互动字段；每个字段保留原始文本和来源 URL。
3. 采集最近 `sampleLimit` 个仍可见作品，去重并记录缺失字段，不用空值替代未知数据。
4. 使用 `scripts/diagnose.mjs` 计算样本内的发布频率、互动率和主题分布；互动率只在分母真实存在时计算。
5. 按 P0（影响定位或合规）、P1（影响内容效率）、P2（可试验）输出问题和三条今日选题。
6. 生成 `account-diagnosis.json` 和 `account-diagnosis.md`，标明 `real` 或 `mock`，并列出证据范围和局限。

## 输出合同

JSON 必须包含 `schemaVersion`、`mode`、`capturedAt`、`account`、`sample`、`metrics`、`findings`、`topicSuggestions`、`limitations`。每条 finding 必须有 `priority`、`claim`、`evidence`；每条 evidence 必须有 `url` 或 `source`。没有证据的结论写入 `limitations`，不得写成事实。

Markdown 用于人工阅读，必须显示：账号、采集时间、样本数量、指标定义、P0/P1/P2问题、三条选题和证据链接。不要把“平台算法规则”“限流原因”“保证涨粉”等未经证实的表达写入报告。

## 页面异常和安全

- 页面要求登录、验证码、实名、风控或权限不足时，停止该账号并返回 `needs_user_action`，不要绕过。
- 页面空白、字段缺失或作品不足时如实记录样本量，不补造数据。
- 不保存 Cookie、密码、令牌或完整个人隐私；报告只保留完成任务所需的公开字段。
- `mode=mock` 只用于自测，报告中必须明确标记 `mock`，不能冒充线上结果。

## 自测

```bash
node scripts/self-test.mjs
```

自测会读取 `fixtures/success.json` 和 `fixtures/failure.json`，验证成功报告和缺少账号 URL 时的失败状态。
