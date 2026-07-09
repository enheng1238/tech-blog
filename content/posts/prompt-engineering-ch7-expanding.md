---
date: 2026-07-09
tags: [Prompt Engineering, LLM, ChatGPT, 吴恩达, 读书笔记]
author: enheng1238
---

# 第7章：扩展（Expanding）

> 吴恩达 & OpenAI — ChatGPT Prompt Engineering for Developers

## 概述

**扩展（Expanding）** 是指将简短的输入文本（如一组指令、几条要点或一段评论）通过 LLM **生成为更长、更详细的文本**。本章以"根据客户评论生成个性化邮件回复"为实战案例，重点展示 **Temperature 参数** 对输出多样性的影响。

核心思想：LLM 不仅是"理解者"，更是"生成者"——可以根据少量输入，结合上下文和风格指令，输出结构完整、语气恰当的长文本。

---

## 7.1 什么是扩展

扩展与第 5 章（推断）和第 6 章（转换）构成完整的 LLM 文本处理 pipeline：

| 章节 | 任务 | 方向 |
|------|------|------|
| 第 5 章 推断 | 从文本中提取信息 | 输入 → 精简 |
| 第 6 章 转换 | 改变文本格式/风格 | 输入 → 变换 |
| **第 7 章 扩展** | **从文本生成更多内容** | **输入 → 扩展** |

典型应用场景：

- 根据客户评论生成客服回复邮件
- 根据关键词/大纲生成文章或报告
- 根据简短指令生成详细说明
- 根据产品描述生成营销文案
- 根据用户问题生成完整的解答

---

## 7.2 Temperature 参数详解

**Temperature** 是控制 LLM 输出随机性和创造力的核心参数。

### 工作原理

| Temperature 值 | 行为 | 适用场景 |
|:---:|---|---|
| **0** | 输出确定、可复现，每次结果相同 | 需要可靠性和可预测性的任务（如事实问答、代码生成、客服回复模板） |
| **0.1–0.5** | 轻微变化，在保留质量的前提下增加少量多样性 | 需要一定变化但仍需一致性的任务 |
| **0.7–1.0** | 明显变化，每次输出不同，更具创造性 | 创意写作、头脑风暴、营销文案、诗歌生成 |
| **> 1.0** | 高度随机，可能产生幻觉或不可靠内容 | 极少使用，仅用于极端的创意探索 |

### 关键理解

- Temperature=0 不意味着"没有创造力"，而是"每次都选概率最高的 Token"
- 温度越高，模型越倾向于选择**概率排名较低**的 Token，从而产生"意外"的结果
- 同一个 prompt + 高 temperature 下，模型可能"更易分心"但"更具创造力"

> **经验法则：** 需要一致性 → temperature=0；需要多样性 → temperature > 0

---

## 7.3 实战：从评论生成客服回复

本章的核心案例：**读取客户的 blender（搅拌机）评论 → 根据情感倾向生成个性化的邮件回复**。

### 7.3.1 准备工作

```python
import openai
import os

def get_completion(prompt, model="gpt-3.5-turbo", temperature=0):
    messages = [{"role": "user", "content": prompt}]
    response = openai.ChatCompletion.create(
        model=model,
        messages=messages,
        temperature=temperature,
    )
    return response.choices[0].message["content"]
```

### 7.3.2 输入：客户评论

```python
review = f"""
So, they still had the 17 piece system on seasonal sale for around $49
in the month of November, about half off, but for some reason (call it
price gouging) around the second week of December the prices all went
up to about anywhere from between $70-$89 for the same system...
After about a year, the motor was making a funny noise. I called customer
service but the warranty expired already, so I had to buy another one.
FYI: The overall quality has gone down in these types of products, so
they are kind of counting on brand recognition and consumer loyalty
to maintain sales.
"""
```

> **评论情感分析结果（第 5 章技能）：** negative（负面）

### 7.3.3 Prompt 设计

```python
sentiment = "negative"

prompt = f"""
You are a customer service AI assistant.
Your task is to send an email reply to a valued customer.

Given the customer email delimited by ```, \
Generate a reply to thank the customer for their review.

If the sentiment is positive or neutral, thank them for \
their review.
If the sentiment is negative, apologize and suggest that \
they can reach out to customer service.

Make sure to use specific details from the review.
Write in a concise and professional tone.
Sign the email as `AI customer agent`.

Customer review: ```{review}```
Review sentiment: {sentiment}
"""
```

**Prompt 设计要点：**

1. **角色设定**：明确指定为"客户服务 AI 助手"
2. **任务描述**：生成邮件回复
3. **条件分支**：根据情感倾向（positive/neutral/negative）执行不同的回复策略
4. **细节要求**：必须引用评论中的具体细节（体现个性化）
5. **风格约束**：简洁 + 专业
6. **签名**：固定签名 `AI customer agent`

### 7.3.4 Temperature=0 的输出（稳定版）

```python
response = get_completion(prompt, temperature=0)
```

```
Dear Valued Customer,

Thank you for taking the time to share your detailed feedback about
our 17 piece system. We sincerely apologize for your negative experience
regarding the price increase in December and the quality concerns you
raised about the motor after a year of use.

We understand your frustration and would like to help. Please reach out
to our customer service team at your earliest convenience so we can
address these issues further.

Thank you again for your loyalty and for bringing these matters to
our attention.

Best regards,
AI customer agent
```

**特点：** 专业、得体、结构化、每次运行结果一致。

### 7.3.5 Temperature=0.7 的输出（多样化版）

```python
response = get_completion(prompt, temperature=0.7)
```

每次运行可能得到不同的回复，例如：

```
版本 A（更正式）：
We greatly appreciate your honest feedback about the 17 piece system.
We're sorry to hear about the pricing concerns and the motor issue you
encountered. Our team would like to make things right — please contact
customer service for further assistance.

版本 B（更亲切）：
Hi there! Thanks for sharing such a detailed review of your blender
experience. We're really sorry about the price hike and the motor
problem. We'd love to help — please get in touch with our support
team and we'll sort it out together.
```

**特点：** 语气、措辞、句式都有变化，适合需要"千人千面"的场景。

---

## 7.4 Temperature 的影响对比

| 维度 | Temperature=0 | Temperature=0.7 |
|------|:---:|:---:|
| 输出一致性 | ✅ 完全一致 | ❌ 每次变化 |
| 可靠性 | ✅ 高 | ⚠️ 中等 |
| 创造力 | ❌ 低 | ✅ 高 |
| 适用于生产环境 | ✅ 首选 | ⚠️ 需审核 |
| 可测试性 | ✅ 容易 | ❌ 难 |

### 何时使用哪个

**使用 Temperature=0：**
- 客服回复模板
- 代码生成
- 数据提取/格式化
- 任何需要确定性输出的场景
- 产品功能中的核心逻辑

**使用 Temperature > 0：**
- 营销文案生成
- 创意写作/故事生成
- 个性化推荐
- 社交媒体内容
- 头脑风暴/探索性任务

---

## 7.5 更复杂的扩展应用

### 7.5.1 多条件分支扩展

可以根据评论中的多个维度（情感、问题类型、客户价值）生成差异化的回复：

```python
prompt = f"""
根据以下客户的评论和画像，生成一封个性化的回复邮件：

客户画像：
- 是否是 VIP 客户: {is_vip}
- 购买次数: {purchase_count}
- 评论情感: {sentiment}

评论内容: ```{review}```
"""
```

### 7.5.2 多语言扩展

可以指定回复语言，实现国际化客户支持：

```python
prompt = f"""
Generate a reply email in {language}.
Customer review: ```{review}```
"""
```

### 7.5.3 分段落扩展

可以要求生成包含特定段落的回复：

```python
prompt = f"""
Generate an email with the following sections:
1. Thank the customer for the review
2. Address their specific concerns
3. Offer a solution or next steps
4. Closing with a positive note
"""
```

---

## 7.6 实战技巧与注意事项

### 1. 结合第 5 章的推断能力

本章的实战流程实际上是**第 5 章（推断）+ 第 7 章（扩展）** 的组合：

> **评论输入 → 情感推断（Ch5）→ 根据结果生成回复（Ch7）**

这是一个完整的 AI 客服 pipeline 原型。

### 2. Temperature 与 Token 预算的权衡

- Temperature 越高，输出越不可预测，生产环境需要增加后处理校验
- 高 Temperature 下建议设置更长的 `max_tokens`，因为创造性的输出往往更长

### 3. 个性化的关键在于"具体细节"

在 prompt 中明确要求"使用评论中的具体细节"，这是让 AI 回复看起来不像模板的关键。

### 4. 安全性考虑

- 客服回复应避免过度承诺
- 敏感场景建议设置回复审查步骤
- 签名和联系方式应固定，防止被 prompt 注入篡改

### 5. 迭代测试

对于生产系统，建议：
1. 先用 Temperature=0 开发功能
2. 测试通过后再逐步调高 Temperature
3. 对每个 Temperature 值做 A/B 测试

---

## 7.7 与前后章节的衔接

```
第5章 推断（Inferring） | 理解输入
        ↓
第6章 转换（Transforming） | 变换格式
        ↓
第7章 扩展（Expanding） | 生成输出
        ↓
第8章 聊天机器人（Chatbot） | 多轮交互
```

> **本章中的评论→回复 pipeline 将在第 8 章演化为完整的对话式客服机器人。**

---

## 关键 Takeaways

| 概念 | 要点 |
|------|------|
| **扩展（Expanding）** | 短文本 → 长文本的生成能力 |
| **Temperature=0** | 确定性输出，适合生产环境 |
| **Temperature>0** | 多样性输出，适合创意场景 |
| **实战案例** | 评论情感推断 → 个性化客服回复生成 |
| **Prompt 设计** | 角色设定 + 条件分支 + 细节引用 + 风格约束 |
| **整体流程** | 推断 + 扩展 构成 AI 客服核心 pipeline |

> **核心洞见：** 扩展能力让 LLM 从"分析器"升级为"生成器"。结合第 5 章的推断能力，可以构建端到端的 AI 客服、内容生成、个性化营销等应用。而 Temperature 参数则是在"稳定可靠"与"创意多样"之间调节的关键旋钮。
