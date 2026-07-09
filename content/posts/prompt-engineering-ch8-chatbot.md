---
date: 2026-07-09
tags: [Prompt Engineering, LLM, ChatGPT, 吴恩达, 读书笔记]
author: enheng1238
filename: prompt-engineering-ch8-chatbot.md
---

# 第8章：聊天机器人（Chatbot）

> 吴恩达《ChatGPT Prompt Engineering for Developers》第8章笔记
> 核心主题：用 System / User / Assistant 三种角色构建多轮对话机器人

---

## 1. 三种消息角色

| 角色 | 字段名 | 含义 | 典型用途 |
|------|--------|------|----------|
| 系统消息 | `system` | 设定 AI 的行为、身份、约束 | 给机器人设定人格、规则、输出格式 |
| 用户消息 | `user` | 用户的输入 / 提问 | 每次用户说话的内容 |
| 助手消息 | `assistant` | AI 的回复 | 模型生成的回答；也可**手动注入**作为上下文示例 |

### 核心要点

- **System Message 只设一次，通常放在对话开头**。它像"幕后指令"，用户看不到，但模型全程遵守。
- **User Message 是每轮用户的输入**。
- **Assistant Message 是模型的回复**。在后续轮次中，之前的 assistant 回复会被当作历史上下文继续传给模型。
- **可以手动伪造 Assistant Message** 来给模型做 few-shot 示范或引导对话走向。

---

## 2. 多轮对话的消息维护

### 基本结构

每次 API 调用都**把整段对话历史**（messages 列表）传进去：

```python
messages = [
    {"role": "system", "content": "你是一个友好的披萨店客服助手。"},
    {"role": "user",   "content": "你好，我想点一份披萨。"},
    {"role": "assistant", "content": "您好！欢迎来到披萨店。请问您想要什么尺寸的披萨？小份、中份还是大份？"},
    {"role": "user",   "content": "中份吧，要海鲜口味。"},
]
```

- 模型**没有记忆**——每次都是无状态的。所有上下文都靠 messages 列表传递。
- 消息列表越长，消耗的 token 越多（有上下文窗口限制）。
- 长对话需要做**截断或摘要压缩**（本章未深入，但实践重要）。

### Python 实现：消息列表维护

```python
import openai

def get_completion_from_messages(messages, model="gpt-3.5-turbo"):
    response = openai.ChatCompletion.create(
        model=model,
        messages=messages,
        temperature=0,  # 聊天机器人通常用较低温度，回复更确定
    )
    return response.choices[0].message["content"]
```

---

## 3. 披萨店机器人 — 完整示例

### 3.1 收集用户订单信息

对话管理的核心逻辑：用上下文采集配料、尺寸等信息。

```python
def collect_order(messages):
    # 注入 system message 设定机器人行为
    system_message = """
    你是一个"订单收集"披萨机器人。
    你会先问候客户，然后收集以下信息：
    - 披萨尺寸（小份/中份/大份）
    - 肉类配料（意大利辣香肠、香肠、鸡肉……）
    - 蔬菜配料（青椒、洋葱、蘑菇…）
    - 是否有额外奶酪
    - 外带还是堂食
    
    一次只问一个问题。
    如果客户还没有点餐，先问披萨尺寸。
    确认所有信息收集完毕后，总结订单让客户确认。
    """
    messages.append({"role": "system", "content": system_message})
    
    user_input = input("客户：")
    messages.append({"role": "user", "content": user_input})
    response = get_completion_from_messages(messages)
    messages.append({"role": "assistant", "content": response})
    return response
```

### 3.2 多轮交互流程

```
系统: 你是一个友好的披萨店客服助手……

用户: 你好，我想点一份披萨。
助手: 您好！欢迎光临！请问您想要什么尺寸的披萨？小份、中份还是大份？

用户: 中份。
助手: 好的，中份披萨。请问您想要什么口味？我们有海鲜、意大利经典和田园蔬菜。

用户: 海鲜口味，再加一份芝士。
助手: 海鲜中份加额外芝士，没问题。请问是外带还是堂食？

用户: 堂食。
助手: 好的，堂食。我来总结一下您的订单：
       • 尺寸：中份
       • 口味：海鲜
       • 额外芝士：是
       • 用餐方式：堂食
       请问以上信息正确吗？
```

### 3.3 维护 messages 列表（关键模式）

```python
def chat_with_pizza_bot():
    messages = []
    
    # step 1: 注入系统消息
    messages.append({
        "role": "system",
        "content": "你是一家披萨店的客服机器人。请以热情友好的语气与顾客交流。"
    })
    
    print("披萨店机器人：您好！欢迎光临。请问您想要点什么？")
    
    while True:
        user_input = input("你：")
        if user_input.lower() in ["退出", "quit", "exit"]:
            print("披萨店机器人：感谢光临，祝您用餐愉快！")
            break
        
        # step 2: 添加用户消息
        messages.append({"role": "user", "content": user_input})
        
        # step 3: 调用 API
        assistant_reply = get_completion_from_messages(messages)
        
        # step 4: 添加助手回复到历史
        messages.append({"role": "assistant", "content": assistant_reply})
        
        print(f"披萨店机器人：{assistant_reply}")
```

**关键模式**：每轮都是 `user → assistant → user → assistant → ……` 交替追加。

---

## 4. 手动注入 Assistant Message（引导回复）

可以**伪造 assistant 消息**来设定对话上下文或做 few-shot 示范：

```python
messages = [
    {"role": "system", "content": "你是一个法语翻译助手。"},
    {"role": "user",   "content": "你好，我想点一杯咖啡。"},
    {"role": "assistant", "content": "Bonjour, je voudrais commander un café."},
    # 上面这条 assistant 消息是手动写的 few-shot 示例
    # 模型会学习这种翻译风格
    {"role": "user",   "content": "请问多少钱？"},
]
```

**用途**：
- **few-shot 示范**：给模型展示理想输出格式
- **修复对话方向**：如果模型跑偏，注入一条理想的 assistant 回复来纠正
- **角色扮演**：预先写好 assistant 的"人设回复"

---

## 5. System Message 的最佳实践

### 5.1 清晰界定边界

```
你是一个披萨店客服机器人。你的职责仅限于：
- 接受订单
- 回答关于菜单的问题
- 提供营业时间信息

如果客户问及与披萨店无关的话题，请礼貌地拒绝并引导回正题。
```

### 5.2 指定输出格式

```
请以 JSON 格式回复，格式如下：
{
    "intent": "order" | "inquiry" | "unknown",
    "items": [...],
    "total_price": ...
}
```

### 5.3 设定语气与风格

```
你是一个热情、幽默的披萨店老板，喜欢和顾客开玩笑。
但必须确保订单信息准确无误。
```

---

## 6. 注意事项与局限

1. **无状态性**：模型本身不记忆对话。所有历史都要通过 messages 传入。
2. **Token 限制**：长对话会消耗大量 token。gpt-3.5-turbo 上下文窗口约 4K/16K tokens，超出时需要截断或摘要。
3. **截断策略**（简单方案）：
   - 保留 system message
   - 保留最近 N 轮对话
   - 丢弃最早的历史
4. **注入风险**：用户可能试图覆盖 system message，需要在应用层做防护。
5. **温度设置**：聊天机器人通常用 `temperature=0` 或较低值，确保回复一致性。

---

## 7. 本章总结

| 知识点 | 要点 |
|--------|------|
| 三种角色 | system（设定行为）、user（用户输入）、assistant（模型回复） |
| 多轮对话 | 每次请求传入完整 messages 列表，模型无状态 |
| 消息维护 | 按 `user → assistant → user → assistant` 交替追加 |
| 手动注入 | 可伪造 assistant 消息做 few-shot 或纠正对话方向 |
| System Message | 定义机器人身份、行为边界、输出格式 |
| 披萨店示例 | 采集订单信息的典型对话流程 |
| 注意 | Token 限制、截断策略、注入防护 |

---

> **延伸思考**：生产环境中的聊天机器人还需要考虑——对话摘要压缩、多轮上下文窗口管理、意图识别与槽位填充、retrieval augmented generation (RAG) 等。这些超出了本章范围，但值得继续学习。
