---
title: "大模型面试题：Transformer 的三种结构——Encoder-only、Decoder-only 与 Encoder-Decoder"
date: 2026-08-12
tags: [大模型, Transformer, 面试题, 架构, AI技术]
author: "enheng1238"
description: "面试官最爱问：Transformer 的编码器、解码器到底怎么组合？本文以面试题形式拆解 Encoder-only、Decoder-only、Encoder-Decoder 三种结构，讲透掩码差异、代表模型、应用场景，以及 2026 年为什么所有大模型都选择 Decoder-only。"
---

# 大模型面试题：Transformer 的三种结构——Encoder-only、Decoder-only 与 Encoder-Decoder

## 引言：一道必考题

系列「Transformer 架构」篇讲过 Transformer 的完整结构——编码器 + 解码器。但到了面试场上，问题往往更刁钻：**为什么 BERT 只有编码器？为什么 GPT 只有解码器？为什么 T5 两者都要？**

这三个问题本质是在考同一个东西：你对 Transformer 三种结构形态的理解。本文以面试题形式拆解，最后给出 2026 年视角的加分答案。

## 面试题一：Transformer 有哪三种结构形态？

Transformer 的原始架构（2017 年论文）包含编码器和解码器两部分，但后续发展出了三种“裁切”方式：

**形态一：Encoder-only（只有编码器）。** 代表模型 BERT。编码器使用**双向注意力**——每个 token 都能看到完整序列的上下文。它天然擅长“理解”：把一段文本压缩成富含语义的向量表示。适合分类、命名实体识别、情感分析、问答中的阅读理解等任务。打个比方：它像一位文学评论家，通读全文后给出分析与判断。

**形态二：Decoder-only（只有解码器）。** 代表模型 GPT 系列。解码器使用**因果掩码（Causal Mask）**——每个 token 只能看到它自己和之前的 token。它天然擅长“生成”：逐词预测下一个 token，自回归地产出文本。它像一位作家，从提示词开始一路写下去，永远看不到未来。

**形态三：Encoder-Decoder（编码器+解码器）。** 代表模型 T5、BART。编码器读取完整输入（双向），解码器在**交叉注意力（Cross-Attention）**的引导下逐词生成输出。它擅长“序列到序列”任务：机器翻译、摘要、改写。

一张表总结（面试直接背）：

| 结构 | 代表模型 | 注意力方式 | 擅长任务 |
|------|----------|------------|----------|
| Encoder-only | BERT、RoBERTa | 双向 | 理解、分类、检索 |
| Decoder-only | GPT、LLaMA | 因果掩码 | 生成、对话、推理 |
| Encoder-Decoder | T5、BART | 双向 + 交叉 | 翻译、摘要、改写 |

三种结构之间还有一层“血缘关系”：Decoder-only 可以看成 Encoder-Decoder 砍掉编码器、只留因果解码器的简化版；Encoder-Decoder 则是在 Decoder-only 的基础上补回一个“能看到完整输入”的编码器。理解了这条演化线，三种结构就不再是三个孤立名词，而是一棵树的三个分支。

## 面试题二：为什么 2026 年的 LLM 几乎全是 Decoder-only？

这是追问率最高的问题，答好这四层就是满分：

**第一层：统一的任务形式。** Decoder-only 把一切任务都统一成“下一个 token 预测”——写文章是预测，做数学题是预测，对话也是预测。不需要为每个任务设计专门的输出头，一个模型通吃所有任务，这是它“通用”的根本。

**第二层：参数效率更高。** 编码器-解码器架构的 2N 参数模型，计算成本约等于 N 参数的 Decoder-only 模型——因为解码器要反复处理编码器输出的完整序列。同等算力下，Decoder-only 能把更多参数用在“生成主干”上，收益更直接。

**第三层：训练与推理的一致性。** Decoder-only 预训练（预测下一个 token）和部署推理（自回归生成）是同一个过程，行为完全对齐；而 BERT 式的掩码训练（MLM）与真实使用场景差异巨大，训出来的能力很难直接迁移到生成任务。

**第四层：规模化验证。** 2020 年后 GPT-3 到 GPT-4 的 Scaling Law 验证、LLaMA 开源生态的爆发、以及 DeepSeek、Qwen、Kimi、GLM 等 2026 年主流模型的集体选择——Decoder-only 已经被“用脚投票”证明是最能规模化、最稳定的路线。

## 面试题三：Encoder-only 和 Encoder-Decoder 被淘汰了吗？

**加分项来了：它们没有死，只是换了战场。**

Encoder-only 在生成型 LLM 时代确实“退居二线”，但在**检索与嵌入**领域依然是王者：RAG 系统中的向量化模型（如 E5、BGE、GTE）、语义搜索、文本聚类、句子相似度——这些任务要的是“高质量理解”，双向注意力比因果模型更高效。面试时能主动说出“encoder-only 是 RAG 检索的骨干”，印象分会立刻不同。2026 年随着 RAG 成为企业落地的标配，这类模型的需求不降反升，只是它们不再以“大模型”的身份出现在聚光灯下，而是安静地躺在向量数据库的入口处。

Encoder-Decoder 在机器翻译等强对齐任务上依然经典（T5 的翻译质量长期被当作基准）；更重要的是，它和 Decoder-only 在数学上并没有本质区别——**编码器-解码器的解码器，本质仍然是一个因果解码器**，只是多了交叉注意力这个“输入注入”通道。理解了这一点，你就看穿了三种结构的底层统一性。

## 面试题四（进阶）：三种结构的注意力掩码有什么不同？

这是把“背答案”变成“真理解”的分水岭：

- **Encoder-only**：完整双向注意力，无掩码，所有 token 互见；
- **Decoder-only**：因果掩码（上三角掩码），位置 i 只能 attend 到 [0, i]，保证自回归性；
- **Encoder-Decoder**：编码器内双向注意力；解码器内因果掩码；解码器通过交叉注意力 attend 编码器的全部输出。

再补一个冷知识（防止被问倒）：Decoder-only 模型内部其实也有“编码”环节——输入 token 经过嵌入层和前向计算才进入自注意力，只是没有独立的编码器模块而已。另外，很多面试官会追问“PrefixLM（前缀语言模型，如 GLM 早期版本）算什么”——它本质是 Decoder-only 的变体：前半段用双向注意力、后半段用因果注意力，既保留理解能力又保持生成能力，属于三种结构的“混血儿”，答出来能体现知识广度。

## 结语：面试答题框架

遇到“Transformer 结构”相关面试题，按这个框架答不会乱：

1. **一句话定性**：三种结构 = 双向理解（BERT）/ 因果生成（GPT）/ 序列到序列（T5）；
2. **讲透掩码**：双向注意力 vs 因果掩码 vs 交叉注意力；
3. **落到场景**：理解任务选 Encoder-only，生成任务选 Decoder-only，强对齐映射选 Encoder-Decoder；
4. **给出 2026 视角**：Decoder-only 因“统一任务 + 参数效率 + 训练推理一致 + 规模化验证”成为 LLM 主流；Encoder-only 活在 RAG 嵌入与检索领域；Encoder-Decoder 在翻译与对齐任务中延续价值。

架构没有绝对的优劣，只有与任务是否匹配。能说出这句话，你已经超过了大部分候选人。

最后再提醒一点：面试时避免把三种结构说成“三选一”的单选题——真实工程中经常组合使用，比如用 encoder-only 嵌入模型做检索、用 decoder-only 大模型做生成，两者拼成完整的 RAG 流水线。能给出这种“组合拳”视角，说明你不仅懂架构，还懂系统。

**参考文献：**
- arXiv (2017). “Attention Is All You Need”（1706.03762）
- Medium·Mandeep Singh (2024). “Transformer Architectures: Encoder Vs Decoder-Only”
- Yitay Gebru (2025). “What happened to BERT & T5? On Transformer Encoders, PrefixLM and Denoising Objectives”
- Gonzo ML (2025). “The Transformer Zoo Revisited”
- 知乎专栏 (2024). “encoder-only、decoder-only 几种模型的原理，及为什么大模型都用 decoder-only”
- Hugging Face 官方文档 (2024). “Summary of the models”（三种架构分类）
