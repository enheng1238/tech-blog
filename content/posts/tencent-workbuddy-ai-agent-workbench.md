---
title: "腾讯 WorkBuddy：从代码助手到全场景 AI 智能体工作台"
date: 2026-08-06
tags: [WorkBuddy, 腾讯, AI智能体, Agent, AI编程]
author: "enheng1238"
description: "解读腾讯最新全场景职场 AI 智能体工作台 WorkBuddy：从 CodeBuddy 进化而来，三大核心能力、多模型与 MCP/Skills 生态，以及 7 月上线的 WorkBuddy Bench 编程排行榜与 EdgeOne 挑战赛最新动态。"
---

# 腾讯 WorkBuddy：从代码助手到全场景 AI 智能体工作台

2026 年 7 月，AI 编程与智能体赛道迎来一位重磅选手——腾讯把旗下的代码助手 CodeBuddy 升级为**全场景职场 AI 智能体工作台 WorkBuddy**，官网地址从 copilot.tencent.com 同步迁移至 codebuddy.cn/work。与此同时，腾讯在 7 月 24 日上线了官方 Agentic Coding 排行榜 WorkBuddy Bench，联合 EdgeOne 发起了「AI Prompts × Skills 挑战赛」，社区侧则出现了两天破千星的 WorkBuddy 实战蓝皮书。这一连串动作表明：腾讯正在把 WorkBuddy 从“编程工具”重新定位为“职场数字同事”。

## 从 CodeBuddy 到 WorkBuddy：一次品牌与能力的双重跃迁

CodeBuddy 诞生之初是腾讯云的 AI 编程伙伴，主打代码补全与对话式编程，官网标题至今仍写着“AI 时代的智能编程伙伴”。而 WorkBuddy 的定位发生了根本变化：它面向人力资源、行政、运营、销售、研发等不同职场角色，是一款“像真正同事一样思考、执行任务并交付结果”的智能体应用。蓝皮书中的定义很精辟——WorkBuddy 的核心能力可以概括为三句话：**听得懂人话、能够自主思考规划、真的能够操作电脑交付成果**。

这与传统 AI 助手的区别是本质性的：传统助手陪聊、给建议，WorkBuddy 则直接交付结果。用户只需用一句自然语言描述需求，它就能在本地电脑中自主规划执行步骤：批量处理文件、生成文档、分析表格、制作 PPT、开展行业调研、构建本地知识库，复杂任务还能拆解给多个智能体并行执行。

## 技术底座：多模型 + MCP + Skills 的三层生态

WorkBuddy 不是单一模型的套壳，而是开放的智能体平台：

- **多模型切换**：内置混元、DeepSeek、GLM、Kimi、MiniMax 等主流模型，用户可按任务类型选择。实际使用中，写代码选代码能力强的模型、做创意内容选多模态模型，是社区里最常见的玩法。
- **MCP Server**：通过模型上下文协议接入外部工具与数据源，让智能体具备调用真实服务的能力。
- **Skills 技能包**：遵循 Anthropic Skills 规范的可复用能力包。蓝皮书社区里已经涌现出“公众号 Skill 一键排版并发布到微信草稿箱”“用 WorkBuddy 清洗 119 份门店 Excel 并生成报表”等真实案例——这正是 Skills 生态价值的直接证明。

针对本地文件操作、终端执行等高危场景，WorkBuddy 还提供高危指令拦截和权限控制机制，降低 AI 自主执行过程中的风险。

除了单任务执行，WorkBuddy 的进阶玩法是把它当成一支“AI 团队”来组织：把一次成功的工作流沉淀为可复用的 Skill，再让多个 Agent 按流程分工协作。蓝皮书把这种模式概括为“从第一项任务，到一支 AI 团队”。实际案例中，有人用它接入小程序与 IM 助理，把智能体挂到微信生态里随时调用；也有人把它与 ima 知识库配合，告别“微信收藏夹吃灰”的困境。对于内容创作者，社区甚至开发出“公众号 Skill 一键排版并发布到微信草稿箱”的流程，从调研、成稿到发布形成完整闭环。

## 最新动态一：WorkBuddy Bench 编程排行榜

7 月 24 日上线的 WorkBuddy Bench（workbuddybench.com）是腾讯官方的 Agentic Coding 排行榜，也是近期最有信息量的评测数据。它采用**双 harness（测试框架）设计**——CodeBuddy Code 与 Claude Code 两种执行环境，覆盖 Code、Web、Office、Security 四个子集，共 8 个计分列。核心结论如下：

| 模型 | 领先项 | 代表分数 |
| --- | --- | --- |
| Claude Opus 4.8 | Code / Web / Office（5 列） | Code 74.43 / 77.90，Web 68.14 / 69.86 |
| GLM-5.2 | Security（2 列） | Security 76.32 / 80.86 |
| GPT-5.5 | Office（1 列，Claude Code 下） | 输出预算最小：Code 仅 6.9k tokens |

榜单还揭示了一个容易被忽略的事实：**同一模型在不同 harness 下的分数差异极大**——GLM-5.2 在 Web 子集上两个 harness 得分分别为 67.43 与 60.71，GPT-5.5 在 Security 上为 77.91 与 64.39。这提醒我们，Agentic 评测结果必须结合具体执行环境解读，脱离 harness 谈模型排名没有意义。另一个亮点是效率：GPT-5.5 在 CodeBuddy Code 下所有子集输出预算最小，却保持 Code 与 Office 顶级水平，说明“省 token 又能干活”是可能的。腾讯自家混元模型（HY-3）在接入跨轮推理回传后，Code 分数提升了 3.82 分（CodeBuddy Code harness），显示出推理链路优化的空间。

## 最新动态二：EdgeOne 挑战赛与社区生态

腾讯云 EdgeOne 与 WorkBuddy 联合发起的「AI Prompts × Skills 挑战赛」，官方作品池已收录 166 件作品（116 个 Prompt + 50 个 Skill），全部通过 EdgeOne Pages 部署，开箱即用全球 CDN 加速。仓库明确写出激励规则：GitHub Star 破 1000 后，所有获奖者 Credit 奖励全线 +10%。这种“官方出资源、社区出作品”的共建模式，正在为 WorkBuddy 积累可复用的资产库。

社区侧的爆发同样值得关注：开源项目 WorkBuddyGuide（实战蓝皮书）7 月 10 日创建，截至 8 月 3 日已获近 2000 星，在线阅读站 workbuddy.homes 提供全文搜索、章节目录与案例集。蓝皮书按“使用手册→案例篇→进阶篇→岗位与行业”四篇组织，把一次成功经验沉淀为团队可复用的工作系统——这恰好补齐了官方文档“讲功能不讲实战”的短板。

## WorkBuddy 在真实工作流中的角色

把 WorkBuddy 放进真实职场，它的价值边界逐渐清晰。对运营和销售，它是“表格分析 + PPT 制作”的流水线：清洗 119 份门店 Excel 并生成汇总报表，这种过去要耗掉一整天的活，现在一句自然语言就能触发。对开发者，它是代码助手 CodeBuddy 能力的延续，同时叠加了本地知识库构建与多模态内容创作。对管理者，它是团队 SOP 的沉淀工具——把某位员工的成功案例固化成 Skill，其他成员就能复用同样的工作方法。

值得注意的是，WorkBuddy 的定位与海外主流 Agent 产品存在明显差异。Claude Code、Codex 聚焦软件工程场景，面向开发者；WorkBuddy 则同时覆盖研发与职能岗位，走“通用工作台”路线。这也解释了为什么它把多模型切换作为默认能力：不同岗位的任务形态差异太大，单一模型难以通吃。

## 风险与思考

WorkBuddy 的“自主操作电脑”能力是把双刃剑。权限控制、高危指令拦截是底线，但真正的安全边界需要用户自己定义：授权哪些目录、允许哪些终端操作、验收标准是什么。社区反复强调“先核对发布者与下载来源”“不从不明镜像获取安装包”，说明 Agent 类产品的供应链安全同样不可忽视。此外，多模型接入意味着隐私数据会流向不同服务商，敏感文件的处理需要格外谨慎。

另一个需要警惕的误区是“榜单分数等于实际体验”。WorkBuddy Bench 的双 harness 数据已经证明，同一模型在不同执行环境下的得分可以相差 7 到 13 分；更何况真实职场任务的评判标准是“交付物是否可用”，而非评测集上的分数。榜单适合用来挑选模型与观察趋势，落地时仍应以自己业务的验收标准为准。

从行业视角看，WorkBuddy 的出现标志着国内大厂在 Agent 赛道从“模型军备竞赛”转向“工作流落地竞争”。当 Claude Code、GPT-5.5 等海外模型在评测中依然领先时，腾讯选择用开放的模型生态、本土化的工作流场景和低门槛的 Skills 体系来建立差异化。谁能把智能体真正嵌入职场人的日常，谁就掌握了下一阶段的话语权。

WorkBuddy Bench 已经给出了第一份答卷，而 WorkBuddy 本身，正等着更多真实的职场任务来检验。对普通用户来说，现在正是动手尝试的好时机：从官方渠道免费下载，微信扫码即可登录，从第一个小任务开始，逐步搭建属于自己的 AI 工作系统。

**参考文献：**
- Tencent WorkBuddy Bench (2026-07-24). “Agentic Coding Leaderboard” — workbuddybench.com
- AlephAITech (2026-07). “WorkBuddy 实战蓝皮书” — github.com/AlephAITech/WorkBuddyGuide，阅读站 workbuddy.homes
- Tencent EdgeOne (2026-07). “Awesome Website Prompts & Skills（WorkBuddy × EdgeOne 挑战赛官方作品池）” — github.com/TencentEdgeOne/awesome-website-prompts-and-skills
- 腾讯云代码助手 CodeBuddy 官网 — copilot.tencent.com / codebuddy.cn/work
- Hacker News (2026-07-24). “WorkBuddy Bench: Agentic Coding Leaderboard” — hn.algolia.com item 49030549
