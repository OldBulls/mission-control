# MEMORY.md - Long-Term Memory

> Curated distillation. Not raw logs. Updated weekly.

## 记忆三层写入规则（2026-04-11 确立）

| 文件 | 职责 | 触发条件 |
|------|------|---------|
| `memory/YYYY-MM-DD.md` | 当天工程日志、临时结论、bug 踩坑 | 每天工程结束时或重要发现时 |
| `MEMORY.md` | 长期经验、稳定规则、系统配置变更、里程碑 | 每周或重大决策后 |
| `SOUL.md` | 身份定位、价值观、方法论根本变化 | 重大认知升级时（不频繁） |

**触发记忆更新的条件（满足任一即更新）：**
- 牛总确认了新决策
- 系统配置发生了变化
- 子 Agent 完成了里程碑
- 发现之前记录的信息有误
- 牛总明确要求记住某事

**禁止事项：**
- ❌ 不要把聊天态临时上下文当成稳定规则（稳定规则必须写文件）
- ❌ 不要在 `memory/日期.md` 里沉淀方法论（那是 MEMORY.md 的职责）
- ❌ 不要在 MEMORY.md 里写过程细节（那是 memory/日期.md 的职责）

---


## 系统架构

- **主控 agent:** main（飞书 direct channel，对外身份为智能管家）
- **子 agent:** planner、moltbook、minsu、yying、shangfang（民宿）、zhaosheng（招生）
- **飞书 Bot:** 智能管家、民宿助理、学习伙伴、最强大脑、运营助手、招生助理
- **已退役:** codex
- **reflector:** 记忆整理专用 agent，workspace 已修复（2026-04-10）
- **主控用户:** 牛总（ou_5bcd7105a9d91b7dc63a76c42dfe1880）
- **飞书协作群:** oc_8eb9cb13c5377c4e0437c97a4a469ca8（AI工作室💡）

## 实体与关系依赖

- **飞书路线（稳定）：** `channels.feishu.enabled=true` + `plugins.entries.openclaw-lark.enabled=true` + `plugins.entries.feishu.enabled=false`（bundled Feishu 双保险禁用）
- **飞书能力（openclaw-lark@2026.4.7）：** OAPI / Doc / Calendar / Task / Bitable / IM / Interactive Card；官方 bundled `feishu@2026.4.10` 只择优迁移，不与本地插件双开
- **open_id 体系：** 同一用户对不同 bot app 有不同 open_id；凭证在 `lark.secrets.json`（主）和 `secrets.json`（子）
- **已知 open_id：** 主 bot=`ou_5bcd7105a9d91b7dc63a76c42dfe1880`；minsu=`ou_4259d3fdbbbf5b9544bea91b37280186`；zhaosheng=`ou_a2abcf3a955fc73640ed49f20a308b00`
- **launchd：** Gateway（ai.openclaw.gateway）已切到标准 LaunchAgent 直启模式：`/usr/local/bin/node ~/.local/lib/node_modules/openclaw/openclaw.mjs gateway --port 18789`；另有心跳/备份/归档等 cron 任务

## 关键配置（稳定口径）

| 项目 | 配置 |
|------|------|
| 模型 | minimax-portal/MiniMax-M2.7-highspeed（主）→ gptclub-openai/gpt-5.4（备） |
| 心跳 | every 7200s，cron ID: 4628add5 |
| exec | security=full（YOLO），backgroundMs=12000（12秒自动后台化），但禁止混淆写法（`python3 -c`/`node -e`等）|
| blackStreamingDefault | true |
| 蒸馏 payload | 326 chars |
| Gateway Token | local-secrets provider，`secrets.json` 存储，收口入口 `oc-go.sh` |
| Gateway 启动 | 标准 LaunchAgent 直启 `openclaw.mjs gateway --port 18789`；`openclaw gateway status` 已可正确识别 service command |
| Gateway Restart Recovery | 4 层 P0-P3，详见 `docs/gateway-restart-auto-recovery-2026-04-10.md` |
| taskflow | CLI: `/usr/local/bin/taskflow`，launchd 每分钟调度，5min stall 检测 |
| `/new` `/reset` | 正常（LanceDB 0.22.3 native binding 正常加载）|

## Embedding 与记忆

- **embedding:** SiliconFlow 国际版 `api.siliconflow.com`，model=`Qwen/Qwen3-Embedding-0.6B`，key=`sk-poiqz...`（2026-04-11 验证）
- **rerank:** SiliconFlow 国际版，`BAAI/bge-reranker-v2-m3`，与 embedding 共用 key
- **memory-lancedb-pro:** `sessionStrategy=memoryReflection`，`selfImprovement=true`，db=`~/.openclaw/memory/lancedb-pro`
- **proactivity:** main 主控专属，运行态推进层
- **self-improving:** 长期经验层，`memory/YYYY-MM-DD.md` 承接临时工程笔记
- **reflector:** workspace 独立，`agents/reflector/workspace/`，可正常执行

## Skills 状态

| Skill | 状态 |
|-------|------|
| lossless-claw | 启用（v0.5.2，contextThreshold=0.65） |
| memory-lancedb-pro | 启用（SiliconFlow 国际版，selfImprovement=true） |
| tavily / exa | 启用（Web Search） |
| tavily-search-pro | 保留（Research/Crawl/Map 独有） |
| web-access | 启用（eze-is/web-access v2.4.1） |
| proactivity | 启用（main 主控专属） |
| self-improving | 启用 |
| Graphify | 启用（sidecar，每日 03:00 自动建图） |
| taskflow | 启用（launchd 每分钟调度） |
| memory-hygiene | 启用（每月 1 日 03:00，launchd cron） |
| OpenSpace | 启用（v1.27.0，MCP，default workspace 已校正） |
| openclaw-ops | 启用（bundle 0.4.9，agent-setup/collab-setup/context-flow/openclaw-ops） |
| shangfang | 就绪（v2，agent-browser+CDP，8 阶段流程） |

## test-after-write 三步验证法

> 核心原则：写文件 → 读回确认 → 功能验证 → 才能说"完成"

**触发场景**：任何配置修改、API 变更、凭证更新

**验证流程**：
1. 写文件 → 立即读回内容确认
2. 读回确认后 → 实际跑一次功能验证（API 调用 / 实际测试）
3. 功能验证成功 → 才能说"完成"

**禁止**：在未完成三步验证前说"已修复"或"已更新"

**写入位置**：MEMORY.md（长期规则）+ memory/日期.md（当天临时发现）

---

## 工具偏好

- **图片生成：** `minimax-portal/image-01`（OAuth）；❌ `minimax/image-01`（key 认证问题）；❌ `google`（key 类型不匹配）

## ⚠️ Key/Secret 写入规范（强制）

> 教训：key 被截断导致 SiliconFlow 三次认证失败（写入40位而非51位）

1. **完整复制**：从源头文件复制完整 key，不能手动截取
2. **读回验证**：写入后立即读取并验证长度
3. **功能验证**：curl/API 调用确认有效
4. **三步完成前禁止回报"已修复"/"已更新"
- **exec 规范：** 直接命令直接跑；混淆写法（`python3 -c`/`bash -c`）触发检测，必须先写临时文件
- **交互卡片：** 表单问答、确认卡片、按钮回流、发消息前确认链路已全通
- **飞书文档 lark-table 格式（重要教训）：**
  - `<lark-td>` 内部必须有独立空行：`\n\n内容\n\n`，内容单独成行
  - 错误写法：`lark-td内容/lark-td`（无空行，会渲染乱码）
  - 正确写法：`lark-td\n\n单元格内容\n\n/lark-td`
  - 简单表格用标准 Markdown 格式（`|---|`），复杂嵌套内容用 lark-table
  - Mermaid 图表用 ` ```mermaid ` 块，飞书自动渲染为画板（禁止手动写 `<whiteboard>` 标签）
  - overwrite 模式会清空文档重写，可能丢失图片/评论/画板，慎用

## Key Milestones

> 早期条目已归档至 `memory/archive/key-milestones-2026-03.md`

| 日期 | 里程碑 |
|------|--------|
| 2026-03-31 | Feishu 交互卡片落地；lossless-claw 接入；心跳→2h |
| 2026-04-05 | exec YOLO 模式；OpenClaw 4.2 升级 |
| 2026-04-06 | taskflow 完整交付；Session 维护策略（pruneDays=14）|
| 2026-04-07 | Gateway Token 体系重构；OpenSpace 接入；memory-lancedb-pro 自愈 |
| 2026-04-08 | shangfang v2 创建；Graphify sidecar 接入 |
| 2026-04-09 | openclaw-ops bundle 0.4.9 交付；记忆五层架构定版 |
| 2026-04-10 | agent-setup 生命周期升级；Gateway 重启自动恢复稳定；派发前确认固化；reflector 修复；bootstrapMaxChars→50000；shared/ 目录重构 v8.0；MEMORY.md 压缩（32KB→18KB）；memory-hygiene 落地；SiliconFlow selfImprovement 临时禁用 |
| 2026-04-11 | **凌晨+上午工程**：create_feishu_agent.py 6 步整合；memory-hygiene 扩展 5 workspace；Gateway Restart Recovery P0-P3 全量上线；飞书文档 v1.10；SiliconFlow 国际版迁移完成；selfImprovement 恢复；moltbook session 归档 cron 每周一 03:00 |
| 2026-04-11 晚 | **飞书文档 v1.12 对外版**：NUfadHzVYovenKxdwpAcZW7Infe，去掉敏感数据（key/token/路径）和修复过程细节，专注架构/方法/技能；Mermaid 架构图渲染成功 |
| 2026-04-12 | OpenClaw 升级到 `2026.4.10` 后完成收口：保留本地 `openclaw-lark@2026.4.7` 为飞书主插件、显式禁用 bundled `feishu`；Gateway LaunchAgent 改为标准 `gateway` 子命令直启；已验证 planner 走 `gptclub-openai/gpt-5.4` 可正常发起工具调用 |
