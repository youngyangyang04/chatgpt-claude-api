# ChatGPT API 国内接入教程：GPT-5.6、Codex API 与中转站指南

本仓库专门整理 **ChatGPT API 国内怎么用、OpenAI API 国内如何接入、GPT-5.6 API 怎么调用、Codex 国内怎么使用** 等常见问题，面向没有海外信用卡、无法顺利创建官方 API Key，或希望使用国内常用支付方式的开发者。

这里提供可以直接复制的 `curl`、Python 和 Node.js 示例，也会讲清楚 ChatGPT 会员、Codex、OpenAI 官方 API 和 API 中转站之间的区别。

> 更新日期：2026-08-23。模型、价格和接口能力都可能变化，请在使用前核对 [OpenAI 官方模型文档](https://developers.openai.com/api/docs/models) 与所选平台的实时说明。

## 先看结论：国内使用 GPT API 有哪些方式？

| 使用方式 | 适合谁 | 需要准备什么 | 需要注意 |
| --- | --- | --- | --- |
| OpenAI 官方 API | 已有官方开发者账号和可用付款方式 | API Key、可访问官方 API 的网络 | 按官方 API 用量单独计费 |
| ChatGPT 账号登录 Codex | 已有包含 Codex 权益的 ChatGPT 套餐 | ChatGPT 账号、Codex CLI 或 IDE 扩展 | 使用套餐权益，不是 API 余额 |
| OpenAI 兼容中转 API | 官方账号、付款或网络路径不方便 | 中转站 Token、API Base URL | 数据会经过服务商，建议先小额测试 |

如果你的目标是把模型接入程序、脚本、网站、Bot 或 AI Agent，使用的是 **API**；如果只是想在官方网页聊天，使用的是 **ChatGPT 产品**。二者账号体系、额度和计费方式不能混用。

## 5 分钟调用一次 GPT API

### 1. 准备 API Key 和接口地址

官方 OpenAI API：

```bash
export OPENAI_API_KEY="替换成你的 OpenAI API Key"
export OPENAI_BASE_URL="https://api.openai.com/v1"
```

如果没有合适的官方账号或付款方式，可以使用 OpenAI 兼容接口。我自己更推荐 [APIDock](https://apidock.ai/)：注册后创建独立 Token，支持国内常用支付方式，接口格式与 OpenAI SDK 兼容，适合先用小额请求跑通。

如果不懂的话，也可以去问 APIDock 的客服，一般接入问题，他们都会回答：

![](https://file1.kamacoder.com/i/web/2026-08-23_17-22-15.jpg)

```bash
export OPENAI_API_KEY="替换成你的 APIDock Token"
export OPENAI_BASE_URL="https://apidock.ai/v1"
```

不要把真实 Token 写进准备提交到 GitHub 的代码文件。

### 2. 使用 curl 测试

`chat/completions` 是很多 OpenAI 兼容服务都支持的接口，适合先测试连通性：

```bash
curl "${OPENAI_BASE_URL}/chat/completions" \
  -H "Authorization: Bearer ${OPENAI_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-5.6-terra",
    "messages": [
      {"role": "user", "content": "用一句话解释什么是 GPT API"}
    ]
  }'
```

如果返回 `401`，优先检查 Token；返回 `404`，检查 Base URL、接口路径和模型名；返回 `429`，检查余额、额度和请求频率。

### 3. 使用 Python 调用

先安装官方 SDK：

```bash
pip install openai
```

```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["OPENAI_API_KEY"],
    base_url=os.getenv("OPENAI_BASE_URL", "https://api.openai.com/v1"),
)

response = client.chat.completions.create(
    model="gpt-5.6-terra",
    messages=[
        {"role": "user", "content": "写一个 Python 快速排序函数"}
    ],
)

print(response.choices[0].message.content)
```

OpenAI 官方新项目更推荐使用 [Responses API](https://developers.openai.com/api/reference/resources/responses/methods/create)。使用兼容服务时，是否支持 `/responses`、工具调用、流式输出和图片输入，需要以平台文档为准。

## GPT-5.6 Sol、Terra、Luna 怎么选？

| 模型 | 官方定位 | 推荐场景 |
| --- | --- | --- |
| `gpt-5.6-luna` | 成本敏感、高吞吐 | 摘要、分类、信息抽取、批量轻任务 |
| `gpt-5.6-terra` | 能力与成本均衡 | 日常开发、内容生成、数据分析，适合作为默认模型 |
| `gpt-5.6-sol` | 复杂专业任务 | 复杂编程、长链路 Agent、多模块排查和难题推理 |

第一次接入建议先用 Terra 跑通，再把批量轻任务切到 Luna，只有真正复杂的任务再升级到 Sol。更完整的模型选择与代码示例见 [GPT-5.6 API 国内接入教程](docs/gpt-5-6-api.md)。

## Codex 和 API 是一回事吗？

不是。

- **Codex** 是能够阅读仓库、修改文件、运行命令和测试的编程 Agent 产品。
- **GPT API** 是模型调用接口。模型只负责生成响应，文件操作、终端执行和权限控制需要由客户端实现。
- Codex 本地客户端支持使用 ChatGPT 账号或官方 API Key 登录；两种方式的计费和功能范围不同。
- 中转站 Token 不等于 ChatGPT 账号，也不会自动获得 Codex 云端或 ChatGPT 套餐权益。

已有 ChatGPT 套餐的用户应优先选择 `Sign in with ChatGPT`，避免误用 API Key 后产生独立 API 账单。详细说明见 [Codex 国内使用与 API 接入指南](docs/codex-china.md)。

## 教程目录

| 教程 | 解决的问题 | 主要关键词 |
| --- | --- | --- |
| [国内接入 ChatGPT / OpenAI API 完整教程](docs/gpt-api-china.md) | 官方 API 与兼容 API 的 curl、Python、Node.js 调用 | ChatGPT API 国内、OpenAI API 国内接入 |
| [GPT-5.6 API 国内接入教程](docs/gpt-5-6-api.md) | Luna、Terra、Sol 选择与常见报错 | GPT-5.6 API、GPT5.6 API 怎么用 |
| [Codex 国内使用与 API 接入指南](docs/codex-china.md) | ChatGPT 登录、API Key、中转 API 的区别 | Codex 国内怎么用、Codex API |
| [没有 ChatGPT / OpenAI 账号怎么使用 GPT API](docs/no-openai-account.md) | 账号、会员与 API 的边界 | 没有 ChatGPT 账号、国内使用 GPT API |
| [GPT API 中转站选择与避坑指南](docs/api-proxy-guide.md) | 计费、稳定性、隐私和 Token 安全 | GPT 中转站、OpenAI API 中转 |

## 国内使用 GPT API 的安全建议

1. 不要把 API Key、Token、邮箱验证码、Cookie 提交到公开仓库。
2. 使用环境变量或密钥管理服务保存 Token；泄露后立即撤销并重新创建。
3. 使用中转 API 时，不要上传客户隐私、生产密钥或未脱敏的公司代码。
4. 中转站先用赠送额度或小额充值测试，不要长期存放大额余额。
5. 核对实际模型名、输入输出 Token、缓存价格、失败请求计费和速率限制。
6. 不购买共享 ChatGPT 账号，不向第三方提供官方账号密码。

建议同时提交一份 `.gitignore`：

```gitignore
.env
.env.*
!.env.example
```

## 常见问题

### 国内可以调用 ChatGPT API 吗？

代码层面可以使用 OpenAI SDK 或兼容协议调用，但实际可用性取决于服务支持地区、账号、网络和付款条件。无法顺利使用官方 API 的开发者，可以考虑 APIDock 这类 OpenAI 兼容 API。

### ChatGPT Plus 包含 OpenAI API 额度吗？

不要把两者当成同一份余额。ChatGPT 套餐是产品订阅，官方 API 通常按平台用量单独计费。Codex 使用 ChatGPT 登录时走套餐权益；使用 API Key 登录时走 API 计费。

### 没有 ChatGPT 账号能使用 GPT API 吗？

可以使用提供独立 Token 的兼容 API，但不会因此获得 ChatGPT 官网账号或 Plus 权益。见 [没有 OpenAI 账号的接入方案](docs/no-openai-account.md)。

### 什么是 OpenAI 兼容 API？

它使用与 OpenAI SDK 相近的请求格式，让现有代码通过修改 `base_url`、`api_key` 和 `model` 切换服务商。兼容程度并不完全相同，尤其要检查 Responses API、工具调用、图片、结构化输出和流式传输。

### API 中转站靠谱吗？

不能只凭“几折”判断。至少检查用量明细、模型列表、服务条款、隐私政策、限额能力和客服响应，并通过小额请求核对账单。详细检查清单见 [GPT API 中转站避坑指南](docs/api-proxy-guide.md)。

## 官方资料

- [OpenAI 模型列表](https://developers.openai.com/api/docs/models)
- [OpenAI Responses API](https://developers.openai.com/api/reference/resources/responses/methods/create)
- [Codex 认证方式](https://learn.chatgpt.com/docs/auth)

## 免责声明

本仓库是面向开发者的技术教程，不是 OpenAI 官方仓库。APIDock 等平台的模型、价格、支付方式和可用性可能调整；请遵守所在地法律法规和各服务的使用条款，并对自己的账号、数据和资金安全负责。

如果内容对你有帮助，欢迎 Star、提交 Issue 或补充新的接入经验。
