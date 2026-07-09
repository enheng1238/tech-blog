---
title: "第4章：摘要（Summarizing）"
date: 2026-07-09
tags: [Prompt Engineering, LLM, ChatGPT, 吴恩达, 读书笔记]
author: enheng1238
---

# 第4章：摘要（Summarizing）

> 吴恩达 & OpenAI — ChatGPT Prompt Engineering for Developers

## 概述

本章聚焦 LLM 的**文本摘要（Summarizing）**能力。核心议题是：如何让 LLM 从一段文本中提取关键信息并输出简洁摘要，以及**如何通过提示词控制摘要的侧重点**——这在实际应用中至关重要。

---

## 4.1 基础摘要（Basic Summarization）

最基本的用法：将一段较长的文本输入给 LLM，要求其输出一个简短的概要。

### 核心 Prompt 模板

```
对以下文本进行摘要，用一句话概括核心内容：

[文本内容]
```

### 示例

> **输入文本：**（一段 200 字的产品评论）
> ​"这款蓝牙耳机音质非常出色，低音浑厚，中高音通透。佩戴舒适，连续使用 4 小时也不会觉得耳朵疼。电池续航大约 8 小时，足够日常通勤使用。唯一的缺点是降噪效果一般，在嘈杂环境中不太够用。总体来说性价比很高。"
>
> **LLM 输出：**
> 一款音质出色、佩戴舒适的蓝牙耳机，续航 8 小时，性价比高，但降噪效果一般。

### Python 示例

```python
import openai

openai.api_key = "your-api-key-here"

text = """
这款蓝牙耳机音质非常出色，低音浑厚，中高音通透。
佩戴舒适，连续使用 4 小时也不会觉得耳朵疼。
电池续航大约 8 小时，足够日常通勤使用。
唯一的缺点是降噪效果一般，在嘈杂环境中不太够用。
总体来说性价比很高。
"""

prompt = f"""
对以下文本进行摘要，输出一句话概括核心内容：

{text}
"""

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": "user", "content": prompt}
    ],
    max_tokens=80,
    temperature=0,
)

print(response.choices[0].message.content.strip())
# 输出: "这款蓝牙耳机音质出色、佩戴舒适、续航持久、性价比高，但降噪效果一般。"
```

---

## 4.2 控制输出长度（Word / Sentence Limit）

可以通过 prompt 精确控制摘要的长度：

| 约束方式 | 示例 |
|---------|------|
| **字数限制** | "用不超过 30 个字总结" |
| **句数限制** | "用一句话总结" / "用最多 3 句话总结" |
| **要点限制** | "输出 3 个核心要点" |

### 示例

**Prompt：**
```
用不超过 20 个字总结以下内容：

杭州是中国浙江省省会，以其美丽的西湖和悠久的历史文化闻名，
是中国著名的旅游城市和电子商务中心。
```

**LLM 输出：**
杭州：浙江省会，以西湖和电商闻名。

> **注意：** 字数限制在 LLM 中不是精确的——模型没有「计数」能力。建议用句数限制（"一句话""最多三句话"）作为替代，可靠性更高。

---

## 4.3 有侧重的摘要（Summarizing with a Focus）

本章**最核心的技巧**：控制 LLM 的摘要**关注点**。同样的文本，不同的关注方向会得到完全不同的摘要。

### 为什么需要侧重？

同一段文本（如产品评论）包含多个维度的信息，不同角色关心的内容不同：

| 角色 | 关注点 |
|------|--------|
| **产品经理** | 用户对功能的反馈、改进建议 |
| **客服主管** | 用户投诉、服务体验问题 |
| **销售** | 用户购买意愿、价格敏感度 |
| **市场** | 用户情感倾向、关键词 |
| **工程师** | Bug 报告、性能问题 |

### 基本 Prompt 模板

```
对以下文本进行摘要。

请特别关注 [X 方面]，输出 2–3 个要点：

[文本内容]
```

### 产品经理视角示例

> **Prompt：**
> 对以下产品评论进行摘要，**特别关注产品功能的反馈和改进建议**，输出 2 个要点：
>
> [同一条蓝牙耳机评论]
>
> **输出：**
> - 优点：音质出色、佩戴舒适、续航够用
> - 缺点：降噪效果不佳，建议在下一代产品中加强主动降噪

### 客服视角示例

> **Prompt：**
> 对以下产品评论进行摘要，**特别关注用户不满或投诉内容**：
>
> [同一条蓝牙耳机评论]
>
> **输出：**
> 用户反映降噪效果一般，在嘈杂环境中体验不佳。

### 工程师视角示例

> **Prompt：**
> 对以下产品评论进行摘要，**特别关注技术指标和性能数据**：
>
> [同一条蓝牙耳机评论]
>
> **输出：**
> - 音质：低音浑厚，中高音通透
> - 佩戴：舒适，4 小时无痛感
> - 续航：约 8 小时
> - 降噪：效果一般

### Python 代码对比

```python
import openai

openai.api_key = "your-api-key-here"

review = """
这款蓝牙耳机音质非常出色，低音浑厚，中高音通透。
佩戴舒适，连续使用 4 小时也不会觉得耳朵疼。
电池续航大约 8 小时，足够日常通勤使用。
唯一的缺点是降噪效果一般，在嘈杂环境中不太够用。
总体来说性价比很高。
"""

# ── 无侧重的通用摘要 ──
prompt_generic = f"""
对以下文本进行摘要，输出一句话：

{review}
"""

response_generic = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[{"role": "user", "content": prompt_generic}],
    max_tokens=60,
    temperature=0,
)
print("【通用摘要】")
print(response_generic.choices[0].message.content.strip())

# ── 有侧重的摘要（工程师视角） ──
prompt_focused = f"""
对以下产品评论进行摘要。

请特别关注技术指标和性能参数（音质、续航、降噪、佩戴），输出要点：

{review}
"""

response_focused = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[{"role": "user", "content": prompt_focused}],
    max_tokens=100,
    temperature=0,
)
print("\n【技术侧重点摘要】")
print(response_focused.choices[0].message.content.strip())
```

---

## 4.4 "提取" vs "摘要"（Extract vs Summarize）

这是一个细微但重要的区别：

| 操作 | 含义 | 适用场景 |
|------|------|---------|
| **摘要（Summarize）** | 用自己的话重述原文要点 | 生成可读性高的概述 |
| **提取（Extract）** | 从原文中摘取原句或特定字段 | 保留原始措辞、证据引用 |

### 示例对比

**Prompt（提取式）：**
```
从以下评论中提取所有关于「缺点」和「改进建议」的原句：

[评论原文]
```

**输出特点：**
- 保留原文措辞，不进行改写
- 适合需要引用原文的场景（如内部报告、问题跟踪）

**Prompt（摘要式）：**
```
总结以下评论中提到的缺点的核心：

[评论原文]
```

**输出特点：**
- 进行概括和重组
- 更简洁、可读性更高
- 可能丢失某些原文细节

### 如何选择？

> **规则：** 需要准确性/可引用的 → 用 Extract；需要简洁/易读的 → 用 Summarize。

---

## 4.5 多文本批量摘要处理

对大量文本（如多篇评论、多篇文章）逐条做摘要时，可以用循环 + Prompt 模板批量处理：

```python
import openai

openai.api_key = "your-api-key-here"

reviews = [
    "评论一：音质好，但太贵了。",
    "评论二：续航很棒，降噪垃圾。",
    "评论三：颜值高，但容易沾指纹。",
    # ... 可以有更多
]

summaries = []

for i, review in enumerate(reviews, 1):
    prompt = f"""
    用一句话总结以下评论：

    {review}
    """
    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[{"role": "user", "content": prompt}],
        max_tokens=40,
        temperature=0,
    )
    summary = response.choices[0].message.content.strip()
    summaries.append(summary)
    print(f"评论 {i}: {summary}")
```

> **优化建议：** 在高吞吐场景下，可将多条评论合并到一个 prompt 中请求批量摘要，以减少 API 调用次数和 Token 消耗。

---

## 4.6 实战技巧

### 1. 指定输出格式

为了让摘要结果可被程序化处理，可以指定结构化输出格式：

```
用以下 JSON 格式输出摘要：
{
  "summary": "一句话总结",
  "key_points": ["要点1", "要点2", "要点3"],
  "sentiment": "正面|负面|中性"
}
```

### 2. 温度参数（Temperature）

| Temperature | 效果 | 适合场景 |
|-------------|------|---------|
| **0** | 确定性输出，每次结果一致 | 生产环境摘要任务（推荐） |
| 0.3–0.5 | 稍有变化 | 创意性总结 |
| > 0.7 | 高度不一致 | **不推荐**用于摘要 |

### 3. 使用分隔符避免上下文干扰

```python
prompt = f"""
对以下文本（由三个反引号分隔）进行摘要：

```{text}```
"""
```

### 4. 多轮迭代优化

先输出通用摘要 → 再追问具体维度：

```
第一轮：对文本进行摘要
第二轮：在上次摘要基础上，特别关注「技术指标」
```

---

## 4.7 与第 5 章的衔接

第 4 章专注于**浓缩信息（摘要）**，第 5 章专注于**推理信息（推断）**。

> **第 4 章：浓缩** → 把长文本变短
> **第 5 章：推断** → 从文本中推理出未明说的信息

两者组合可以形成强大的信息处理 pipeline：先摘要降噪 → 再推断提取深层信息。

---

## 关键 Takeaways

| 概念 | 要点 |
|------|------|
| **基础摘要** | 一句话或几句话说清核心内容 |
| **长度控制** | 用句数限制（而非字数限制）更可靠 |
| **有侧重的摘要（⭐ 核心）** | 通过 prompt 指定关注维度，不同角色获得不同摘要 |
| **Extract vs Summarize** | Extract 保留原句用于引用；Summarize 重述用于易读 |
| **批量处理** | 循环或合并 prompt 处理多条文本 |
| **温度参数** | 摘要任务推荐 temperature=0 |

> **核心洞见：** 摘要任务的关键不是「让 LLM 总结」，而是「告诉 LLM 从什么角度总结」。同样一段文本，针对产品经理、客服、工程师给出不同的关注点 prompt，得到的摘要质量天差地别。

*参考来源：吴恩达（Andrew Ng）& OpenAI — "ChatGPT Prompt Engineering for Developers" 第 4 章*
