---
title: "第1章：提示工程导论 — Base LLM 与 Instruction Tuned LLM"
date: 2026-07-09
tags: [Prompt Engineering, LLM, ChatGPT, 吴恩达, 读书笔记]
author: enheng1238
---

# 第1章：提示工程导论

## 概览

本章介绍提示工程（Prompt Engineering）的基本概念，核心是理解 **Base LLM（基础语言模型）** 与 **Instruction Tuned LLM（指令调优语言模型）** 之间的本质区别。这两种模型对同一段提示词（prompt）的反应截然不同，理解它们的差异是写好提示词的基石。

---

## Base LLM（基础语言模型）

**Base LLM** 通过海量文本数据训练，目标是**预测最可能的下一个 token**。给定一段文本，它会根据训练语料中的统计规律，输出「最合理」的续写内容。

### 特点

| 特性 | 说明 |
|------|------|
| 训练目标 | 预测下一个 token（自回归语言建模） |
| 输出风格 | 模仿训练数据的分布，可能继续提问、编故事、输出无关内容 |
| 典型代表 | GPT-3（原始版）、LLaMA 基础版 |
| 适用场景 | 文本续写、生成、补全 |

### 示例

**提示词：**  
> 中国的首都是北京。

**Base LLM 可能输出：**  
> 上海是中国最大的城市。广州是南方的经济中心……

——Base LLM 只是在「做续写」，它认为用户期望它列举更多中国城市的信息，而不是回答问题。

---

## Instruction Tuned LLM（指令调优语言模型）

**Instruction Tuned LLM** 在 Base LLM 的基础上，经过了**指令微调（Instruction Fine-tuning）** 和 **RLHF（基于人类反馈的强化学习）**，被专门训练以**遵循指令**、理解用户的意图。

### 训练流程

1. **Base LLM 预训练** → 获得语言理解能力
2. **指令微调** → 在大规模（指令, 期望输出）数据集上微调
3. **RLHF** → 用人类偏好排序进一步对齐模型行为

### 特点

| 特性 | 说明 |
|------|------|
| 训练目标 | 遵循指令、理解用户意图 |
| 输出风格 | 直接回答用户问题，拒绝有害指令 |
| 典型代表 | ChatGPT、GPT-4、Claude |
| 适用场景 | 问答、翻译、摘要、代码生成等任务导向场景 |

### 示例

**提示词：**  
> 中国的首都是北京。

**Instruction Tuned LLM 可能输出：**  
> 是的，正确！北京是中华人民共和国的首都，也是政治、文化和国际交流中心。

——模型识别出这是一个「事实判断 + 期望确认」的指令，直接给出肯定回答并补充信息。

---

## 核心区别对比

| 维度 | Base LLM | Instruction Tuned LLM |
|------|----------|----------------------|
| **本质** | 文本续写器 | 指令跟随器 |
| **对「提问」的反应** | 可能继续提问或输出相关文本 | 直接回答 |
| **对「不完整输入」的反应** | 尝试补全成合理的文本 | 尝试理解意图后回应 |
| **安全性** | 无安全对齐，可能输出有害内容 | 经 RLHF 对齐，拒绝有害请求 |
| **提示方式** | 需要精心设计补全格式 | 可以用自然语言直接下达指令 |

---

## Python 代码示例

以下代码用 `openai` 库模拟两者的区别（使用 `text-davinci-003` 模拟 Base LLM 风格，`gpt-3.5-turbo` 模拟 Instruction Tuned LLM 风格）。实际使用中请替换为你自己的 API Key。

### 示例 1：同样的提示，不同的反应

```python
import openai

openai.api_key = "your-api-key-here"

# ── 模拟 Base LLM（text completion API）──
prompt = "中国的首都是北京。"

response_base = openai.Completion.create(
    model="text-davinci-003",   # 接近 Base LLM 行为
    prompt=prompt,
    max_tokens=50,
    temperature=0.7,
)

print("【Base LLM 输出】")
print(repr(response_base.choices[0].text.strip()))
# 可能输出: "上海是中国最大的城市。广州是南方的经济中心。\n深圳是改革开放的窗口。"


# ── 模拟 Instruction Tuned LLM（chat completion API）──
response_instruct = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",      # Instruction Tuned LLM
    messages=[
        {"role": "system", "content": "你是一个有用的助手。"},
        {"role": "user", "content": prompt},
    ],
    max_tokens=50,
    temperature=0.7,
)

print("\n【Instruction Tuned LLM 输出】")
print(repr(response_instruct.choices[0].message.content.strip()))
# 可能输出: "是的，北京是中国的首都。它是中国的政治和文化中心。"
```

### 示例 2：指令形式的提示

```python
# ── Base LLM 对指令的反应 ──
instruction = "将下面的句子翻译成英文：中国的首都是北京。"

response_base2 = openai.Completion.create(
    model="text-davinci-003",
    prompt=instruction,
    max_tokens=100,
    temperature=0,
)

print("【Base LLM — 收到翻译指令】")
print(repr(response_base2.choices[0].text.strip()))
# 可能输出: 'The capital of China is Beijing.'（正确，但不可靠）
# 也可能输出: "句子翻译：The capital of China is Beijing. 接下来我们来看另一个例子……"


# ── Instruction Tuned LLM 对指令的反应 ──
response_instruct2 = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": "system", "content": "你是一个翻译助手。"},
        {"role": "user", "content": instruction},
    ],
    max_tokens=100,
    temperature=0,
)

print("\n【Instruction Tuned LLM — 收到翻译指令】")
print(repr(response_instruct2.choices[0].message.content.strip()))
# 输出: 'The capital of China is Beijing.' 精确执行指令，不添加无关内容。
```

### 示例 3：用系统角色（System Prompt）控制行为

```python
# 没有 system prompt 时，Instruction Tuned LLM 可能仍会「多想一步」
response_no_system = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": "user", "content": "用一句话介绍北京。"}
    ],
    max_tokens=50,
)

print("【无 System Prompt】")
print(response_no_system.choices[0].message.content.strip())
# 输出: "北京是中华人民共和国的首都，拥有3000多年建城史，是政治、文化中心。"

# 有 system prompt 约束时，模型严格遵循
response_with_system = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": "system", "content": "你只能回答「是」或「否」，不要做任何解释。"},
        {"role": "user", "content": "北京是中国的首都吗？"}
    ],
    max_tokens=10,
)

print("\n【有 System Prompt 约束】")
print(response_with_system.choices[0].message.content.strip())
# 输出: "是"
```

---

## 为什么提示工程很重要？

理解了 Base LLM 和 Instruction Tuned LLM 的区别后，提示工程的核心原则就清晰了：

1. **对 Instruction Tuned LLM，要用「指令」而非「续写」的口吻写提示**  
   - ❌ `"法国的首都是"`（Base LLM 风格）  
   - ✅ `"法国的首都是什么？"`（Instruction Tuned LLM 风格）

2. **利用 System Prompt 设定角色和约束**  
   - 给模型一个「身份」能显著提升输出质量

3. **提示词就是「编程」**  
   - 对 Instruction Tuned LLM 而言，提示词就像代码，清晰、结构化、有明确输入输出格式的提示词效果最好

---

## 本章小结

| 概念 | 要点 |
|------|------|
| **Base LLM** | 预测下一个 token，擅长续写，不擅指令跟随 |
| **Instruction Tuned LLM** | 经过指令微调 + RLHF，擅长任务导向对话 |
| **提示工程的关键** | 理解模型类型，使用正确的提示风格 |
| **实践建议** | 使用 Chat API + System Prompt，以指令形式明确表达需求 |

---

*参考来源：吴恩达（Andrew Ng）& OpenAI — "ChatGPT Prompt Engineering for Developers" 第1章*
