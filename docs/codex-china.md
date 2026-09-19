# Codex 国内怎么用？ChatGPT 登录、API Key 与中转 API 指南

国内使用 Codex 时，最容易混淆的不是安装步骤，而是三种完全不同的东西：ChatGPT 账号登录、OpenAI 官方 API Key、第三方 GPT API Token。

根据 [Codex 官方认证文档](https://learn.chatgpt.com/docs/auth)，Codex 本地客户端支持两种 OpenAI 登录方式：使用 ChatGPT 登录获得套餐访问，或使用官方 API Key 按 API 用量计费；Codex Cloud 则要求使用 ChatGPT 登录。

## 一张表看懂 Codex 的几种路径

| 路径 | 凭证 | 计费 | 能获得什么 |
| --- | --- | --- | --- |
| Codex + ChatGPT 登录 | ChatGPT 账号 | 使用对应套餐的 Codex 权益 | 本地 Codex，符合条件时可用云端功能 |
| Codex + OpenAI API Key | 官方 API Key | OpenAI Platform 按量计费 | 本地 Codex 工作流，部分云端功能不可用 |
| 兼容客户端 + 中转 API | 平台 Token | 平台按量计费 | 取决于客户端的读写文件、终端和 Agent 能力 |

第三条路径不能自动解锁官方 Codex Cloud，也不会生成 ChatGPT 账号。它只是让一个支持自定义模型提供商的编程客户端调用 GPT 模型。

## 已有 ChatGPT 套餐，怎么登录 Codex？

安装 Codex CLI 后运行：

```bash
codex login
```

在浏览器中登录包含 Codex 权益的同一个 ChatGPT 账号。检查当前认证方式：

```bash
codex login status
```

如果登录错了，可以清理后重新登录：

```bash
codex logout
codex login
```

选择 ChatGPT 登录时，使用的是 ChatGPT 套餐访问；不要误填 API Key，否则会切换到独立的 API 用量计费。

## 使用 OpenAI API Key 登录 Codex

官方认证文档给出的 CLI 方式是通过标准输入传入 Key：

```bash
printenv OPENAI_API_KEY | codex login --with-api-key
```

这种方式适合需要官方 API 按量计费的本地工作流或自动化环境。不要把 Key 直接写进 shell 历史、公开脚本或仓库。

## 远程服务器打不开浏览器怎么办？

官方文档提供设备码认证：

```bash
codex login --device-auth
```

在可以打开浏览器的设备上访问提示的地址，登录后输入一次性验证码。若该功能在账号或工作区中不可用，再按官方认证页面提供的其他方式处理。

## 中转 API 能直接填进官方 Codex 吗？

不要默认任何 OpenAI 兼容 Token 都能替代官方账号或 API Key。

Codex 支持配置模型提供商，但平台必须与客户端所需的协议、认证、模型和 Agent 行为兼容。具体配置应以 Codex 当前配置说明和平台文档为准。即使模型能返回内容，也不代表 Codex 云端、插件、OAuth 连接或套餐权益可用。

如果希望少折腾，可以参考[第三方平台接入文档](https://apidock.ai/docs/apidock-easy-install)。先确认它当前支持的客户端和模型，再使用独立 Token 小额测试。

## API 模型为什么不等于 Codex？

GPT 模型可以理解代码并生成修改建议，但完整的编程 Agent 还需要：

- 读取和搜索仓库；
- 创建、修改、删除文件；
- 执行终端命令；
- 运行构建和测试；
- 展示 diff 并让用户审查；
- 隔离权限和保护密钥。

因此，“某个平台支持 GPT-6 Astra 或 GPT-5.6 API”和“这个平台提供完整 Codex 体验”不是同一个结论。

## 国内使用 Codex 的安全建议

1. 优先使用自己控制的账号，不购买共享号。
2. 不向任何第三方提供 ChatGPT 密码、邮箱验证码或浏览器 Cookie。
3. `~/.codex/auth.json` 可能包含访问凭证，不要提交、截图或发送给他人。
4. 中转 API 处理私有仓库前，先确认公司合规要求和平台隐私条款。
5. 给 Agent 最小必要权限，提交前审查 diff，重要操作保留人工确认。

## 常见问题

### ChatGPT 会员和 OpenAI API Key 应该选哪个？

已有包含 Codex 权益的 ChatGPT 套餐，通常选择 ChatGPT 登录。需要按 API 用量计费或做程序化本地工作流时，使用官方 API Key。两者账单独立。

### 中转 Token 能登录 ChatGPT 或 Codex Cloud 吗？

不能把它当成 ChatGPT 账号。它只能用于平台明确支持的 API 或客户端配置。

### Codex API 是一个单独的固定接口吗？

“Codex API”常被用来泛指适合代码和 Agent 任务的模型 API。实际开发时要确认模型 ID、Responses API 能力、客户端工具能力和认证方式，不要只看宣传名称。

### 手机验证失败怎么办？

先确认卡在 ChatGPT 登录、OpenAI Platform 创建 API Key，还是账号安全验证。不要购买来路不明的接码号。若只是需要模型 API，可以评估第三方兼容接口，但它不会替代官方 Codex 账号。

## 相关阅读

- [国内接入 ChatGPT / OpenAI API 完整教程](gpt-api-china.md)
- [GPT-5.6 API 国内接入教程](gpt-5-6-api.md)
- [没有 ChatGPT / OpenAI 账号怎么使用 GPT API](no-openai-account.md)
- [返回项目首页](../README.md)
