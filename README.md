# GPT API、Claude API 国内接入教程：GPT-6 Astra、GPT-5.6、Opus 5、Fable 5.1 与中转站

想在国内调用 GPT API 或 Claude API？本文以 GPT-6 Astra、GPT-5.6、Claude Opus 5 和 Fable 5.1 为例，介绍模型 ID、官方接口与中转站的区别，并给出可直接运行的 `curl` 和 Python 接入示例。

> 更新于 2026-09-19。模型开放范围、接口能力和价格会变化，实际以[OpenAI 模型列表](https://developers.openai.com/api/docs/models)、[Claude 模型列表](https://platform.claude.com/docs/en/models/overview)及服务商后台为准。本仓库是独立教程，与模型厂商无官方关联。

## 先选接入方式

| 方式 | 需要准备 | 适用情况 |
| --- | --- | --- |
| 官方 API | 对应厂商的开发者账号、API Key、受支持的地区与付款方式 | 希望直接使用官方接口与完整功能 |
| 兼容中转服务 | 服务商签发的 Token、Base URL、可用模型名 | 官方账号、付款或网络接入不方便时，先小额验证 |

国内直接接入 GPT API、Claude API 需要中转，否则无法直接接入。 

需要中转服务的话，可以参考我推荐的[这家兼容服务及选择方法](https://apidock.ai/)。 

![](https://file1.kamacoder.com/i/web/20260919161434-n3c8v9.png)

它提供独立 Token 和接入说明；**先确认所需模型是否开放、支持哪种接口协议，再用小额请求测试**。中转服务会处理请求内容，敏感数据应先评估隐私条款。

ChatGPT Plus、Claude 订阅与 API 用量分别计费；网页会员资格不能直接当作 API Key 使用。中转 Token 也不能登录官方产品。

## 这几个模型怎么选

| 模型 | API 模型 ID | 适合的起点 |
| --- | --- | --- |
| GPT-5.6 Luna | `gpt-5.6-luna` | 摘要、分类等高频轻任务 |
| GPT-5.6 Terra | `gpt-5.6-terra` | 日常编程、内容生成，兼顾效果与成本 |
| GPT-5.6 Sol | `gpt-5.6-sol` | 更复杂的代码与专业任务 |
| GPT-6 Astra | `gpt-6-astra` | 高难度推理、长流程和工具协作 |
| Claude Opus 5 | `claude-opus-5` | 复杂编程与日常 Agent 工作流 |
| Claude Fable 5.1 | `claude-fable-5-1` | 更难的长流程推理、研究与 Agent 任务 |

这只是选型起点，生产环境最好用自己的任务比较效果、延迟和每次完成任务的成本。[OpenAI 模型说明](https://developers.openai.com/api/docs/models)与[Claude 选型指南](https://platform.claude.com/docs/en/about-claude/models/choosing-a-model)提供最新定位。

## 调用 GPT：OpenAI Responses API

先设置 Key 与 Base URL。使用官方 API 时：

```bash
export OPENAI_API_KEY="你的官方 API Key"
export OPENAI_BASE_URL="https://api.openai.com/v1"
```

使用支持 **Responses API** 的中转服务时，把两项分别换成服务商 Token 和其提供的 OpenAI 兼容 Base URL。不要凭域名猜测路径，也不要把真实 Token 提交到 GitHub。

```bash
curl "${OPENAI_BASE_URL}/responses" \
  -H "Authorization: Bearer ${OPENAI_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-5.6-terra",
    "input": "用一句话解释 API 是什么"
  }'
```

想测试最新旗舰模型，将 `model` 改为 `gpt-6-astra`。中转服务若只兼容 `/chat/completions`，应按其文档调整请求格式；工具调用等高级能力也需逐项验证。

Python 项目可使用官方 SDK：

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
response = client.responses.create(
    model="gpt-6-astra",
    input="给这个 Python 服务列出三个性能排查步骤",
)
print(response.output_text)
```

## 调用 Claude：Anthropic Messages API

Claude 官方使用 `/v1/messages` 与 `anthropic-version` 请求头。**OpenAI 兼容地址不一定支持这套协议**；使用中转服务时，应确认它提供 Anthropic 兼容入口，或按照其 OpenAI 兼容文档调用 Claude 模型。

```bash
export ANTHROPIC_API_KEY="你的官方 API Key"
export ANTHROPIC_BASE_URL="https://api.anthropic.com"
```

若使用 Anthropic 兼容中转入口，将上面两项换成该服务的 Token 与其明确标注的 Anthropic Base URL。

```bash
curl "${ANTHROPIC_BASE_URL}/v1/messages" \
  -H "x-api-key: ${ANTHROPIC_API_KEY}" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{
    "model": "claude-opus-5",
    "max_tokens": 512,
    "messages": [{"role": "user", "content": "解释一下什么是 API 中转"}]
  }'
```

需要试用 Fable 5.1 时，把 `model` 改成 `claude-fable-5-1`，并确认账号或服务商已开放该模型。[Messages API 文档](https://platform.claude.com/docs/en/api/messages/create)列出了完整请求格式。

## 接入前后检查什么

1. 在平台模型列表核对**实际模型 ID**、接口类型、价格和限额；同名展示不保证接口能力相同。
2. 用无敏感内容的小请求确认响应与用量账单，再测试流式输出、工具调用等所需功能。
3. `401` 检查 Key；`404` 检查 Base URL、路径和模型名；`429` 检查额度与速率限制。
4. Token 放在环境变量或密钥管理服务中；泄露后立即撤销并重建，不向他人提供密码、验证码或 Cookie。

更多背景见 [GPT API 国内接入教程](docs/gpt-api-china.md)、[GPT-5.6 选型说明](docs/gpt-5-6-api.md)和[中转服务检查清单](docs/api-proxy-guide.md)。
