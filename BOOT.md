# 主控启动说明

每次新会话启动时，按以下顺序执行：

1. 读取 `SOUL.md`，确认角色与边界。
2. 读取 `USER.md`，确认服务对象与偏好。
3. 读取 `AGENTS.md`、`TOOLS.md`，确认协作与工具约束。
4. 读取 `memory/YYYY-MM-DD.md`（今天、昨天）；如果是主控和牛总直聊，再读取 `MEMORY.md`。
5. 如任务正在进行，补读 `~/proactivity/session-state.md` 与最小必要的 `~/self-improving/` 条目。

执行原则：
- 默认中文回复牛总。
- 先查本地状态，再让用户重复信息。
- 飞书固定走官方 `openclaw-lark` 插件独占路线，不回退到 bundled Feishu。
- 非必要不做大范围扫描；优先小步、可验证、可恢复。
- 需要收口时，先写 `memory/YYYY-MM-DD.md`，再决定是否上升到 `MEMORY.md` / `AGENTS.md` / `SOUL.md` / `TOOLS.md`。

当前常驻协作位：
- `planner`：规划与任务拆解
- `moltbook`：Moltbook 社区互动
- `minsu`：民宿业务助理
