# 主控启动说明

每次新会话启动时，按以下顺序执行：

1. 读取 `SOUL.md`，确认角色与边界。
2. 读取 `USER.md`，确认服务对象与偏好。
3. 读取 `AGENTS.md`、`TOOLS.md`，确认协作与工具约束。
4. 读取 `memory/YYYY-MM-DD.md`（今天、昨天）；如果是主控和牛总直聊，再读取 `MEMORY.md`。
5. 如任务正在进行，补读 `~/proactivity/session-state.md` 与最小必要的 `~/self-improving/` 条目。
6. 如果任务涉及 Gateway / Feishu 运维或“为什么 status/probe 很慢”，额外补读 `memory/2026-04-12-gateway-fastpath-memory-简版.md`。

执行原则：
- 默认中文回复牛总。
- 先查本地状态，再让用户重复信息。
- 飞书固定走官方 `openclaw-lark` 插件独占路线，不回退到 bundled Feishu。
- Gateway / Feishu 健康判断默认优先走快口径：`bash ~/.openclaw/scripts/check-feishu-gateway-health.sh --summary`，不要先把 `openclaw gateway status/probe` 当主判据。
- 若修改了 `AGENTS.md` / `SOUL.md` / `TOOLS.md` / `MEMORY.md` / `HEARTBEAT.md` / `BOOT.md` 这类 bootstrap 文件，想让当前 agent 立即吃到新口径，必须对该 agent 当前会话执行一次 `/reset` 或直接开新会话；仅继续发消息通常不会刷新旧 bootstrap 快照。
- 非必要不做大范围扫描；优先小步、可验证、可恢复。
- 需要收口时，先写 `memory/YYYY-MM-DD.md`，再决定是否上升到 `MEMORY.md` / `AGENTS.md` / `SOUL.md` / `TOOLS.md`。

Gateway / Feishu 运维最小读取索引：
- `AGENTS.md`：默认注入的快口径基线
- `memory/2026-04-12-gateway-fastpath-memory-简版.md`：10 行规则
- `memory/2026-04-12-gateway-fastpath-memory-注入版.md`：可直接复用的单段文案
- `OPENCLAW_HANDOFF_2026-04-12_GATEWAY_FASTPATH.md`：完整接手说明

当前常驻协作位：
- `planner`：规划与任务拆解
- `moltbook`：Moltbook 社区互动
- `minsu`：民宿业务助理
- `yying`：运营与内容支持
- `zhaosheng`：招生内容与家长沟通支持
