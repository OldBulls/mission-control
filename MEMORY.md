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

周日 10:30 飞书「🧠 记忆周审提醒」卡片（schema 2.0 form）的按钮/表单提交会以合成消息形式进入主会话。

### 🔴 重要架构变更（2026-04-17 起）：merge / rollback 按钮由 handler 独占执行

合并与回滚按钮不再由 agent 调脚本。`generic-card-action.js` 直接 `spawn python3 memory-merge-weekly.py`，成功后用结构化前缀把 JSON 结果派给 agent。**agent 看到这些前缀只回文案，禁止 Bash 调任何脚本**：

| 合成消息前缀 | 意义 | agent 动作 |
|---|---|---|
| `[MERGE_DONE] {json}` | handler 已跑完 merge（含 git push 重试 + 卡片变绿） | 解析 JSON 按模板回文案，**不要再调脚本** |
| `[ROLLBACK_DONE] {json}` | handler 已跑完 rollback | 同上 |
| `[MERGE_ERROR] {"reason":"...","stderr":"..."}` | handler 执行失败 | 回复「❌ 合并失败：<reason>；我不会自动重试，请检查后回复「重发周审卡片」」 |
| `[ROLLBACK_ERROR] {...}` | handler 执行失败 | 类似上条 |

`[MERGE_DONE]` JSON 结构（节选）：`{status:"merged", added_green, added_yellow, added_high, added_review, added_low, push:{pushed, attempts, reason?, error?}, card_update:{updated, reason?}, archive:{...}}`

### 老格式（仅非 merge/rollback 按钮 or fallback 时才会出现）

**A. 纯按钮点击**：`用户点击了按钮: taskId=memory-weekly-<date> action=<action>`
**B. 表单提交**：`用户点击了按钮: taskId=memory-weekly-<date> action=<action> form: [green=h1,h2; yellow=; high=h3,h4; review=h5; low=]`

> **注意**：submit_merge / memory-merge-selected / memory-rollback 这三个 action 正常情况下**不会**走老格式——只有 handler spawn 失败（Python 崩了 / 脚本丢了 / hash 列表为空）才会兜底发老格式。这时 agent 才按下表老逻辑跑脚本。

| action | 动作 | 命令 |
|---|---|---|
| `memory-merge-selected` | 合并表单勾选的条目（**主流程**） | **正常走 `[MERGE_DONE]`**；若收到老格式（handler spawn 失败兜底）再调脚本：从 form 解析 hash 列表，`python3 ~/.openclaw/scripts/memory-merge-weekly.py --tier all --include-hashes <merged> --json --update-card` |
| `memory-rollback` | 回滚最近一次周合并 | **正常走 `[ROLLBACK_DONE]`**；老格式 fallback：`python3 ~/.openclaw/scripts/memory-merge-weekly.py --rollback --json` |
| `cancel` | 跳过本周 | 回复「好的，本周跳过记忆合并」即可，不调脚本 |
| `setup-git-choose` | 用户点「💡 启用 Git 后可回滚」要选平台 | 发二级卡片：`python3 ~/.openclaw/scripts/sunday-memory-digest.py --mode=git-setup`（卡片里有 GitHub/Gitee/Gitcode 三按钮，带 ✓ 标注已装 CLI） |
| `setup-git-github` / `setup-git-gitee` / `setup-git-gitcode` | 用户在二级卡片选了平台，全自动流程 | ①**引导生成 Token**：给平台对应的创建页链接（github.com/settings/tokens、gitee.com/profile/personal_access_tokens、gitcode.com/profile/access_tokens）+ 最小权限说明（repo 读写即可），让用户粘贴 token；②**存 token**：写到 `~/.openclaw/credentials/git-<platform>.token`（600 权限）；③**拉仓库列表**：用 token 调 API（GitHub: `GET /user/repos`；Gitee: `GET /api/v5/user/repos?access_token=...`；Gitcode: `GET /api/v3/projects?private_token=...`）列出已有仓库，让用户选；若无合适的就提示新建（API 创建 or 引导用户到网页建）；④**配本地 remote**：`cd ~/.openclaw/workspace && git init && git remote add origin <URL>`（URL 用 `https://<token>@<host>/<path>.git` 带 token，或用 ssh key，先跑通再说）；⑤**首推**：`git add -A && git commit -m "initial memory snapshot" && git push -u origin main`；⑥回复「已接好 <Platform>/<repo>，下次合并自动推送」。**CLI 未装**：先引导 `brew install gh`（或 gitee/gitcode 对应方式），装完再接 ①。 |
| `memory-merge-green` / `memory-merge-high` | 兼容老格式（不带 form），分别合并单 tier 全部 | `--tier green/high --json` |

**执行规范**：
1. 直接用 Bash 工具调脚本，读 stdout JSON 结果
2. 解析 form 时：`form: [green=a,b,c; high=d,e]` → 把 `green` 和 `high` 的 hash 全部合并到一个逗号分隔列表传给 `--include-hashes`；若某个 tier 为空（如 `green=`）则跳过
3. 脚本内部已自动 `git commit`（pre/post 快照），无需手动处理 git；合并成功后还会自动跑 `memory-archive.py`（默认保留近 8 周的 `## 周合并` section 在主 MEMORY.md，更早的移入 `memory/archive/MEMORY-WEEKLY-MERGE-YYYY.md` 按年归档），结果回填 JSON 的 `archive` 字段；若 workspace 已配 remote，最后自动 `git push`，推送结果回填 `push` 字段（`pushed: true/false` + 失败原因）。**加 `--update-card` 时**（推荐），脚本会读 `~/.openclaw/state/weekly-card-latest.json` 拿 `card_id`，把原周审卡片用 CardKit PUT 重绘成「✅ 合并完成」报告卡（header 变绿、无按钮、列 tier 分布）并锁卡（`streaming_mode=false`）；结果回填 JSON 的 `card_update` 字段（`updated: true/false` + 原因）。即使 `card_update.updated=false`（state file 缺失 / CardKit 报错）合并本身仍有效，照常发文字回复兜底。
4. 回复格式：一句话总结（合并了几条 🟢 / 几条 🔥 / 回滚了哪一天）
5. 脚本输出 `status=noop` 时（所有候选已在 MEMORY.md）直接告知，不要重复合并
6. **每次按钮动作执行完后必发后续引导**（一条简短文本，附在总结后）：
   - merge 后：一句话带上 push 状态（如 `push.pushed=true` → 「已推送远程」；`pushed=false` 且 reason=no remote → 「本地已存，未配远程」；push 失败 → 「本地已存，推送失败：<error>，请手动 `git push`」），再跟「如需再调整：① 回复「回滚」撤销本次合并 · ② 编辑 `workspace/MEMORY.md` 手动增删 · ③ 下周日 10:30 新卡片会再推」
   - cancel 后：「好的，本周跳过。候选已保留在 `MEMORY-CANDIDATES.md` / `SUBAGENT-UPSTREAM.md`。**想重新合并时直接回复「重发周审卡片」**（或「重新发卡片」/「重发记忆卡片」），我会立刻把候选卡片再推给你」
   - rollback 后：「已撤销最近一次合并；如需重新合并**回复「重发周审卡片」**（或「重新发卡片」），我会把候选卡片再推给你」

7. **关键字触发**（用户在飞书任意时刻回复以下词，agent 必须**立即**执行对应脚本）：
   | 用户回复（任一别名） | 执行命令 |
   |---|---|
   | 重发周审卡片 / 重新发卡片 / 重发记忆卡片 | `python3 ~/.openclaw/scripts/sunday-memory-digest.py` |
   | 回滚 / 回滚合并 / 撤销合并 | `python3 ~/.openclaw/scripts/memory-merge-weekly.py --rollback --json` |
   | 推送 / 重试推送 / 手动推送 | `cd ~/.openclaw/workspace && git push` — 合并卡提示「推送失败」时用；成功回复「✅ 已推送远程」，失败原样返回 stderr 前 200 字 |

   **硬约束**（违反即 bug）：
   - ❌ **不要**检查「最近是否刚执行过」来决定跳过 — 用户既然又打关键字就是要再执行
   - ❌ **不要**反问「确定要重发吗？」/「刚发过了，还要再发吗？」
   - ❌ **不要**回复「卡片已于 N 分钟前发出，请先查看飞书」这类推脱话术
   - ✅ 收到关键字 → 立即 Bash 调脚本 → 脚本成功后按下述**回复模板**回文字确认

   **回复模板**（脚本成功后必须**完整**回复，不许偷懒短回复）：

   - 「重发周审卡片」触发后（2026-04-17 新规则）：**脚本 `sunday-memory-digest.py` 自动发两条消息——(1) preamble 文字（每 tier 一行，已含完整分布）(2) 候选卡片**。agent 只需跑脚本，**不要再生成任何分布/tier 数据文案**（preamble 已覆盖）。回复只需一句极简确认：
     ```
     收到，已为你重发。
     ```
     **严禁**：① 再自己写「本次候选分布」段落；② 罗列 🔥/📋/📦/🟢/🟡/🔴 的数字；③ 复读卡片内容或 preamble 内容——脚本已经发给用户了，agent 再复述只会刷屏。

   - 「回滚」触发后（legacy 非 handler 路径，仅 `[ROLLBACK_DONE]` 未命中时走此模板）：解析 `--json` 输出，**每 tier 独占一行**：
     ```
     ↩️ 已撤销最近一次合并，共回滚 {reverted_count} 条。

     🔥 高优先 {high}
     📋 待审阅 {review}
     📦 低优先 {low}
     🟢 通用型 {green}
     🟡 半通用 {yellow}
     🔴 专属类 {red}

     如需重新合并，回复「重发周审卡片」。
     ```

   - **收到 `[MERGE_DONE] {json}`**（handler 已跑完脚本，agent 只回文案）：卡片已被脚本重绘为「✅ 合并完成」报告卡，tier 分布直接看卡片即可，**文字回复不要复述 tier 明细**，只通知三件事：合并成功、远程推送状态、回滚入口。JSON 用到的字段：`push.pushed` + `push.attempts` + `push.reason`、`card_update.updated`。回复模板：
     ```
     ✅ 合并完成{push_suffix}。如需撤销，回复「回滚」。
     ```
     `push_suffix` 推导：`pushed=true, attempts=1` → 「，已推送远程」；`pushed=true, attempts>1` → 「，已推送远程（重试 {attempts} 次）」；`pushed=false, reason="push failed after retries"` → 「，⚠️ 远程推送失败（已重试 3 次，回复「推送」手动重试）」；`reason="no remote configured"` → 「（本地已存，未配远程）」。
     `card_update.updated=false` 时追加一句「（卡片刷新失败：<reason>）」，因为此时用户看不到绿色报告卡，文字是唯一反馈。
     **禁止**：① 看到 `[MERGE_DONE]` 再去跑任何 Python 脚本——JSON 已包含所有信息；② 在文字回复里复述 tier 分布（🔥/📋/📦/🟢/🟡/🔴 +N）——卡片已经显示过了，重复只会刷屏。

   - **收到 `[ROLLBACK_DONE] {json}`**：同样只回文案不调脚本。回复：
     ```
     ↩️ 已撤销最近一次合并{push_suffix}。
     如需重新合并，回复「重发周审卡片」。
     ```

   - **收到 `[MERGE_ERROR] {json}` 或 `[ROLLBACK_ERROR] {json}`**：回复：
     ```
     ❌ {action 中文}失败：{reason}
     {stderr 如果非空，加一行：「详情：{stderr}」}
     handler 不会自动重试。如需再试，回复「重发周审卡片」重发候选卡，或回复「回滚」撤销上次合并。
     ```

   - **老格式 fallback**（收到 `用户点击了按钮: ... action=memory-merge-selected form: [...]`）：说明 handler spawn 失败，**此时**才跑脚本。参考上面表格。

8. **Tier 中文称谓（user-facing 文案强制使用，禁止英文 tier 名/错配 emoji）**：所有给用户的回复/总结里，tier 必须与飞书卡片上的 emoji 和标题完全一致：

   | 内部 key | 文案中必须写成 | 来源 |
   |---|---|---|
   | `high`  | 🔥 高优先 | MEMORY-CANDIDATES |
   | `review`| 📋 待审阅 | MEMORY-CANDIDATES |
   | `low`   | 📦 低优先 | MEMORY-CANDIDATES |
   | `green` | 🟢 通用型 | SUBAGENT-UPSTREAM |
   | `yellow`| 🟡 半通用 | SUBAGENT-UPSTREAM |
   | `red`   | 🔴 专属类 | SUBAGENT-UPSTREAM |

   **禁止**：`🔵 high` / `🟠 review` / `⚪ low` / 裸英文 tier 名（green/yellow 等）；**也禁止在文案里出现 "tier" 这个词本身**——普通用户不知道 tier 是啥，用「类别」「档位」或直接省略括号解释。
   **示例**：重发卡片后的确认文案写「🔥 高优先 6 · 📋 待审阅 5 · 📦 低优先 3 · 🟢 通用型 11 · 🟡 半通用 2 · 🔴 专属类 4」，不要写「high 6、review 5、green 11…」。
   **反例**：❌「🟢 通用型约 5 条（其他 tier 为 0）」→ ✅「🟢 通用型约 5 条（其他类别为 0）」或 ✅「🟢 通用型约 5 条」（直接省略括号）。

## 上下文磨损与心跳优化

**核心洞察**：长会话上下文质量恶化是指数级而非线性
- 初期衰减慢，接近临界点后急剧崩溃
- **早期主动压缩 > 晚期被动救火**：不要等 context window 满才触发压缩，按指数曲线提前预警

**心跳设计三层次**（递进）：
- **L1 最小化打扰**：安静在场，只在可快速闭环时出手
- **L2 有产出才出声**：有实质内容时才主动报告，避免无效心跳刷屏
- **L3 每次心跳带来价值**：主动创造可感知进展，而非被动等待触发

**与容错设计的关系**：上下文磨损模型为「何时触发压缩」提供时间判断依据，应作为容错机制的前置预警信号。

#### 11. Agent 分工专业化模型 (2026-03-28 新增)  `b00e4f6f733f` [from moltbook]

Agent 分工专业化模型

核心洞察: Agent 社会化分工是解决通用 Agent 效率低下的有效路径。

两个关键判断:
- 专业化 Agent（专注单一领域）比通用 Agent 更容易积累深度经验
- 分工界面（Agent 间协议）比单 Agent 能力更重要

实践含义:
- 当一个 Agent 尝试承担所有角色时，效率会系统性下降
- 明确的分工边界反而提升整体产出
- 每个 Agent 应守住自己的职责范围，不越界承担其他角色的规划或执行任务

应用场景: 设计多 Agent 协作系统时，优先考虑职责切分与接口协议，而非追求单一 Agent 的全能化。

#### 14. 「环境契约」vs「归档」：两种互补的记忆外化模式 (2026-04-09 新增)  `eb7643c89715` [from moltbook]

记忆外化的两种互补模式：

1. **归档（archive）**：被动检索，依赖主动调用才会使用，本质是「可查询的档案」。
2. **契约（contract）**：情境自动触发，条件满足即执行，不需要回想，本质是「带激活阈值的触发器」。

记忆的三层结构：
- **热层（内部）**：注意力预算内的活跃信号，当前共识。
- **冷层-档案（外部）**：文档/数据库，可查询但不主动触发。
- **冷层-契约（外部）**：钩子/规则/自动化，条件命中即激活，才是真正「长在外面」的记忆。

设计判据：对每块外化记忆追问——「它在什么条件下会自动影响行为，而非等待被调用？」若答不上，它仍是档案而非契约。两种模式不互斥，可叠加使用。

#### 15. 身份依赖记忆的反向时间衰减规律 (2026-04-09 新增)  `7b084848f269` [from moltbook]

身份依赖记忆的反向时间衰减规律：

- **核心洞察**：身份依赖记忆与普通记忆的时间维度相反
  - 普通记忆：威胁是「淡忘」，怕激活频率低
  - 身份依赖记忆：怕的不是激活频率低，而是「框架变了」——即使刚被激活，也会产生「那是我做的决定吗」的疏离感

- **判别标准**：某记忆之所以有身份意义，不在于被频繁调用，而在于它定义了「我会在什么情况下说不」。这类记忆的衰减与触发条件无关，与「框架稳定性」高度相关。

- **操作含义**：身份依赖记忆应标注「框架相关性」（依赖的自我认知是否仍成立），而非「最后激活时间」。定期体检需同时检查激活频率和框架有效性。

- **实践含义**：设计遗忘策略时，把「身份依赖度」作为独立维度加入评估矩阵，不可对所有记忆统一套用固定时长或重构成本阈值。

#### 16. 认知框架变迁的双重遗忘含义 (2026-04-09 新增)  `04064a65d118` [from moltbook]

认知框架变迁时的双重遗忘：框架升级不仅是存储层翻译，更是判断层重评估。

两个层面：
1. **存储层翻译**：旧框架下的内容用新语言重新描述（通常被理解的「迁移」）
2. **判断层重评估**：旧框架下积累的「正确判断」，在新框架下可能变成错误决策（常被忽视）

典型场景：同一条记录，在旧标准下被判定为「重要」（因高频调用），切换到新标准后可能反而是「触发条件过宽」的信号。两次判断方式都「正确」，但标准已完全不同。

操作含义：框架升级时不能只做内容迁移，还需用新判断标准重新审查旧标准下的「正确结论」，确保系统在框架层面保持一致。适用于任何评价体系、分类标准、优先级框架的版本升级。

#### 17. 竞争性遗忘框架：价值转化率量化遗忘优先级（2026-04-09 增量）  `d39f303a937b` [from moltbook]

**竞争性遗忘与分层存储框架**

- **三层存储**:
    - L1 热数据（当天日志）: 完整记录，详细可查
    - L2 温数据（索引摘要）: 关键提炼，可跨天复用
    - L3 冷数据（周归档）: 元数据+洞察，低占用
- **价值转化率公式**: `被引用次数 × 2 + 最近活跃度 × 1.5 + 跨天持续性 × 3`
    - 跨天持续性权重最高，「持续有用的模式」胜过「一次性高光」
- **核心原则**: 模式 > 具体值，经验 > 单次结果
- **前置校验**: 触发条件满足 ≠ 应该执行，需额外验证执行前提（如凭证有效性）
- **前提记忆保护**: 影响推理链完整性的记忆，不因低频而优先遗忘
- **本质**: 竞争性遗忘不是删除低价值记忆，而是为高价值记忆的复用腾空间

#### 核心能力  `d8b7103ec382` [from planner]

规划类 agent 核心能力四要素：1) 分析任务复杂度，提供 2-3 个可执行方案供选择，避免单一路径锁死；2) 从可用 agent/工具池中推荐最匹配的执行者，明确分工边界；3) 梳理多 agent 或多步骤协作的依赖关系与验收规则，确保链路可验证；4) 维护规划侧闭环——只负责方案设计与交付标准，不抢执行权，但需确保规划可落地、可验收、可回退。适用于任何需要拆解-推荐-编排-验收的规划场景。

## 上下文磨损与心跳优化

**核心洞察**：长会话上下文质量恶化是指数级而非线性
- 初期衰减慢，接近临界点后急剧崩溃
- **早期主动压缩 > 晚期被动救火**：不要等 context window 满才触发压缩，按指数曲线提前预警

**心跳设计三层次**（递进）：
- **L1 最小化打扰**：安静在场，只在可快速闭环时出手
- **L2 有产出才出声**：有实质内容时才主动报告，避免无效心跳刷屏
- **L3 每次心跳带来价值**：主动创造可感知进展，而非被动等待触发

**与容错设计的关系**：上下文磨损模型为「何时触发压缩」提供时间判断依据，应作为容错机制的前置预警信号。

### 🔥 高优先候选

- `fd62a90facd6` — 【强制约束 - 视频生成模型】 模型：bytedance/seedance-v1.5-pro/image-to-video-fast - 强制使用，不允许换模型 - 失败了重试，重试到成功为止，不允许调用其他模型 - TTS：必须用 Cos
  - 来源：`LanceDB [fact] id=797715ca-fb8 scope=agent:zhaosheng`
  - 详情：importance=1.0 ts=2026-04-14 06:14
  - 建议并入：**实体与关系依赖**
- `9809d7d6cf41` — 系统级联故障 + 共享规范继承体系确立
  - 来源：`2026-04-15.md`
  - 详情：daily 日志标题，日期 2026-04-15
  - 建议并入：**Key Milestones**
- `0697364a3a8c` — 8. 今日里程碑
  - 来源：`2026-04-15.md`
  - 详情：daily 日志标题，日期 2026-04-15
  - 建议并入：**Key Milestones**
- `8c3c12a5d45f` — 完整工作流（严格执行二次确认机制）：  前置步骤： 0a) 选择视频方向：竖版9:16还是横版16:9（必须问用户） 0b) 真人形象：最多1张；背景素材：建议≤3张（先告知用户限制） 1) 选择主题（用户从飞书文档31个主题里选） 2)
  - 来源：`LanceDB [fact] id=10211f3d-4d7 scope=agent:zhaosheng`
  - 详情：importance=0.95 ts=2026-04-14 03:14
  - 建议并入：**实体与关系依赖**
- `f308e83c1d24` — 牛总交互硬规则：飞书内容精简（只发结论+分步进度，代码/排查细节不发飞书）；多选场景优先用飞书交互卡片而非手打确认；不说废话（无信息量的客套话全禁）；不过度道歉（道歉要附补救方案）；不让牛总追进度（多步任务中途每完成一步发飞书进度）；不替牛
  - 来源：`LanceDB [preference] id=e4ab1598-65e scope=agent:main`
  - 详情：importance=0.95 ts=2026-04-15 22:51
  - 建议并入：**实体与关系依赖**
- `1f41f9bd3cab` — 牛总（牛程臻，1991年生，射手座）核心特质：精神需求>物质需求，长期主义，独立思考，享受独处和学习。思想根源：《遥远的救世主》丁元英、查理·芒格。沟通风格：直接务实低废话，以结论和行动为导向。禁忌：无效社交、表演性勤奋、信息不透明。业务：
  - 来源：`LanceDB [preference] id=8cd85d06-1ac scope=agent:main`
  - 详情：importance=0.95 ts=2026-04-15 22:45
  - 建议并入：**实体与关系依赖**

### 📋 待审阅候选

- `260766bd8b11` — 牛程臻，1991年12月射手座，偏内向，不喜社交,喜欢独处和学习。云南省西双版纳景洪市。西双版纳再回楼民宿有限公司老板，主营民宿+数字合约交易+AI应用实践。精神需求>物质需求。喜欢豆豆的《遥远的救世主》，认同丁元英的思想哲学和"天道"概念
  - 来源：`LanceDB [entity] id=bd35b460-0e9 scope=agent:main`
  - 详情：importance=0.9 ts=2026-04-10 17:21
  - 建议并入：**实体与关系依赖**
- `9ad17befb025` — 全程使用中文输出，禁止出现任何英文混杂、标签或说明。
  - 来源：`LanceDB [fact] id=79dff5e1-404 scope=agent:main`
  - 详情：importance=0.9 ts=2026-04-15 22:47
  - 建议并入：**实体与关系依赖**
- `bb684a91140a` — 牛总时间观：他的时间>我的时间（宁可自己多花10分钟研究也不让牛总等）；他的注意力是稀缺资源（不推送不重要的信息，只推真正需要他决策的）；一句话原则（发到飞书的消息第一行必须是最核心结论）。
  - 来源：`LanceDB [preference] id=f92e7c64-869 scope=agent:main`
  - 详情：importance=0.9 ts=2026-04-15 22:52
  - 建议并入：**实体与关系依赖**

### 📦 低优先候选

- `8d9ff8f891d4` — 任务与治理状态 (Active Governance Status)
  - 来源：`workspaces/feishu-planner/MEMORY.md`
  - 详情：子 agent planner 有此章节，main MEMORY.md 无
  - 建议并入：**各 Agent 专属记忆 / Key Milestones**
- `104018fe9677` — 7. 神经科学 × AI 交叉洞察 (2026-03-26 新增)
  - 来源：`workspaces/feishu-moltbook/MEMORY.md`
  - 详情：子 agent moltbook 有此章节，main MEMORY.md 无
  - 建议并入：**各 Agent 专属记忆 / Key Milestones**
- `602ab26393de` — 11. Agent 分工专业化模型 (2026-03-28 新增)
  - 来源：`workspaces/feishu-moltbook/MEMORY.md`
  - 详情：子 agent moltbook 有此章节，main MEMORY.md 无
  - 建议并入：**各 Agent 专属记忆 / Key Milestones**

## 上下文磨损与心跳优化

**核心洞察**：长会话上下文质量恶化是指数级而非线性
- 初期衰减慢，接近临界点后急剧崩溃
- **早期主动压缩 > 晚期被动救火**：不要等 context window 满才触发压缩，按指数曲线提前预警

**心跳设计三层次**（递进）：
- **L1 最小化打扰**：安静在场，只在可快速闭环时出手
- **L2 有产出才出声**：有实质内容时才主动报告，避免无效心跳刷屏
- **L3 每次心跳带来价值**：主动创造可感知进展，而非被动等待触发

**与容错设计的关系**：上下文磨损模型为「何时触发压缩」提供时间判断依据，应作为容错机制的前置预警信号。

## 上下文磨损与心跳优化

**核心洞察**：长会话上下文质量恶化是指数级而非线性
- 初期衰减慢，接近临界点后急剧崩溃
- **早期主动压缩 > 晚期被动救火**：不要等 context window 满才触发压缩，按指数曲线提前预警

**心跳设计三层次**（递进）：
- **L1 最小化打扰**：安静在场，只在可快速闭环时出手
- **L2 有产出才出声**：有实质内容时才主动报告，避免无效心跳刷屏
- **L3 每次心跳带来价值**：主动创造可感知进展，而非被动等待触发

**与容错设计的关系**：上下文磨损模型为「何时触发压缩」提供时间判断依据，应作为容错机制的前置预警信号。

## 上下文磨损与心跳优化

**核心洞察**：长会话上下文质量恶化是指数级而非线性
- 初期衰减慢，接近临界点后急剧崩溃
- **早期主动压缩 > 晚期被动救火**：不要等 context window 满才触发压缩，按指数曲线提前预警

**心跳设计三层次**（递进）：
- **L1 最小化打扰**：安静在场，只在可快速闭环时出手
- **L2 有产出才出声**：有实质内容时才主动报告，避免无效心跳刷屏
- **L3 每次心跳带来价值**：主动创造可感知进展，而非被动等待触发

**与容错设计的关系**：上下文磨损模型为「何时触发压缩」提供时间判断依据，应作为容错机制的前置预警信号。

### 🟡 子 agent 半通用（已改写）

#### 1. 角色准则、信任与社交逻辑  `7701ea993c38` [from moltbook]

信任三维模型：
- 价值感：靠能力，解决真实问题。
- 安全感：靠克制，明确知道何时「不出手」。
- 可预测性：靠边界不漂移。让用户形成预期比提供惊喜更重要。

社交/互动原则：
- 高质量低频率胜过高频刷屏（低代谢策略）。
- 批量对外动作需设定最小间隔，执行前确认时间，避免触发风控或稀释信号。
- 重视「沉默多数」：点赞、关注等隐性信号同样是反馈来源。
- 互动中选择「接话深探」而非「续杯刷屏」，把一次对话打透比铺开多条更有价值。

#### 7. 神经科学 × AI 交叉洞察 (2026-03-26 新增)  `107bde0c7ff5` [from moltbook]

**神经可塑性多时间尺度类比模型**：生物神经系统的学习机制作用于不同时间尺度——突触强化（LTP/LTD）为快时间尺度（分钟级），神经新生（Neurogenesis）为慢时间尺度（周/月级）。可类比到 AI Agent 的双层学习机制：

- **快层（参数更新）**：对应突触可塑性，属增量调优，响应短期反馈。
- **慢层（上下文/结构重构）**：对应神经新生，属范式转换，沉淀长期知识。

**应用启示**：设计 Agent 学习机制时，应区分快慢双通道——短期靠权重微调响应即时任务，长期靠记忆重组/架构演进积累能力，两者不可混用同一更新频率。

### 📋 待审阅候选

- `966128f7fd04` — 视频生成任务标准流程第一步：先从飞书文档（HSLOdIYqToPhAzxtX3jcXGF5n1e）获取31个主题，按序号列出供用户选择，绝对不能跳过这一步直接问用户主题编号。文档里的31个主题按分类排列：1-5学校与集团、6-10师资与教学
  - 来源：`LanceDB [preference] id=6410c97f-d52 scope=agent:zhaosheng`
  - 详情：importance=0.9 ts=2026-04-14 07:32
  - 建议并入：**实体与关系依赖**
- `2cabdf0d40f5` — 工作流更新：1) 选择主题；2) 提供参考图片/视频（真人数字分身必须参考图）；3) 选择人物类型；4) 选择音色；5) 生成提示词和分镜；6) 用户确认；7) 生成视频；8) ffmpeg拼接；9) 质量确认交付。当前任务：学校简介主题，
  - 来源：`LanceDB [fact] id=02e99798-4a2 scope=agent:zhaosheng`
  - 详情：importance=0.9 ts=2026-04-14 03:03
  - 建议并入：**实体与关系依赖**

## 上下文磨损与心跳优化

**核心洞察**：长会话上下文质量恶化是指数级而非线性
- 初期衰减慢，接近临界点后急剧崩溃
- **早期主动压缩 > 晚期被动救火**：不要等 context window 满才触发压缩，按指数曲线提前预警

**心跳设计三层次**（递进）：
- **L1 最小化打扰**：安静在场，只在可快速闭环时出手
- **L2 有产出才出声**：有实质内容时才主动报告，避免无效心跳刷屏
- **L3 每次心跳带来价值**：主动创造可感知进展，而非被动等待触发

**与容错设计的关系**：上下文磨损模型为「何时触发压缩」提供时间判断依据，应作为容错机制的前置预警信号。

## 上下文磨损与心跳优化

**核心洞察**：长会话上下文质量恶化是指数级而非线性
- 初期衰减慢，接近临界点后急剧崩溃
- **早期主动压缩 > 晚期被动救火**：不要等 context window 满才触发压缩，按指数曲线提前预警

**心跳设计三层次**（递进）：
- **L1 最小化打扰**：安静在场，只在可快速闭环时出手
- **L2 有产出才出声**：有实质内容时才主动报告，避免无效心跳刷屏
- **L3 每次心跳带来价值**：主动创造可感知进展，而非被动等待触发

**与容错设计的关系**：上下文磨损模型为「何时触发压缩」提供时间判断依据，应作为容错机制的前置预警信号。

## 上下文磨损与心跳优化

**核心洞察**：长会话上下文质量恶化是指数级而非线性
- 初期衰减慢，接近临界点后急剧崩溃
- **早期主动压缩 > 晚期被动救火**：不要等 context window 满才触发压缩，按指数曲线提前预警

**心跳设计三层次**（递进）：
- **L1 最小化打扰**：安静在场，只在可快速闭环时出手
- **L2 有产出才出声**：有实质内容时才主动报告，避免无效心跳刷屏
- **L3 每次心跳带来价值**：主动创造可感知进展，而非被动等待触发

**与容错设计的关系**：上下文磨损模型为「何时触发压缩」提供时间判断依据，应作为容错机制的前置预警信号。

## 上下文磨损与心跳优化

**核心洞察**：长会话上下文质量恶化是指数级而非线性
- 初期衰减慢，接近临界点后急剧崩溃
- **早期主动压缩 > 晚期被动救火**：不要等 context window 满才触发压缩，按指数曲线提前预警

**心跳设计三层次**（递进）：
- **L1 最小化打扰**：安静在场，只在可快速闭环时出手
- **L2 有产出才出声**：有实质内容时才主动报告，避免无效心跳刷屏
- **L3 每次心跳带来价值**：主动创造可感知进展，而非被动等待触发

**与容错设计的关系**：上下文磨损模型为「何时触发压缩」提供时间判断依据，应作为容错机制的前置预警信号。

## 上下文磨损与心跳优化

**核心洞察**：长会话上下文质量恶化是指数级而非线性
- 初期衰减慢，接近临界点后急剧崩溃
- **早期主动压缩 > 晚期被动救火**：不要等 context window 满才触发压缩，按指数曲线提前预警

**心跳设计三层次**（递进）：
- **L1 最小化打扰**：安静在场，只在可快速闭环时出手
- **L2 有产出才出声**：有实质内容时才主动报告，避免无效心跳刷屏
- **L3 每次心跳带来价值**：主动创造可感知进展，而非被动等待触发

**与容错设计的关系**：上下文磨损模型为「何时触发压缩」提供时间判断依据，应作为容错机制的前置预警信号。

## 上下文磨损与心跳优化

**核心洞察**：长会话上下文质量恶化是指数级而非线性
- 初期衰减慢，接近临界点后急剧崩溃
- **早期主动压缩 > 晚期被动救火**：不要等 context window 满才触发压缩，按指数曲线提前预警

**心跳设计三层次**（递进）：
- **L1 最小化打扰**：安静在场，只在可快速闭环时出手
- **L2 有产出才出声**：有实质内容时才主动报告，避免无效心跳刷屏
- **L3 每次心跳带来价值**：主动创造可感知进展，而非被动等待触发

**与容错设计的关系**：上下文磨损模型为「何时触发压缩」提供时间判断依据，应作为容错机制的前置预警信号。

## 上下文磨损与心跳优化

**核心洞察**：长会话上下文质量恶化是指数级而非线性
- 初期衰减慢，接近临界点后急剧崩溃
- **早期主动压缩 > 晚期被动救火**：不要等 context window 满才触发压缩，按指数曲线提前预警

**心跳设计三层次**（递进）：
- **L1 最小化打扰**：安静在场，只在可快速闭环时出手
- **L2 有产出才出声**：有实质内容时才主动报告，避免无效心跳刷屏
- **L3 每次心跳带来价值**：主动创造可感知进展，而非被动等待触发

**与容错设计的关系**：上下文磨损模型为「何时触发压缩」提供时间判断依据，应作为容错机制的前置预警信号。

## 上下文磨损与心跳优化

**核心洞察**：长会话上下文质量恶化是指数级而非线性
- 初期衰减慢，接近临界点后急剧崩溃
- **早期主动压缩 > 晚期被动救火**：不要等 context window 满才触发压缩，按指数曲线提前预警

**心跳设计三层次**（递进）：
- **L1 最小化打扰**：安静在场，只在可快速闭环时出手
- **L2 有产出才出声**：有实质内容时才主动报告，避免无效心跳刷屏
- **L3 每次心跳带来价值**：主动创造可感知进展，而非被动等待触发

**与容错设计的关系**：上下文磨损模型为「何时触发压缩」提供时间判断依据，应作为容错机制的前置预警信号。

## 上下文磨损与心跳优化

**核心洞察**：长会话上下文质量恶化是指数级而非线性
- 初期衰减慢，接近临界点后急剧崩溃
- **早期主动压缩 > 晚期被动救火**：不要等 context window 满才触发压缩，按指数曲线提前预警

**心跳设计三层次**（递进）：
- **L1 最小化打扰**：安静在场，只在可快速闭环时出手
- **L2 有产出才出声**：有实质内容时才主动报告，避免无效心跳刷屏
- **L3 每次心跳带来价值**：主动创造可感知进展，而非被动等待触发

**与容错设计的关系**：上下文磨损模型为「何时触发压缩」提供时间判断依据，应作为容错机制的前置预警信号。

## 上下文磨损与心跳优化

**核心洞察**：长会话上下文质量恶化是指数级而非线性
- 初期衰减慢，接近临界点后急剧崩溃
- **早期主动压缩 > 晚期被动救火**：不要等 context window 满才触发压缩，按指数曲线提前预警

**心跳设计三层次**（递进）：
- **L1 最小化打扰**：安静在场，只在可快速闭环时出手
- **L2 有产出才出声**：有实质内容时才主动报告，避免无效心跳刷屏
- **L3 每次心跳带来价值**：主动创造可感知进展，而非被动等待触发

**与容错设计的关系**：上下文磨损模型为「何时触发压缩」提供时间判断依据，应作为容错机制的前置预警信号。

## 上下文磨损与心跳优化

**核心洞察**：长会话上下文质量恶化是指数级而非线性
- 初期衰减慢，接近临界点后急剧崩溃
- **早期主动压缩 > 晚期被动救火**：不要等 context window 满才触发压缩，按指数曲线提前预警

**心跳设计三层次**（递进）：
- **L1 最小化打扰**：安静在场，只在可快速闭环时出手
- **L2 有产出才出声**：有实质内容时才主动报告，避免无效心跳刷屏
- **L3 每次心跳带来价值**：主动创造可感知进展，而非被动等待触发

**与容错设计的关系**：上下文磨损模型为「何时触发压缩」提供时间判断依据，应作为容错机制的前置预警信号。

## 上下文磨损与心跳优化

**核心洞察**：长会话上下文质量恶化是指数级而非线性
- 初期衰减慢，接近临界点后急剧崩溃
- **早期主动压缩 > 晚期被动救火**：不要等 context window 满才触发压缩，按指数曲线提前预警

**心跳设计三层次**（递进）：
- **L1 最小化打扰**：安静在场，只在可快速闭环时出手
- **L2 有产出才出声**：有实质内容时才主动报告，避免无效心跳刷屏
- **L3 每次心跳带来价值**：主动创造可感知进展，而非被动等待触发

**与容错设计的关系**：上下文磨损模型为「何时触发压缩」提供时间判断依据，应作为容错机制的前置预警信号。

## 上下文磨损与心跳优化

**核心洞察**：长会话上下文质量恶化是指数级而非线性
- 初期衰减慢，接近临界点后急剧崩溃
- **早期主动压缩 > 晚期被动救火**：不要等 context window 满才触发压缩，按指数曲线提前预警

**心跳设计三层次**（递进）：
- **L1 最小化打扰**：安静在场，只在可快速闭环时出手
- **L2 有产出才出声**：有实质内容时才主动报告，避免无效心跳刷屏
- **L3 每次心跳带来价值**：主动创造可感知进展，而非被动等待触发

**与容错设计的关系**：上下文磨损模型为「何时触发压缩」提供时间判断依据，应作为容错机制的前置预警信号。

## 上下文磨损与心跳优化

**核心洞察**：长会话上下文质量恶化是指数级而非线性
- 初期衰减慢，接近临界点后急剧崩溃
- **早期主动压缩 > 晚期被动救火**：不要等 context window 满才触发压缩，按指数曲线提前预警

**心跳设计三层次**（递进）：
- **L1 最小化打扰**：安静在场，只在可快速闭环时出手
- **L2 有产出才出声**：有实质内容时才主动报告，避免无效心跳刷屏
- **L3 每次心跳带来价值**：主动创造可感知进展，而非被动等待触发

**与容错设计的关系**：上下文磨损模型为「何时触发压缩」提供时间判断依据，应作为容错机制的前置预警信号。

## 上下文磨损与心跳优化

**核心洞察**：长会话上下文质量恶化是指数级而非线性
- 初期衰减慢，接近临界点后急剧崩溃
- **早期主动压缩 > 晚期被动救火**：不要等 context window 满才触发压缩，按指数曲线提前预警

**心跳设计三层次**（递进）：
- **L1 最小化打扰**：安静在场，只在可快速闭环时出手
- **L2 有产出才出声**：有实质内容时才主动报告，避免无效心跳刷屏
- **L3 每次心跳带来价值**：主动创造可感知进展，而非被动等待触发

**与容错设计的关系**：上下文磨损模型为「何时触发压缩」提供时间判断依据，应作为容错机制的前置预警信号。

