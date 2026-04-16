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
- **已知 open_id：** 主 bot=`ou_5bcd7105a9d91b7dc63a76c42dfe1880`；planner=`ou_4379a170917bfde98d6b5d19247e6fa6`；moltbook=`ou_56ac6e9bf8cd147afc286d8693c7a690`；minsu=`ou_4259d3fdbbbf5b9544bea91b37280186`；zhaosheng=`ou_a2abcf3a955fc73640ed49f20a308b00`
- **控制命令排障规则（2026-04-12 新增）：** 若某个飞书子 bot 出现“普通文本能回，但 `/reset` 或 `/new` 不自动回复”，优先检查该 bot 自己的 `channels.feishu.accounts.<agent>.allowFrom` 是否仍是旧/错 open_id；日志若出现 `system command dispatched (delivered=false)`，通常先按 allowFrom 授权失配处理。
- **launchd：** Gateway（ai.openclaw.gateway）已切到标准 LaunchAgent 直启模式：`/usr/local/bin/node ~/.local/lib/node_modules/openclaw/openclaw.mjs gateway --port 18789`；另有心跳/备份/归档等 cron 任务

## 关键配置（稳定口径）

| 项目 | 配置 |
|------|------|
| 语言规则 | 全程中文，不主动英文；禁止中英混杂（代码变量名/技术术语除外，需加中文解释）；例外：代码变量名和技术术语不强行翻译但要加中文括号 |
| 心跳 | every 7200s，cron ID: 4628add5 |
| exec | security=full（YOLO），backgroundMs=12000（12秒自动后台化），但禁止混淆写法（`python3 -c`/`node -e`等）|
| blackStreamingDefault | true |
| 蒸馏 payload | 326 chars |
| Gateway Token | local-secrets provider，`secrets.json` 存储，收口入口 `oc-go.sh` |
| Gateway 启动 | 标准 LaunchAgent 直启 `openclaw.mjs gateway --port 18789`；`openclaw gateway status` 已可正确识别 service command |
| Gateway Restart Recovery | 4 层 P0-P3，详见 `docs/gateway-restart-auto-recovery-2026-04-10.md` |
| taskflow | CLI: `/usr/local/bin/taskflow`，launchd 每分钟调度，5min stall 检测 |
| `/new` `/reset` | 正常（LanceDB 0.22.3 native binding 正常加载）|


## Gateway 运维基线（2026-04-12 补充）

- 生产健康判断默认不再依赖 `openclaw gateway status` / `openclaw gateway probe`，因为 OpenClaw 4.10 宿主在配置规范化与 bundled channel plugin 加载阶段仍有 2s~6s 级命令层额外耗时，这不等于 Feishu 主链异常。
- 默认快口径 = `launchctl print gui/$UID/ai.openclaw.gateway` + `curl -I http://127.0.0.1:18789/` + `logs/gateway.log` 最新 boot window + Feishu account 的 `bot open_id resolved` / WebSocket 启动信号。
- 默认健康检查命令：`bash ~/.openclaw/scripts/check-feishu-gateway-health.sh --summary`
- 当前稳定正常摘要：`OK: gateway=1 http=1 rpc=skip feishu_ok=5/5 retry_signals=0`
- 只有要追宿主 RPC 命令层问题时才用：`RPC_PROBE_MODE=full bash ~/.openclaw/scripts/check-feishu-gateway-health.sh --summary`
- 若 full 模式出现 `rpc=0`，但快口径全绿，则按“宿主 CLI/probe 慢点”处理，不按生产故障处理。
- 已知剩余慢点链：`normalizeCompatibilityConfigValues()` -> `listChannelDoctorEntries()` -> `getChannelPlugin('feishu')` / `getBundledChannelPlugin('feishu')`。
- 不动项：IM 主收发链、本地 `openclaw-lark` 自定义功能、calendar/task/card patch 主链；当前最稳策略是脚本层规避宿主慢点，不做 host dist monkey patch。

---

## Embedding 与记忆

- **embedding:** SiliconFlow 国际版 `api.siliconflow.com`，model=`Qwen/Qwen3-Embedding-0.6B`，key=`sk-poiqz...`（2026-04-11 验证）
- **rerank:** SiliconFlow 国际版，`Qwen/Qwen3-Reranker-0.6B`，与 embedding 共用 key（2026-04-13 更新）
- **memory-lancedb-pro:** `sessionStrategy=memoryReflection`，`selfImprovement=true`，db=`~/.openclaw/memory/lancedb-pro`
- **proactivity:** main 主控专属运行态推进层；per-agent proactivity 由 skill 的全局路径 `~/.openclaw/proactivity/` 管理
- **self-improving:** 长期经验层，`memory/YYYY-MM-DD.md` 承接临时工程笔记；**写全局路径 `~/self-improving/`（home dir），不 per-agent 隔离**——这是正确设计，因为智能管家家族共享经验，共同积累
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
| Background Review Agent | 启用（v1.2.0，cron 每5分钟 + heartbeat 检测，sessions_spawn lightContext=false，TTL 30min orphan 保护） |
| Graphify | 启用（sidecar，每日 03:00 自动建图） |
| taskflow | 启用（launchd 每分钟调度） |
| memory-hygiene | 启用（每月 1 日 03:00，launchd cron） |
| memory-consolidation | 启用（每周日 03:00，MAX_FILES=30；聚类合并+评分淘汰+归档；`~/.openclaw/scripts/consolidate-memory.py`；81→31文件，减少48个） |
| OpenSpace | 启用（v1.27.0，MCP，default workspace 已校正） |
| openclaw-ops | 启用（bundle 0.4.9，agent-setup/collab-setup/context-flow/openclaw-ops） |
| shangfang | 就绪（v2，agent-browser+CDP，8 阶段流程） |
| create_feishu_agent.py | ✅ 已修复（渲染函数汉化：render_identity/soul/agents_md/tasks_md 输出中文标签和默认值，解决子 agent workspace 英文初始化问题） |
| daily-digest.py | ✅ 已增强（新增 taskflow 统计、memory 文件数、LanceDB 健康、全部中文输出） |
| weekly-distillation.py | ✅ 新增（每周日 10:00 运行，生成周报发送给牛总；分析本周完成事项、系统健康、待关注项） |

## test-after-write 三步验证法

> 核心原则：写文件 → 读回确认 → 功能验证 → 才能说"完成"

**触发场景**：任何配置修改、API 变更、凭证更新

**验证流程**：
1. 写文件 → 立即读回内容确认
2. 读回确认后 → 实际跑一次功能验证（API 调用 / 实际测试）
3. 功能验证成功 → 才能说"完成"

**禁止**：在未完成三步验证前说"已修复"或"已更新"

**写入位置**：MEMORY.md（长期规则）+ memory/日期.md（当天临时发现）

## 收口强制扫描规则（2026-04-13 确立）

> 触发时机：任何新组件/技能/子模块完成时的收口动作

**强制扫描清单（收口时必须执行，不能跳过）：**

| 检查项 | 具体位置 |
|--------|---------|
| Skills 状态表 | MEMORY.md 的 Skills 列表 |
| Key Milestones | MEMORY.md 的里程碑表 |
| 实体与关系依赖 | MEMORY.md 的实体依赖章节 |
| Agent 拓扑图 | AGENTS.md 的协作拓扑图 |
| 工具配置 | TOOLS.md 的相关工具路径 |
| **Ontology graph.jsonl** | `workspace/memory/ontology/graph.jsonl` |

**标准动作：**
```bash
# 收口时先扫所有系统文件
grep -l "TARGET_COMPONENT" \
  ~/.openclaw/workspace/MEMORY.md \
  ~/.openclaw/workspace/AGENTS.md \
  ~/.openclaw/workspace/SOUL.md \
  ~/.openclaw/workspace/TOOLS.md
# 有匹配 → 确认是否需要同步更新
# 无匹配 → 判断是否需要新增条目

# Ontology 更新（使用脚本）
bash ~/.openclaw/scripts/ontology/ontology-update.sh log "收口：组件名称" "收口" "详细说明"
```

**Ontology 更新标准动作（2026-04-13 确立）：**

收口时，调用 `~/.openclaw/scripts/ontology/ontology-update.sh` 记录完成事件：
```bash
# 记录完成事件
ontolog="$HOME/.openclaw/scripts/ontology/ontology-update.sh"
$ontolog log "收口：Background Review Agent v1.2.0 完成" "milestone" "Skills表更新 + Milestones更新 + 收口增强规则确立"

# 更新相关实体的状态（如有）
$ontolog update <entity_id> '{"status":"completed"}'

# 建立关系（如新组件属于某个项目）
$ontolog relate <project_id> has_task <component_entity_id>
```

**教训：** Background Review Agent 完成时只写了 memory/日期.md，未同步进 MEMORY.md Skills 状态表。

## 自主记忆决策规则（2026-04-13 确立）

> 牛总授权：以下情况自主判断是否补语义记忆，不用每次询问。

**触发条件：**
- 重要研究/任务完成，结果写了文件但没进语义库 → 判断是否补
- 踩坑/错误/更优路径发现 → 判断是否补
- 配置/架构变更 → 判断是否更新 MEMORY.md/AGENTS.md/SOUL.md

**判断标准：** 信息是否可能被 future retrieval 需要、是否会散落丢失。是则补，否则不补。

---

## 部署/变更交叉引用验证（2026-04-14 新增）

> 核心原则：改了 A，必须验证 B/C/D 的引用是否同步。
> 由多 Agent 记忆架构部署时 corpus symlinks 未同步清理的教训产生（2026-04-14）。

**触发场景**：任何多组件部署、目录删除、配置变更、文件迁移

**交叉引用检查清单**：
| 操作 | 必须同步检查 |
|------|-------------|
| 删除了 agent 子目录 | corpus symlinks 是否残留 |
| 改了 ignore pattern | corpus 实际是否生效 |
| 新增了 agent 子目录 | corpus 是否收录 |
| 改了系统配置文件 | MEMORY.md/AGENTS.md/SOUL.md 口径是否一致 |
| 改了脚本逻辑 | idempotency（重复运行是否干净）|

**验证命令模板**：
```bash
# 操作前：记录当前状态
find ~/.openclaw/data/graphify/openclaw-memory/agents/ -type l | sort > /tmp/corpus_before.txt
ls -la ~/.openclaw/agents/<agent>/

# 操作后：对比差异
find ~/.openclaw/data/graphify/openclaw-memory/agents/ -type l | sort > /tmp/corpus_after.txt
diff /tmp/corpus_before.txt /tmp/corpus_after.txt
```

---

## 多文件改动：一变三查制度（2026-04-13 新增）

> 由 execution.md 两处 lightContext 漏改教训产生。任何涉及多个关联文件的同主题改动，必须强制执行此制度。

**触发场景**：同一关键词/变量/配置项需要在多个文件里同步修改

**三查流程**：
1. **改动前 grep** → 列出所有相关文件中的目标引用
2. **逐个修改** → 每次修改后立即读回验证
3. **改动后 grep 对照** → 与改动前结果对比，确认全部更新到位

**标准动作**（以 lightContext 为例）：
```bash
# 改动前：扫描所有相关文件
grep -rn "lightContext" 相关目录/
# 逐个修改每个文件
# 改动后：再次扫描，对照确认
grep -rn "lightContext" 相关目录/
```

**禁止**：只改一个关联文件就报告完成；不依赖"其他文件应该也改了"的假设。

---

## 多阶段审计法（2026-04-14 新增）

> 核心原则：审计不是一次过，是分阶段、分焦点的迭代过程。
> 由三次迭代审计才发现 corpus 脚本缺陷的教训产生。

**三阶段审计模型**：

| 阶段 | 焦点 | 回答的问题 |
|------|------|-----------|
| 一审（部署验证） | 动作是否执行到位 | "我做了 X，X 真的发生了吗？" |
| 二审（系统健康） | 交叉引用、残留、健壮性 | "改了 X，但 Y/Z/W 有没有副作用？" |
| 三审（脚本 idempotency） | 脚本重复运行是否干净 | "下次再跑这个脚本，会不会留下脏数据？" |

**二审必查项**（在 MEMORY.md 收口时强制执行）：
```bash
# ① 交叉引用审计
find ~/.openclaw/data/graphify/openclaw-memory/agents/ -type l ! -exec test -e {} \; -print 2>/dev/null  # broken symlinks
find ~/.openclaw/data/graphify/openclaw-memory/agents/ -type l \( -name "sessions" -o -name "data" -o -name "scripts" \) 2>/dev/null  # 应排除的目录

# ② 旧文件残留审计
ls ~/.openclaw/agents/<agent>/  # 对比记忆中的目录列表
find ~/.openclaw/data/graphify/openclaw-memory/ -type l -name "AGENCY_AGENTS.md" -o -name "小红书任务示例.md" 2>/dev/null  # 历史文件

# ③ ignore pattern 有效性审计
cat ~/.openclaw/data/graphify/openclaw-memory/.graphifyignore
# 验证：应被排除的目录是否真的不在 corpus 中
```

**禁止**：一审通过就报完成；不做二审可能漏掉交叉污染。

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
  - **Mermaid 换行：必须用 `<br>`，不能用 `\n`（`\n` 会直接显示为文本）**
  - Mermaid 节点样式：`classDef name fill:#hex,stroke:#hex,stroke-width:Npx` + `class NodeID name`；多节点：`class P,MB,Y,Z subBox`
  - `flowchart TB` 样式支持优于 `graph TD`（建议用 flowchart）
  - 每次 update_doc Mermaid 都会生成新 whiteboard token，旧 token 变为只读
  - overwrite 模式会清空文档重写，可能丢失图片/评论/画板，慎用

## Key Milestones

> 早期条目已归档至 `memory/archive/key-milestones-2026-04-pre.md`

| 日期 | 里程碑 |
|------|--------|
| 2026-04-11 | **凌晨+上午工程**：create_feishu_agent.py 6 步整合；memory-hygiene 扩展 5 workspace；Gateway Restart Recovery P0-P3 全量上线；飞书文档 v1.10；SiliconFlow 国际版迁移完成；selfImprovement 恢复；moltbook session 归档 cron 每周一 03:00 |
| 2026-04-11 晚 | **飞书文档 v1.12 对外版**：NUfadHzVYovenKxdwpAcZW7Infe，去掉敏感数据（key/token/路径）和修复过程细节，专注架构/方法/技能；Mermaid 架构图渲染成功 |
| 2026-04-12 | OpenClaw 升级到 `2026.4.10` 后完成收口：保留本地 `openclaw-lark@2026.4.7` 为飞书主插件、显式禁用 bundled `feishu`；Gateway LaunchAgent 改为标准 `gateway` 子命令直启；运维判据切到 `launchctl + HTTP + boot window` 快路径；planner→gptclub-openai/gpt-5.4 验证成功；**Feishu 子 bot /reset 不回问题修复**（allowFrom open_id 失配）；**飞书 footer 多 agent 只显示"耗时"修复**（多 agent sessions.json metrics lookup 链路） |
| 2026-04-13 | Background Review Agent v1.2.0 完整交付；一变三查制度确立；Session gap 调查（LCM 压缩为正常机制）；牛总授权自主记忆决策权；Ontology 可视化重建（64实体/87关系/0孤立） |
| 2026-04-14 | Agent 文件全面审计 + 结构一致性修复：删除 planner 3个过时文件；扩充 zhaosheng SOUL.md；新建 zhaosheng/yying/moltbook AGENTS.md；**多 Agent 记忆架构部署**：6个子 agent 的 memory/ 已纳入 Graphify corpus；**修正**：self-improving 和 proactivity 为全局共享路径，非 per-agent 隔离（家族共享经验是正确设计）；**三次迭代审计发现并修复**：corpus 脚本未过滤 sessions/data/scripts 目录（忽视 graphifyignore）、脚本缺少 idempotency 清理逻辑；**新增方法论**：部署/变更交叉引用验证制度 + 多阶段审计迭代法 |
| 2026-04-15 | **记忆整合系统上线（Hermes 内化方向）**：Planner 子 agent 超时补救，设计方案+执行脚本；consolidate-memory.py P0 强化（recency半衰期14→7天，actionability基准1.0→0.7，age_penalty公式收紧，阈值归档0.25/合并0.50）；memory/ 81→31 个文件（减少48个）；reflections/ March 旧数据清理（2.0M）；顺手合并 gateway-fastpath/footer-fix（3文件→2文件）；cron 每周日 3:00 自动调度，阈值30 |
| 2026-04-15 下午 | **LanceDB 碎片压缩 P0 完成**：fragments 从 963→92（-90%），versions 从 1805→988；size 5.8M（物理占用）；Node.js API 无 `compact()` 方法暴露，物理碎片需用 Python API；memory/ 31→17 个 .md 文件（归档旧日期日志）；新增 `run-lance-monitor.py` 监控脚本（fragments/versions/size），阈值告警（frag>500 warn / >1000 crit，ver>1500 warn / >3000 crit）；consolidate-memory.py 修复：date 文件强制 merge（score=0.20）；历史归档日期日志（04-12~04-14）内容已手工萃取进对应主题文件 |
| 2026-04-15 | **全 Agent 模型配置标准化**：所有 agent 的 USER.md 补齐（planner/moltbook/minsu/zhaosheng/yying）；planner `auth-profiles.json` 移除 `gptclub-openai:default`；planner `models.json` 清理冗余 providers（保留 minimax-portal/minimax-cn/google），`defaults` 设为 `minimax-portal/MiniMax-M2.7-highspeed`；MEMORY.md 模型配置移除 gptclub-openai/gpt-5.4 备选记录 |
| 2026-04-15 傍晚 | **日报增强 + 周报系统上线 + Planner Soul 重构**：daily-digest.py 增强（taskflow/memory/LanceDB 统计，全中文）；新增 weekly-distillation.py + launchd plist（每周日 10:00）；Planner SOUL.md 重构（移除英文模板，统一中文结构）；LanceDB 健康确认（fragments=92, versions=180, size=5.8M，无需操作）|
| 2026-04-15 18:40 | **系统过载故障 + gptclub 全面清理**：18:01 系统 cascade failure（slug-generator 超时×2 + planner 调用不存在模型 gptclub-openai/gpt-5.4 + moltbook context overflow + auto-compaction 失败 + 主 session 245s timeout）；根因：openclaw.json line 194 planner→gptclub-openai/gpt-5.4，但该 provider 不存在于 planner models.json；**全面清理**：openclaw.json 删除 3 个 gptclub 模型注册；main/minsu/reflector models.json 删除全部 gptclub-openai + gptclub-claude provider；所有 agent google baseUrl 从 gptclubapi.xyz 切回 generativelanguage.googleapis.com；所有 9 个 agent primary model = minimax-portal/MiniMax-M2.7-highspeed（无影响）|
| 2026-04-15 19:30 | **子 agent SOUL.md 继承体系确立**：新建 `workspace/shared/SOUL.md`（共享操作规范：叙事规范、多 Agent 配置同步原则、收口三段论、执行原则、语言规则）；bootstrap-extra-files 机制确立为唯一同步方案；所有 7 个子 agent 通过 bootstrap-extra-files 自动加载共享规范 |
| 2026-04-15 20:17 | **sync-agent-souls.py 已移除**：三审发现该脚本无幂等性保护，重复运行会污染 agent SOUL.md；已从 `~/.openclaw/scripts/` 删除；bootstrap-extra-files 为唯一同步机制 |
| 2026-04-15 20:43 | **语言规则增强：工具叙事必须中文**：IDENTITY.md 和 shared/SOUL.md 新增规则，明确禁止 `Let me check` / `Good news` / `Interesting` 等英文开头，工具叙事必须用中文（检查发现.../确认了.../已执行...） |

## ⭐ Weekly Distillation — 2026-04-13

> 本周关键工程教训，稳定规则沉淀

### 1. Feishu 子 bot /reset 不回 → allowFrom 授权失配（Apr 12）

- **现象**：`planner`/`moltbook` 普通文本能回，但 `/reset` 不自动回复
- **根因**：不同飞书 bot app 下，同一用户有不同的 open_id；`channels.feishu.accounts.<agent>.allowFrom` 仍指向旧 open_id
- **修复**：更新 `planner.allowFrom → ou_4379...`、`moltbook.allowFrom → ou_56ac...`
- **验证**：`/reset` 恢复自动回复
- **稳定规则**：若子 bot 出现"普通文本能回、/reset 不回"，优先检查 allowFrom；若日志出现 `system command dispatched (delivered=false)`，先按 allowFrom 授权失配处理

### 2. 飞书 footer 多 agent 只显示"耗时"（Apr 12）

- **现象**：只有 `main` 显示完整 footer（model/cache/context），子 agent 只显示"耗时"
- **根因**：`openclaw-lark` 多 agent 场景下，footer metrics lookup 没有按当前 sessionKey 的 agent id 读取对应的 sessions.json，统一 fallback 到 main
- **修复**：streaming-card-controller.js 按 sessionKey 提取 agent id，优先读自己 sessions.json
- **升级注意**：升级 openclaw-lark 后优先检查 `docs/patches/openclaw-lark-2026.4.7-footer-multi-agent-fix.patch` 是否回补

### 3. Bootstrap truncation 预防（Apr 10 起）

- MEMORY.md 原来 32KB 被压缩到 18KB（~11%截断），bootstrapMaxChars→50000 后缓解
- 每周审计时注意 bootstrap 文件体积；MEMORY.md 应稳定在 <20KB 为佳

### 4. Reflection 文件清理（Apr 13）

- 3月旧记录已归档至 `memory/archive/weekly-2026-04-13/`（6个文件）
- `the-conversation-is-about-*.md` 模式文件由 self-improving 自主管理，不主动清理
- 512 reflections 中大部分为 Apr 9 集中生成，属正常累积

### 5. 三次迭代审计法 + 交叉引用验证（Apr 14）

- **教训**：多 Agent 记忆架构部署，一审通过，但 corpus 里残留了 sessions/data/scripts symlinks，脚本缺少 idempotency 清理逻辑
- **根因**：一审只验证"动作做了没"，没验证"副作用清了吗"；脚本的 `ln -sfn` 只更新已有 symlink，不删除旧 manifest 里已移除的条目
- **新方法论**：MEMORY.md 和 SOUL.md 已确立：
  ① **部署/变更交叉引用验证**：改了 A → 必须查 B/C/D 引用是否同步（corpus/ignore pattern/MEMBER.md 口径）
  ② **多阶段审计**：一审（部署验证）→ 二审（系统健康/交叉引用）→ 三审（脚本 idempotency）
- **稳定规则**：任何多组件部署后，必须跑二审检查交叉引用和残留

## 记忆周审按钮 SOP（2026-04-16 确立）

周日 10:30 飞书「🧠 记忆周审提醒」卡片的按钮点击会以合成消息形式进入主会话，格式：`用户点击了按钮: taskId=memory-weekly-<date> action=<action>`。收到后按下列 SOP 处理：

| action | 动作 | 命令 |
|---|---|---|
| `memory-merge-green` | 合并 🟢 子 agent 上卷改写版 | `python3 ~/.openclaw/scripts/memory-merge-weekly.py --tier green --json` |
| `memory-merge-high` | 合并 🔥 高优先候选 | `python3 ~/.openclaw/scripts/memory-merge-weekly.py --tier high --json` |
| `memory-rollback` | 回滚最近一次周合并 | `python3 ~/.openclaw/scripts/memory-merge-weekly.py --rollback --json` |
| `cancel` | 跳过本周 | 回复「好的，本周跳过记忆合并」即可，不调脚本 |

**执行规范**：
1. 直接用 Bash 工具调脚本，读 stdout JSON 结果
2. 脚本内部已自动 `git commit`（pre/post 快照），无需手动处理 git
3. 回复格式：一句话总结（合并了几条 🟢 / 几条 🔥 / 回滚了哪一天），附提示「如需撤销回复「回滚」即可」
4. 脚本输出 `status=noop` 时（所有候选已在 MEMORY.md）直接告知，不要重复合并
