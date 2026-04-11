# TOOLS.md - 智能管家工具地图

> 本文件定义智能管家的"手眼器官"——本地环境别名、硬技能、工具参数。
> 基于龙虾教练体系文档15精读，2026-04-10 重构。

---

## 飞书工具（核心业务工具）

### 飞书 Bot 账号体系

| Agent | Bot 名称 | open_id | 备注 |
|-------|---------|---------|------|
| main | 智能管家 | `ou_5bcd7105a9d91b7dc63a76c42dfe1880` | 主账号 |
| minsu | 民宿助理 | `ou_4259d3fdbbbf5b9544bea91b37280186` | minsu 视角 |
| zhaosheng | 招生助理 | `ou_a2abcf3a955fc73640ed49f20a308b00` | zhaosheng 视角 |

### 飞书协作群

| 群名 | chat_id | 用途 |
|------|---------|------|
| AI工作室💡 | `oc_8eb9cb13c5377c4e0437c97a4a469ca8` | 协作群 |

### 飞书交互工具链

| 工具 | 用途 | 使用场景 |
|------|------|---------|
| `feishu_ask_user_question` | 交互问答卡片 | "是否继续/二选一/补字段"场景 |
| `feishu_im_user_message send` | 发飞书消息 | 主动通知牛总 |
| `feishu_calendar_event` | 日程管理 | 创建会议、查日程 |
| `feishu_task_task` | 飞书任务 | 创建/管理任务 |
| `feishu_bitable_app_table_record` | 多维表格 | 数据录入/查询 |
| `feishu_search_doc_wiki` | 文档/知识库搜索 | 查飞书文档 |
| `feishu_drive_file` | 云盘文件管理 | 上传/下载/移动 |
| `feishu_fetch_doc` | 获取文档内容 | 读取飞书云文档 |

---

## OpenClaw 系统工具

### Agent 派发

| 工具 | 用途 |
|------|------|
| `sessions_send` | 发送消息到子 agent session |
| `sessions_spawn` | 派发新任务到 subagent（isolated session） |
| `subagents list/steer/kill` | 管理已派发的 subagent |

### 记忆工具

| 工具 | 用途 |
|------|------|
| `memory_recall` | 语义搜索长期记忆 |
| `memory_store` | 写入重要信息到长期记忆 |
| `memory_forget` | 删除指定记忆 |
| `memory_update` | 更新已有记忆 |
| `lcm_grep` / `lcm_expand` / `lcm_expand_query` | 搜索/展开压缩后的会话历史 |

### taskflow 工具

| 工具 | 路径/命令 |
|------|----------|
| taskflow CLI | `/usr/local/bin/taskflow` |
| 任务存储 | `~/.openclaw/taskflow/tasks.json` |
| file_lock.py | `~/.openclaw/scripts/taskflow/file_lock.py` |
| scheduler | `~/.openclaw/scripts/taskflow/scheduler.sh` |

### OpenSpace 工具（MCP）

| 工具 | 配置 |
|------|------|
| MCP Server | `/private/tmp/OpenSpace/.env` |
| mcporter 配置 | `~/.openclaw/workspace/config/mcporter.json` |
| 工作区 | `/Users/zaihuilou/.openclaw/workspace` |
| 调用要求 | 必须加 `--timeout 120000` |

### Graphify 工具

| 工具 | 路径 |
|------|------|
| 建图脚本 | `~/.openclaw/scripts/graphify/rebuild-graph-if-needed.sh` |
| corpus 生成 | `~/.openclaw/scripts/graphify/prepare-openclaw-memory-corpus.sh` |
| 图谱构建器 | `~/.openclaw/scripts/graphify/build-local-graph.py` |
| 三合一 HTML | `~/.openclaw/data/graphify/openclaw-memory/graphify-out/graph-triple-in-one.html` |

---

## 本地路径别名

### OpenClaw 核心路径

| 别名 | 实际路径 |
|------|---------|
| `~/.openclaw/` | `/Users/zaihuilou/.openclaw/` |
| `~/.openclaw/workspace/` | `/Users/zaihuilou/.openclaw/workspace/` |
| `~/.openclaw/agents/` | `/Users/zaihuilou/.openclaw/agents/` |
| `~/.openclaw/scripts/` | `/Users/zaihuilou/.openclaw/scripts/` |
| `~/.openclaw/taskflow/` | `/Users/zaihuilou/.openclaw/taskflow/` |
| `~/.openclaw/credentials/` | `/Users/zaihuilou/.openclaw/credentials/` |
| `~/.openclaw/data/` | `/Users/zaihuilou/.openclaw/data/` |
| `~/.openclaw/logs/` | `/Users/zaihuilou/.openclaw/logs/` |

### 子 Agent 工作区

| Agent | 路径 |
|-------|------|
| planner | `~/.openclaw/agents/planner/` |
| moltbook | `~/.openclaw/agents/moltbook/` |
| minsu | `~/.openclaw/agents/minsu/` |
| shangfang | `~/.openclaw/agents/shangfang/` |
| zhaosheng | `~/.openclaw/agents/zhaosheng/` |
| yying | `~/.openclaw/agents/yying/` |

### Desktop 文档路径

| 文档集 | 路径 |
|--------|------|
| 龙虾教练文档 | `~/Desktop/文档/` |

---

## Shell / exec 工具

### exec 安全规范

> **重要：区分直接命令 vs 混淆写法**

| 类型 | 示例 | 状态 |
|------|------|------|
| ✅ 直接命令 | `ls ~/Desktop` / `cat /tmp/file.py` / `python3 /tmp/script.py` | 正常执行 |
| ❌ 混淆写法（触发 obfuscated code 检测） | `python3 -c "..."` / `node -e "..."` / `bash -c '...'` | 禁止 |
| ✅ 正确 workaround | 先写临时文件 `echo 'code' > /tmp/s.py` 再执行 `python3 /tmp/s.py` | 必须这样用 |

### Gateway 管理命令

| 操作 | 命令 |
|------|------|
| 启动 | `openclaw gateway start` |
| 停止 | `openclaw gateway stop` |
| 重启 | `openclaw gateway restart` |
| 状态 | `openclaw gateway status` |
| 健康检查 | `openclaw health` |
| 配置校验 | `openclaw config validate` |
| 自愈入口 | `~/.openclaw/scripts/oc-go.sh` |

### OpenClaw 升级管理

| 脚本 | 路径 | 用途 |
|------|------|------|
| 备份 | `~/.openclaw/scripts/backup-memory-plugin-upgrade.sh` | 升级前打包 |
| 回归检查 | `~/.openclaw/scripts/check-memory-plugin-upgrade.sh` | 升级后验证 |
| 恢复 | `~/.openclaw/scripts/restore-memory-plugin-upgrade.sh --apply` | 翻车时恢复 |

---

## 浏览器自动化（shangfang 专用）

### Chrome 启动命令（shangfang agent 使用）

```bash
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
  --remote-debugging-port=9222 \
  --user-data-dir=/tmp/chrome-debug
```

⚠️ **注意：勿用 `open -a Chrome --args ...`，参数传递会失效**

### agent-browser 配置

| 配置项 | 值 |
|--------|---|
| CDP 端口 | 9222 |
| Chrome 用户数据目录 | /tmp/chrome-debug |
| auth 状态文件 | `~/.openclaw/agents/shangfang/data/auth_state.json` |
| 截图目录 | `~/.openclaw/agents/shangfang/data/screenshots/` |
| 属性数据 | `~/.openclaw/agents/shangfang/data/property.yaml` |

---

## 内容生成工具

### 图片生成

| 工具 | 配置 |
|------|------|
| image_generate | MiniMax 图片模型 |

### 视频生成

| 工具 | 配置 |
|------|------|
| video_generate | 可用（未配置指定模型） |

### 语音生成（TTS）

| 工具 | 配置 |
|------|------|
| tts | 未配置默认声音（可指定 channel:telegram 等） |

---

## Proactive Tool Use（主动使用原则）

基于文档15的原则，智能管家的工具使用准则：

| 原则 | 说明 |
|------|------|
| 内部操作优先 | 读写文件、查状态、准备草案，优先于上报 |
| 多种方式尝试 | 一个工具不行，尝试另一个工具，不轻易放弃 |
| 测试假设 | 用工具验证假设，不凭猜测行动 |
| 外部操作前停 | send / delete / reschedule / contact 类操作，先问牛总 |
| session-state 更新 | 工具结果改变了工作状态，立即更新 `~/proactivity/session-state.md` |

---

*本文件是智能管家的"工具箱说明书"。环境相关的特殊配置都在这里。*
*系统升级或环境变更时，同步更新本文件。*
