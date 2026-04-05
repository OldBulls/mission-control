# Errors

Command failures, exceptions, and unexpected tool behavior.

**Priorities**: low | medium | high | critical
**Areas**: frontend | backend | infra | tests | docs | config
**Statuses**: pending | in_progress | resolved | wont_fix | promoted

## Priority Guidelines

| Priority | When to Use |
|----------|-------------|
| `critical` | Blocks core functionality, data loss risk, security issue |
| `high` | Significant impact, affects common workflows, recurring issue |
| `medium` | Moderate impact, workaround exists |
| `low` | Minor inconvenience, edge case, nice-to-have |

---


## [ERR-20260402-001]

**Logged**: 2026-04-02T14:44:24.449Z
**Priority**: high
**Status**: pending
**Area**: config

### Summary
exec-approvals.json 模式匹配失败：main agent 有 "*" allow-always 但 git/node python 多词命令仍被拒

### Details
根因已确认：exec 安全需要双层配置：①openclaw.json tools.exec.security="allowlist" ②exec-approvals.json main agent 有"*" source:allow-always。两层都满足后，单词命令(echo, git --version, python3 --version, git status)有效，但多词命令(git config --list, git status -s, python3 -c "...")在 main agent 有显式 pattern 情况下仍被拒。可能是 gateway 在读取了多次写入后，socket 状态和文件不一致。

### Suggested Action
用 openclaw approvals CLI 命令（而非直接写文件）来管理 allowlist 规则；或者完全重启 Gateway（kill + start 而非 SIGUSR1 热重载）以确保 socket 和文件状态一致。

### Metadata
- Source: memory-lancedb-pro/self_improvement_log
---


## [ERR-20260402-002]

**Logged**: 2026-04-02T20:29:53.394Z
**Priority**: medium
**Status**: pending
**Area**: config

### Summary
gateway restart打断了正在处理的飞书消息session

### Details
在处理飞书用户消息（open_id: ou_5bcd7105a9d91b7dc63a76c42dfe1880）的过程中，调用了 `openclaw gateway restart`。重启时session绑定被打断，导致后续所有回复被路由到webchat而非飞书，用户在飞书端看不到任何回复。

根因：gateway restart 会话断开时，如果当前有活跃session在处理消息，该session的channel binding会丢失。重启完成后session恢复，但channel binding可能错乱。

预防措施：
1. 永远不在消息处理过程中调用gateway restart
2. 配置变更后先发完回复，再告知用户需要重启，等下一条消息再执行restart
3. 或者使用 `openclaw gateway restart --force` 配合适当的时机

### Suggested Action
处理飞书/其他实时消息时，先完成回复，再考虑restart。或者告知用户需要重启，等下一条消息再执行。

### Metadata
- Source: memory-lancedb-pro/self_improvement_log
---
