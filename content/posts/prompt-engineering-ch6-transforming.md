---
title: "第6章：文本转换（Transforming）"
date: 2026-07-09
tags: [Prompt Engineering, LLM, ChatGPT, 吴恩达, 读书笔记]
author: enheng1238
---

# 第6章：文本转换（Transforming）

> 课程：ChatGPT Prompt Engineering for Developers  
> 讲师：Andrew Ng (DeepLearning.AI) & Isa Fulford (OpenAI)

---

## 概述

大语言模型（LLM）非常擅长将输入转换为不同格式的输出。与传统的正则表达式或模板方法相比，LLM 在翻译、拼写校正、格式转换等任务上表现得更加**准确、灵活、高效**。

本章涵盖四类核心转换能力：

1. **语言翻译** —— 多语种互译、多目标语言翻译、语言识别
2. **语气/风格转换** —— 口语→商务信函、正式/非正式转换
3. **格式转换** —— JSON → HTML 表格等跨格式转换
4. **拼写与语法校正** —— 校对、纠错、变更对比

---

## 1. 语言翻译（Translation）

### 1.1 单语翻译

将英文翻译为西班牙语：

```python
prompt = f"""
Translate the following English text to Spanish: \
```Hi, I would like to order a blender```
"""
response = get_completion(prompt)
print(response)
```

### 1.2 多目标翻译

同时将一句话翻译为多个语种：

```python
prompt = f"""
Translate the following text to French and Spanish and English pirate:
"I would like to order a blender"
"""
response = get_completion(prompt)
print(response)
```

### 1.3 语言识别

让模型判断输入文本属于哪种语言：

```python
prompt = f"""
Tell me which language this is:
```Combien coûte le lampadaire?```
"""
response = get_completion(prompt)
print(response)
```

### 1.4 正式/非正式风格翻译

控制译文风格，例如将英文句子同时翻译为正式和非正式中文：

```python
prompt = f"""
Translate the following text to Chinese in both the \
formal and informal forms:
'Would you like to dinner with me?'
"""
response = get_completion(prompt)
print(response)
```

**预期输出：**

```
Formal: 您愿意与我共进晚餐吗？
Informal: 你想跟我一起吃晚饭吗？
```

> 💡 **实用场景**：将用户评论从多语种统一翻译成同一种语言，方便后续的批量分析（情感分类、主题提取）。

---

## 2. 语气/风格转换（Tone & Style Transformation）

LLM 可以将一种语气/风格转换为另一种，无需编写复杂的 NLP 流水线。

### 示例：口语 → 商务信函

```python
prompt = f"""
Translate the following from slang to a business letter:
'Dude, This is Joe, check out this spec on this standing lamp.'
"""
response = get_completion(prompt)
print(response)
```

**预期输出：**

```
Dear Sir/Madam,

I am writing to bring to your attention a standing lamp that I believe
may be of interest to you. Please find attached the specifications
for your review.

Thank you for your time and consideration.

Sincerely,
Joe
```

> 💡 **实用场景**：自动将聊天记录、客户反馈或口语评论转换为正式的商务文档或邮件。

---

## 3. 格式转换（Format Conversion）

LLM 可以将数据从一种格式转换为另一种格式，典型场景是 **JSON → HTML 表格**。

```python
data_json = { "resturant employees" :[\
    {"name":"Shyam", "email":"shyamjaiswal@gmail.com"},\
    {"name":"Bob", "email":"bob32@gmail.com"},\
    {"name":"Jai", "email":"jai87@gmail.com"}\
]}

prompt = f"""
Translate the following python dictionary from JSON to an HTML \
table with column headers and title: {data_json}
"""
response = get_completion(prompt)
print(response)
```

**预期输出**（HTML 表格）：

```html
<table>
  <caption>Restaurant Employees</caption>
  <thead>
    <tr><th>Name</th><th>Email</th></tr>
  </thead>
  <tbody>
    <tr><td>Shyam</td><td>shyamjaiswal@gmail.com</td></tr>
    <tr><td>Bob</td><td>bob32@gmail.com</td></tr>
    <tr><td>Jai</td><td>jai87@gmail.com</td></tr>
  </tbody>
</table>
```

> 💡 **实用场景**：将 API 返回的 JSON 数据快速渲染为可视化的 HTML 或 Markdown 表格，省去手动拼装模板的繁琐工作。

---

## 4. 拼写检查与语法校正（Spelling & Grammar Correction）

### 4.1 基础校对

```python
text = f"""
Got this for my daughter for her birthday cuz she keeps taking \
mine from my room. Yes that also has the light at the back of the \
stand that goes into the body of the book, even then it's \
not that big. It takes AAA batteries, not included. \
I found this hard to use because it's so small.
"""
prompt = f"""
Proofread and correct this review: ```{text}```
"""
response = get_completion(prompt)
print(response)
```

#### 中文校对示例

```python
text = f"""郭艾伦是我最稀饭的球星之一"""
prompt = f"""校对并纠正这段文本: ```{text}```"""
response = get_completion(prompt)
print(response)
```

**预期输出：**

```
郭艾伦是我最喜欢的球星之一。
```

> ⚠️ **注意**：中文校对效果因模型而异，有时会过度修正或修正错误，建议对中文任务增加人工复核。

### 4.2 无错误时的兜底输出

指定当没有发现错误时模型的输出，避免模型凭空修改：

```python
prompt = f"""
Proofread and correct this review: ```{text}```
If you don't find any errors, just say "No errors found".
"""
```

### 4.3 可视化变更差异

使用 `redlines` 库高亮显示修改前后的差异：

```python
from redlines import Redlines
from IPython.display import display, Markdown

diff = Redlines(text, response)
display(Markdown(diff.output_markdown))
```

> 💡 **实用场景**：自动校对用户评论、文档草稿、翻译初稿，并可视化展示修改内容，提升审校效率。

---

## 核心要点总结

| 转换类型 | 关键 Prompt 技巧 | 典型应用 |
|---------|----------------|---------|
| **语言翻译** | 指定源语言和目标语言；可同时翻译为多种语言 | 多语种评论统一分析、国际化产品 |
| **风格转换** | 描述源风格和目标风格（口语→书面、正式→非正式） | 客服消息转正式文档、语气调整 |
| **格式转换** | 描述输入格式和期望输出格式（JSON→HTML, CSV→Markdown） | 数据可视化、API 输出格式化 |
| **拼写/语法校正** | 使用 "Proofread and correct" 指令；提供兜底输出 | 评论审核、文档校对、翻译审校 |

- LLM 的文本转换能力**显著优于**传统规则方法（正则表达式、模板引擎），特别是在处理模糊、多变、跨语种任务时。
- 结合 Prompt 原则（清晰指令 + 分隔符 + 指定输出格式），转换结果更可控。
- 通过这些转换能力，可以将 LLM 嵌入数据处理管道（ETL）、国际化工作流、内容生成管线中，替代大量手工程序。

---

*下一章：第7章 —— 文本扩展（Expanding）*
