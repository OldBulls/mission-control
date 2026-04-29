## 2026-04-27 11:46 — Background Review 哨兵与历史读取

**内容**：后台 Review 子流程先读 `.running` 导致 ENOENT；之后用 `sessions_history` 检查当前会话时返回大体积消息，出现 transcript repair synthetic error。
**教训**：后台 Review 开始时先原子创建 `.running` 哨兵；巡检优先用轻量来源，避免在 review 自身里调用可能回填超长会话体的工具。
**状态**：⏳ candidate

## 2026-04-27 11:46 — Background Review 哨兵与历史读取

**内容**：后台 Review 子流程先读 `.running` 导致 ENOENT；在 review 自身里调用 `sessions_history` 时返回大体积消息，出现 transcript repair synthetic error。
**教训**：后台 Review 开始时先原子创建 `.running` 哨兵；巡检优先用轻量来源，避免使用会回填超长会话体的历史工具。
**状态**：⏳ candidate

## 2026-04-27 12:31 — Background Review transcript repair 连续失败

**内容**：后台 Review 重试期间，多个轻量工具连续返回 `missing tool result in session history; inserted synthetic error result for transcript repair`，说明问题不在单个工具，而在当前会话历史修复链。
**教训**：一旦后台 Review 触发链出现连续 transcript repair / synthetic error 信号，应立刻停止继续读取历史，直接写入候选记录并原子清理 flag 与 `.running`，避免放大故障。
**状态**：⏳ candidate

## 2026-04-27 12:31 — Background Review 重试环快速退出

**内容**：`pending-background-review` 在同一分钟反复触发；此前已确认连续 transcript repair / synthetic error。
**教训**：后台 Review 自身进入重试环时，应跳过历史扫描，直接执行原子清理；不要为了满足扫描步骤继续调用可能污染会话的工具。
**状态**：⏳ candidate

## 2026-04-27 13:38 — Background Review 连续失败信号

**内容**：后台 review 会话在读取历史/工具结果时连续出现中止与 transcript repair。
**教训**：当 review 会话连续出现多次超时或缺失 tool result，应立即记录并停止追加扫描，优先原子清理哨兵，避免形成重复触发链。
**状态**：⏳ candidate


## 2026-04-27 13:51 — Background Review transcript repair 重试链

**内容**：后台 Review 在重试过程中连续遇到历史扫描/读取工具返回 synthetic transcript repair 错误，说明当前会话修复链不稳定。
**教训**：遇到该类连续信号时，不再继续调用历史/读取类工具；直接记录候选经验并原子清理 pending flag、`.running` 与 counter。
**状态**：⏳ candidate

## 2026-04-27 14:05 — 后台 Review 连续失败信号

**内容**：pending-background-review 重试期间，sessions/lcm 检索多次返回 missing tool result synthetic repair，未能完成正常会话扫描。
**教训**：遇到连续 missing tool result / No result provided 时，不继续堆叠检索工具；改用单条 shell 聚合读取状态并快速收口，随后清理 pending/running/counter，避免形成重试风暴。
**状态**：⏳ candidate
