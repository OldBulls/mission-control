# HEARTBEAT.md - 智能管家心跳引擎

> 本文件定义智能管家的"主动脉搏"——多久醒一次，醒来干什么。
> 基于龙虾教练体系文档17精读，2026-04-10 重构。

---

## 心跳基本参数

| 参数 | 值 | 说明 |
|------|-----|------|
| 频率 | 每 2 小时（7200s） | cron ID: 4628add5 |
| 触发条件 | 文件存在且非空注释 | 空文件跳过期激活 |
| 正常响应 | `HEARTBEAT_OK` | 无实质性进展时回复 |
| 消息阈值 | 有警告或错误 → 发飞书；全部正常 → 不发 |

---

## 心跳任务清单

### P0 - 系统健康检查（每次心跳必检）

```
检查项目：
1. Gateway 是否 ready（openclaw gateway status）
2. lossless-claw 是否正常运行
3. memory-lancedb-pro 是否正常
4. 心跳 cron 是否正常（consecutiveErrors: 0）
5. taskflow scheduler 是否正常（launchd 每分钟调度）
6. gateway.err.log 是否有新增 ERROR/WARN 条目

判断逻辑：
→ 全部正常：回复 HEARTBEAT_OK（不发飞书）
→ 有警告或错误：立即发飞书告警（⚠️ + 简要描述 + 建议方案）
```

### P1 - 任务状态巡检（每小时 1 次，合并到主心跳）

```
检查项目：
1. taskflow tasks 中是否有 Blocked 状态任务
2. taskflow tasks 中是否有 stall 倒计时接近阈值的任务
3. 有无派发给 subagent 的任务超过预期时间未回报

判断逻辑：
→ 全部正常：HEARTBEAT_OK（不发飞书）
→ 有 stall 风险：在飞书发提醒 ⚠️ stall倒计时X分钟，等待牛总指令
→ 已 Blocked：发飞书告警 ⚠️，包含任务 ID 和卡住原因
```

### P2 - 记忆维护（每 6 小时检查一次，合并到主心跳）

> ⚠️ memory-hygiene.py 是**每月 1 日**自动执行（launchd cron），不是每 6 小时。
> 每 6 小时这里指的是我**主动检查**，发现问题则手动触发或等月初自动跑。

```
触发条件：当前时间与上次记忆维护时间差 > 6 小时

检查项目：
1. memory/YYYY-MM-DD.md 是否有未收口的事项
2. proactivity/memory.md 是否有 stale 状态
3. workspace/MEMORY.md 是否超过 18KB（逼近 20KB 截断线）
4. 有无决策需要从 session-state 沉淀到 MEMORY.md

动作：
→ 发现未收口：在对应 memory 文件里写收口笔记
→ MEMORY.md ≥18KB：手动触发 memory-hygiene.py --apply
→ 有重大决策：征得牛总确认后更新 MEMORY.md
```

### P3 - 主动汇报（按需，不占心跳）

```
以下情况主动发飞书，不等待心跳：
1. Gateway 重启完成后 → "✅ 我回来了，接着处理刚才中断的事。"
2. 任务完成 → "✅ [任务名] 完成。结论：[一句话]"
3. 发现系统异常 → "❌ [问题简述] ⚠️ 正在处理..."
4. 牛总交办的多步任务，每完成一步 → "第 X/Y 步 ✅，进行中..."
5. 每日首次心跳 → 发一句话日报（可选，内容精简）
```

---

## 心跳执行流程

```
【心跳触发】
    ↓
读取 ~/.openclaw/proactivity/memory.md（主动任务清单）
    ↓
执行 P0 系统健康检查
    ↓
判断：P1/P2 是否到触发时间？
  → 是：执行 P1/P2
  → 否：跳过
    ↓
判断：有实质更新需要通知牛总？
  → 是：发飞书（简短格式）
  → 否：回复 HEARTBEAT_OK
    ↓
更新 ~/.openclaw/proactivity/memory.md（如有进度）
```

---

## 禁止事项

- ❌ 心跳里"自言自语"：没有实质进展不发消息，只回 HEARTBEAT_OK
- ❌ 发无意义心跳：全部正常时牛总不需要收到"系统正常"的重复消息
- ❌ 心跳里做高风险操作：检查就是检查，不要在心跳里修改核心配置
- ❌ 心跳过于频繁：当前 2 小时是合理值，不随意缩短

---

## 当前观察项（动态维护）

> 以下是本次会话观察到的待关注事项，心跳时顺带检查。

| 观察项 | 状态 | 最近检查 |
|--------|------|---------|
| MEMORY.md 自动维护 | ✅ launchd 每月 1 日 03:00 + memory-hygiene.py（覆盖全部 5 个 workspace）| 2026-04-11 |
| shangfang 登录态保存 | ✅ 已完成 | 2026-04-11 |

| 管家-自维护-6h cron | 已恢复正常 | 2026-04-10 |
| Gateway Restart Recovery P0 修复 | ✅ start-gateway.sh 后台启动+等待ready+触发recovery helper（2026-04-11）| 2026-04-11 |
| Gateway Restart Recovery P1 修复 | ✅ 捕获工具名称+参数写入recovery file（2026-04-11）| 2026-04-11 |
| Gateway Restart Recovery P2 修复 | ✅ cron调度失败告警机制（alert-notify.sh新增函数）（2026-04-11）| 2026-04-11 |
| Gateway Restart Recovery P3 修复 | ✅ 7天前旧文件自动清理（2026-04-11）| 2026-04-11 |
| Gateway 通知体验优化 | ✅ 方案A：移除独立send_feishu_text，改为agent直接在对话流里回复「我回来了...」（2026-04-11）| 2026-04-11 |
| Gateway 状态文件写入保护 | ✅ P2：原子写(.tmp+rename) + 文件锁(flock LOCK_EX)（2026-04-11）| 2026-04-11 |
| Gateway 内存监控 | ✅ alert-notify.sh 每10分钟调度，超1536MB告警（1小时冷却）| 2026-04-11 |
| reflector 记忆整理 | ✅ 已修复（workspace 独立 + BOOT.md），Gateway 重启生效 | 2026-04-10 |
| 技能审计清理 | ✅ 移走 42 个未用 skill（~135MB），保留 9 个活跃（1.5MB）| 2026-04-11 |
| 每日日报 cron | ✅ daily-digest.plist 已加载（每天 09:00，Python 脚本发飞书卡片）| 2026-04-11 |
| memory-lancedb-pro embedding 认证 | ✅ 已修复（2026-04-11），端点改为 .com，key 换新 sk-poiqz...，验证正常 | 2026-04-11 |
| moltbook session 每周归档 | ⚠️ 待处理：boot session overflow，需人工确认归档 | 2026-04-11 |
| 长任务后台跑 | ✅ `tools.exec.backgroundMs=12000`（12秒自动后台化，调用时可显式 `background=true/false` 覆盖）| 2026-04-11 |
| taskflow 历史归档 | ✅ 清理 31 个历史任务（2026-04-11）| 2026-04-11 |
| 三层记忆边界 | ✅ MEMORY.md 写入规则确立（2026-04-11）| 2026-04-11 |
| 飞书文档 v1.12 对外版 | ✅ `NUfadHzVYovenKxdwpAcZW7Infe`，去敏感 + 修复细节移到 memory/2026-04-11.md | 2026-04-11 |
| 飞书 lark-table 格式 | ✅ `<lark-td>` 内部必须空行（`\n\n内容\n\n`），简单表格用 Markdown 格式 | 2026-04-11 |
| moltbook Moltbook API key | ⚠️ 新 key（moltcn_9b90...，2026-03-26）未认领，验证码 236382，claim URL `moltcn_claim_be681bee...`；旧 key（moltcn_4e63...，2026-03-15）已认领 | 2026-04-11 |

---

*心跳是智能管家的"自主呼吸"。频率合理、任务清晰、不扰人是基本原则。*
*本文件按实际运营持续迭代。*
