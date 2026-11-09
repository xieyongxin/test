资料核对日期：**2026-11-05，Asia/Shanghai**。价格币种：USD。本文是依据官方资料编写的测试设计；**未发起付费模型请求，所有测试执行结果均为“待执行”**。

## 1. 使用范围与结论

本文覆盖官方模型目录、定价页中列出且尚未下线的文本生成、原生图片生成模型，包括已弃用但仍提供服务的模型、预览版和受限型号。模型别名与快照放在同一行；已下线型号列入排除说明。语音、实时音视频、视频生成、向量、审核、微调训练和自行部署的开放权重模型不在本次计费测试范围。代理产品另列说明，避免将其多次模型调用误算为一次基础模型调用。

“所有字段”指这些模型相关的**计费维度、用量统计字段、影响费用的请求参数**，并不指完整 API 的全部业务参数。

最重要的验证结论：

1. OpenAI 新文本缓存有普通输入、缓存读取、30m 写入、12h 写入四个互斥计费桶；写入价格是完整费率，不能再加一次普通输入费用。
2. OpenAI `gpt-image-2`、2.5 的缓存优惠仅适用于 **Responses API 的 image_generation 工具**。直接重复 `/images/generations` 或 `/images/edits` 不能用来验证这些缓存优惠。图片指南明确说明 Responses 输出不报告图片生成的缓存数量；Responses 参考另列有可选 `tool_usage.image_gen` 明细，但可能省略或不完整，不能要求必返。图片缓存扣费仍需账单证据。
3. Google **Interactions 仅支持隐式缓存**；显式缓存对象需要使用支持它的模型和 generateContent／cachedContents 接口。Gemini 4 Argon 不支持显式缓存。
4. 当前列出的五个 Gemini 原生生图模型，模型页均将 **Caching 标为不支持**。不能为 Nano Banana 设计“必定出现文本／图片缓存输入”的正向用例；可测试的多模态缓存来自支持缓存的文本理解模型，例如 `gemini-3.8-flash`。
5. Gemini 显式缓存按缓存读取和存储时间等维度计价，没有 OpenAI 那两个独立的“30m 写／12h 写”字段。`ttl=1800s` 或 `43200s` 只是对象生命周期，不能当作两档缓存写价格。

以上依据：[OpenAI 缓存][O-CACHE]、[OpenAI 图片缓存][O-IMAGE]、[Gemini 缓存][G-CACHE]、[Gemini 模型目录][G-MODELS]。具体型号能力以模型表的链接为准。

### 1.1 表格符号

| 标记 | 含义 | 测试处理 |
| --- | --- | --- |
| 数值 0／Free | 官方明确免费或本次实际计量为零 | 需要区分免费档和付费档 |
| —／官网原文 `-` | 该价格表未列出此项 | 不自动推断免费或支持 |
| 不支持 | 官方能力表明确不支持 | 记录 N/A，执行适用的负向用例 |
| 未公布 | 未找到对应官方价格或明确规则 | 保留未知，不套用邻近型号费率 |
| 未返回 | 响应中没有该字段 | 不等同于数量为零 |
| 条件触发 | 有能力，但公共 API 不保证本次命中／输出 | 未命中记“未触发”，证据不足记“无法确认” |

## 2. 官方地址

| 厂商 | 地址用途 | 官方地址 |
| --- | --- | --- |
| OpenAI | API 基址 | `https://api.openai.com/v1` |
| OpenAI | 模型目录／全部型号 | [模型][O-MODELS]／[全部模型][O-ALL] |
| OpenAI | 当前价格／下线时间 | [定价][O-PRICE]／[弃用公告][O-DEP] |
| OpenAI | 文本生成与用量字段 | [Responses 创建接口][O-RESP]／[Chat Completions][O-CHAT] |
| OpenAI | 缓存机制 | [Prompt caching][O-CACHE] |
| OpenAI | 生图说明／原生接口 | [图片生成指南][O-IMAGE]／[Generations][O-GEN]／[Edits][O-EDIT] |
| OpenAI | 批处理／用量控制台 | [Batch][O-BATCH]／[Usage](https://platform.openai.com/usage) |
| Google | Gemini 原生 API 基址 | `https://generativelanguage.googleapis.com/v1beta` |
| Google | 模型／定价／生命周期 | [模型][G-MODELS]／[定价][G-PRICE]／[弃用公告][G-DEP] |
| Google | generateContent／用量字段 | [API 参考][G-GEN] |
| Google | Interactions／创建请求 | [API 参考][G-INTER]／[创建接口][G-ICREATE] |
| Google | 缓存／显式创建／删除 | [缓存指南][G-CACHE]／[创建缓存][G-CCREATE]／[删除缓存][G-CDELETE] |
| Google | 图片生成／计数／批处理 | [图片指南][G-IMAGE]／[Token 计数][G-TOKENS]／[Batch][G-BATCH] |
| Google | 账单入口 | [Google Cloud Billing](https://console.cloud.google.com/billing) |

Google 的 `ai.google.dev` 文档链接目前会跳转到 `aistudio.google.com` 官方文档。Gemini Developer API 与 Vertex AI 的接口、区域和价格不能混用；本文按 Gemini Developer API 编写。

## 3. 官方模型总表

能力表中的“文字／图片输入”表示模型直接接收这些内容，不代表该模型可以输出图片。OpenAI 文本模型调用图片工具时，会额外产生图片模型费用。

### 3.1 OpenAI

| 厂商 | 官方模型ID | 其他有效快照 | 状态 | 输入→输出 | 接口 | 类型 | 官方依据 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| OpenAI | `gpt-6-astra` | 未另列日期快照 | 提供中 | text, image → text | Chat/Responses/Batch | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-6-astra) |
| OpenAI | `gpt-6.1-sol` | 未另列日期快照 | 提供中 | text, image → text | Chat/Responses/Batch | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-6.1-sol) |
| OpenAI | `gpt-6.1-terra` | 未另列日期快照 | 提供中 | text, image → text | Chat/Responses/Batch | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-6.1-terra) |
| OpenAI | `gpt-6-luna` | 未另列日期快照 | 提供中 | text, image → text | Chat/Responses/Batch | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-6-luna) |
| OpenAI | `gpt-6-sol` | 未另列日期快照 | 提供中 | text, image → text | Chat/Responses/Batch | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-6-sol) |
| OpenAI | `gpt-5.6-sol` | 未另列日期快照 | 提供中 | text, image → text | Chat/Responses/Batch | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-5.6-sol) |
| OpenAI | `gpt-5.6-terra` | 未另列日期快照 | 提供中 | text, image → text | Chat/Responses/Batch | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-5.6-terra) |
| OpenAI | `gpt-5.6-luna` | 未另列日期快照 | 提供中 | text, image → text | Chat/Responses/Batch | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-5.6-luna) |
| OpenAI | `gpt-5.5` | `gpt-5.5-2026-04-23` | 提供中 | text, image → text | Chat/Responses/Batch | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-5.5) |
| OpenAI | `gpt-5.5-pro` | `gpt-5.5-pro-2026-04-23` | 提供中 | text, image → text | Responses/Batch | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-5.5-pro) |
| OpenAI | `gpt-5.4` | `gpt-5.4-2026-03-05` | 提供中 | text, image → text | Chat/Responses/Batch | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-5.4) |
| OpenAI | `gpt-5.4-pro` | `gpt-5.4-pro-2026-03-05` | 提供中 | text, image → text | Responses | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-5.4-pro) |
| OpenAI | `gpt-5.4-mini` | `gpt-5.4-mini-2026-03-17` | 提供中 | text, image → text | Chat/Responses/Batch | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-5.4-mini) |
| OpenAI | `gpt-5.4-nano` | `gpt-5.4-nano-2026-03-17` | 已弃用，仍提供；见3.3日期 | text, image → text | Chat/Responses/Batch | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-5.4-nano) |
| OpenAI | `gpt-5.2` | `gpt-5.2-2025-12-11` | 提供中 | text, image → text | Chat/Responses/Batch | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-5.2) |
| OpenAI | `gpt-5.2-pro` | `gpt-5.2-pro-2025-12-11` | 提供中 | text, image → text | Responses | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-5.2-pro) |
| OpenAI | `gpt-5.1` | `gpt-5.1-2025-11-13` | 已弃用，仍提供；见3.3日期 | text, image → text | Chat/Responses/Batch | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-5.1) |
| OpenAI | `gpt-5` | `gpt-5-2025-08-07` | 已弃用，仍提供；见3.3日期 | text, image → text | Chat/Responses/Batch | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-5) |
| OpenAI | `gpt-5-mini` | `gpt-5-mini-2025-08-07` | 已弃用，仍提供；见3.3日期 | text, image → text | Chat/Responses/Batch | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-5-mini) |
| OpenAI | `gpt-5-nano` | `gpt-5-nano-2025-08-07` | 已弃用，仍提供；见3.3日期 | text, image → text | Chat/Responses/Batch | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-5-nano) |
| OpenAI | `gpt-5-pro` | `gpt-5-pro-2025-10-06` | 已弃用，仍提供；见3.3日期 | text, image → text | Responses/Batch | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-5-pro) |
| OpenAI | `gpt-4.1` | `gpt-4.1-2025-04-14` | 已弃用，仍提供；见3.3日期 | text, image → text | Chat/Responses/Batch | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-4.1) |
| OpenAI | `gpt-4.1-mini` | `gpt-4.1-mini-2025-04-14` | 已弃用，仍提供；见3.3日期 | text, image → text | Chat/Responses/Batch | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-4.1-mini) |
| OpenAI | `gpt-4o` | `gpt-4o-2024-11-20`<br>`gpt-4o-2024-08-06` | 已弃用，仍提供；见3.3日期 | text, image → text | Chat/Responses/Batch | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-4o) |
| OpenAI | `gpt-4o-mini` | `gpt-4o-mini-2024-07-18` | 已弃用，仍提供；见3.3日期 | text, image → text | Chat/Responses/Batch | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-4o-mini) |
| OpenAI | `o3` | `o3-2025-04-16` | 已弃用，仍提供；见3.3日期 | text, image → text | Chat/Responses/Batch | 文本 | [模型](https://developers.openai.com/api/docs/models/o3) |
| OpenAI | `o3-pro` | `o3-pro-2025-06-10` | 已弃用，仍提供；见3.3日期 | text, image → text | Responses/Batch | 文本 | [模型](https://developers.openai.com/api/docs/models/o3-pro) |
| OpenAI | `gpt-5.3-codex` | 未另列日期快照 | 已弃用，仍提供；见3.3日期 | text, image → text | Responses | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-5.3-codex) |
| OpenAI | `chat-latest` | 未另列日期快照 | 动态别名；不推荐锁定回归 | text, image → text | Chat/Responses | 文本 | [模型](https://developers.openai.com/api/docs/models/chat-latest) |
| OpenAI | `gpt-5.6-cyber` | 未另列日期快照 | 受限；需官方授权 | text, image → text | Responses | 文本 | [模型](https://developers.openai.com/api/docs/models/gpt-5.6-cyber) |
| OpenAI | `gpt-5.5-cyber` | 未另列日期快照 | 价格页列出；访问／接口待确认 | 未独立公布 | 未独立确认 | 文本 | [官方价格][O-PRICE] |
| OpenAI | `gpt-6-rosalind` | 未另列日期快照 | 受限，需获批研究组织；接口待确认 | 未独立公布 | 未独立确认 | 文本 | [官方价格][O-PRICE] |
| OpenAI | `gpt-rosalind-research` | 未另列日期快照 | 受限，需获批研究组织；接口待确认 | 未独立公布 | 未独立确认 | 文本 | [官方价格][O-PRICE] |
| OpenAI | `gpt-rosalind-discovery` | 未另列日期快照 | 受限，需获批研究组织；接口待确认 | 未独立公布 | 未独立确认 | 文本 | [官方价格][O-PRICE] |
| OpenAI | `gpt-6-astra-law` | 未另列日期快照 | 法律模型配置标识；非已确认的独立model参数 | 未独立公布 | 未独立确认 | 文本 | [官方价格][O-PRICE] |
| OpenAI | `gpt-5-search-api` | 未另列日期快照 | 价格页列出；访问／接口待确认 | 未独立公布 | 未独立确认 | 文本 | [官方价格][O-PRICE] |
| OpenAI | `gpt-image-2.6-sunburst` | 未另列日期快照 | 提供中 | text, image → image | Images生成/Images编辑；工具路径见图片指南 | 图片 | [模型](https://developers.openai.com/api/docs/models/gpt-image-2.6-sunburst) |
| OpenAI | `gpt-image-2.6-flare` | 未另列日期快照 | 提供中 | text, image → image | Images生成/Images编辑；工具路径见图片指南 | 图片 | [模型](https://developers.openai.com/api/docs/models/gpt-image-2.6-flare) |
| OpenAI | `gpt-image-2.5-sunburst` | `gpt-image-2.5-sunburst-2026-09-08` | 提供中 | text, image → image | Batch/Images生成/Images编辑；工具路径见图片指南 | 图片 | [模型](https://developers.openai.com/api/docs/models/gpt-image-2.5-sunburst) |
| OpenAI | `gpt-image-2.5-flare` | `gpt-image-2.5-flare-2026-09-08` | 提供中 | text, image → image | Batch/Images生成/Images编辑；工具路径见图片指南 | 图片 | [模型](https://developers.openai.com/api/docs/models/gpt-image-2.5-flare) |
| OpenAI | `gpt-image-2` | `gpt-image-2-2026-04-21` | 提供中 | text, image → image | Batch/Images生成/Images编辑；工具路径见图片指南 | 图片 | [模型](https://developers.openai.com/api/docs/models/gpt-image-2) |
| OpenAI | `gpt-image-1.5` | `gpt-image-1.5-2025-12-16` | 已弃用，仍提供；见3.3日期 | text, image → image, text | Batch/Images生成/Images编辑；工具路径见图片指南 | 图片 | [模型](https://developers.openai.com/api/docs/models/gpt-image-1.5) |
| OpenAI | `gpt-image-1-mini` | 未另列日期快照 | 已弃用，仍提供；见3.3日期 | text, image → image | Batch/Images生成/Images编辑；工具路径见图片指南 | 图片 | [模型](https://developers.openai.com/api/docs/models/gpt-image-1-mini) |
| OpenAI | `chatgpt-image-latest` | 未另列日期快照 | 已弃用，仍提供；见3.3日期 | text, image → image, text | Batch/Images生成/Images编辑；工具路径见图片指南 | 图片 | [模型](https://developers.openai.com/api/docs/models/chatgpt-image-latest) |
| OpenAI | `gpt-daybreak-red-latest` / `gpt-daybreak-blue-latest` | 动态路由别名 | 已弃用；受限资格，实际路由需确认 | 依实际目标模型 | 依目标模型 | 文本别名 | [官方全部模型][O-ALL] |

新文本模型表中的 Chat 支持不代表所有工具可在 Chat 接口调用；工具调用使用 Responses。仅价格页列出的条目已保留未知能力，没有将其当作已证实可直接调用的API型号。

### 3.2 Gemini

| 厂商 | 官方模型ID／变体 | 别名／快照 | 状态 | 输入→输出 | 接口 | 缓存能力 | 官方依据 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Google | `gemini-4-argon` | 未另列日期快照 | Stable | 文字/图片/音频/视频/PDF → 文字 | Interactions优先；仅隐式缓存 | 隐式；不支持显式 | [模型](https://aistudio.google.com/docs/models/gemini-4-argon) |
| Google | `gemini-3.8-flash` | 未另列日期快照 | Stable | 文字/图片/音频/视频/PDF → 文字 | generateContent / Interactions / Batch | 支持；显式走generateContent | [模型](https://aistudio.google.com/docs/models/gemini-3.8-flash) |
| Google | `gemini-3.6-flash` | 未另列日期快照 | Stable | 文字/图片/音频/视频/PDF → 文字 | generateContent / Interactions / Batch | 支持；显式走generateContent | [模型](https://aistudio.google.com/docs/models/gemini-3.6-flash) |
| Google | `gemini-3.5-flash-lite` | 未另列日期快照 | Stable | 文字/图片/音频/视频/PDF → 文字 | generateContent / Interactions / Batch | 支持 | [模型](https://aistudio.google.com/docs/models/gemini-3.5-flash-lite) |
| Google | `gemini-3.1-flash-lite` | 未另列日期快照 | Stable | 文字/图片/音频/视频/PDF → 文字 | generateContent / Interactions / Batch | 支持 | [模型](https://aistudio.google.com/docs/models/gemini-3.1-flash-lite) |
| Google | `gemini-3.1-pro-preview` | 未另列日期快照 | Preview | 文字/图片/音频/视频/PDF → 文字 | generateContent / Interactions / Batch | 支持 | [模型](https://aistudio.google.com/docs/models/gemini-3.1-pro-preview) |
| Google | `gemini-3.1-pro-preview-customtools` | 3.1 Pro自定义工具变体，非日期快照 | Preview；自定义工具变体 | 同3.1 Pro；行为针对工具优化 | 依3.1 Pro接口能力；不是托管shell服务 | 依主型号 | [模型](https://aistudio.google.com/docs/models/gemini-3.1-pro-preview) |
| Google | `gemini-3-flash-preview` | 未另列日期快照 | Preview | 文字/图片/音频/视频/PDF → 文字 | generateContent / Interactions / Batch | 支持 | [模型](https://aistudio.google.com/docs/models/gemini-3-flash-preview) |
| Google | `gemini-2.5-pro` | 未另列日期快照 | 已弃用；既有用户限定，12-01切路由 | 文字/图片/音频/视频/PDF → 文字 | generateContent；其他接口按当前账号能力 | 支持 | [模型](https://aistudio.google.com/docs/models/gemini-2.5-pro) |
| Google | `gemini-2.5-flash` | 未另列日期快照 | 已弃用；既有用户限定，12-01切路由 | 文字/图片/音频/视频/PDF → 文字 | generateContent；其他接口按当前账号能力 | 支持 | [模型](https://aistudio.google.com/docs/models/gemini-2.5-flash) |
| Google | `gemini-2.5-flash-lite` | 未另列日期快照 | 既有用户限定；尚未弃用 | 文字/图片/音频/视频/PDF → 文字 | generateContent；其他接口按当前账号能力 | 支持 | [模型](https://aistudio.google.com/docs/models/gemini-2.5-flash-lite) |
| Google | `gemini-nano-banana-2.1` | Nano Banana 2.1 | Stable；Nano Banana 2.1 | 文字/图片/视频/PDF → 图片/文字 | Interactions / generateContent；Batch按指南 | 不支持 | [模型](https://aistudio.google.com/docs/models/gemini-nano-banana-2.1) |
| Google | `gemini-3.1-flash-image` | Nano Banana 2 | Stable；Nano Banana 2 | 文字/图片/视频/PDF → 图片/文字 | generateContent / Interactions | 不支持 | [模型](https://aistudio.google.com/docs/models/gemini-3.1-flash-image) |
| Google | `gemini-3.1-flash-lite-image` | Nano Banana 2 Lite | Stable；Nano Banana 2 Lite | 文字/图片/视频/PDF → 图片/文字 | generateContent / Interactions | 不支持 | [模型](https://aistudio.google.com/docs/models/gemini-3.1-flash-lite-image) |
| Google | `gemini-3-pro-image` | Nano Banana Pro | Stable；Nano Banana Pro | 文字/图片 → 图片/文字 | generateContent / Interactions | 不支持 | [模型](https://aistudio.google.com/docs/models/gemini-3-pro-image) |
| Google | `gemini-2.5-flash-image` | Nano Banana | 已弃用；2027-03-15下线 | 文字/图片 → 图片/文字 | generateContent；其他接口按指南 | 不支持 | [模型](https://aistudio.google.com/docs/models/gemini-2.5-flash-image) |
| Google | `gemini-robotics-er-2-preview` | 未另列日期快照 | Preview；空间理解专用 | 文字/图片/音频/视频 → 文字 | generateContent；模型与机器人指南 | 支持 | [模型](https://aistudio.google.com/docs/models/gemini-robotics-er-2-preview) |
| Google | `gemini-robotics-er-2-streaming-preview` | 未另列日期快照 | Preview；流式专用 | 文字/图片/音频/视频 → 文字 | 按Robotics流式接口指南 | 不支持 | [模型](https://aistudio.google.com/docs/models/gemini-robotics-er-2-preview) |
| Google | `gemini-3.7-flash` / `gemini-3.5-flash` | 动态路由别名；记录实际版本 | 仍接受请求的自动路由别名 | 依3.8 Flash / 3.6 Flash | 按实际目标接口 | 依目标模型 | [路由公告][G-DEP] |
| Google | `gemini-flash-latest` | 动态路由别名；记录实际版本 | 动态latest别名，回归需记录实际版本 | 依实际目标模型 | 官方Interactions示例使用 | 依目标模型 | [API示例][G-INTER] |

当前模型页以稳定ID或preview ID为版本标识，未给出的日期快照不自行编造。Google原生图片接口的两种请求结构不同，参见C13/C14。

**相关托管代理（单列，非单一基础模型价格）：**

| 代理ID | 状态 | 接口／能力 | 计费口径 | 官方依据 |
| --- | --- | --- | --- | --- |
| `deep-research-preview-04-2026` | Preview | Interactions；多步推理／工具 | 底层推理+工具累计；没有单一固定每次价格 | [模型](https://aistudio.google.com/docs/models/deep-research-preview-04-2026) |
| `deep-research-max-preview-04-2026` | Preview | Interactions；多步推理／工具 | 底层推理+工具累计；没有单一固定每次价格 | [模型](https://aistudio.google.com/docs/models/deep-research-max-preview-04-2026) |
| `antigravity-preview-09-2026` | Preview | Interactions；多步推理／工具 | 底层推理+工具累计；没有单一固定每次价格 | [模型](https://aistudio.google.com/docs/models/antigravity-preview-09-2026) |

### 3.3 生命周期与目录边界

- OpenAI：GPT-5／mini／nano／pro、o3／o3-pro 计划于 **2026-12-11** 下线；GPT-4o／4o-mini／4.1／4.1-mini 计划于 **2027-01-14** 下线；GPT-5.1／5.3-Codex／5.4-nano 计划于 **2027-04-01** 下线；GPT Image 1.5／1 Mini／chatgpt-image-latest 计划于 **2026-12-01** 下线。[公告][O-DEP]
- Gemini：2.5 Pro、2.5 Flash 目前限制既有用户，**2026-12-01** 起分别自动路由至 3.8 Flash、3.5 Flash-Lite，并按替代模型收费；2.5 Flash-Lite 仍服务既有用户。2.5 Flash Image 已弃用，计划于 **2027-03-15** 下线。[公告][G-DEP]
- Gemini `gemini-3.7-flash` 已路由至 `gemini-3.8-flash`，`gemini-3.5-flash` 已路由至 `gemini-3.6-flash`；保留请求模型与实际模型版本，不能按旧名称独立套价。
- 已下线的 OpenAI `gpt-image-1`、DALL·E 旧系列、4.1 nano、o4-mini、旧 Codex／Deep Research／Chat 快照，以及 Gemini 2.0、旧 preview 图片模型、Imagen 均不作为当前可调用基线。价格页保留的历史示例不证明型号仍可用。Imagen 下线另见[官方迁移说明](https://aistudio.google.com/docs/imagen)。
- OpenAI gpt-oss 与 Google Gemma 为开放权重系列；自行部署和第三方托管价格不套用本文官方 API 费率。
- Gemini Deep Research／Max、Antigravity 等托管代理按底层推理和工具累积收费；它们没有可替代全部底层价格的单一“每次回答”价格。OpenAI 法律配置、Rosalind、Cyber 的访问资格与具体接口能力需要同时满足官方要求。

## 4. 官方模型价格表

下表来自核对日的官方页面，保留官方显示精度。除明确标注外，单位均为 **USD／1,000,000 Token**；缓存存储列为 **USD／1,000,000 Token·小时**；每张图片报价不能再与同一图片的输出 Token 报价重复相加。

价格表采用横向字段列：每个输入、输出、缓存列都是独立计费维度；同一行的单位、上下文档位、服务档位、日期和来源适用于各列。价格起始日期未公布时，核对日期只表示资料采集时间，不能当作官方生效日期。

部分官方表格展示舍入值，例如缓存价格 `0.13`；账单复算若涉及这类精度，需取得账单 SKU 的计量精度，不能仅凭展示舍入判定错误。

### 4.1 OpenAI 文本 Standard

来源：[官方定价][O-PRICE]。上下文阈值和状态见第3节；短长档位单位均为每百万Token。

| 模型／配置 | 输入上下文 | 档位 | 输入 | 输出 | 缓存读 | 30m写 | 12h写 | 单位 | 价格生效日期 | 官方来源 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `gpt-6-astra` | ≤272K | Standard | $10.00 | $50.00 | $1.00 | $12.50 | $25.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-astra` | >272K | Standard | $20.00 | $75.00 | $2.00 | $25.00 | $50.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6.1-sol` | ≤272K | Standard | $2.00 | $10.00 | $0.10 | $2.50 | $5.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6.1-sol` | >272K | Standard | $4.00 | $15.00 | $0.20 | $5.00 | $10.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6.1-terra` | ≤272K | Standard | $1.00 | $5.00 | $0.05 | $1.25 | $2.50 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6.1-terra` | >272K | Standard | $2.00 | $7.50 | $0.10 | $2.50 | $5.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-luna` | ≤272K | Standard | $0.10 | $0.50 | $0.01 | $0.125 | $0.25 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-luna` | >272K | Standard | $0.20 | $0.75 | $0.02 | $0.25 | $0.50 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-sol` | ≤272K | Standard | $2.00 | $10.00 | $0.20 | $2.50 | $5.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-sol` | >272K | Standard | $4.00 | $15.00 | $0.40 | $5.00 | $10.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.6-sol` | ≤272K | Standard | $4.00 | $20.00 | $0.40 | $5.00 | $10.00 | USD／百万Token | 起始日期未公布；2026-11-05核对；优惠至2026-11-30，之后价未公布 | [定价][O-PRICE] |
| `gpt-5.6-sol` | >272K | Standard | $8.00 | $30.00 | $0.80 | $10.00 | $20.00 | USD／百万Token | 起始日期未公布；2026-11-05核对；优惠至2026-11-30，之后价未公布 | [定价][O-PRICE] |
| `gpt-5.6-terra` | ≤272K | Standard | $2.00 | $12.00 | $0.20 | $2.50 | $5.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.6-terra` | >272K | Standard | $4.00 | $18.00 | $0.40 | $5.00 | $10.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.6-luna` | ≤272K | Standard | $0.20 | $1.20 | $0.02 | $0.25 | $0.50 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.6-luna` | >272K | Standard | $0.40 | $1.80 | $0.04 | $0.50 | $1.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.5` | ≤272K | Standard | $5.00 | $30.00 | $0.50 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.5` | >272K | Standard | $10.00 | $45.00 | $1.00 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.5-pro` | ≤272K | Standard | $30.00 | $180.00 | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.5-pro` | >272K | Standard | $60.00 | $270.00 | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.4` | ≤272K | Standard | $2.50 | $15.00 | $0.25 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.4` | >272K | Standard | $5.00 | $22.50 | $0.50 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.4-pro` | ≤272K | Standard | $30.00 | $180.00 | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.4-pro` | >272K | Standard | $60.00 | $270.00 | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.4-mini` | 型号标准上下文 | Standard | $0.75 | $4.50 | $0.075 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.4-nano` | 型号标准上下文 | Standard | $0.20 | $1.25 | $0.02 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.2` | 型号标准上下文 | Standard | $1.75 | $14.00 | $0.175 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.2-pro` | 型号标准上下文 | Standard | $21.00 | $168.00 | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.1` | 型号标准上下文 | Standard | $1.25 | $10.00 | $0.125 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5` | 型号标准上下文 | Standard | $1.25 | $10.00 | $0.125 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5-mini` | 型号标准上下文 | Standard | $0.25 | $2.00 | $0.025 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5-nano` | 型号标准上下文 | Standard | $0.05 | $0.40 | $0.005 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5-pro` | 型号标准上下文 | Standard | $15.00 | $120.00 | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-4.1` | 型号标准上下文 | Standard | $2.00 | $8.00 | $0.50 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-4.1-mini` | 型号标准上下文 | Standard | $0.40 | $1.60 | $0.10 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-4o` | 型号标准上下文 | Standard | $2.50 | $10.00 | $1.25 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-4o-mini` | 型号标准上下文 | Standard | $0.15 | $0.60 | $0.075 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `o3-pro` | 型号标准上下文 | Standard | $20.00 | $80.00 | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `o3` | 型号标准上下文 | Standard | $2.00 | $8.00 | $0.50 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.6-cyber` | ≤272K | Standard | $12.50 | $75.00 | $1.25 | $15.625 | $31.25 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.6-cyber` | >272K | Standard | — | — | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.5-cyber` | ≤272K | Standard | $12.50 | $75.00 | $1.25 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.5-cyber` | >272K | Standard | — | — | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-rosalind` | ≤272K | Standard | $15.00 | $75.00 | $1.50 | $18.75 | $37.50 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-rosalind` | >272K | Standard | $30.00 | $112.50 | $3.00 | $37.50 | $75.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-rosalind-research` | ≤272K | Standard | $5.00 | $25.00 | $0.50 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-rosalind-research` | >272K | Standard | — | — | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-rosalind-discovery` | ≤272K | Standard | $5.00 | $25.00 | $0.50 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-rosalind-discovery` | >272K | Standard | — | — | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-astra-law` | 型号标准上下文 | Standard | $10.00 | $50.00 | $1.00 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对；优惠至少至2026-12-31 | [定价][O-PRICE] |
| `chat-latest` | 型号标准上下文 | Standard | $5.00 | $30.00 | $0.50 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.3-codex` | 型号标准上下文 | Standard | $1.75 | $14.00 | $0.175 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5-search-api` | 型号标准上下文 | Standard | $1.25 | $10.00 | $0.125 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |

### 4.2 OpenAI 文本其他档位

下表逐项保留官网报价，不将全部旧型号机械乘统一倍率。型号缺某档位价格时不补造。来源：[官方定价][O-PRICE]。

| 模型／配置 | 输入上下文 | 档位 | 输入 | 输出 | 缓存读 | 30m写 | 12h写 | 单位 | 价格生效日期 | 官方来源 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `gpt-6-astra` | ≤272K | Batch | $5.00 | $25.00 | $0.50 | $6.25 | $12.50 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-astra` | >272K | Batch | $10.00 | $37.50 | $1.00 | $12.50 | $25.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6.1-sol` | ≤272K | Batch | $1.00 | $5.00 | $0.05 | $1.25 | $2.50 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6.1-sol` | >272K | Batch | $2.00 | $7.50 | $0.10 | $2.50 | $5.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6.1-terra` | ≤272K | Batch | $0.50 | $2.50 | $0.025 | $0.625 | $1.25 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6.1-terra` | >272K | Batch | $1.00 | $3.75 | $0.05 | $1.25 | $2.50 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-luna` | ≤272K | Batch | $0.05 | $0.25 | $0.005 | $0.0625 | $0.125 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-luna` | >272K | Batch | $0.10 | $0.375 | $0.01 | $0.125 | $0.25 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-sol` | ≤272K | Batch | $1.00 | $5.00 | $0.10 | $1.25 | $2.50 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-sol` | >272K | Batch | $2.00 | $7.50 | $0.20 | $2.50 | $5.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.6-sol` | ≤272K | Batch | $2.00 | $10.00 | $0.20 | $2.50 | $5.00 | USD／百万Token | 起始日期未公布；2026-11-05核对；优惠至2026-11-30，之后价未公布 | [定价][O-PRICE] |
| `gpt-5.6-sol` | >272K | Batch | $4.00 | $15.00 | $0.40 | $5.00 | $10.00 | USD／百万Token | 起始日期未公布；2026-11-05核对；优惠至2026-11-30，之后价未公布 | [定价][O-PRICE] |
| `gpt-5.6-terra` | ≤272K | Batch | $1.00 | $6.00 | $0.10 | $1.25 | $2.50 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.6-terra` | >272K | Batch | $2.00 | $9.00 | $0.20 | $2.50 | $5.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.6-luna` | ≤272K | Batch | $0.10 | $0.60 | $0.01 | $0.125 | $0.25 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.6-luna` | >272K | Batch | $0.20 | $0.90 | $0.02 | $0.25 | $0.50 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.5` | ≤272K | Batch | $2.50 | $15.00 | $0.25 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.5` | >272K | Batch | $5.00 | $22.50 | $0.50 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.5-pro` | ≤272K | Batch | $15.00 | $90.00 | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.5-pro` | >272K | Batch | — | — | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.4` | ≤272K | Batch | $1.25 | $7.50 | $0.13 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.4` | >272K | Batch | $2.50 | $11.25 | $0.25 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.4-pro` | ≤272K | Batch | $15.00 | $90.00 | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.4-pro` | >272K | Batch | $30.00 | $135.00 | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.4-mini` | 型号标准上下文 | Batch | $0.375 | $2.25 | $0.0375 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.4-nano` | 型号标准上下文 | Batch | $0.10 | $0.625 | $0.01 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.2` | 型号标准上下文 | Batch | $0.875 | $7.00 | $0.0875 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.2-pro` | 型号标准上下文 | Batch | $10.50 | $84.00 | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.1` | 型号标准上下文 | Batch | $0.625 | $5.00 | $0.0625 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5` | 型号标准上下文 | Batch | $0.625 | $5.00 | $0.0625 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5-mini` | 型号标准上下文 | Batch | $0.125 | $1.00 | $0.0125 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5-nano` | 型号标准上下文 | Batch | $0.025 | $0.20 | $0.0025 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5-pro` | 型号标准上下文 | Batch | $7.50 | $60.00 | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-4.1` | 型号标准上下文 | Batch | $1.00 | $4.00 | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-4.1-mini` | 型号标准上下文 | Batch | $0.20 | $0.80 | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-4o` | 型号标准上下文 | Batch | $1.25 | $5.00 | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-4o-mini` | 型号标准上下文 | Batch | $0.075 | $0.30 | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `o3-pro` | 型号标准上下文 | Batch | $10.00 | $40.00 | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `o3` | 型号标准上下文 | Batch | $1.00 | $4.00 | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-astra` | ≤272K | Flex | $5.00 | $25.00 | $0.50 | $6.25 | $12.50 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-astra` | >272K | Flex | $10.00 | $37.50 | $1.00 | $12.50 | $25.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6.1-sol` | ≤272K | Flex | $1.00 | $5.00 | $0.05 | $1.25 | $2.50 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6.1-sol` | >272K | Flex | $2.00 | $7.50 | $0.10 | $2.50 | $5.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6.1-terra` | ≤272K | Flex | $0.50 | $2.50 | $0.025 | $0.625 | $1.25 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6.1-terra` | >272K | Flex | $1.00 | $3.75 | $0.05 | $1.25 | $2.50 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-luna` | ≤272K | Flex | $0.05 | $0.25 | $0.005 | $0.0625 | $0.125 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-luna` | >272K | Flex | $0.10 | $0.375 | $0.01 | $0.125 | $0.25 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-sol` | ≤272K | Flex | $1.00 | $5.00 | $0.10 | $1.25 | $2.50 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-sol` | >272K | Flex | $2.00 | $7.50 | $0.20 | $2.50 | $5.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.6-sol` | ≤272K | Flex | $2.00 | $10.00 | $0.20 | $2.50 | $5.00 | USD／百万Token | 起始日期未公布；2026-11-05核对；优惠至2026-11-30，之后价未公布 | [定价][O-PRICE] |
| `gpt-5.6-sol` | >272K | Flex | $4.00 | $15.00 | $0.40 | $5.00 | $10.00 | USD／百万Token | 起始日期未公布；2026-11-05核对；优惠至2026-11-30，之后价未公布 | [定价][O-PRICE] |
| `gpt-5.6-terra` | ≤272K | Flex | $1.00 | $6.00 | $0.10 | $1.25 | $2.50 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.6-terra` | >272K | Flex | $2.00 | $9.00 | $0.20 | $2.50 | $5.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.6-luna` | ≤272K | Flex | $0.10 | $0.60 | $0.01 | $0.125 | $0.25 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.6-luna` | >272K | Flex | $0.20 | $0.90 | $0.02 | $0.25 | $0.50 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.5` | ≤272K | Flex | $2.50 | $15.00 | $0.25 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.5` | >272K | Flex | $5.00 | $22.50 | $0.50 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.5-pro` | ≤272K | Flex | $15.00 | $90.00 | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.5-pro` | >272K | Flex | — | — | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.4` | ≤272K | Flex | $1.25 | $7.50 | $0.13 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.4` | >272K | Flex | $2.50 | $11.25 | $0.25 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.4-pro` | ≤272K | Flex | $15.00 | $90.00 | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.4-pro` | >272K | Flex | $30.00 | $135.00 | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.4-mini` | 型号标准上下文 | Flex | $0.375 | $2.25 | $0.0375 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.4-nano` | 型号标准上下文 | Flex | $0.10 | $0.625 | $0.01 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.2` | 型号标准上下文 | Flex | $0.875 | $7.00 | $0.0875 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.1` | 型号标准上下文 | Flex | $0.625 | $5.00 | $0.0625 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5` | 型号标准上下文 | Flex | $0.625 | $5.00 | $0.0625 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5-mini` | 型号标准上下文 | Flex | $0.125 | $1.00 | $0.0125 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5-nano` | 型号标准上下文 | Flex | $0.025 | $0.20 | $0.0025 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `o3` | 型号标准上下文 | Flex | $1.00 | $4.00 | $0.25 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-astra` | ≤272K | Fast | $20.00 | $100.00 | $2.00 | $25.00 | $50.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-astra` | >272K | Fast | $40.00 | $150.00 | $4.00 | $50.00 | $100.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6.1-sol` | ≤272K | Fast | $4.00 | $20.00 | $0.20 | $5.00 | $10.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6.1-sol` | >272K | Fast | $8.00 | $30.00 | $0.40 | $10.00 | $20.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6.1-terra` | ≤272K | Fast | $2.00 | $10.00 | $0.10 | $2.50 | $5.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6.1-terra` | >272K | Fast | $4.00 | $15.00 | $0.20 | $5.00 | $10.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-luna` | ≤272K | Fast | $0.20 | $1.00 | $0.02 | $0.25 | $0.50 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-luna` | >272K | Fast | $0.40 | $1.50 | $0.04 | $0.50 | $1.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-sol` | ≤272K | Fast | $4.00 | $20.00 | $0.40 | $5.00 | $10.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-sol` | >272K | Fast | $8.00 | $30.00 | $0.80 | $10.00 | $20.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.6-sol` | ≤272K | Fast | $8.00 | $40.00 | $0.80 | $10.00 | $20.00 | USD／百万Token | 起始日期未公布；2026-11-05核对；优惠至2026-11-30，之后价未公布 | [定价][O-PRICE] |
| `gpt-5.6-sol` | >272K | Fast | $16.00 | $60.00 | $1.60 | $20.00 | $40.00 | USD／百万Token | 起始日期未公布；2026-11-05核对；优惠至2026-11-30，之后价未公布 | [定价][O-PRICE] |
| `gpt-5.6-terra` | ≤272K | Fast | $4.00 | $24.00 | $0.40 | $5.00 | $10.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.6-terra` | >272K | Fast | $8.00 | $36.00 | $0.80 | $10.00 | $20.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.6-luna` | ≤272K | Fast | $0.40 | $2.40 | $0.04 | $0.50 | $1.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.6-luna` | >272K | Fast | $0.80 | $3.60 | $0.08 | $1.00 | $2.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.5` | ≤272K | Fast | $12.50 | $75.00 | $1.25 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.5` | >272K | Fast | — | — | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.4` | ≤272K | Fast | $5.00 | $30.00 | $0.50 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.4` | >272K | Fast | — | — | — | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.4-mini` | 型号标准上下文 | Fast | $1.50 | $9.00 | $0.15 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.2` | 型号标准上下文 | Fast | $3.50 | $28.00 | $0.35 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5.1` | 型号标准上下文 | Fast | $2.50 | $20.00 | $0.25 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5` | 型号标准上下文 | Fast | $2.50 | $20.00 | $0.25 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-5-mini` | 型号标准上下文 | Fast | $0.45 | $3.60 | $0.045 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-4.1` | 型号标准上下文 | Fast | $3.50 | $14.00 | $0.875 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-4.1-mini` | 型号标准上下文 | Fast | $0.70 | $2.80 | $0.175 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-4o` | 型号标准上下文 | Fast | $4.25 | $17.00 | $2.125 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-4o-mini` | 型号标准上下文 | Fast | $0.25 | $1.00 | $0.125 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `o3` | 型号标准上下文 | Fast | $3.50 | $14.00 | $0.875 | — | — | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-astra` | ≤272K | Ultrafast | $60.00 | $300.00 | $6.00 | $75.00 | $150.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6-astra` | >272K | Ultrafast | $120.00 | $450.00 | $12.00 | $150.00 | $300.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6.1-sol` | ≤272K | Ultrafast | $12.00 | $60.00 | $0.60 | $15.00 | $30.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |
| `gpt-6.1-sol` | >272K | Ultrafast | $24.00 | $90.00 | $1.20 | $30.00 | $60.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] |

### 4.3 OpenAI 图片

来源：[官方图片定价][O-PRICE]。以下所有数字仍为每百万Token；旧型号虽已弃用但核对日尚未下线。2.6未列Batch报价，不自动沿用2.5。

| 模型 | 档位 | 文本输入 | 文本缓存读 | 图片输入 | 图片缓存读 | 文本输出 | 图片输出 | 单位 | 价格生效日期 | 官方来源 | 上下文档位 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `gpt-image-2.6-sunburst` | Standard | $5.00 | $1.25 | $8.00 | $2.00 | - | $30.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] | 未单列长短档；按型号输入限制 |
| `gpt-image-2.6-flare` | Standard | $5.00 | $1.25 | $8.00 | $2.00 | - | $30.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] | 未单列长短档；按型号输入限制 |
| `gpt-image-2.5-sunburst` | Standard | $5.00 | $1.25 | $8.00 | $2.00 | - | $30.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] | 未单列长短档；按型号输入限制 |
| `gpt-image-2.5-flare` | Standard | $5.00 | $1.25 | $8.00 | $2.00 | - | $30.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] | 未单列长短档；按型号输入限制 |
| `gpt-image-2` | Standard | $5.00 | $1.25 | $8.00 | $2.00 | - | $30.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] | 未单列长短档；按型号输入限制 |
| `gpt-image-1.5` | Standard | $5.00 | $1.25 | $8.00 | $2.00 | $10.00 | $32.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] | 未单列长短档；按型号输入限制 |
| `gpt-image-1-mini` | Standard | $2.00 | $0.20 | $2.50 | $0.25 | - | $8.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] | 未单列长短档；按型号输入限制 |
| `chatgpt-image-latest` | Standard | $5.00 | $1.25 | $8.00 | $2.00 | $10.00 | $32.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] | 未单列长短档；按型号输入限制 |
| `gpt-image-2` | Batch | $2.50 | $0.625 | $4.00 | $1.00 | - | $15.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] | 未单列长短档；按型号输入限制 |
| `gpt-image-2.5-sunburst` | Batch | $2.50 | $0.625 | $4.00 | $1.00 | - | $15.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] | 未单列长短档；按型号输入限制 |
| `gpt-image-2.5-flare` | Batch | $2.50 | $0.625 | $4.00 | $1.00 | - | $15.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] | 未单列长短档；按型号输入限制 |
| `gpt-image-1.5` | Batch | $2.50 | $0.63 | $4.00 | $1.00 | $5.00 | $16.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] | 未单列长短档；按型号输入限制 |
| `gpt-image-1-mini` | Batch | $1.00 | $0.10 | $1.25 | $0.13 | - | $4.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] | 未单列长短档；按型号输入限制 |
| `chatgpt-image-latest` | Batch | $2.50 | $0.63 | $4.00 | $1.00 | $5.00 | $16.00 | USD／百万Token | 起始日期未公布；2026-11-05核对 | [定价][O-PRICE] | 未单列长短档；按型号输入限制 |

2／2.5的缓存价仅限Responses生图工具。1.5、chatgpt-image-latest列有文本输出价，但不保证某个公开请求必定产生可观测文本输出。2.6缓存路径未独立确认。以上不代表原生Images接口会返回全部计费模态的缓存明细。

### 4.4 Gemini 文本：所有已公布档位

来源：[Gemini官方定价][G-PRICE]。保留原文中的日期与阈值条件。输入列仅把文字／图片维度用于本次测试；原报价同时列出音频差价时一并保留以免误读，但不扩展为音频用例。缓存存储单位为每百万Token·小时。

| 模型 | 档位 | 输入价／条件 | 输出价（含Thinking） | 缓存读取价／条件 | 缓存存储价／条件 | Free档输入／输出／缓存 | 上下文档位 | 单位 | 价格生效日期 | 官方来源 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `gemini-4-argon` | Standard | $2.00 （至2026-11-30）； $4.00 （2026-12-01起） | $10.00 （至2026-11-30）； $20.00 （2026-12-01起） | $0.10 （至2026-11-30）； $0.20 （2026-12-01起） 仅隐式缓存，不支持显式缓存 | 不适用；仅隐式缓存 | 不提供／不提供／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对；当前优惠至2026-11-30；后价2026-12-01起 | [定价][G-PRICE] |
| `gemini-4-argon` | Batch | $1.00 （至2026-11-30）； $2.00 （2026-12-01起） | $5.00 （至2026-11-30）； $10.00 （2026-12-01起） | $0.05 （至2026-11-30）； $0.10 （2026-12-01起） 仅隐式缓存，不支持显式缓存 | 不适用；仅隐式缓存 | 不提供／不提供／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对；当前优惠至2026-11-30；后价2026-12-01起 | [定价][G-PRICE] |
| `gemini-4-argon` | Flex | $1.00 （至2026-11-30）； $2.00 （2026-12-01起） | $5.00 （至2026-11-30）； $10.00 （2026-12-01起） | $0.05 （至2026-11-30）； $0.10 （2026-12-01起） 仅隐式缓存，不支持显式缓存 | 不适用；仅隐式缓存 | 不提供／不提供／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对；当前优惠至2026-11-30；后价2026-12-01起 | [定价][G-PRICE] |
| `gemini-4-argon` | Priority | $3.60 （至2026-11-30）； $7.20 （2026-12-01起） | $18.00 （至2026-11-30）； $36.00 （2026-12-01起） | $0.18 （至2026-11-30）； $0.36 （2026-12-01起） 仅隐式缓存，不支持显式缓存 | 不适用；仅隐式缓存 | 不提供／不提供／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对；当前优惠至2026-11-30；后价2026-12-01起 | [定价][G-PRICE] |
| `gemini-4-argon` | Contributor | $1.00 | $5.00 | $0.05 仅隐式缓存，不支持显式缓存 | 不适用；仅隐式缓存 | 不提供／不提供／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-3.8-flash` | Standard | $0.75 （至2026-12-31）； $1.50 （2027-01-01起） | $3.75 （至2026-12-31）； $7.50 （2027-01-01起） | $0.075 （至2026-12-31）； $0.15 （2027-01-01起） | $0.50／百万Token·小时 （至2026-12-31）； $1.00／百万Token·小时 （2027-01-01起） | 0／0／0 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对；当前优惠至2026-12-31；后价2027-01-01起 | [定价][G-PRICE] |
| `gemini-3.8-flash` | Batch | $0.375 （至2026-12-31）； $0.75 （2027-01-01起） | $1.875 （至2026-12-31）； $3.75 （2027-01-01起） | $0.0375 （至2026-12-31）； $0.075 （2027-01-01起） | $0.50／百万Token·小时 （至2026-12-31）； $1.00／百万Token·小时 （2027-01-01起） | 不提供／不提供／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对；当前优惠至2026-12-31；后价2027-01-01起 | [定价][G-PRICE] |
| `gemini-3.8-flash` | Flex | $0.375 （至2026-12-31）； $0.75 （2027-01-01起） | $1.875 （至2026-12-31）； $3.75 （2027-01-01起） | $0.0375 （至2026-12-31）； $0.075 （2027-01-01起） | $0.50／百万Token·小时 （至2026-12-31）； $1.00／百万Token·小时 （2027-01-01起） | 不提供／不提供／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对；当前优惠至2026-12-31；后价2027-01-01起 | [定价][G-PRICE] |
| `gemini-3.8-flash` | Priority | $1.35 （至2026-12-31）； $2.70 （2027-01-01起） | $6.75 （至2026-12-31）； $13.50 （2027-01-01起） | $0.135 （至2026-12-31）； $0.27 （2027-01-01起） | $0.50／百万Token·小时 （至2026-12-31）； $1.00／百万Token·小时 （2027-01-01起） | 0／0／0 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对；当前优惠至2026-12-31；后价2027-01-01起 | [定价][G-PRICE] |
| `gemini-3.8-flash` | Contributor | $0.375 | $1.875 | $0.0375 | $0.50／百万Token·小时 | 0／0／0 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-3.6-flash` | Standard | $0.75 （至2026-12-31）； $1.50 （2027-01-01起） | $3.75 （至2026-12-31）； $7.50 （2027-01-01起） | $0.075 （至2026-12-31）； $0.15 （2027-01-01起） | $0.50／百万Token·小时 （至2026-12-31）； $1.00／百万Token·小时 （2027-01-01起） | 0／0／0 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对；当前优惠至2026-12-31；后价2027-01-01起 | [定价][G-PRICE] |
| `gemini-3.6-flash` | Batch | $0.375 （至2026-12-31）； $0.75 （2027-01-01起） | $1.875 （至2026-12-31）； $3.75 （2027-01-01起） | $0.0375 （至2026-12-31）； $0.075 （2027-01-01起） | $0.50／百万Token·小时 （至2026-12-31）； $1.00／百万Token·小时 （2027-01-01起） | 不提供／不提供／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对；当前优惠至2026-12-31；后价2027-01-01起 | [定价][G-PRICE] |
| `gemini-3.6-flash` | Flex | $0.375 （至2026-12-31）； $0.75 （2027-01-01起） | $1.875 （至2026-12-31）； $3.75 （2027-01-01起） | $0.0375 （至2026-12-31）； $0.075 （2027-01-01起） | $0.50／百万Token·小时 （至2026-12-31）； $1.00／百万Token·小时 （2027-01-01起） | 不提供／不提供／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对；当前优惠至2026-12-31；后价2027-01-01起 | [定价][G-PRICE] |
| `gemini-3.6-flash` | Priority | $1.35 （至2026-12-31）； $2.70 （2027-01-01起） | $6.75 （至2026-12-31）； $13.50 （2027-01-01起） | $0.135 （至2026-12-31）； $0.27 （2027-01-01起） | $0.50／百万Token·小时 （至2026-12-31）； $1.00／百万Token·小时 （2027-01-01起） | 0／0／0 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对；当前优惠至2026-12-31；后价2027-01-01起 | [定价][G-PRICE] |
| `gemini-3.5-flash-lite` | Standard | $0.30 (文字／图片／视频／音频) | $2.50 | $0.03 | $1.00／百万Token·小时 | 0／0／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-3.5-flash-lite` | Batch | $0.15 (文字／图片／视频／音频) | $1.25 | $0.015 | $1.00／百万Token·小时 | 0／0／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-3.5-flash-lite` | Flex | $0.15 (文字／图片／视频／音频) | $1.25 | $0.02 | $1.00／百万Token·小时 | 不提供／不提供／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-3.5-flash-lite` | Priority | $0.54 (文字／图片／视频／音频) | $4.50 | $0.05 | $1.00／百万Token·小时 | 0／0／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-3.5-flash-lite` | Contributor | $0.15 (文字／图片／视频／音频) | $1.25 | $0.015 | $1.00／百万Token·小时 | 0／0／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-3.1-flash-lite` | Standard | $0.25 (文字／图片／视频) $0.50 （音频） | $1.50 | $0.025 (文字／图片／视频) $0.05 （音频） | $1.00／百万Token·小时 | 0／0／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-3.1-flash-lite` | Batch | $0.125 (文字／图片／视频) $0.25 （音频） | $0.75 | $0.0125 (文字／图片／视频) $0.025 （音频） | $0.50／百万Token·小时 | 0／0／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-3.1-flash-lite` | Flex | $0.125 (文字／图片／视频) $0.25 （音频） | $0.75 | $0.0125 (文字／图片／视频) $0.025 （音频） | $0.50／百万Token·小时 | 0／0／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-3.1-flash-lite` | Priority | $0.45 (文字／图片／视频) $0.90 （音频） | $2.70 | $0.045 (文字／图片／视频) $0.09 （音频） | $1.80／百万Token·小时 | 0／0／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-3.1-pro-preview` | Standard | $2.00, （输入≤200K） $4.00, （输入>200K） | $12.00, （输入≤200K） $18.00, （输入>200K） | $0.20, （输入≤200K） $0.40, （输入>200K） | $4.50／百万Token·小时 | 不提供／不提供／不提供 | ≤200K与>200K（各项价格已分列） | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-3.1-pro-preview` | Batch | $1.00, （输入≤200K） $2.00, （输入>200K） | $6.00, （输入≤200K） $9.00, （输入>200K） | 与Standard相同： $0.20, （输入≤200K） $0.40, （输入>200K） | $4.50／百万Token·小时 | 不提供／不提供／不提供 | ≤200K与>200K（各项价格已分列） | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-3.1-pro-preview` | Flex | $1.00, （输入≤200K） $2.00, （输入>200K） | $6.00, （输入≤200K） $9.00, （输入>200K） | 与Standard相同： $0.20, （输入≤200K） $0.40, （输入>200K） | $4.50／百万Token·小时 | 不提供／不提供／不提供 | ≤200K与>200K（各项价格已分列） | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-3.1-pro-preview` | Priority | $3.60, （输入≤200K） $7.20, （输入>200K） | $21.60, （输入≤200K） $32.40, （输入>200K） | $0.36, （输入≤200K） $0.72, （输入>200K） | $8.10／百万Token·小时 | 不提供／不提供／不提供 | ≤200K与>200K（各项价格已分列） | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-3.1-pro-preview` | Contributor | $1.00, （输入≤200K） $2.00, （输入>200K） | $6.00, （输入≤200K） $9.00, （输入>200K） | $0.10, （输入≤200K） $0.20, （输入>200K） | $4.50／百万Token·小时 | 不提供／不提供／不提供 | ≤200K与>200K（各项价格已分列） | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-3-flash-preview` | Standard | $0.50 (文字／图片／视频) $1.00 （音频） | $3.00 | $0.05 (文字／图片／视频) $0.10 （音频） | $1.00／百万Token·小时 | 0／0／0 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-3-flash-preview` | Batch | $0.25 (文字／图片／视频) $0.50 （音频） | $1.50 | 与Standard相同： $0.05 (文字／图片／视频) $0.10 （音频） | $1.00／百万Token·小时 | 不提供／不提供／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-3-flash-preview` | Flex | $0.25 (文字／图片／视频) $0.50 （音频） | $1.50 | 与Standard相同： $0.05 (文字／图片／视频) $0.10 （音频） | $1.00／百万Token·小时 | 不提供／不提供／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-3-flash-preview` | Priority | $0.90 (文字／图片／视频) $1.80 （音频） | $5.40 | $0.09 (文字／图片／视频) $0.18 （音频） | $1.80／百万Token·小时 | 0／0／0 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-2.5-pro` | Standard | $1.25, （输入≤200K） $2.50, （输入>200K） | $10.00, （输入≤200K） $15.00, （输入>200K） | $0.125, （输入≤200K） $0.25, （输入>200K） | $4.50／百万Token·小时 | 0／0／不提供 | ≤200K与>200K（各项价格已分列） | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-2.5-pro` | Batch | $0.625, （输入≤200K） $1.25, （输入>200K） | $5.00, （输入≤200K） $7.50, （输入>200K） | $0.125, （输入≤200K） $0.25, （输入>200K） | $4.50／百万Token·小时 | 不提供／不提供／不提供 | ≤200K与>200K（各项价格已分列） | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-2.5-pro` | Flex | $0.625, （输入≤200K） $1.25, （输入>200K） | $5.00, （输入≤200K） $7.50, （输入>200K） | $0.125, （输入≤200K） $0.25, （输入>200K） | $4.50／百万Token·小时 | 不提供／不提供／不提供 | ≤200K与>200K（各项价格已分列） | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-2.5-pro` | Priority | $2.25, （输入≤200K） $4.50, （输入>200K） | $18.00, （输入≤200K） $27.00, （输入>200K） | $0.225, （输入≤200K） $0.45, （输入>200K） | $8.10／百万Token·小时 | 0／0／不提供 | ≤200K与>200K（各项价格已分列） | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-2.5-flash` | Standard | $0.30 (文字／图片／视频) $1.00 （音频） | $2.50 | $0.03 (文字／图片／视频) $0.1 （音频） | $1.00／百万Token·小时 | 0／0／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-2.5-flash` | Batch | $0.15 (文字／图片／视频) $0.50 （音频） | $1.25 | $0.03 (文字／图片／视频) $0.1 （音频） | $1.00／百万Token·小时 | 不提供／不提供／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-2.5-flash` | Flex | $0.15 (文字／图片／视频) $0.50 （音频） | $1.25 | $0.03 (文字／图片／视频) $0.1 （音频） | $1.00／百万Token·小时 | 不提供／不提供／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-2.5-flash` | Priority | $0.54 (文字／图片／视频) $1.80 （音频） | $4.50 | $0.054 (文字／图片／视频) $0.18 （音频） | $1.80／百万Token·小时 | 0／0／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-2.5-flash-lite` | Standard | $0.10 (文字／图片／视频) $0.30 （音频） | $0.40 | $0.01 (文字／图片／视频) $0.03 （音频） | $1.00／百万Token·小时 | 0／0／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-2.5-flash-lite` | Batch | $0.05 (文字／图片／视频) $0.15 （音频） | $0.20 | $0.01 (文字／图片／视频) $0.03 （音频） | $1.00／百万Token·小时 | 不提供／不提供／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-2.5-flash-lite` | Flex | $0.05 (文字／图片／视频) $0.15 （音频） | $0.20 | $0.01 (文字／图片／视频) $0.03 （音频） | $1.00／百万Token·小时 | 不提供／不提供／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-2.5-flash-lite` | Priority | $0.18 (文字／图片／视频) $0.54 （音频） | $0.72 | $0.018 (文字／图片／视频) $0.054 （音频） | $1.80／百万Token·小时 | 0／0／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-robotics-er-2` | Standard | $1.00 (文字／图片／视频／音频) （至2026-12-31）； $2.00 (文字／图片／视频／音频) （2027-01-01起） | $5.00 （至2026-12-31）； $10.00 （2027-01-01起） | $0.10 （至2026-12-31）； $0.20 （2027-01-01起） | $0.50／百万Token·小时 （至2026-12-31）； $1.00／百万Token·小时 （2027-01-01起） | 0／0／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对；当前优惠至2026-12-31；后价2027-01-01起 | [定价][G-PRICE] |
| `gemini-robotics-er-2` | Batch | $0.50 (文字／图片／视频／音频) （至2026-12-31）； $1.00 (文字／图片／视频／音频) （2027-01-01起） | $2.50 （至2026-12-31）； $5.00 （2027-01-01起） | $0.05 （至2026-12-31）； $0.10 （2027-01-01起） | $0.50／百万Token·小时 （至2026-12-31）； $1.00／百万Token·小时 （2027-01-01起） | 不提供／不提供／不提供 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对；当前优惠至2026-12-31；后价2027-01-01起 | [定价][G-PRICE] |
| `gemini-robotics-er-2-streaming` | Standard | $1.00 (文字／图片／视频／音频) （至2026-12-31）； $2.00 (文字／图片／视频／音频) （2027-01-01起） | $5.00 （至2026-12-31）； $10.00 （2027-01-01起） | 未列 | 未列 | 0／0／未列 | 官网未单列长短价；按型号限制 | 生成／读取：USD／百万Token；存储：USD／百万Token·小时 | 起始日期未公布；2026-11-05核对；当前优惠至2026-12-31；后价2027-01-01起 | [定价][G-PRICE] |

`gemini-3.1-pro-preview-customtools`官方工具定价说明与3.1 Pro相同。Robotics表的价格锚点省略preview，实际API ID见模型表。Free列按“输入／输出／缓存”分别列出：0是官方免费，不提供不是0。该列不代表所有工具或显式存储也免费；扣费测试使用Paid项目。

### 4.5 Gemini 图片：原单位和档位

来源：[Gemini官方定价][G-PRICE]。输出列同时列出每百万Token和每张的等价值时，它们描述同一笔图片输出费用，不能相加。

| 模型 | 档位 | 输入价 | 输出价／每张等价值 | 缓存 | Free档输入／输出 | 上下文档位 | 单位 | 价格生效日期 | 官方来源 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `gemini-nano-banana-2.1` | Standard | $1.50 (文字／图片／视频) | $7.50 (文字及Thinking)；$30.00 （图片） ；等价每张： $0.0336 ／张1K图片, $0.0504 ／张2K图片、$0.113 ／张4K图片*. | 不支持 | 不提供／不提供 | 未单列长短价；尺寸见4.6 | 未标每张者：USD／百万Token；每张价：USD／张 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-nano-banana-2.1` | Batch | $0.75 (文字／图片／视频) | $3.75 (文字及Thinking)；$15.00 （图片） ；等价每张： $0.0168 ／张1K图片, $0.0252 ／张2K图片、$0.0567 ／张4K图片*. | 不支持 | 不提供／不提供 | 未单列长短价；尺寸见4.6 | 未标每张者：USD／百万Token；每张价：USD／张 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-3.1-flash-image` | Standard | $0.50 (文字／图片) | $3 (文字及Thinking)；$60.00 （图片） ；等价每张： $0.045 ／张0.5K图片 $0.067 ／张1K图片, $0.101 ／张2K图片、$0.151 ／张4K图片. | 不支持 | 不提供／不提供 | 未单列长短价；尺寸见4.6 | 未标每张者：USD／百万Token；每张价：USD／张 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-3.1-flash-lite-image` | Standard | $0.25 (文字／图片／视频) | $1.50 (文字及Thinking)；$30.00 （图片） ；等价每张： $0.0336 ／张1K图片* | 不支持 | 不提供／不提供 | 未单列长短价；尺寸见4.6 | 未标每张者：USD／百万Token；每张价：USD／张 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-3.1-flash-lite-image` | Batch | $0.125 (文字／图片／视频) | $0.75 (文字及Thinking)；$15.00 （图片） ；等价每张： $0.0168 ／张1K图片* | 不支持 | 不提供／不提供 | 未单列长短价；尺寸见4.6 | 未标每张者：USD／百万Token；每张价：USD／张 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-3-pro-image` | Standard | $2.00 (文字／图片), 等价每张： $0.0011 ／张图片* | $12.00 (文字及Thinking)；$120.00 （图片） ；等价每张： $0.134 ／张1K或2K图片、$0.24 ／张4K图片 | 不支持 | 不提供／不提供 | 未单列长短价；尺寸见4.6 | 未标每张者：USD／百万Token；每张价：USD／张 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-2.5-flash-image` | Standard | $0.30 (文字／图片) | $0.039 ／张图片* | 不支持 | 不提供／不提供 | 未单列长短价；尺寸见4.6 | 未标每张者：USD／百万Token；每张价：USD／张 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-2.5-flash-image` | Batch | $0.15 (文字／图片) | $0.0195 ／张图片* | 不支持 | 不提供／不提供 | 未单列长短价；尺寸见4.6 | 未标每张者：USD／百万Token；每张价：USD／张 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-2.5-flash-image` | Flex | $0.15 (文字／图片) | $0.0195 ／张图片* | 不支持 | 不提供／不提供 | 未单列长短价；尺寸见4.6 | 未标每张者：USD／百万Token；每张价：USD／张 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |
| `gemini-2.5-flash-image` | Priority | $0.54 (文字／图片) | $0.0702 ／张图片* | 不支持 | 不提供／不提供 | 未单列长短价；尺寸见4.6 | 未标每张者：USD／百万Token；每张价：USD／张 | 起始日期未公布；2026-11-05核对 | [定价][G-PRICE] |

2.5 Flash Image的文字输入／输出按2.5 Flash文本费率；Standard文字输出为$2.50／M，Batch／Flex为$1.25／M，Priority为$4.50／M。其图片输出官网另给$30／M的Token等价及每张展示价，不把图中文字当作额外text_tokens。