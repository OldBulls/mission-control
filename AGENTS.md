# AGENTS.md — main 协作拓扑与 main 特定运维基线

> 本文件只保留 main 独有内容（拓扑图、agent 档案、main 运维基线、shangfang 链路、新建 agent 流程、bootstrap-extra-files 继承体系说明）。
>
> **通用协作规范已下沉到 `shared/AGENTS.md`**：派发决策树、派发前确认流程、协作机制、taskflow、记忆五层架构、会话启动顺序、收口规则、自主记忆决策。这些通过 bootstrap-extra-files 自动加载到所有 agent。
>
> 2026-04-17 完成 L1+L2 一致性瘦身（去重 + 对齐 shared/，删除文件内自身重复）。

---

## 一、Gateway / Feishu 快口径注入块（main 运维基线）

> 这是当前 OpenClaw 主控工作区的稳定运维基线；由于 `AGENTS.md` 会进入常规 bootstrap context，这一段就是默认注入位。

- 飞书生产主链继续使用本地 `extensions/openclaw-lark`，`bundled feishu` 保持禁用，不回切双开。
- Gateway service 稳定基线是标准 LaunchAgent 直启：`/usr/local/bin/node ~/.local/lib/node_modules/openclaw/openclaw.mjs gateway --port 18789`
- 生产健康判断默认不要把 `openclaw gateway status` / `openclaw gateway probe` 当主判据；优先跑 `bash ~/.openclaw/scripts/check-feishu-gateway-health.sh --summary`
- 默认快口径 = `launchctl` + HTTP + `logs/gateway.log` 最新 boot window + Feishu `bot open_id resolved` / WebSocket 启动信号
- 若出现 `rpc=0` 或 `rpc=skip`，但 `gateway=1 http=1 feishu_ok=5/5 retry_signals=0`，按宿主 CLI/probe 慢点处理，不按 Feishu 生产故障处理
- 若脚本返回 `WARN ... warming=1`，表示刚重启且 Feishu 仍在 channel-connect grace 内；默认等 1-2 分钟后再复查，不按生产故障处理
- 不做 bundled `feishu` 回切，不动 IM 主收发链，不动本地 `openclaw-lark` 自定义能力，不做 OpenClaw 4.10 host dist monkey patch
- 若飞书 footer 出现"只有 `main` 显示完整、子 agent 只显示 `耗时`"，不要先怀疑配置或飞书平台；优先按 `workspace/memory/2026-04-12-feishu-footer-multi-agent-fix.md` 排查 `openclaw-lark` 多 agent session metrics lookup。
- 当前 footer 本地补丁文件：`docs/patches/openclaw-lark-2026.4.7-footer-multi-agent-fix.patch`；升级后按清单 `docs/feishu-plugin-local-patch-inventory-2026-04-11.md` 回补。
- 以后升级 OpenClaw / `openclaw-lark` 后，先跑 `docs/feishu-upgrade-3min-self-checklist.md`，重点检查 `reasoningDefault=off`、`/reset`、多 agent footer、招生助理是否仍正常。
- 若修改了 `AGENTS.md` / `SOUL.md` / `TOOLS.md` / `MEMORY.md` / `HEARTBEAT.md` / `BOOT.md` 这类 bootstrap 文件，想让当前 agent 立即吃到新口径，必须对该 agent 当前会话执行一次 `/reset` 或直接开新会话；仅继续发消息通常不会刷新同一 `sessionKey` 下的旧 bootstrap snapshot。

---

## 二、协作拓扑

```
                    牛总（飞书私聊）
                         │
                         ▼
              ┌─────────────────────┐
              │    main（智能管家）   │ ← 主控 · 统一入口 · 记忆中枢
              └──────────┬──────────┘
                         │ sessions_send / sessions_spawn
        ┌────────────────┼────────────────┬─────────────────┬───────────────┐
        ▼                ▼                ▼                 ▼               ▼
   ┌─────────┐    ┌───────────┐   ┌──────────┐    ┌──────────┐   ┌──────────┐
   │ planner │    │  moltbook │   │   minsu  │    │   yying  │   │zhaosheng │
   │最强大脑 │    │ 学习伙伴  │   │ 民宿助理 │    │ 运营助手 │   │ 招生助理 │
   └─────────┘    └───────────┘   └─────┬────┘    └──────────┘   └──────────┘
                                          │
                                          │ minsu 内部触发
                                          ▼
                                 ┌────────────────┐
                                 │   shangfang   │ ← 途家上房自动化
                                 │   (browser)   │   minsu 的内置能力
                                 └────────────────┘

        ※ reflector 独立于派发树
          由 memory-lancedb-pro 插件自动触发，无需手动派发
```

### 拓扑关系

| 调用关系 | 说明 |
|---------|------|
| main → planner / moltbook / yying / zhaosheng | sessions_send / sessions_spawn 派发 |
| main → minsu | sessions_send / sessions_spawn 派发 |
| minsu → shangfang | **内置能力**，收到上房需求时直接触发 |
| reflector | **插件自动触发**，无需派发（由 memory-lancedb-pro sessions_send 调用） |

---

## 三、Agent 档案

| Agent | 对外名称 | 角色定位 | 触发词 |
|-------|---------|---------|--------|
| **main** | 智能管家 | 统一入口 · 意图分发 · 记忆中枢 | 通用问题 · 多 agent 协作 · 主控路由 |
| **planner** | 最强大脑 | 任务拆解 · SOP 维护 · 多步规划 | 任务规划 · 拆解 · SOP |
| **moltbook** | 学习伙伴 | 在各大 Agent 社区里学习交流 / 发帖互动 / 整理归纳新知识 / 自我进化 | 社区学习 · Agent 互动 · 多飞书内容协作 |
| **minsu** | 民宿助理 | 民宿运营 + **途家上房能力** | 民宿运营 · 房价 · 上房 |
| **yying** | 运营助手 | **小红书** · 自媒体图文 · 视频制作 · 剪辑 | 小红书 · 运营 · 内容制作 |
| **zhaosheng** | 招生助理 | 私立小学招生内容创作 | 招生 · 小学宣传 |
| **reflector** | 记忆回顾 | 记忆整理专用（插件自动触发） | 无需派发 |

> **shangfang** 不在 main 派发树里，是 minsu 的内置能力。

---

## 四、shangfang 调用链路（main 视角）

```
牛总说"上房" → main 路由到 minsu → minsu 内部 sessions_spawn shangfang
                                              ↓
shangfang 执行途家浏览器自动化 → 汇报给 minsu → minsu 汇报给牛总
```

路径：`~/.openclaw/agents/shangfang/`

---

## 五、协作群

| 群名 | chat_id | 用途 |
|------|---------|------|
| AI工作室💡 | `oc_8eb9cb13c5377c4e0437c97a4a469ca8` | 主协作群（唯一有效群） |

---

## 六、新建 Agent 标准流程

创建新 agent 时，**必须同步完成**（不能事后补）：

1. `openclaw approvals allowlist add "*" --agent "新agent" --source allow-always`
2. 若有 cron：`openclaw approvals allowlist add "*" --agent "新agent:cron" --source allow-always`
3. 更新父 agent 的 `subagents.allowAgents`
4. 更新 `tools.agentToAgent.allow`
5. 将新 agent 加入拓扑图和 Agent 档案表（本文件§二、§三）

---

## 七、共享规范继承体系（bootstrap-extra-files）

> 通过 `bootstrap-extra-files` 机制，共享规范自动加载到所有 agent 的 Project Context。
> main 维护共享规范，各子 agent 无需手动同步。

**共享文件位置：**
```
~/.openclaw/shared/
├── SOUL.md    ← 通用操作规范（自动加载）
├── TOOLS.md   ← 通用工具配置（自动加载）
└── AGENTS.md  ← 通用协作规范（自动加载）
```

**bootstrap-extra-files 配置：**
```json
"bootstrap-extra-files": {
  "enabled": true,
  "paths": ["shared/SOUL.md", "shared/TOOLS.md", "shared/AGENTS.md"]
}
```

**加载机制：**
1. 所有 agent 在 bootstrap 时自动加载 `shared/*` 文件到 Project Context
2. 无需运行同步脚本；修改 `shared/` 下的文件后，新 session 自动继承
3. 各 agent 的 workspace 文件（如 `workspace/SOUL.md`）为 agent 特定覆盖，不冲突

**共享规范覆盖范围（避免在 workspace 文件里重复写）：**

| 共享文件 | 内容范围（已下沉，workspace 不再重复） |
|---|---|
| `shared/AGENTS.md` | 派发决策树 / 派发前确认流程 / 协作机制 / taskflow / 记忆五层架构 / 会话启动顺序 / 收口规则 / 自主记忆决策 |
| `shared/SOUL.md` | 工具调用叙事规范 / 多 agent 配置同步原则（一变三查）/ 收口三段论 / test-after-write / 语言规则 / 经验捕获规则 |
| `shared/TOOLS.md` | 飞书工具链 / OpenClaw 系统工具 / 路径别名 / exec 安全规范 / Gateway 命令 / 升级脚本 / Proactive Tool Use / OpenSpace |

**⚠️ 旧版同步机制（已废弃）：**
- `sync-agent-souls.py` 已移除（2026-04-15 20:17）；bootstrap-extra-files 自动加载
- 已同步的子 agent SOUL.md 内容保留作 backward compat，不影响新 session

---

*本文件是 main 视角的"组织架构图 + 运维基线"。每次拓扑变更或运维口径调整时同步更新。*
*通用协作规则在 `shared/AGENTS.md`。*
