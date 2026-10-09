# AI Campaign 3.0 参赛队伍云资源与 AI API 使用指南

> **版本**：2026-10-09  
> **适用周期**：2026 年 10 月–11 月  
> **主要地域**：阿里云华北 2（北京）  
> **适用对象**：AI Campaign 3.0 参赛队伍

## 1. 比赛提供的资源

每支队伍默认或按需可获得以下资源：

| 资源 | 配置 | 提供方式 |
| --- | --- | --- |
| ECS | `ecs.g9i.large`，2 vCPU / 8 GiB RAM | 每队 1 台 |
| 系统盘 | 80 GiB ESSD Entry | 随 ECS 提供 |
| 百炼 API Key | 每队 1 个独立 Key | 默认提供 |
| API Key 费用上限 | **RMB 2,000 / 两个月** | 适用于比赛提供的百炼 Key，不按月重新计算 |
| PostgreSQL | 共享 RDS PostgreSQL Serverless | 按需申请 |
| OSS | 私有对象存储 | 按需申请 |
| Web Search | 百炼内置联网搜索 | 按需使用 |
| TTS | 百炼语音合成模型 | **默认开放** |

比赛提供的 API Key 并不是唯一允许使用的模型来源。队伍可以根据项目需要接入其他外部模型或 API；相关账号、费用、使用条款和凭证由队伍自行管理，并仍需遵守公司的数据、安全和合规要求。

---

## 2. ECS 开发环境

### 2.1 默认规格

```text
Instance family: Alibaba Cloud ECS g9i
Instance type:   ecs.g9i.large
vCPU:            2
Memory:          8 GiB
System disk:     80 GiB ESSD Entry
Region:          China North 2 (Beijing)
Lifecycle:       Competition period (Oct–Nov 2026)
```

适合运行：

- Python / FastAPI / Flask / Django
- Node.js / TypeScript
- 中轻量 Java / Spring Boot
- Docker / Docker Compose
- Agent orchestration、MCP 和工具服务
- Web 前端与 API 后端
- 轻量 Redis、SQLite 或本地开发数据库
- RAG 服务、任务队列和基础数据处理

大模型推理由百炼 API 或队伍自行接入的外部模型提供，因此默认不提供 GPU ECS。

### 2.2 登录与安全

建议使用 SSH Key 登录：

- 不要共享 SSH 私钥。
- 不要把私钥、API Key 或数据库密码提交到 Git 仓库。
- 只开放应用实际需要的安全组端口。
- 如需公网入口、域名、证书或额外端口，请联系组织方。

### 2.3 升级申请

默认 2C8G 无法满足实际测试时，可以申请：

- 更高 CPU / RAM
- 更大云盘
- 临时测试实例
- 特殊网络配置

申请时请说明实际瓶颈、用途和预计使用时间。

---

## 3. 数据库与文件存储

### 3.1 共享 PostgreSQL

需要持久化数据库时，可以申请共享 RDS PostgreSQL Serverless：

```text
Service:          RDS PostgreSQL Serverless
Compute range:    0.5–8 RCU
Storage:          ESSD PL1
Network:          Private VPC
Region:           China North 2 (Beijing)
```

组织方将为申请队伍提供独立的 Database、账号和连接信息。队伍之间不共享数据库账号。

常见用途：

- 业务数据
- Agent / workflow state
- Conversation metadata
- RAG metadata
- JSONB 数据
- 向量检索

需要向量检索时，可以申请启用 `pgvector`。Embedding 模型负责把文本生成向量，`pgvector` 负责在 PostgreSQL 中保存并按相似度检索这些向量。

### 3.2 OSS

需要保存 PDF、Word、Excel、扫描件、图片或其他文件时，可以申请 OSS。

建议：

- 原始文件存 OSS。
- 文件 metadata、chunk 和向量信息存 PostgreSQL。
- Bucket 默认使用私有访问。
- 不要把敏感文件设置为公网可读。

---

## 4. 百炼 API Key

每支队伍获得一个独立 API Key：

```text
Workspace: AI-Campaign-2026
Region:    China North 2 (Beijing)
Key scope: Team-specific
Budget cap: RMB 2,000 for the full two-month campaign
```

**RMB 2,000 是比赛提供的百炼 API Key 在整个两个月周期内的费用上限，不是每月 RMB 2,000。模型、Embedding、Web Search 和 TTS 等通过该 Key 产生的费用均计入该上限。**

当调用量接近上限时，请先检查：

- 是否存在 Agent 无限循环或过多 retry
- 是否所有步骤都在使用高价模型
- 是否重复传入完整历史对话或长文档
- 是否设置了过大的输出长度
- 是否存在不必要的批量或自动化调用

如确有合理业务需求需要追加额度，请联系组织方评估。

### 4.1 API Key 安全

- 使用环境变量或 Secret 管理 API Key。
- 不要硬编码在源码中。
- 不要提交到 Git / GitHub / GitLab。
- 不要放在浏览器前端、移动端或其他可被最终用户读取的位置。
- 不要与其他队伍共享。
- 怀疑泄露时立即联系组织方轮换 Key。

推荐环境变量：

```bash
export BAILIAN_API_KEY="<your-team-api-key>"
export BAILIAN_BASE_URL="https://<workspace-id>.cn-beijing.maas.aliyuncs.com/compatible-mode/v1"
```

Python 示例：

```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["BAILIAN_API_KEY"],
    base_url=os.environ["BAILIAN_BASE_URL"],
)

response = client.chat.completions.create(
    model="qwen3.8-flash",
    messages=[{"role": "user", "content": "Hello"}],
)

print(response.choices[0].message.content)
```

### 4.2 使用其他外部模型

比赛不要求队伍只能使用比赛提供的百炼 API。

队伍可以自行接入其他模型平台或外部 API。需要注意：

- 外部服务费用不计入比赛提供的 RMB 2,000 百炼 Key 上限。
- 外部账号、凭证、配额和费用由队伍自行管理。
- 不论使用哪家模型，均需遵守相同的数据分类、安全、隐私和合规要求。
- 未经批准，不要向外部服务发送生产客户数据、敏感信息、凭证或机密数据。

---

## 5. 可用模型与公开价格

> 以下为华北 2（北京）公开原价，截至 2026-10-09。单位为 **人民币 / 每 100 万 Token**。实际账单以阿里云出账为准。

### 5.1 生成模型

| Model ID | 输入 | 输出 | Cache-hit 输入 | 建议用途 |
| --- | ---: | ---: | ---: | --- |
| `qwen3.8-flash` | ¥0.8 | ¥2.7 | ¥0.1 | 默认低成本模型；Agent、RAG、代码和多模态任务 |
| `deepseek-v4.1-flash` | 闲时 ¥1 / 忙时 ¥2 | 闲时 ¥4 / 忙时 ¥8 | 闲时 ¥0.1 / 忙时 ¥0.2 | 复杂推理和 Agent；支持百炼内置联网搜索 |
| `ZHIPU/GLM-5.3-FlashX` | ¥2 | ¥7 | ¥0.57 | 低延迟、多模态和文件场景 |
| `glm-5.3` | ¥8 | ¥28 | ¥2 | 复杂代码、长程 Agent 和高难度任务 |
| `qwen3.8-max` | ¥12 | ¥36 | ¥1.5 | 质量敏感的复杂专业任务 |

DeepSeek V4.1 Flash 北京时间 08:00–22:00 为忙时，其余时间为闲时。

### 5.2 Embedding / RAG

| Model ID | 价格 | 建议 |
| --- | ---: | --- |
| `qwen3.7-text-embedding-flash` | ¥0.125 / 百万输入 Token | 默认高性价比选择 |
| `qwen3.7-text-embedding` | ¥0.5 / 百万输入 Token | 需要更多向量维度配置时使用 |

### 5.3 选择建议

- 默认先使用 `qwen3.8-flash`。
- 需要更强推理或联网能力时测试 `deepseek-v4.1-flash`。
- 对低延迟、多模态或文件输入，可测试 `ZHIPU/GLM-5.3-FlashX`。
- 只有在质量收益明显时再使用 `glm-5.3` 或 `qwen3.8-max`。

---

## 6. Web Search

百炼提供内置联网搜索，适合查询政策、法规、新闻、市场动态和其他公开时效信息。

当前模型支持情况：

| Model ID | 内置 Web Search |
| --- | :---: |
| `qwen3.8-flash` | ✅ |
| `deepseek-v4.1-flash` | ✅ |
| `qwen3.8-max` | ✅ |
| `ZHIPU/GLM-5.3-FlashX` | ❌ |
| `glm-5.3` | ❌ |

Chat Completions 示例：

```python
response = client.chat.completions.create(
    model="qwen3.8-flash",
    messages=[
        {"role": "user", "content": "查询最近一周跨境数据监管政策变化并总结"}
    ],
    extra_body={
        "enable_search": True,
        "search_options": {
            "search_strategy": "turbo",
            "freshness": 7
        }
    }
)
```

需要多轮搜索并打开网页正文时，可使用 Responses API 的 `web_search` 和 `web_extractor`。

公开价格：

- `turbo`：¥3 / 1,000 次搜索
- `max`：¥4 / 1,000 次搜索
- Responses API `web_search`：¥4 / 1,000 次搜索

搜索结果进入模型上下文后，还会产生正常的输入 Token 费用。联网搜索账号级限流为 15 RPS，由同一阿里云账号下的所有 API Key 共享。

---

## 7. TTS 语音合成（默认开放）

以下 TTS 模型在比赛业务空间中默认开放，无需单独申请：

| Model ID | 北京公开价格 | 适用场景 |
| --- | ---: | --- |
| `qwen3-tts-instruct-flash` | ¥0.8 / 万字符 | 默认 TTS；Agent 播报、语音回复、风格和情绪控制 |
| `cosyvoice-v3.5-flash` | ¥0.8 / 万字符 | 声音复刻、声音设计和自定义音色 |
| `qwen-audio-3.1-tts-flash` | 输入 ¥1.5/M Token；输出 ¥12/M Token | 实时语音助手、流式输出、方言和细粒度声音控制 |
| `qwen-audio-3.1-tts-next` | 输入 ¥6/M Token；输出 ¥12/M Token | 多人物播客、音效、环境声和完整音频生成 |

选择建议：

1. 普通语音播报：`qwen3-tts-instruct-flash`
2. 需要克隆或设计音色：`cosyvoice-v3.5-flash`
3. 需要实时对话、方言和低延迟：`qwen-audio-3.1-tts-flash`
4. 需要播客、多人对话、音效和环境声：`qwen-audio-3.1-tts-next`

中文汉字在按字符计费的模型中通常按 2 个计费字符计算。约 5,000 个纯中文汉字约等于 10,000 个计费字符，对应公开原价约 ¥0.8。

TTS 调用费用计入本队比赛百炼 API Key 的 RMB 2,000 两个月上限。声音复刻涉及个人声音特征时，必须获得明确授权。

---

## 8. 调用与费用控制

建议应用记录每次请求的：

```text
prompt_tokens
completion_tokens
total_tokens
model
request timestamp
```

同时建议：

- 为 Agent 设置 `max_steps`、retry 上限和 timeout。
- 避免每一步重复发送完整历史和文档。
- 对长对话做摘要，对 RAG 只传相关片段。
- 为输出设置合理的 token 上限。
- 将分类、路由、抽取等简单步骤交给低成本模型。
- 高价模型只用于确实需要更高质量的步骤。
- Web Search、TTS 和其他工具调用也应记录用量。

---

## 9. 数据与安全要求

- 只使用已获批准的数据进行测试和模型调用。
- 未经批准，不得发送生产客户数据、敏感信息、凭证或机密数据。
- API Key、数据库密码和 SSH 私钥均属于 Secret。
- ECS、RDS 和 OSS 尽量使用私网访问。
- 发现凭证泄露、异常调用或异常费用时，立即停止相关服务并联系组织方。

---

## 10. 申请额外资源

请提供：

1. Team 名称
2. 需要的资源或模型
3. 使用场景
4. 默认配置无法满足的原因
5. 预计使用时间
6. 预计调用量或资源使用量

可申请的项目包括：

- ECS 升级或更大云盘
- RDS / pgvector
- OSS
- 公网入口、域名或证书
- 其他有明确业务必要的服务

---

## 11. 官方参考资料

- Model Studio API endpoint: https://help.aliyun.com/zh/model-studio/base-url
- Model pricing: https://help.aliyun.com/zh/model-studio/model-pricing
- Web Search: https://help.aliyun.com/zh/model-studio/web-search
- TTS models: https://help.aliyun.com/zh/model-studio/tts-model
- Qwen-Audio 3.1 TTS Flash: https://help.aliyun.com/zh/model-studio/qwen-audio-3-1-tts-flash
- Qwen-Audio 3.1 TTS Next: https://help.aliyun.com/zh/model-studio/qwen-audio-3-1-tts-next
- CosyVoice 3.5 Flash: https://help.aliyun.com/zh/model-studio/cosyvoice-v3-5-flash
- RDS PostgreSQL Serverless: https://help.aliyun.com/zh/rds/apsaradb-rds-for-postgresql/serverless-apsaradb-rds-for-postgresql-instances/
- ECS g9i: https://help.aliyun.com/zh/ecs/user-guide/general-purpose-instance-families/
