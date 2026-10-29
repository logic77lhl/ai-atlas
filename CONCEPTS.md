# 概念清单 v1（与 index.html 同源）

> 本文档由 `index.html` 的 `DATA` 数组生成，两边永远一致。改内容请改 `index.html`，再重跑生成脚本。
>
> 每条回答四件事：它替代了什么（前身）→ 今天实际怎么用 → 状态 → 什么时候不该用。
> 状态是判断，不是事实；每条标「最后核对」月份。

## 状态说明

| 标签 | 含义 | 新项目怎么对待 |
|---|---|---|
| **标准** | 事实上的默认选择 | 直接用 |
| **主流** | 多数项目在用，有明确替代 | 先评估替代品 |
| **过渡** | 正在被替换 | 新项目谨慎 |
| **衰退** | 维护模式，少用 | 除非 legacy，否则别选 |
| **历史** | 理解脉络用 | 知道即可，不要用于新项目 |
| **争议** | 工程界没有共识 | 看场景，别信单一结论 |

## 基础（7）

> 需要多少系统知识才够

| 概念 | 前身 · 它替代了什么 | 现在怎么用 | 状态 | 边界 / 替代 | 深链 |
|---|---|---|---|---|---|
| **反向传播**<br>`Backpropagation` | 手推梯度不现实，网络一深就算不动 | 框架自动微分，你只定义前向；调试时仍要看梯度 | 标准 | 几乎无替代；梯度消失要靠归一化和残差 | [CS231n：反向传播](https://cs231n.github.io/optimization-2/) |
| **AdamW**<br>`Adam with decoupled weight decay` | SGD 对学习率太敏感，调参成本高 | 默认优化器；AdamW 把权重衰减和梯度解耦 | 标准 | 超大 batch 偶用 LAMB / Adafactor；收敛异常先查 lr | [论文：Decoupled Weight Decay](https://arxiv.org/abs/1711.05101)<br>[PyTorch AdamW](https://pytorch.org/docs/stable/generated/torch.optim.AdamW.html) |
| **混合精度**<br>`Mixed precision` | fp32 又慢又吃显存 | bf16 默认；fp16 需要 loss scaling | 标准 | 新卡上用 fp8；老卡 fp16 容易溢出 | [论文：Mixed Precision Training](https://arxiv.org/abs/1710.03740)<br>[PyTorch AMP](https://pytorch.org/docs/stable/amp.html) |
| **显存带宽**<br>`Memory bandwidth` | 算力涨了，数据却搬不动 | 推理优化先看内存带宽，不只看 FLOPs | 标准 | 计算密集的训练才以算力为主 | [FlashAttention（IO 视角）](https://arxiv.org/abs/2205.14135)<br>[Making Deep Learning Go Brrrr](https://horace.io/brrr_intro.html) |
| **Scaling law**<br>`Scaling laws` | 不知道该投多少算力和数据 | 预估 loss；Chinchilla 之后又被过量训练修正 | 主流 | 数据质量和推理算力改变了曲线，不能直接外推 | [Kaplan et al. 2020](https://arxiv.org/abs/2001.08361)<br>[Chinchilla](https://arxiv.org/abs/2203.15556) |
| **分布式训练**<br>`DDP / ZeRO / FSDP` | 单卡装不下模型 | 切参数、梯度、优化器状态，多卡协同 | 标准 | 小模型单卡更快；通信瓶颈时切分反而更慢 | [ZeRO](https://arxiv.org/abs/1910.02054)<br>[PyTorch FSDP](https://pytorch.org/docs/stable/fsdp.html) |
| **梯度检查点**<br>`Gradient checkpointing` | 激活值吃显存，大模型训不动 | 前向不存激活，反向重算，用算力换显存 | 标准 | 会慢 20–30%，显存够就别开 | [论文：Memory-Efficient Backprop](https://arxiv.org/abs/1604.06174)<br>[PyTorch checkpoint](https://pytorch.org/docs/stable/checkpoint.html) |

## 模型（8）

> 为什么是 Transformer

| 概念 | 前身 · 它替代了什么 | 现在怎么用 | 状态 | 边界 / 替代 | 深链 |
|---|---|---|---|---|---|
| **RNN / LSTM**<br>`Recurrent networks` | 序列模型只能一个词一个词递归地算 | NLP 主干已不用；只在极短序列、边缘设备偶见 | 历史 | 不能并行、长依赖衰减；新项目一律从 Transformer 起步 | [Colah：理解 LSTM](https://colah.github.io/posts/2015-08-Understanding-LSTMs/)<br>[Seq2Seq](https://arxiv.org/abs/1409.3215) |
| **Transformer**<br>`Transformer` | RNN 不能并行、长依赖弱 | 所有主流 LLM 的骨架 | 标准 | 状态空间模型在超长序列有优势，但生态小 | [Attention Is All You Need](https://arxiv.org/abs/1706.03762)<br>[The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/) |
| **注意力变体**<br>`MHA / MQA / GQA` | 多头注意力表达力强，但 KV 太贵 | GQA 成为默认；MQA 更省但质量降 | 标准 | MLA 是另一条路，框架支持少 | [GQA](https://arxiv.org/abs/2305.13245)<br>[MQA](https://arxiv.org/abs/1911.02150) |
| **旋转位置编码**<br>`RoPE` | 绝对位置编码外推能力差 | 旋转位置编码；长上下文靠 YaRN / NTK 插值 | 标准 | 外推不是免费的，超过训练长度仍会退化 | [RoFormer](https://arxiv.org/abs/2104.09864)<br>[YaRN](https://arxiv.org/abs/2309.00071) |
| **分词器**<br>`BPE / WordPiece / Unigram` | 词表外词（OOV）频发 | byte-level BPE 主流；中文仍占多个 token | 标准 | tokenizer-free 还在研究；换 tokenizer 等于重训 | [BPE 原论文](https://arxiv.org/abs/1508.07909)<br>[tiktoken（byte-level BPE）](https://github.com/openai/tiktoken) |
| **词向量**<br>`Word2Vec` | one-hot 词表又大又不带语义 | 被上下文相关的 embedding 取代；只在极轻量检索里还见得到 | 历史 | 一词一向量，分不清多义词；别拿它做新系统的语义层 | [Word2Vec 论文](https://arxiv.org/abs/1301.3781)<br>[BERT（上下文表示）](https://arxiv.org/abs/1810.04805) |
| **混合专家**<br>`Mixture of Experts` | 想加参数，但不想等比例加算力 | 稀疏激活：总参数大、激活参数小 | 主流 | 部署复杂，显存按总参数算 | [DeepSeekMoE](https://arxiv.org/abs/2401.06066)<br>[Switch Transformer](https://arxiv.org/abs/2101.03961) |
| **多模态融合**<br>`Vision-language fusion` | 图文各自为政，模型看不见图 | 视觉编码器 + projector；原生统一架构在增加 | 主流 | 拼接式多模态在细粒度任务上仍弱 | [CLIP](https://arxiv.org/abs/2103.00020)<br>[LLaVA](https://arxiv.org/abs/2304.08485) |

## 训练（9）

> 怎么让模型学会一件事

| 概念 | 前身 · 它替代了什么 | 现在怎么用 | 状态 | 边界 / 替代 | 深链 |
|---|---|---|---|---|---|
| **预训练**<br>`Pre-training` | 从零获得语言能力，没有捷径 | 千卡级，少数团队做；多数人做继续预训练 | 标准 | 没有数据工程能力不要碰 | [GPT-3](https://arxiv.org/abs/2005.14165)<br>[LLaMA](https://arxiv.org/abs/2302.13971) |
| **监督微调**<br>`Supervised Fine-Tuning` | 基座模型不会听指令 | 几万条高质量对话即可见效 | 标准 | 数据质量 > 数量；格式错等于白训 | [InstructGPT](https://arxiv.org/abs/2203.02155)<br>[Hugging Face TRL（SFT 实现）](https://github.com/huggingface/trl) |
| **RLHF / PPO**<br>`RLHF with PPO` | 人类偏好很难写成损失函数 | 奖励模型 + PPO，工程复杂、不稳定 | 过渡 | 多数场景已被 DPO / GRPO 替代，只有大团队还在跑 PPO | [Christiano et al. 2017](https://arxiv.org/abs/1706.03741)<br>[InstructGPT](https://arxiv.org/abs/2203.02155) |
| **直接偏好优化**<br>`DPO` | PPO 太重、不稳定，还要单独训奖励模型 | 偏好对直接优化，不需要奖励模型 | 主流 | 推理任务让位 GRPO；偏好对质量差会更差 | [DPO 论文](https://arxiv.org/abs/2305.18290)<br>[DPO 参考实现](https://github.com/eric-mitchell/direct-preference-optimization) |
| **组相对策略优化**<br>`GRPO` | PPO 的 critic 太贵，显存翻倍 | 组内相对优势，适合可验证奖励 | 主流 | 通用对话仍多用 DPO；奖励设计不好会刷分 | [DeepSeekMath](https://arxiv.org/abs/2402.03300)<br>[verl（GRPO 训练框架）](https://github.com/verl-project/verl) |
| **低秩适配**<br>`LoRA` | 全量微调太贵，每个任务存一份权重 | 低秩 adapter，可训练参数 <1% | 标准 | rank 不是越大越好；数据少时先加数据 | [LoRA 论文](https://arxiv.org/abs/2106.09685)<br>[Hugging Face PEFT](https://github.com/huggingface/peft) |
| **4bit 微调**<br>`QLoRA` | 单卡显存不够，7B 都跑不起来 | 4bit 底座 + LoRA，显存换精度 | 标准 | 8GB 跑 7B 是极限；多卡仍用 bf16 | [QLoRA 论文](https://arxiv.org/abs/2305.14314)<br>[bitsandbytes（4bit 后端）](https://github.com/bitsandbytes-foundation/bitsandbytes) |
| **数据清洗与配比**<br>`Data curation` | 堆数据边际收益递减 | 去重、质量分类、配比实验 | 标准 | 没有替代，且是真正的护城河 | [去重论文](https://arxiv.org/abs/2107.06499)<br>[FineWeb 数据集论文](https://arxiv.org/abs/2406.17557) |
| **蒸馏**<br>`Distillation` | 大模型推理太贵，小模型又不够聪明 | 用大模型输出训小模型 | 主流 | 教师错误会被学生学走；合成数据要过滤 | [Hinton 2015](https://arxiv.org/abs/1503.02531)<br>[MiniLLM](https://arxiv.org/abs/2306.08543) |

## 推理（8）

> 为什么同样模型快慢差十倍

| 概念 | 前身 · 它替代了什么 | 现在怎么用 | 状态 | 边界 / 替代 | 深链 |
|---|---|---|---|---|---|
| **KV 缓存**<br>`KV cache` | 每生成一个 token 都要重算整个历史 | 缓存 K/V，用显存换速度 | 标准 | 长上下文显存吃紧，才有 GQA / 量化 KV | [PagedAttention](https://arxiv.org/abs/2309.06180)<br>[Databricks：推理性能工程](https://www.databricks.com/blog/llm-inference-performance-engineering-best-practices) |
| **分页注意力**<br>`PagedAttention` | KV 碎片浪费大量显存 | 分页分配，vLLM 的核心 | 标准 | 调度层仍在优化，但范式没变 | [PagedAttention](https://arxiv.org/abs/2309.06180)<br>[vLLM 文档](https://docs.vllm.ai) |
| **静态批处理**<br>`Static batching` | 离线推理按固定 batch 跑，吞吐靠堆 batch size | 只在纯离线批量任务里用；在线服务已全面换成连续批处理 | 历史 | 按最长请求对齐，短请求白等；在线服务用它会浪费显存和时间 | [Anyscale：连续批处理](https://www.anyscale.com/blog/continuous-batching-llm-inference)<br>[Orca（OSDI'22）](https://www.usenix.org/conference/osdi22/presentation/yu) |
| **连续批处理**<br>`Continuous batching` | 静态 batch 被最慢的请求拖住 | 请求级调度，完成即换下一个 | 标准 | 单请求低延迟场景收益小 | [Orca（OSDI'22）](https://www.usenix.org/conference/osdi22/presentation/yu)<br>[vLLM 文档](https://docs.vllm.ai) |
| **量化**<br>`Quantization` | 显存和带宽是瓶颈 | GGUF 本地 / AWQ·GPTQ GPU / FP8 新卡 | 标准 | 没有单一赢家，按硬件选格式 | [GPTQ](https://arxiv.org/abs/2210.17323)<br>[AWQ](https://arxiv.org/abs/2306.00978) |
| **投机解码**<br>`Speculative decoding` | 大模型逐 token 生成太慢 | 小模型起草 + 大模型验证 | 主流 | 草稿质量差时反而更慢 | [Leviathan et al.](https://arxiv.org/abs/2211.17192)<br>[Chen et al.](https://arxiv.org/abs/2302.01318) |
| **前缀缓存**<br>`Prompt caching` | 相同 system prompt 每次都要重算 | API 层缓存 KV，命中后直接跳过 | 主流 | 跨请求缓存有隐私边界 | [vLLM 前缀缓存](https://docs.vllm.ai/en/latest/features/automatic_prefix_caching.html)<br>[论文：Prompt Cache](https://arxiv.org/abs/2311.04934) |
| **推理引擎**<br>`Inference engines` | 不同框架吞吐差十倍 | vLLM / SGLang / TensorRT-LLM / llama.cpp | 争议 | 按硬件、并发、模型支持度选，没有统一答案 | [vLLM](https://docs.vllm.ai)<br>[SGLang](https://github.com/sgl-project/sglang) |

## 检索 / RAG（6）

> 模型不知道的东西怎么给它

| 概念 | 前身 · 它替代了什么 | 现在怎么用 | 状态 | 边界 / 替代 | 深链 |
|---|---|---|---|---|---|
| **向量表示**<br>`Embeddings` | 关键词匹配不懂语义 | 向量检索；中文常用 BGE / GTE / Qwen-Embedding | 标准 | 领域词和长文本仍要微调；稀疏 + 稠密混合常见 | [Sentence-BERT](https://arxiv.org/abs/1908.10084)<br>[BGE](https://arxiv.org/abs/2309.07597) |
| **向量索引**<br>`HNSW / IVF` | 全量比对太慢 | 近似最近邻检索 | 标准 | 小数据直接暴力算更快；召回率和内存要权衡 | [HNSW 论文](https://arxiv.org/abs/1603.09320)<br>[hnswlib](https://github.com/nmslib/hnswlib) |
| **分块**<br>`Chunking` | 文档太长，塞不进上下文 | 按语义 / 结构切块，加 overlap | 主流 | 长上下文削弱它，但没有消除；切法决定上限 | [Pinecone：Chunking](https://www.pinecone.io/learn/chunking-strategies/)<br>[Dense X Retrieval](https://arxiv.org/abs/2312.06648) |
| **重排**<br>`Reranking` | 召回够但排序粗 | cross-encoder 重排 top-k | 主流 | 成本敏感可省略，质量会降 | [SBERT：Retrieve & Re-rank](https://www.sbert.net/examples/sentence_transformer/applications/retrieve_rerank/README.html)<br>[ColBERT](https://arxiv.org/abs/2004.12832) |
| **检索增强生成**<br>`RAG` | 模型知识过期、容易幻觉 | 检索 → 拼上下文 → 生成 | 标准 | 长上下文和微调是替代路线，不是替代品 | [RAG 原论文](https://arxiv.org/abs/2005.11401)<br>[RAG 综述](https://arxiv.org/abs/2312.10997) |
| **RAG 评估**<br>`RAG evaluation` | 不知道检索到底有没有用 | 召回率、faithfulness、答案相关性 | 争议 | 仍靠人工抽检 + LLM judge，指标不统一 | [RAGAS](https://arxiv.org/abs/2309.15217)<br>[Ragas 文档](https://docs.ragas.io) |

## Agent（7）

> 怎么让模型做事而不只是说话

| 概念 | 前身 · 它替代了什么 | 现在怎么用 | 状态 | 边界 / 替代 | 深链 |
|---|---|---|---|---|---|
| **ReAct**<br>`Reason + Act` | 模型只会答，不会查和做 | 思考 → 行动 → 观察 循环 | 标准 | 工具多时容易绕圈；框架封装了它，但范式没变 | [ReAct 论文](https://arxiv.org/abs/2210.03629)<br>[LangGraph](https://github.com/langchain-ai/langgraph) |
| **函数调用**<br>`Function calling` | 自由文本无法可靠调工具 | JSON schema 约束输出 | 标准 | schema 太复杂会降低调用成功率 | [OpenAI Cookbook：函数调用](https://github.com/openai/openai-cookbook/blob/main/examples/How_to_call_functions_with_chat_models.ipynb)<br>[ToolLLM](https://arxiv.org/abs/2307.16789) |
| **模型上下文协议**<br>`MCP` | 每个工具一套接口，重复适配 | 统一工具 / 资源协议，一次接入多处复用 | 主流 | A2A 负责 agent 间通信；权限和注入是新风险 | [MCP 官网](https://modelcontextprotocol.io)<br>[Anthropic 发布说明](https://www.anthropic.com/news/model-context-protocol) |
| **智能体记忆**<br>`Agent memory` | 上下文窗口有限，聊久了就忘 | 短期上下文 + 长期向量库 + 摘要 | 争议 | 没有公认最佳实践，别过度设计 | [MemGPT](https://arxiv.org/abs/2310.08560)<br>[Letta 文档](https://docs.letta.com) |
| **规划**<br>`CoT / ToT` | 多步任务一步错、步步错 | CoT 提示；ToT 做搜索 | 主流 | 推理模型内置思考，外显 CoT 的重要性下降 | [Chain-of-Thought](https://arxiv.org/abs/2201.11903)<br>[Tree of Thoughts](https://arxiv.org/abs/2305.10601) |
| **多智能体**<br>`Multi-agent` | 单 agent 能力有上限 | 角色分工 + 消息传递 | 争议 | 生产上「单 agent + 工具」往往更稳 | [AutoGen](https://arxiv.org/abs/2308.08155)<br>[Don't Build Multi-Agents](https://cognition.com/blog/dont-build-multi-agents) |
| **操作电脑**<br>`Computer use` | API 覆盖不到的操作只能人工 | 看屏幕、点鼠标、填表单 | 争议 | 可靠性不足，先做辅助而不是替代 | [OSWorld](https://arxiv.org/abs/2404.07972)<br>[Anthropic Computer use](https://www.anthropic.com/news/3-5-models-and-computer-use) |

## 评估（5）

> 怎么知道它真的变好了

| 概念 | 前身 · 它替代了什么 | 现在怎么用 | 状态 | 边界 / 替代 | 深链 |
|---|---|---|---|---|---|
| **基准测试**<br>`Benchmarks` | 没有统一比较口径 | MMLU / SWE-bench / τ-bench 等 | 标准 | 污染严重，私有 eval 更重要 | [MMLU](https://arxiv.org/abs/2009.03300)<br>[SWE-bench](https://arxiv.org/abs/2310.06770) |
| **模型评判**<br>`LLM-as-judge` | 人工评估太贵、太慢 | 用强模型打分 | 主流 | 位置 / 长度偏见，必须校准 | [Judging LLM-as-a-Judge](https://arxiv.org/abs/2306.05685)<br>[OpenAI Evals](https://github.com/openai/evals) |
| **竞技场**<br>`Arena / pairwise` | 绝对分数不可比 | 两两对比算 Elo | 主流 | 主观偏好 ≠ 任务能力 | [Chatbot Arena](https://arxiv.org/abs/2403.04132)<br>[LMSYS（Arena 团队）](https://www.lmsys.org/) |
| **回归测试**<br>`Regression evals` | 改一句 prompt 就悄悄退化 | 固定 eval set + CI | 标准 | 多数团队做得不够 | [HELM](https://arxiv.org/abs/2211.09110)<br>[lm-evaluation-harness](https://github.com/EleutherAI/lm-evaluation-harness) |
| **数据污染**<br>`Benchmark contamination` | 分数虚高，无法判断真实能力 | n-gram 重合、成员推断 | 争议 | 没有完美方法 | [数据污染调查](https://arxiv.org/abs/2311.09783) |

## 工程（5）

> 怎么让它稳定、便宜、可观测

| 概念 | 前身 · 它替代了什么 | 现在怎么用 | 状态 | 边界 / 替代 | 深链 |
|---|---|---|---|---|---|
| **Prompt 版本管理**<br>`Prompt versioning` | prompt 散落在代码和文档里 | 模板 + 版本 + 灰度 | 争议 | 部分被代码仓库顺带解决，工具仍在收敛 | [promptfoo](https://promptfoo.dev)<br>[Langfuse](https://github.com/langfuse/langfuse) |
| **可观测性**<br>`Tracing` | Agent 是黑盒，出错无法定位 | 用 span 记录每一步工具调用 | 主流 | OpenTelemetry GenAI 约定仍在收敛 | [OTel GenAI 约定](https://opentelemetry.io/docs/specs/semconv/gen-ai/)<br>[Langfuse 文档](https://langfuse.com/docs) |
| **成本 / 延迟**<br>`Cost & latency` | 全部用大模型太贵、太慢 | 路由、缓存、小模型、批处理 | 标准 | 路由判断错会伤质量 | [RouteLLM](https://arxiv.org/abs/2406.18665)<br>[论文：FrugalGPT（成本级联）](https://arxiv.org/abs/2305.05176) |
| **提示注入**<br>`Prompt injection` | 工具让模型能做事，也能被利用 | 输入隔离、权限最小化、人工确认 | 争议 | 没有银弹 | [Greshake et al.](https://arxiv.org/abs/2302.12173)<br>[OWASP LLM Top 10](https://owasp.org/projects/top-10-for-large-language-model-applications) |
| **部署形态**<br>`Deployment` | 自建、托管、API 各有取舍 | API / Space / 专用实例 / 自建 | 标准 | 按流量、合规、成本选 | [vLLM（自建推理服务）](https://github.com/vllm-project/vllm)<br>[vLLM](https://docs.vllm.ai) |

## 边界（4）

> 能不能用、该不该用

| 概念 | 前身 · 它替代了什么 | 现在怎么用 | 状态 | 边界 / 替代 | 深链 |
|---|---|---|---|---|---|
| **对齐**<br>`Alignment` | 能力越强，目标越要可靠 | RLHF / DPO / Constitutional AI | 争议 | 没有定论 | [InstructGPT](https://arxiv.org/abs/2203.02155)<br>[Constitutional AI](https://arxiv.org/abs/2212.08073) |
| **许可与开源定义**<br>`Licensing` | open weights 常被当成 open source | 看 LICENSE + 使用政策；OSAID 给判据 | 争议 | 商用必须逐条读原文 | [SPDX 许可证列表](https://spdx.org/licenses/)<br>[OSAID](https://opensource.org/ai/open-source-ai-definition) |
| **隐私与训练数据**<br>`Privacy` | 模型可能记住敏感信息 | PII 过滤、脱敏、差分隐私 | 主流 | 法规驱动，技术不完整 | [Extracting Training Data](https://arxiv.org/abs/2012.07805)<br>[Differential Privacy](https://arxiv.org/abs/1607.00133) |
| **版权**<br>`Copyright` | 训练数据来源不清，输出归属不明 | 诉讼进行中，各法域规则不同 | 争议 | 商用需要自评，不能只看模型许可 | [WIPO：前沿技术知识产权](https://www.wipo.int/en/web/frontier-technologies)<br>[论文：Foundation Models and Fair Use](https://arxiv.org/abs/2303.15715) |

---

## 统计

- 领域：9
- 概念：59
- 深链：116 条（平均 2.0 条/概念）
- 状态分布：标准 28 · 主流 16 · 过渡 1 · 衰退 0 · 历史 3 · 争议 11
- 全部条目的「最后核对」：2026-10

## 链接策略

- 优先 arXiv 摘要页、GitHub 仓库、官方文档根路径——这三类地址最稳。
- 2026-10 实测本机网络：`huggingface.co`、`lmarena.ai`、`platform.openai.com`、`copyright.gov`、`docs.anthropic.com` 不可达或被区域拦截，相关深链已换成同等内容的可达来源（论文 / GitHub / WIPO / SPDX）。
- GitHub 在批量探测时会限流超时，属正常现象，不代表链接失效。

## 硬规则（维护时）

1. 新增一条必须删一条，总数不超过 60。
2. 状态判断必须能指向一个来源；指不出来就不写状态。
3. 每条正文不超过 120 字，原理交给深链。
4. 每季度抽查 20% 的「主流 / 过渡」条目，改完更新 `v`。
