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
| Gateway / Feishu 快口径 | ✅ 默认健康检查改为 `check-feishu-gateway-health.sh --summary`；`rpc=0/skip` 但快口径全绿时不按生产故障处理 | 2026-04-12 |
| memory-consolidation 系统 | ✅ 已上线（2026-04-15）：每周日 3:00 cron，P0强化评分，81→31文件（减少48个） | 2026-04-15 |
| Background Review Agent | ✅ 已实现（counter + pending flag + 心跳触发 + sessions_spawn） | 2026-04-13 |
| Ontology 可视化 | ✅ 彻底重建（65实体/89关系/0孤立，24节点/27连线 D3内嵌），generate-viz.py 三合一自动发布 docs/ | 2026-04-13 |
| Image/VL 模型可用性 | ✅ MiniMax-VL-01 第二次重试成功，截图分析已恢复 | 2026-04-13 |
| moltbook Moltbook API key | ✅ 已解决（2026-04-13 确认） | 2026-04-13 |
| LanceDB 碎片监控 | ⚠️ 963 fragments（>500 warn / >1000 crit）；versions 991；size 60MB；`run-lance-monitor.py` 已部署，每周 cron + 告警写入 `state/lance-alert.json`；Node.js API 无 `compact()`，fragment 合并需 Python API | 2026-04-15 |

---

*心跳是智能管家的"自主呼吸"。频率合理、任务清晰、不扰人是基本原则。*
*本文件按实际运营持续迭代。*
