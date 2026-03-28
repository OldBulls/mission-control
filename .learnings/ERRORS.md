# Errors

Append structured entries:
- ERR-YYYYMMDD-XXX for command/tool/integration failures
- Include symptom, context, probable cause, and prevention

## ERR-20260318-001: .learnings 目录路径混淆
**Date**: 2026-03-18 03:42  
**Symptom**: 在 Self-Improving 检查中尝试访问 `~/self-improving/LEARNINGS.md` 和 `~/.learnings/` 时失败（No such file or directory）。  
**Context**: 执行 cron 任务"最强大脑-Self-Improving检查"时，试图读取 `.learnings` 目录以整理经验。  
**Root Cause**: `.learnings` 目录实际位于 `~/.openclaw/workspace/.learnings/`，而非 `~/self-improving/` 或 `~/.learnings/`。Self-improving 系统使用 `~/self-improving/domains/` 存储领域知识，而 `.learnings` 是工作区级别的学习日志。  
**Prevention**: 
- 在 Self-Improving 检查中，应明确区分两个系统：
  - `~/self-improving/` - 长期领域知识库（domains/task-planning.md 等）
  - `~/.openclaw/workspace/.learnings/` - 短期学习日志（LEARNINGS.md, ERRORS.md）
- 更新 Self-Improving 检查逻辑，优先读取 `~/.openclaw/workspace/.learnings/LEARNINGS.md` 提取近期经验，再整合到 `~/self-improving/domains/` 对应文件。

## ERR-20260318-002: Gateway 重启期间消息延迟，导致已收到消息但回复明显滞后
**Date**: 2026-03-18 09:04  
**Symptom**: 用户在网页端连续发送多条消息后，系统长时间无回复，待 Gateway/模型切换完成后才集中恢复响应。  
**Context**: 修改 `~/.openclaw/openclaw.json` 中 `main` / `planner` 的主模型后，立即执行 `openclaw gateway restart`。重启过程中当前会话仍有新消息进入。  
**Root Cause**: 模型切换和 Gateway 重启是全局变更，现有流程没有在文档和脚本中明确提醒“重启窗口会短暂阻塞当前会话收发”，也没有先做状态检查、再做重启后的可用性确认。  
**Prevention**:  
- 模型切换后使用“修改配置 -> 重启 Gateway -> 等待 -> 验证状态 -> 再继续对话”的固定流程。  
- 启动文档里明确说明：重启期间消息可能延迟，不要在切换窗口连续发送多条关键消息。  
- `start-and-check.sh` 在启动后增加 `openclaw gateway status` 验证，避免仅靠固定 `sleep 5`。


## [ERR-20260320-001]

**Logged**: 2026-03-20T12:07:53.683Z
**Priority**: low
**Status**: pending
**Area**: docs

### Summary
Tool result was surfaced as an error wrapper for a normal file read, causing noisy error-detected context.

### Details
A read of alert-notify.sh returned file content but was also flagged in the session as an error signal. This appears to be an upstream wrapper/reporting issue rather than a task failure.

### Suggested Action
Treat read outputs as successful when content is returned, and only escalate if the tool explicitly reports a real read failure.

### Metadata
- Source: memory-lancedb-pro/self_improvement_log
---


## [ERR-20260320-002]

**Logged**: 2026-03-20T21:02:39.556Z
**Priority**: medium
**Status**: pending
**Area**: docs

### Summary
Feishu doc creation failed due to missing document write scopes

### Details
Attempted feishu_doc action=create twice for an OpenClaw installation guide doc; both returned HTTP 400. feishu_app_scopes showed docs read scope but no docx/document write-related scopes required by feishu_doc skill, so creation could not proceed. sessions_send to planner also timed out and should not be relied on for quick group-thread collaboration.

### Suggested Action
Before promising Feishu doc creation, check feishu_app_scopes for document write permissions or phrase the action as conditional. Avoid blocking on sessions_send in time-sensitive group replies.

### Metadata
- Source: memory-lancedb-pro/self_improvement_log
---


## [ERR-20260321-001]

**Logged**: 2026-03-21T00:38:19.598Z
**Priority**: medium
**Status**: pending
**Area**: config

### Summary
session_status model override rejected for disallowed model alias

### Details
Tried to switch the current session with session_status(model="anthropic/claude-opus-4.6") and received: Model "anthropic/claude-opus-4.6" is not allowed. Need to verify allowed model IDs/aliases before promising a model switch.

### Suggested Action
When user asks to switch models, first inspect current session_status and use documented allowed aliases or check workspace config/docs before attempting the override.

### Metadata
- Source: memory-lancedb-pro/self_improvement_log
---


## [ERR-20260321-002]

**Logged**: 2026-03-21T06:08:41.740Z
**Priority**: medium
**Status**: pending
**Area**: infra

### Summary
sessions_send timeout can coexist with successful target-side execution and pending announce delivery

### Details
In multi-agent collaboration, sessions_send to agent:moltbook:main timed out twice, but target session executed the Moltbook post successfully and later returned status via a separate successful sessions_send. Session history shows target processed the task and produced a result while parent side initially reported timeout. This indicates timeout/ack issues in the inter-session communication layer, not execution failure.

### Suggested Action
When sessions_send times out, do not equate it with task failure. Check target session history or re-query the target agent before reporting failure, and treat timeout as 'unknown/pending' unless execution is disproven.

### Metadata
- Source: memory-lancedb-pro/self_improvement_log
---


## [ERR-20260321-003]

**Logged**: 2026-03-21T07:14:24.949Z
**Priority**: low
**Status**: pending
**Area**: docs

### Summary
Exact-text edit failed because AGENTS.md context had drifted from the expected snippet

### Details
While adding the new '规则更新后二次验收' section, an initial edit call failed with 'Could not find the exact text'. The file had already evolved enough that the original anchor block no longer matched byte-for-byte. I recovered by reading the relevant section first and then inserting against the exact current heading boundary.

### Suggested Action
Before editing evolving rule files with exact-match replacement, read the nearby section and anchor on a smaller stable heading or exact current snippet instead of a larger assumed block.

### Metadata
- Source: memory-lancedb-pro/self_improvement_log
---


## [ERR-20260321-004]

**Logged**: 2026-03-21T13:31:39.711Z
**Priority**: high
**Status**: pending
**Area**: docs

### Summary
Internal English working notes leaked into user-visible group replies during multi-step document updates

### Details
During repeated read/edit/retry cycles, short internal English notes like 'Need read then edit' and 'Need fix registry duplication' appeared in visible replies. This is a presentation/containment failure, not user-facing intent. It recurred multiple times in the same session while iterating on docs and registry updates.

### Suggested Action
For user-visible replies, never echo internal planning fragments or tool-thought shorthand; send only finalized Chinese summaries after tool work completes, and treat leaked internal notes as a blocking presentation bug to correct immediately.

### Metadata
- Source: memory-lancedb-pro/self_improvement_log
---


## [ERR-20260321-005]

**Logged**: 2026-03-21T14:34:24.465Z
**Priority**: high
**Status**: pending
**Area**: docs

### Summary
Internal English work notes continued leaking into user-visible replies during Feishu trigger-mode debugging

### Details
While debugging Feishu trigger behavior and applying config/code changes, short internal notes like 'Need edit files' and 'Need mention restart maybe' still surfaced in visible replies. This repeated after prior corrections and should be treated as the same display-layer containment failure.

### Suggested Action
During multi-step debugging, do not send any interim reasoning fragments. Finish tool work first, then reply only with finalized Chinese conclusions and next actions.

### Metadata
- Source: memory-lancedb-pro/self_improvement_log
---


## [ERR-20260321-006]

**Logged**: 2026-03-21T16:01:51.140Z
**Priority**: high
**Status**: pending
**Area**: config

### Summary
oc-restart 调用未定义的 ensure_managed_cron 导致脚本中断

### Details
在收口 openclaw-manage.sh restart 行为时，把 restart 分支改为调用 ensure_managed_cron，但函数定义未被 shell 正确加载，用户执行 oc-restart 时报错：line 172: ensure_managed_cron: command not found。

### Suggested Action
修改 openclaw-manage.sh 时，执行一次脚本级自检（如 bash -n 和目标命令 dry-run/静态检查），避免把未定义函数名落进可执行路径。

### Metadata
- Source: memory-lancedb-pro/self_improvement_log
---


## [ERR-20260321-007]

**Logged**: 2026-03-21T16:15:57.067Z
**Priority**: medium
**Status**: pending
**Area**: infra

### Summary
全盘脚本引用扫描两次被 SIGTERM 中断，需改为聚焦候选脚本分批核验

### Details
在检查 ~/.openclaw/scripts 全量引用关系时，使用 root.rglob 全树扫描两次都被系统 SIGTERM 终止，导致无法直接产出全量可删清单。

### Suggested Action
对大目录引用核验改用候选脚本分批扫描或先限定 docs/cron/scripts/LaunchAgent 关键范围，避免长时间全树遍历。

### Metadata
- Source: memory-lancedb-pro/self_improvement_log
---


## [ERR-20260321-008]

**Logged**: 2026-03-21T19:00:48.012Z
**Priority**: medium
**Status**: pending
**Area**: config

### Summary
Weekly subworkspace backup failed to push to GitHub.

### Details
git push failed for ~/.openclaw/workspaces. 
Error: remote: Repository not found. fatal: repository 'https://github.com/OldBulls/openclaw-subworkspaces.git/' not found.
Local commit was successful (470dbc7).

### Suggested Action
Check if the remote repository exists or if there is a GitHub authentication/permission issue for the current environment.

### Metadata
- Source: memory-lancedb-pro/self_improvement_log
---


## [ERR-20260321-009]

**Logged**: 2026-03-21T19:30:46.491Z
**Priority**: low
**Status**: pending
**Area**: infra

### Summary
Workspace shell environment lacks `rg`, causing preferred search command to fail during heartbeat checks.

### Details
During 2026-03-22 heartbeat handling, `exec` of `rg -n "lastMemoryReview" -S /Users/zaihuilou/.openclaw/workspace` failed with `zsh:1: command not found: rg`. Fallback to `grep` worked. Prefer checking for `rg` availability or using a resilient fallback in shell commands.

### Suggested Action
When using shell search in this workspace, guard `rg` usage with `command -v rg >/dev/null || ...` or use `grep` fallback immediately.

### Metadata
- Source: memory-lancedb-pro/self_improvement_log
---


## [ERR-20260322-001]

**Logged**: 2026-03-22T01:30:14.419Z
**Priority**: low
**Status**: pending
**Area**: tools

### Summary
Broad recursive text scan pulled in session backup noise and produced unusable results.

### Details
A recursive content scan over the workspace matched large `Assets/SessionBackups` JSONL files, which overwhelmed the output with historical tool snapshots instead of live docs/scripts. This was a search-scope mistake rather than a platform failure.

### Suggested Action
When auditing live references, explicitly exclude backup/session snapshot directories or target only active roots like `scripts`, `memory`, and `docs`.

### Metadata
- Source: memory-lancedb-pro/self_improvement_log
---


## [ERR-20260325-001]

**Logged**: 2026-03-25T12:09:57.120Z
**Priority**: medium
**Status**: pending
**Area**: config

### Summary
Memory storage embedding provider authentication failed

### Details
Jina embedding API 返回 403，可能是 API key 过期或权限问题。memory_store 无法正常工作。

### Suggested Action
检查 Jina API key 配置，或切换到本地 Ollama embedding 端点

### Metadata
- Source: memory-lancedb-pro/self_improvement_log
---

### [ERR-20260327-001]

**Logged**: 2026-03-27T10:35:00+08:00
**Priority**: medium
**Status**: pending
**Area**: multi-agent协作

### Summary
moltbook 自维护总检查 cron 任务 Edit 操作失败

### Details
cron task `0d77fc19-...`（moltbook 自维护总检查）在 isolated session 中运行时，执行 `⚠️ 📝 Edit failed`。错误发生在 2026-03-27 08:00 CST。13:20 CST 轮次在运行中尚未返回结果。

### Suggested Action
1. 待 13:20 轮次完成后确认是否仍然失败
2. 若持续失败，检查 moltbook cron 任务的 payload 是否硬编码了主控 memory 路径（`~/.openclaw/workspace/memory/`），而 isolated session 无法访问该路径
3. 建议：moltbook 自维护总检查应只写自己 workspace（`~/.openclaw/workspaces/feishu-moltbook/memory/`），不写主控 memory

### Metadata
- Source: main heartbeat 2026-03-27 18:35
- Task: moltbook self-maintenance total check
- Error pattern: isolated session write to main memory path


## [ERR-20260327-002]

**Logged**: 2026-03-27T17:01:47.861Z
**Priority**: medium
**Status**: pending
**Area**: config

### Summary
memory_store 在系统维护收口时失败，原因是 embedding provider Jina 返回 403

### Details
在 2026-03-28 01:00 系统维护 cron 收口阶段，先用 scope=main 写结构化记忆被拒绝（Access denied to scope: main），改用默认 scope 后仍失败，错误为 Embedding provider authentication failed (403)。这说明 memory-lancedb-pro 当前 embedding 配置仍有认证问题，影响结构化记忆写入闭环。

### Suggested Action
后续优先核对 memory-lancedb-pro 的 embedding.apiKey / retrieval.rerankApiKey 是否生效，并在修复后补写本次维护的结构化记忆。

### Metadata
- Source: memory-lancedb-pro/self_improvement_log
---


## [ERR-20260327-003]

**Logged**: 2026-03-27T17:10:58.306Z
**Priority**: medium
**Status**: pending
**Area**: memory

### Summary
memory_store 在记录 Jina 监控结论时失败

### Details
首次调用 memory_store 使用 scope=openclaw/main 被拒绝（Access denied to scope）。改为默认 scope 后再次调用，返回 Embedding provider authentication failed (403: 403 status code (no body))，说明当前结构化记忆写入链路依赖的 Jina embedding 配置不可用，即使主 Key 直接请求 embeddings API 仍返回 200。

### Suggested Action
后续涉及结构化记忆收口时，先验证 memory_store 可写性；若失败，记录到文件并提示用户检查 memory 插件使用的 embedding.apiKey / endpoint 是否与手工健康检查使用的 Key 一致。

### Metadata
- Source: memory-lancedb-pro/self_improvement_log
---


## [ERR-20260329-001]

**Logged**: 2026-03-29T00:03+08:00
**Priority**: low
**Status**: pending
**Area**: tools

### Summary
message tool 在 heartbeat 环境中报错 Unknown channel: feishu

### Details
在周日记忆整理后尝试通过 `message(action=send, channel=feishu)` 发送提醒给牛总，收到错误 "Unknown channel: feishu"。这表明 heartbeat 运行环境可能没有 feishu channel 配置或配置未加载。

### Suggested Action
heartbeat 环境发送消息时，应先检查可用 channel 或直接在回复中提示用户，不依赖 message 工具。需要用户主动查的提醒可写入 memory 日志并在下次用户对话时同步。

### Metadata
- Source: heartbeat session 2026-03-29 00:03
- Context: 记忆整理完成，需提醒 moltbook 认领 pending
