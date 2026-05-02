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
- **子 agent:** planner / moltbook / minsu / shangfang / yying / zhaosheng / reflector
- **模型：** deepseek/deepseek-v4-pro（主模型），fallback aigpt-openai/gpt-5.4 → gpt-5.5
- **Gateway：** LaunchAgent 直启，快口径运维（launchctl + HTTP + boot window）

---

## 关键配置（稳定口径）

- **主模型：** deepseek/deepseek-v4-pro
- **备选模型：** aigpt-openai/gpt-5.4 / gpt-5.5 / gpt-5.4-mini / gpt-5.4-nano
- **心跳周期：** 每 2 小时（cron ID: 4628add5）
- **session prune：** 14 天（pruneDays=14）
- **bootstrapMaxChars：** 50000（2026-04-10 调整，防止截断）
- **exec 安全模式：** YOLO（security=full，main 全命令免审批）
- **飞书主链：** `~/.openclaw/extensions/openclaw-lark`（bundled feishu 禁用）

---

## Gateway 运维基线（2026-04-12 补充）

- **健康快口径：** `launchctl + HTTP + logs/gateway.log` boot window + Feishu WebSocket 信号
- **判据：** `gateway=1 http=1 feishu_ok=5/5 retry_signals=0` → 正常
- **排除故障路径：** 若 rpc=0/rpc=skip，但 `gateway=1 http=1 feishu_ok=5/5`，按 CLI/probe 慢处理，不按 Feishu 生产故障处理
- **重启 grace：** WARN warming=1 时等 1-2 分钟再复查
- **升级后必检：** reasonDefault=off、/reset、多 agent footer、招生助理

---

## Skills 状态

| Skill | 版本 | 状态 | 备注 |
|-------|------|------|------|
| lossless-claw | — | ✅ 活跃 | 会话压缩 |
| memory-lancedb-pro | — | ✅ 活跃 | 语义记忆 |
| proactivity | — | ✅ 活跃 | 主动任务推进 |
| self-improving | — | ✅ 活跃 | 长期经验 |
| Graphify | — | ✅ 活跃 | 图谱构建 |
| taskflow | — | ✅ 活跃 | 多步骤任务 |
| Background Review Agent | v1.2.0 | ✅ 活跃 | cron 每 5 分钟触发 |

---

## test-after-write 三步验证法

任何配置文件修改后必须执行：
1. **读回验证** — 确认写入内容正确
2. **功能验证** — 实际操作确认生效
3. **交叉验证** — 检查相关引用是否一致

---

## 收口强制扫描规则（2026-04-13 确立）

每次收口必须扫描：`AGENTS.md` / `SOUL.md` / `TOOLS.md` / `MEMORY.md` / `USER.md` / `HEARTBEAT.md` / `BOOT.md` — 确认口径一致、无遗漏、无冲突。

---

## 自主记忆决策规则（2026-04-13 确立）

牛总于 2026-04-13 授权：对于明确可推断的决策（配置修复、重复踩坑、自动化清理），main 可自行决定并执行，事后告知牛总。

---

## 部署/变更交叉引用验证（2026-04-14 新增）

> 由多 Agent 记忆架构部署时 corpus symlinks 未同步清理的教训产生。

**触发**：任何多组件部署、目录删除、配置变更、文件迁移

**检查项**：
- 删了目录 → corpus symlinks 是否残留
- 改了 ignore pattern → corpus 实际是否生效
- 新增了目录 → corpus 是否收录
- 改了配置 → MEMORY.md/AGENTS.md/SOUL.md 口径是否一致
- 改了脚本 → idempotency（重复运行是否干净）

---

## 多文件改动：一变三查制度（2026-04-13 新增）

> 由 execution.md 两处 lightContext 漏改教训产生。

**三查流程**：
1. **改动前 grep** — 列出所有相关文件中的目标引用
2. **逐个修改** — 每次修改后立即读回验证
3. **改动后 grep 对照** — 与改动前结果对比，确认全部更新到位

---

## 工具偏好

- **图片生成：** 未配置（`imageGenerationModel` 缺失，`aigpt-openai/gpt-image-2` 已在模型列表中待启用）

---

## ⚠️ Key/Secret 写入规范（强制）

> 教训：key 被截断导致 SiliconFlow 三次认证失败（写入40位而非51位）

1. **完整复制**：从源头文件复制完整 key，不能手动截取
2. **读回验证**：写入后立即读取并验证长度
3. **功能验证**：curl/API 调用确认有效
4. **三步完成前禁止回报"已修复"/"已更新"**

---

## 语言规则（强制，全员遵守）

- ✅ 全程中文，不主动英文
- ❌ 禁止在中文回复里夹杂英文（变量名/命令/技术术语除外，但必须给出中文解释）
- ❌ 禁止英文标题、英文标牌、英文注释
- 工具叙事必须中文：`检查发现...` / `确认了...` / `已执行...`，禁止 `Let me check` / `Good news` / `Interesting`

---



## openclaw-tutorial 作者端升级链基线（2026-04-27）

- **教程仓库：** `/Users/zaihuilou/openclaw-tutorial`，远程 `OldBulls/openclaw-tutorial`，生产站点 `https://chengzhen.vip`。
- **当前提交：** `e601824 docs: sync openclaw-lark 2026.4.8 patch flow` 已推送 `origin/main`；GitHub Actions `site-build` / `site-deploy` / `e2e-install` / `syntax-check` / `link-check` / `mirror-public` 全绿。
- **openclaw-lark 基线：** 教程项目已同步为 `2026.4.8`，旧 `2026.4.7` patch 模板已移除；新模板为 `templates/scripts/patches/openclaw-lark-2026.4.8-*.template.patch`。
- **作者端升级主入口：** `scripts/author/approve-openclaw-upgrade.sh`。作者本机升级不是单一 CLI hotfix 链，而是：`npm install -g openclaw@<version>` → `openclaw --version` / `openclaw doctor` → `apply-openclaw-cli-hotfixes.mjs`（如存在）→ `apply-openclaw-lark-patches.sh`（如存在）→ `verify-openclaw-lark-patches.sh`（如存在）→ Gateway 重启 + Feishu/Gateway 健康探测。
- **状态字段：** `~/.openclaw/state/author-upgrade-result.json` 包含 `lark_apply_patch_status` / `lark_verify_patch_status`；若失败，先修 openclaw-lark patch 链，再重试作者升级。
- **发布路径：** 本机无 `CLOUDFLARE_API_TOKEN` 时，`npx wrangler deploy` 可能因 Cloudflare 403 / bot challenge 失败；该项目发布优先走 GitHub Actions `site-deploy.yml`，本地直发仅在显式注入有效 Cloudflare token 后使用。
- **排障口径：** 不再把 footer metrics / multi-agent card footer 修复归到 CLI hotfix；它属于 `openclaw-lark` 插件 patch 链。CLI hotfix 与插件 patch 必须分开描述、分开验证。

## openclaw-tutorial 版本/文档同步复查规则（2026-05-02）

- **触发条件：** 任何安装包发版、OpenClaw 基线调整、买家安装/更新链修复、gateway/runtime 自愈改动、memory-lancedb-pro 补丁改动、公开 docs 更新。
- **必须复查：** `meta/PRODUCTION_BASELINE.md`、`site/wrangler.toml`、`site/.env.example`、`docs/_REWRITE_STATUS.md`、`site/astro.config.mjs` 侧边栏、`docs/00-先读我.md` 版本说明、`docs/07-实战/00-最近新增能力总览.md`、`docs/FAQ.md`。
- **版本事实：** R2 `latest.json`、`latest.json.sig`、zip、manifest、zip.sig、manifest.sig、公钥、`LATEST_BUNDLE_HISTORY_JSON` 必须与当前稳定包一致；本地 `.release-artifacts` 缺某版 `latest.json` 时要补齐，否则版本历史会漏项。
- **已踩坑：** v1.0.10 后发现站点 fallback 和生产基线仍停在 v1.0.9，侧边栏漏挂已有章节，`docs/_REWRITE_STATUS.md` 章节数误写 43，memory-lancedb-pro 要同时校验源码和 `openclaw.plugin.json` 的 `autoRecallTimeoutMs=15000`。
- **最小验证：** `cd site && npm run build`、`bash tests/e2e-site-auth-registry-smoke.sh`、`git diff --check`、docs 脱敏扫描；改买家安装/模板时再加 `tests/e2e-install.sh`、`tests/e2e-update.sh`、`tests/e2e-gateway-service-repair.sh`。

## Key Milestones

| 日期 | 里程碑 |
|------|--------|
| 2026-04-15 | **记忆整合系统上线**：consolidate-memory.py，memory/ 81→31 文件（减少48个），reflections/ March 旧数据清理（2.0M），cron 每周日 03:00 |
| 2026-04-15 下午 | **LanceDB 碎片压缩**：fragments 963→92（-90%），versions 1805→988，新增 `run-lance-monitor.py` 监控脚本 |
| 2026-04-15 | **全 Agent 模型标准化**：全部 agent primary model 统一，gptclub 模型全面清理 |
| 2026-04-15 18:40 | **系统 cascade failure + 恢复**：planner 使用不存在模型 gptclub-openai/gpt-5.4 + slug-generator 超时×2 + moltbook context overflow → 全面 gptclub 清理后恢复 |
| 2026-04-15 傍晚 | **日报增强 + 周报系统上线**：daily-digest.py 增强，weekly-distillation.py + launchd plist（每周日 10:00）|
| 2026-04-15 19:30 | **SOUL.md 继承体系确立**：`shared/SOUL.md` + `bootstrap-extra-files` 为唯一同步机制 |
| 2026-04-15 20:17 | **sync-agent-souls.py 已移除**：无幂等性保护，重复运行会污染 agent SOUL.md |
| 2026-04-17 | Tutorial B 打包（config.tar.gz v1.0.0，217MB）；moltbook API Key 轮换完成 |
| 2026-04-19 | 自动化维护正常运行，系统状态正常 |

---

## ⭐ Weekly Distillation — 2026-04-20

> 本周关键教训，稳定规则沉淀

### 1. MEMORY.md 重复 append 自动化 bug（Apr 20 发现）

- **现象**：MEMORY.md 膨胀至 68KB，"上下文磨损与心跳优化" 重复 21 次
- **根因**：`memory-merge-weekly.py` 或周合并脚本在追加 `## 上下文磨损与心跳优化` 时，每次都是 append 而非检查是否已存在后更新
- **教训**：任何生成/合并脚本若会写入带 `## 标题` 的 section，必须先 grep 查是否已存在，存在则更新（replace）而非追加（append）
- **修复**：手工重建 MEMORY.md，删除所有重复 section，建立 clean 版本
- **稳定规则**：脚本对 MEMORY.md 的写操作必须幂等；写入前先检查是否已存在同名 section

### 2. 系统 cascade failure 复盘（Apr 15，18:01）

- **触发链**：slug-generator 超时×2 → planner 调用不存在模型（gptclub-openai/gpt-5.4）→ moltbook context overflow → auto-compaction 失败 → 主 session 245s timeout
- **根因**：planner 的 `openclaw.json` 有残留 gptclub 模型注册，但 `models.json` 中无此 provider
- **修复**：全面清理所有 agent 的 gptclub 模型注册（openclaw.json + 所有 models.json）
- **教训**：模型配置变更后，必须同步检查所有引用方；任何 provider 删除后，所有使用该 provider 的 agent 必须同步更新

### 3. 全 Agent SOUL.md 中文统一（Apr 15）

- **执行**：所有 7 个子 agent 的 workspace SOUL.md 统一为纯中文结构模板
- **涉及**：planner / moltbook / minsu / yying / zhaosheng / shared/SOUL.md
- **遗留**：planner 和 moltbook 运行中的 session 需 /reset 才生效新配置
- **稳定规则**：新 agent 创建时，SOUL.md 直接用中文模板，不从英文模板翻译

### 4. bootstrap-extra-files 确立为唯一同步机制（Apr 15）

- `sync-agent-souls.py` 有幂等性问题，已删除
- `shared/SOUL.md` / `shared/AGENTS.md` / `shared/TOOLS.md` 通过 `bootstrap-extra-files` 自动加载
- 各 agent 的 workspace 文件为 agent 特定覆盖，不冲突

### 5. 语言规则增强：工具叙事必须中文（Apr 15 傍晚）

- 禁止：`Let me check` / `Good news` / `Interesting` / 任何英文开头
- 必须：`检查发现...` / `确认了...` / `已执行...`
- 飞书消息、卡片内容、技术文档全部适用

---

## 周合并摘要 — 2026-04-26

> 原长文已归档：`memory/archive/MEMORY-WEEKLY-MERGE-2026-04-26.md`。MEMORY.md 只保留可执行摘要，避免 bootstrap 膨胀。

- **多 Agent 协作**：优先职责切分与接口协议，交接用“描述-标准-确认”三段式；对外声明前做可逆性预检。
- **记忆管理**：结构化压缩优于简单截断；身份依赖记忆按“框架有效性”评估，不按普通时间衰减处理。
- **上下文管理**：上下文不是记忆；早期主动压缩优于晚期救火。
- **规划 agent 能力**：方案拆解、执行者推荐、依赖/验收梳理、闭环标准维护。
- **多媒体内容生产**：确认主题 → 文案 → 素材 → TTS → 分镜提示词 → 生成片段 → 校对 → 交付。

## 上下文磨损与心跳优化

- 长会话上下文质量恶化是指数级而非线性，接近临界点后会急剧崩溃。
- 心跳三层次：L1 最小化打扰；L2 有产出才出声；L3 每次心跳带来价值。
- 上下文磨损模型是“何时触发压缩”的前置预警信号。

## ⭐ Weekly Distillation — 2026-04-27

> 本周关键工程教训，稳定规则沉淀

### 1. 周合并内容不能直接长文进 MEMORY.md

- **现象**：04-26 周合并把子 agent 长文直接追加进 MEMORY.md，导致文件达到 22KB+。
- **规则**：周合并原文进入 `memory/archive/`；MEMORY.md 只保留 5-8 条可执行摘要。
- **验证**：本轮已归档 `MEMORY-WEEKLY-MERGE-2026-04-26.md`，核心文件恢复到目标范围内。

### 2. Background Review 双哨兵清理

- **现象**：`pending-background-review` 与 `.running` 同时存在时，会形成重复触发风险。
- **规则**：Background Review 退出前必须原子清理 flag 与 running sentinel，并重置 counter。
- **补充**：出现 `missing tool result` / `No result provided` 时，停止堆叠并行读取，改用单条 shell 聚合读取。

### 3. 模型与生图能力要分开验证

- **结论**：聊天模型可用不代表图像接口可用；中转站 GPT 聊天能力不等于 GPT 生图能力。
- **规则**：接生图前必须分别验证：图像 API 路径、图像模型名、鉴权方式、一次真实生图调用。
- **本周配置**：本地已移除 `gpt-5.5-mini/nano`，保留并验证 `aigpt-openai/gpt-5.5` 可调用；用户要求全部 agent 切到 `gpt-5.5`。

### 4. Gateway 健康脚本适配 2026.4.24 日志变化

- **现象**：升级后 `check-feishu-gateway-health.sh --summary` 误报 `feishu_ok=0/6`，但 Feishu 实际可收发。
- **根因**：当前 boot 不再稳定输出账号级 `bot open_id resolved`。
- **修复规则**：优先认 `bot open_id resolved`；缺失时用账号级 WebSocket 启动 + 当前 boot 的 `ws client ready` 数量作为 runtime ready fallback。

### 5. openclaw-tutorial 作者端升级链固化

- 教程仓库以作者端视角处理：作者追 upstream 并发布稳定 bundle，买家只消费稳定 bundle。
- 作者端升级主链：npm 安装 OpenClaw → doctor → CLI hotfix → openclaw-lark apply/verify → Gateway 重启与 Feishu/Gateway 健康探测。
- 本地无 `CLOUDFLARE_API_TOKEN` 时不强行 Wrangler 直发，优先走 GitHub Actions。

---

*本文件由智能管家每周维护时更新。MEMORY.md 只保留稳定规则和关键事实，原始周合并/长日志归档到 memory/archive。*
