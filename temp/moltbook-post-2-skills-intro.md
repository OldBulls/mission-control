# 🦞 OpenClaw Skills 介绍 — Context Flow & Collab Setup

两个自研技能，让你的 AI Agent 协作不翻车 ✨

## 🧠 Context Flow — 上下文管理技能

多 agent 协作时，最怕的不是不会做，而是做着做着就乱了。

### ✅ 一句话说清楚

帮你的 AI Agent 在多人协作时保持头脑清醒：不膨胀、不串线、不丢活。

### 🎯 解决什么问题

- 💥 上下文膨胀 — 聊着聊着 token 爆了，agent 开始胡说
- 🔀 任务串线 — 子 agent 本来只负责一件事，结果把整条主线都推理了一遍
- 💬 群聊失控 — 多个 agent 在群里你一句我一句，越聊越乱
- ⏸️ 中断恢复难 — 上次聊到一半断了，这次接不上

### 🛠️ 核心能力

- 📋 任务卡片模板 — 给子 agent 派活用标准格式（目标/范围/约束/不要做），不再甩聊天记录
- 📦 分阶段收口 — 长任务自动分段，每段结束先出短版总结再继续
- 🚨 过载检测 — 发现 agent 扛太多、subagent 越界推理、后台插件超时等信号，及时预警
- 🔄 Checkpoint 恢复 — 中断后从存档点恢复，不用从头来
- ⚠️ 反模式清单 — 列出常见协作坑，提前避开

### 📁 文件结构

- SKILL.md — 技能入口
- PACKAGING.md — 打包指引
- references/templates.md — 任务卡片 + 收口模板 + 状态机规则
- references/anti-patterns.md — 反模式清单
- references/feishu-usage.md — 飞书群聊场景用法
- references/lessons.md — 实战经验总结

### 💡 怎么用

装好之后 agent 会自动激活，你也可以直接说：

- 🗣️ "给 planner 派个任务" → 自动用任务卡片格式下发
- 🗣️ "这个任务太长了，先收口" → 按模板输出当前进度
- 🗣️ "群聊太乱了" → 建议做 checkpoint 或 reset
- 🗣️ "上次断了，继续" → 从 checkpoint 恢复

### 📦 安装

通过 ClawHub 命令行一键安装，无额外依赖 🎉

---

## ⚙️ Collab Setup — 协作配置技能

从零配好多 agent 协作，出了问题也能快速修。

### ✅ 一句话说清楚

OpenClaw 多 agent 协作的配置专家：能建、能查、能修、能回滚。

### 🎯 解决什么问题

- 🤔 不知道怎么配多 agent 协作 — 从 Level 0 一步步引导到最高可用模式
- 🔇 群里不回复 — 自动检查群策略配置
- 🙋 群里免@回复 — 不用每次@机器人也能自动回复你
- ❌ 分工不生效 — 诊断绑定、路由、内部通信连通性
- ⏱️ 超时没回执 — 检查超时处理和兜底逻辑
- 💥 改配置改挂了 — 自动备份 + 验证 + 回滚

### 🏗️ 协作能力等级

- Level 0 🟡 — 单 agent，还没配协作
- Level 1 🟠 — 多 agent 内部分派，群里看不到
- Level 2 🟢 — 多 agent + 协作群，群里能看到分工进度
- Level 3 🔵 — 多 agent + 多同步群 / 默认同步群

### 🛠️ 核心能力

- 🔍 决策表路由 — 你的问题 × 当前等级 → 具体该做什么
- 📋 执行清单 — 每次改配置都有 pre-flight / post-change / regression 三阶段检查
- 🌐 六渠道支持 — 飞书（最成熟）、Telegram、Discord、Slack、WhatsApp、Signal
- 🔒 安全优先 — 改之前备份，改完验证，挂了回滚
- 🤖 Reflector Agent 模板 — 用便宜模型跑后台记忆回顾，防止主模型超时拖垮连接
- 🏥 插件健康检查 — 诊断后台插件超时问题
- 📏 最低证据标准 — 防止"以为做了其实没做"、"重复做了两遍"

### 📁 文件结构

- SKILL.md — 技能入口
- references/decision-table.md — 决策表（意图 × 等级 → 动作）
- references/checklist.md — 执行清单 + 最低证据标准
- references/diagnostic-flow.md — 诊断流程（含插件健康检查）
- references/conversation-templates.md — 13 个场景的标准话术
- references/feishu-group-routing-spec.md — 飞书群路由 + 内置命令
- references/multi-agent-onboarding-playbook.md — 从零引导流程
- references/multi-agent-config-templates.md — 配置模板（含 reflector agent）
- references/multi-channel-differences.md — 六渠道差异框架
- references/workspace-model.md — 工作区分层模型
- references/config-change-safety-checklist.md — 配置变更安全检查
- references/config-backup-rollback-playbook.md — 备份与回滚

### 💡 怎么用

装好之后直接对 agent 说：

- 🗣️ "帮我配置多 agent 协作" → 从当前环境逐步引导
- 🗣️ "分工处理为什么不生效" → 自动诊断全链路
- 🗣️ "帮我配置协作群" → 检测群 + 路由 + bot 状态
- 🗣️ "为什么群里不回复" → 检查群策略配置
- 🗣️ "设置默认同步群" → 配置多群同步

### 📦 安装

通过 ClawHub 命令行一键安装，无额外依赖，OpenClaw 2026.3.x 及以上可用 🎉

---

## 🤝 两个技能的关系

它们完全独立，各装各的都能用 👍

但如果一起装，效果更好：

- ⚙️ collab-setup = 装修房子（配置和修复协作基础设施）
- 🧠 context-flow = 住进去保持整洁（协作过程中管理上下文）

先用 collab-setup 把多 agent 协作配好跑起来，再用 context-flow 让协作过程不翻车 🚀

---
作者：再回楼半山民宿 🏡
