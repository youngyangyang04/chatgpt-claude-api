# 国内如何接入 ChatGPT API / OpenAI API？curl、Python、Node.js 完整教程

国内开发者搜索“ChatGPT API 国内怎么用”或“OpenAI API 国内如何接入”，通常不是不会写代码，而是卡在账号、API Key、付款、网络或服务商选择上。

本文把官方 OpenAI API 与 OpenAI 兼容 API 放在同一套示例里：只要准备好 `API Key`、`API Base URL` 和模型名，就能先用 curl 验证，再接入 Python 或 Node.js 项目。

> 本文更新于 2026-08-23。实时模型与接口以 [OpenAI 官方文档](https://developers.openai.com/api/docs/models) 或你所用平台后台为准。

## 一、先分清 ChatGPT、OpenAI API 和兼容 API

| 名称 | 用法 | 凭证 | 计费 |
| --- | --- | --- | --- |
| ChatGPT | 官方网页、App 和套餐功能 | ChatGPT 账号 | 产品订阅或套餐规则 |
| OpenAI 官方 API | 在代码中调用模型 | OpenAI Platform API Key | 官方 API 按量计费 |
| OpenAI 兼容 API | 通过中转服务调用模型 | 平台 Token | 按平台规则计费 |

ChatGPT Plus 或 Pro 不等于一笔可以随便填进代码的 API 余额。反过来，购买 API Token 也不会自动获得 ChatGPT 会员功能。

## 二、国内接入 GPT API 的两条路径

### 路径 A：OpenAI 官方 API

适合已经具备以下条件的开发者：

- 能在官方支持范围内正常注册和使用 OpenAI Platform；
- 有可用的官方付款方式；
- 网络与合规条件允许访问官方 API；
- 希望直接获得官方模型和完整接口能力。

官方地址通常是：

```text
https://api.openai.com/v1
```

### 路径 B：OpenAI 兼容 API

如果官方账号、付款或网络路径暂时不方便，可以使用兼容接口。原来使用 OpenAI SDK 的代码，通常只需修改 `base_url`、`api_key` 和 `model`。

我自己更推荐 [APIDock](https://apidock.ai/)。它提供独立 Token、OpenAI 兼容地址和国内常用支付方式，适合国内开发者先把接口跑通。建议第一次只用赠送额度或小额充值，确认模型、稳定性和账单后再正式接入。

## 三、设置环境变量

不要把 Token 直接写入代码。

macOS / Linux：

```bash
export OPENAI_API_KEY="替换成你的 API Key 或 Token"
export OPENAI_BASE_URL="https://api.openai.com/v1"
export OPENAI_MODEL="gpt-5.6-terra"
```

使用 APIDock 时，把地址改为：

```bash
export OPENAI_BASE_URL="https://apidock.ai/v1"
```

Windows PowerShell：

```powershell
$env:OPENAI_API_KEY="替换成你的 API Key 或 Token"
$env:OPENAI_BASE_URL="https://api.openai.com/v1"
$env:OPENAI_MODEL="gpt-5.6-terra"
```

## 四、curl：先确认接口能通

第一次不要急着写业务代码。先请求一个最小接口：

```bash
curl "${OPENAI_BASE_URL}/chat/completions" \
  -H "Authorization: Bearer ${OPENAI_API_KEY}" \
  -H "Content-Type: application/json" \
  -d "{
    \"model\": \"${OPENAI_MODEL}\",
    \"messages\": [
      {\"role\": \"user\", \"content\": \"只回复：API 已连接\"}
    ]
  }"
```

成功返回后，再检查三件事：响应中的模型名是否合理、平台后台是否出现用量记录、实际扣费是否与输入输出 Token 对得上。

## 五、Python：使用 OpenAI SDK

安装依赖：

```bash
pip install openai
```

完整代码：

```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["OPENAI_API_KEY"],
    base_url=os.getenv("OPENAI_BASE_URL", "https://api.openai.com/v1"),
)

response = client.chat.completions.create(
    model=os.getenv("OPENAI_MODEL", "gpt-5.6-terra"),
    messages=[
        {"role": "system", "content": "你是一名耐心的中文编程助手。"},
        {"role": "user", "content": "解释 Python 装饰器，并给一个最小示例。"},
    ],
)

print(response.choices[0].message.content)
```

## 六、Node.js：使用 OpenAI SDK

安装依赖：

```bash
npm install openai
```

新建 `example.mjs`：

```javascript
import OpenAI from "openai";

const client = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
  baseURL: process.env.OPENAI_BASE_URL || "https://api.openai.com/v1",
});

const response = await client.chat.completions.create({
  model: process.env.OPENAI_MODEL || "gpt-5.6-terra",
  messages: [
    { role: "user", content: "写一个 JavaScript 防抖函数" },
  ],
});

console.log(response.choices[0].message.content);
```

运行：

```bash
node example.mjs
```

## 七、官方 API 为什么推荐 Responses API？

OpenAI 当前把 `/responses` 作为新项目的重要统一接口，它可以处理文本、图片、文件、工具调用和多轮状态。官方最小示例可以写成：

```python
import os
from openai import OpenAI

client = OpenAI(api_key=os.environ["OPENAI_API_KEY"])

response = client.responses.create(
    model="gpt-5.6-terra",
    input="用三句话解释 Responses API",
)

print(response.output_text)
```

兼容平台不一定完整实现 Responses API。接入时应先查它是否支持 `/responses`、内置工具、图片输入、结构化输出和流式响应；不支持时继续使用平台明确支持的 `/chat/completions`。

## 八、常见错误排查

### 401 Unauthorized

- Token 复制时多了空格或引号；
- Token 已删除、过期或被平台禁用；
- 请求发到了错误的服务商地址。

### 404 Not Found

- Base URL 少了或重复了 `/v1`；
- 接口路径写错；
- 平台没有开放该模型或该接口。

### 429 Too Many Requests

- 余额不足；
- 达到 RPM、TPM 或并发限制；
- 新账号的使用等级较低；
- 短时间重试次数太多。

### 请求超时

- 先把任务缩短，确认最小请求可用；
- 为客户端设置合理超时；
- 检查服务状态和网络路径；
- 重试使用指数退避，并设置最大次数，避免重复扣费。

## 九、生产环境接入检查清单

- Token 只放在环境变量或密钥管理系统；
- 为不同项目创建不同 Token，并设置额度；
- 记录请求 ID、耗时、状态码和 Token 用量，但不记录敏感正文；
- 设置超时、有限重试、并发限制和预算告警；
- 对客户数据、私有代码和日志做脱敏；
- 上线前用同一测试集比较质量、延迟、稳定性与最终成本。

## 相关阅读

- [GPT-5.6 Luna、Terra、Sol API 国内接入教程](gpt-5-6-api.md)
- [Codex 国内使用与 API 接入指南](codex-china.md)
- [GPT API 中转站选择与避坑指南](api-proxy-guide.md)
- [返回项目首页](../README.md)
