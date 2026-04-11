# Self-Improving / Proactivity / 临时工程笔记分层

## 最终分工

### 1. `~/proactivity/`

主控运行态。

负责：
- 主动推进
- 上下文恢复
- 当前任务状态
- heartbeat 跟进
- working buffer

适合写入：
- 当前任务下一步
- 卡点
- 最近决策
- 需要后续跟进的事项

### 2. `~/self-improving/`

长期执行经验层。

负责：
- 用户偏好
- 纠错沉淀
- 可复用工作流经验
- 跨任务稳定规则

适合写入：
- “以后都这样做”
- “这个坑以后别再踩”
- “这个领域应该优先用哪套方法”

## 当前推荐

- `proactivity` 继续作为主控行为层
- `self-improving` 作为长期经验层
- 旧的 `.learnings` 方案已停用；工程临时笔记先落 `memory/YYYY-MM-DD.md`

## 使用规则

- 需要“继续推进、恢复上下文、留下下一步” → 写 `~/proactivity/`
- 需要“形成长期经验、偏好、稳定规则” → 写 `~/self-improving/`
- 只是“先记下当天临时工程问题，后面再整理” → 写 `memory/YYYY-MM-DD.md`
