# SHORTCUTS.md

给牛总的远控快捷口令映射。

核心原则：尽量复用 `~/.openclaw/scripts/remote-actions.sh` 和既有脚本，不重复发明轮子。

## 标准调用方式

由于当前环境里脚本不适合裸执行，统一使用下面这种方式：

```bash
bash ~/.openclaw/scripts/remote-actions.sh <action> [args]
```

以后我内部默认也按这个口径执行。

## 推荐口令

### OpenClaw 运维

- `检查状态`
  - 对应：`bash ~/.openclaw/scripts/remote-actions.sh status`

- `重启 openclaw`
  - 对应：`bash ~/.openclaw/scripts/remote-actions.sh restart-openclaw`

- `检查健康`
  - 对应：`bash ~/.openclaw/scripts/remote-actions.sh check-health`

- `分析故障`
  - 对应：`bash ~/.openclaw/scripts/remote-actions.sh analyze-failure`

- `执行维护`
  - 对应：`bash ~/.openclaw/scripts/remote-actions.sh maintain`

- `轮转日志`
  - 对应：`bash ~/.openclaw/scripts/remote-actions.sh rotate-logs`

- `检查未回复消息`
  - 对应：`bash ~/.openclaw/scripts/remote-actions.sh check-unanswered`

- `看性能报告`
  - 对应：`bash ~/.openclaw/scripts/remote-actions.sh perf-report`

- `运行监控`
  - 对应：`bash ~/.openclaw/scripts/remote-actions.sh monitor`

### 文件 / 日志

- `看最近 200 行日志`
  - 对应：`bash ~/.openclaw/scripts/remote-actions.sh logs 200`

- `备份数据`
  - 对应：`bash ~/.openclaw/scripts/remote-actions.sh backup-data`

### Mac 应用控制

- `打开 Safari`
  - 对应：`bash ~/.openclaw/scripts/remote-actions.sh open-app Safari`

- `退出 Safari`
  - 对应：`bash ~/.openclaw/scripts/remote-actions.sh quit-app Safari`

- `打开飞书`
  - 对应：`bash ~/.openclaw/scripts/remote-actions.sh open-app Feishu`

- `打开微信`
  - 对应：`bash ~/.openclaw/scripts/remote-actions.sh open-app WeChat`

### 通知

- `弹通知：任务完成`
  - 对应：`bash ~/.openclaw/scripts/remote-actions.sh notify 任务完成`

### 自定义脚本

- `执行脚本 /绝对路径/xxx.sh`
  - 对应：`bash ~/.openclaw/scripts/remote-actions.sh run-script /绝对路径/xxx.sh`

## 已抽测动作

- `status`
- `check-health`
- `logs 20`

如果后续继续抽测更多动作，再往这里补。

## 备注

- 这些口令是“建议映射”，不是必须逐字匹配
- 以后如果有高频动作，优先往 `remote-actions.sh` 里加，而不是散落到多个新脚本
- 对外发消息、删除重要文件、覆盖内容，仍建议保留确认
