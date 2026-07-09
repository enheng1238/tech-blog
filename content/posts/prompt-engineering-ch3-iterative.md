---
date: 2026-07-09
tags: [Prompt Engineering, LLM, ChatGPT, 吴恩达, 读书笔记]
author: enheng1238
---

# 第3章：迭代式 Prompt 开发 (Iterative Prompt Development)

> **课程**: ChatGPT Prompt Engineering for Developers — 吴恩达 (Andrew Ng) & OpenAI
> **核心思想**: 好的 Prompt 不是一次写成的，而是**迭代打磨**出来的。

---

## 为什么需要迭代？

LLM 无法一次性读懂你的全部意图。第一次写的 Prompt 往往：
- 太长或太短
- 格式不对
- 遗漏关键字段
- 包含无关信息

**解决方式**：像写代码一样，跑一轮 → 看结果 → 改 Prompt → 再跑。

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│  Idea    │ ──▶ │  Prompt  │ ──▶ │   LLM    │ ──▶ │ Analyze  │
└──────────┘     └──────────┘     └──────────┘     └──────────┘
                                                       │
                                          ┌────────────┘
                                          ▼
                                    ┌──────────┐
                                    │ Refine   │
                                    │ Prompt   │
                                    └──────────┘
```

---

## 实战案例：商品描述 → 结构化产品卡片

### 原始输入文本

```text
原文（关于一把椅子）：
"Got this for my home office. The chair is very comfortable.
It's an ergonomic mesh chair with adjustable armrests and lumbar support.
Assembly was straightforward — took about 15 minutes.
The mesh back keeps me cool during long work sessions. It's sleek black,
looks professional, and supports up to 300 lbs. I contacted customer service
about a missing screw and they shipped one immediately. Highly recommend!"
```

---

### 🔁 第1轮 — 原始 Prompt

**Prompt**:
```python
prompt = f"""
从以下产品评论中提取产品信息：
```{text}```
"""
```

**问题**: 输出太"自由发挥"了 —— 一段自然语言段落，没有结构，**长度不受控**，甚至把用户评价也混了进来。

**教训**: 告诉 LLM 要做什么，但没告诉**怎么做**，它就按自己理解来。

---

### 🔁 第2轮 — 限制长度

**Prompt**:
```python
prompt = f"""
从以下产品评论中提取产品信息，限制在 50 词以内：
```{text}```
"""
```

**问题**: 强行限词导致关键字段被裁掉（尺寸、材质、承重等）。拼凑感强，可读性差。

**教训**: 光限长度不够，要明确**输出什么内容**。

---

### 🔁 第3轮 — 指定输出格式

**Prompt**:
```python
prompt = f"""
从以下评论中提取产品信息，并按以下格式输出：

- 产品名称：
- 主要特性：
- 材质：
- 尺寸/承重：
- 用户评价摘要：

评论：
```{text}```
"""
```

**效果**: 结构清晰了，但仍有问题：
- 字段粒度不够细（"主要特性" 里包含了调节功能、散热、安装等）
- 部分字段缺少明确指引（材质没有显式出现在原文，LLM 开始"脑补"）

**教训**: 格式有了，但**字段定义**还不够精确。

---

### 🔁 第4轮 — 精确定义 + 条件处理

**Prompt**:
```python
prompt = f"""
从以下评论中提取产品信息。若某字段在原文中未提及，请明确写"未提及"。

输出 JSON 格式，包含以下字段：
1. name: 产品名称/型号
2. category: 产品类别（如椅子、桌子）
3. primary_features: 列表，至少包括：材质、是否可调节、支撑类型
4. dimensions: 尺寸信息（未提及则填 null）
5. weight_capacity: 承重上限（未提及则填 null）
6. assembly: 安装难度描述
7. pros: 正面评价（列表）
8. cons: 负面评价或不足（列表）
9. summary: 一句话总结

评论：
```{text}```
"""
```

**输出**:
```json
{
  "name": "人体工学网面办公椅",
  "category": "椅子",
  "primary_features": ["网面材质", "可调节扶手", "腰部支撑"],
  "dimensions": null,
  "weight_capacity": "300 lbs",
  "assembly": "简单，约15分钟",
  "pros": ["舒适", "透气", "外观专业", "客服响应好"],
  "cons": ["缺一颗螺丝（已补发）"],
  "summary": "一款舒适透气的网面人体工学椅，安装简单，承重300磅，客服体验好。"
}
```

✅ **达到预期**。

---

## 迭代方法论总结

| 步骤 | 做什么 | 典型问题 |
|------|--------|----------|
| 1. 写第一个 Prompt | 清晰描述任务 | 输出太随意、缺少结构 |
| 2. 分析输出 | 检查长度、格式、完整性 | 信息缺失或格式错误 |
| 3. 精炼指引 | 加约束、加格式、加边界条件 | 字段粒度不够 |
| 4. 重复 2↔3 | 直到满意 | — |

### 四个常用精炼方向

1. **长度控制** — `限制在 X 词以内` / `不超过 Y 句话`
2. **格式约束** — `输出 JSON` / `用表格` / `按编号列表`
3. **字段定义** — 明确列出每个字段及其含义
4. **边界条件** — `若原文未提及，写"未提及"` / `若不确定，留空`

---

## 完整可运行代码

```python
import openai  # pip install openai
import os

# 先设定 API Key（推荐用环境变量）
openai.api_key = os.getenv("OPENAI_API_KEY")

# ====== 工具函数 ======
def ask_gpt(prompt, model="gpt-3.5-turbo", temperature=0.0):
    """封装一次 ChatCompletion 调用"""
    response = openai.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": prompt}],
        temperature=temperature,
    )
    return response.choices[0].message.content


# ====== 原始输入 ======
review_text = """
Got this for my home office. The chair is very comfortable.
It's an ergonomic mesh chair with adjustable armrests and lumbar support.
Assembly was straightforward — took about 15 minutes.
The mesh back keeps me cool during long work sessions. It's sleek black,
looks professional, and supports up to 300 lbs. I contacted customer service
about a missing screw and they shipped one immediately. Highly recommend!
"""


# ====== 第1轮：基本信息提取 ======
prompt_v1 = f"""
从以下产品评论中提取产品信息：
```{review_text}```
"""
print("=== 第1轮 ===")
print(ask_gpt(prompt_v1))
print()


# ====== 第2轮：限制长度 ======
prompt_v2 = f"""
从以下产品评论中提取产品信息，限制在 50 词以内：
```{review_text}```
"""
print("=== 第2轮（限50词）===")
print(ask_gpt(prompt_v2))
print()


# ====== 第3轮：指定格式 ======
prompt_v3 = f"""
从以下评论中提取产品信息，并按以下格式输出：

- 产品名称：
- 主要特性：
- 材质：
- 尺寸/承重：
- 用户评价摘要：

评论：
```{review_text}```
"""
print("=== 第3轮（指定格式）===")
print(ask_gpt(prompt_v3))
print()


# ====== 第4轮：精确定义 + JSON + 缺省处理 ======
prompt_v4 = f"""
从以下评论中提取产品信息。若某字段在原文中未提及，请明确写"未提及"。

输出 JSON 格式，包含以下字段：
1. name: 产品名称/型号
2. category: 产品类别（如椅子、桌子）
3. primary_features: 列表，至少包括：材质、是否可调节、支撑类型
4. dimensions: 尺寸信息（未提及则填 null）
5. weight_capacity: 承重上限（未提及则填 null）
6. assembly: 安装难度描述
7. pros: 正面评价（列表）
8. cons: 负面评价或不足（列表）
9. summary: 一句话总结

评论：
```{review_text}```
"""
print("=== 第4轮（精确定义+JSON）===")
print(ask_gpt(prompt_v4))
```

> ⚠️ 运行前需设置环境变量 `OPENAI_API_KEY`。

---

## 关键 takeaways

| # | 要点 |
|---|------|
| 1 | **Prompt 是迭代的** — 没有人第一次就写出完美 Prompt |
| 2 | **从一个清晰的 Prompt 开始**，不要追求一步到位 |
| 3 | **分析 LLM 的输出** — 错在哪就改哪 |
| 4 | **逐步增加约束** — 长度 → 格式 → 字段 → 边界条件 |
| 5 | **设定 `temperature=0`** 在开发阶段，确保输出可复现 |
| 6 | **用系统化的方法调试 Prompt**，就像调试代码一样 |

---

## 实用英语 Prompt 模板（可直接复用）

```text
# 迭代式精炼
Step 1: "Extract product information from the following review."
Step 2: "Keep it under 50 words."
Step 3: "Use a bullet-point format with these fields: name, features, material, dimensions, summary."
Step 4: "Output JSON. If a field is missing, use null. Include: name, category, features (list), dimensions, weight_capacity, assembly, pros (list), cons (list), summary."
```

---

## 延伸阅读

- [ChatGPT Prompt Engineering for Developers (DeepLearning.AI)](https://www.deeplearning.ai/short-courses/chatgpt-prompt-engineering-for-developers/)
- [OpenAI Cookbook — Techniques to improve reliability](https://cookbook.openai.com/articles/techniques_to_improve_reliability)
- 本书其他章节笔记参见本目录下的 `prompt-engineering-ch*.md` 系列
