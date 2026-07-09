---
title: 第2章 提示工程指导原则 — Guidelines for Prompting
date: 2026-07-09
tags: [Prompt Engineering, LLM, ChatGPT, 吴恩达, 读书笔记]
author: enheng1238
---

# 第2章 提示工程指导原则

> **课程来源**: [ChatGPT Prompt Engineering for Developers] — Andrew Ng (吴恩达) & Isa Fulford, DeepLearning.AI

---

## 一、两大核心原则

| 原则 | 核心思想 | 一句话总结 |
|------|----------|------------|
| **原则一**：编写清晰、具体的指令 | 提示词应足够详细，不模糊不简略 | **指令明确，不要吝啬措辞** |
| **原则二**：给模型思考的时间 | 让模型逐步推理，而不是仓促给出结论 | **慢想深究，拒绝急推** |

---

## 二、原则一：编写清晰、具体的指令（Write Clear and Specific Instructions）

> 模型不会读心术。提示词越模糊，输出越随机。清晰 ≠ 简短，详细 ≠ 啰嗦。

### 策略 1：使用分隔符清晰标识输入的不同部分

用 `"""`, `---`, `<>`, `XML 标签` 等分隔符将指令与输入文本明确分开，防止 Prompt Injection。

```python
import openai

# ❌ 不推荐：没有分隔符，模型可能混淆指令和文本
prompt = "将以下文本总结为一句话：今天天气真好啊，适合出去散步。"

# ✅ 推荐：使用三重引号分隔
prompt = """
将以下文本总结为一句话：

文本：\"\"\"今天天气真好啊，适合出去散步。\"\"\"
"""

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[{"role": "user", "content": prompt}]
)
print(response.choices[0].message["content"])
# 输出: 今天天气很好，适合外出散步。
```

**其他分隔符示例：**

| 分隔符 | 用法 |
|--------|------|
| `"""文本"""` | 三重引号，最常用 |
| `<tag>文本</tag>` | XML/HTML 标签 |
| `---文本---` | 三个连字符 |
| ``` ```文本``` ``` | 代码块标记 |

---

### 策略 2：要求结构化输出（JSON、HTML 等）

指定输出格式使下游程序更容易解析，减少解析错误。

```python
prompt = """
从以下文本中提取信息，并以 JSON 格式输出，包含三个字段：
- name（姓名）
- age（年龄）
- city（城市）

文本：\"\"\"张三今年28岁，住在北京，是一名软件工程师。\"\"\"

请只输出 JSON，不要附加其他文字。
"""

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[{"role": "user", "content": prompt}]
)

import json
data = json.loads(response.choices[0].message["content"])
print(data["name"])   # 张三
print(data["age"])    # 28
print(data["city"])   # 北京
```

> **💡 提示**：指定 `请只输出 JSON，不要附加其他文字` 可避免模型在 JSON 外额外输出解释性文字。

---

### 策略 3：检查条件是否满足（条件验证）

在提示词中明确说明：如果输入不符合要求，应该怎么做。这可以防止模型在无效输入上强行生成答案。

```python
prompt = """
你将从用户输入的文本中提取指令并执行。
如果文本中没有明确包含"步骤"或"指令"等关键词，请回复：
"未检测到有效指令，请提供包含步骤的文本。"

文本：\"\"\"今天天气不错。\"\"\"
"""

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[{"role": "user", "content": prompt}]
)
print(response.choices[0].message["content"])
# 输出: 未检测到有效指令，请提供包含步骤的文本。
```

**实战场景**：
- 客服系统：要求输入必须包含订单号，否则返回错误提示
- 数据清洗：要求输入必须符合特定格式，否则跳过
- 安全审查：检测输入是否包含敏感词，拒绝处理

---

### 策略 4：少样本提示（Few-Shot Prompting）

在提示词中提供几个输入→输出的示例，让模型"照猫画虎"。

```python
prompt = """
请将以下中文词语翻译为英文。以下是一些示例：

中文: 猫 -> 英文: cat
中文: 狗 -> 英文: dog
中文: 鸟 -> 英文: bird

中文: 电脑 ->
"""

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[{"role": "user", "content": prompt}]
)
print(response.choices[0].message["content"])
# 输出: computer
```

**少样本 vs 零样本对比**：

| 方式 | 示例数量 | 适用场景 | 准确率 |
|------|----------|----------|--------|
| **零样本** | 0 | 简单通用任务 | 中等 |
| **少样本** | 1-5 | 格式要求严格/领域特殊 | 较高 |
| **多样本** | 5+ | 复杂 pattern 学习 | 最高 |

---

## 三、原则二：给模型思考的时间（Give the Model Time to Think）

> 模型和人类一样，被催促时会犯错。引导模型分步推理可以获得更准确的答案。

### 策略 5：指定完成任务所需的步骤

将复杂任务拆解为多个明确的子步骤，引导模型一步步执行。

```python
prompt = """
请按以下步骤执行：
1. 将以下中文文本翻译成英文。
2. 用一句话概括翻译后的英文文本。
3. 列出该英文文本中出现的所有名词。

文本：\"\"\"深度学习是机器学习的一个分支，它使用多层神经网络来学习数据的表示。\"\"\"

请严格按照上述步骤输出结果，每步用 [步骤X] 标记。
"""

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[{"role": "user", "content": prompt}]
)
print(response.choices[0].message["content"])
# 输出示例:
# [步骤1] Deep learning is a branch of machine learning that uses multi-layer neural networks to learn representations of data.
# [步骤2] Deep learning uses multi-layer neural networks to learn data representations.
# [步骤3] deep learning, branch, machine learning, multi-layer neural networks, representations, data
```

**进阶技巧**：使用 Inner Monologue（内心独白）让模型在输出最终答案前先写下推理过程，然后对用户隐藏中间步骤：

```python
prompt = """
用户会提供一段文本。请按以下步骤执行：
1. 分析文本中的情绪（积极/消极/中性）。
2. 根据情绪分析结果，决定回复策略。
3. 只输出最终回复，不要输出分析过程。

用户文本：\"\"\"你们的产品质量太差了，用了一个月就坏了！\"\"\"
"""
```

---

### 策略 6：引导模型先自己推理，再下结论（Chain-of-Thought）

这是最关键的一个策略。在一个问答任务中，不要直接问模型"答案是对还是错"，而是让模型**先从零推导自己的答案**，再将你的答案和它的答案对比。

```python
# ❌ 错误做法：让模型直接判断对错
prompt = """
判断以下解答是否正确：
问题：一个球和一个球拍共 1.10 美元，球拍比球贵 1.00 美元，球多少钱？
解答：球 0.10 美元。

答案（对/错）：
"""
# 模型很可能回答"对"——因为 0.10 听起来直觉正确

# ✅ 正确做法：让模型先自己推导
prompt = """
问题：一个球和一个球拍共 1.10 美元，球拍比球贵 1.00 美元，球多少钱？

请先自己一步一步计算，得到答案后，再判断以下解答是否正确：
解答：球 0.10 美元。
"""

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[{"role": "user", "content": prompt}]
)
print(response.choices[0].message["content"])
# 输出会先推导：设球为 x，球拍为 x+1.00，则 2x+1.00=1.10 → x=0.05
# 然后判断原解答错误
```

**为什么有效？**
- 直接判断 → 模型走捷径，依赖表面模式匹配
- **链式思维（Chain-of-Thought）** → 模型必须经过完整推理链，不容易被误导

---

## 四、幻觉问题（Hallucinations）

### 什么是幻觉？

模型在回答时**编造事实**，生成看似合理但完全错误的内容。

> **经典例子**：问模型"Boie 牌牙刷的说明"，模型会生成一份详细但完全虚构的说明书，因为该品牌并不存在。

### 如何减少幻觉？

最有效的方法：**要求模型引用原文来源**。

```python
# 基础指令（容易产生幻觉）
prompt = """
告诉我 Boie 牌牙刷的使用说明。
"""

# ✅ 防幻觉指令：要求引用原文
prompt = """
请从以下产品描述中提取牙刷的使用说明。
如果你找到说明，请逐字引用原文并标注行号。
如果产品描述中没有明确的使用说明，请回答：
"产品描述中未包含使用说明。"

产品描述：\"\"\"这是一款环保竹制牙刷，刷毛柔软，适合敏感牙龈。\"\"\"
"""

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[{"role": "user", "content": prompt}]
)
print(response.choices[0].message["content"])
# 输出: 产品描述中未包含使用说明。
```

**防幻觉 Checklist**：

| 措施 | 说明 |
|------|------|
| ✅ 引用原文 | 要求输出必须包含原文引用，附行号 |
| ✅ 限定范围 | "只根据以下文本回答"，添加分隔符 |
| ✅ 设置"未知"兜底 | "如果不知道，就说不知道" |
| ✅ 少样本 + 否定示例 | 给一个"不知道就承认"的示例 |
| ✅ 问不确定时直接回答"不知道" | 降低模型强行填充的动机 |

---

## 五、两大原则速查表

```
原则一：编写清晰、具体的指令
├── 策略 1：使用分隔符（""", ---, <tag>）
├── 策略 2：要求结构化输出（JSON, HTML）
├── 策略 3：检查条件是否满足
└── 策略 4：少样本提示

原则二：给模型思考的时间
├── 策略 5：指定子步骤
└── 策略 6：引导模型先自己推理（CoT）
```

---

## 六、常见错误与最佳实践

### ❌ 常见错误

1. **提示词过短**：只说"总结一下"，不指定长度、语言、格式
2. **没有兜底逻辑**：输入不合法时模型仍然硬编答案
3. **直接判断对错**：不给模型推理空间
4. **不处理幻觉**：盲目相信模型输出的"事实"

### ✅ 最佳实践

1. **先写草稿，迭代优化**：没有完美的第一版提示词
2. **使用分隔符隔离输入与指令**：安全 + 清晰
3. **指定输出格式**：便于程序消费
4. **提供示例**：少样本提示在格式控制上效果显著
5. **设定期望行为**：告诉模型"不确定时就说不知道"

---

> **📌 核心心法**：
> 写提示词就像给一个聪明但有缺陷的新人下指令——
> **越清晰、越结构化、越有回旋余地，模型的输出就越可控。**
