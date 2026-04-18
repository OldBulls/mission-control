# TOOLS.md - main 特定工具地图

> 本文件只保留 main 独有的工具配置（飞书 Bot 账号、协作群 chat_id、子 agent 工作区、Graphify 脚本、shangfang 浏览器自动化、内容生成、Desktop 文档路径）。
>
> **通用工具配置已下沉到 `shared/TOOLS.md`**：飞书工具链 / OpenClaw 系统工具（派发/记忆/taskflow/LCM）/ 路径别名 / exec 安全规范 / Gateway 管理命令 / 升级脚本 / Proactive Tool Use / OpenSpace MCP。这些通过 bootstrap-extra-files 自动加载到所有 agent。
>
> 2026-04-17 完成 L1+L2 一致性瘦身（删除已被 shared 覆盖的通用段，对齐 Gateway 健康检查口径）。

---

## 一、飞书 Bot 账号体系（main 维护）

> 通用飞书工具链在 `shared/TOOLS.md`。本表只记录各 bot 的账号信息，main 在路由时需要这些 open_id。

| Agent | Bot 名称 | open_id | 备注 |
|-------|---------|---------|------|
| main | 智能管家 | `ou_5bcd7105a9d91b7dc63a76c42dfe1880` | 主账号 |
| minsu | 民宿助理 | `ou_4259d3fdbbbf5b9544bea91b37280186` | minsu 视角 |
| zhaosheng | 招生助理 | `ou_a2abcf3a955fc73640ed49f20a308b00` | zhaosheng 视角 |

---

## 二、飞书协作群

| 群名 | chat_id | 用途 |
|------|---------|------|
| AI工作室💡 | `oc_8eb9cb13c5377c4e0437c97a4a469ca8` | 协作群 |

---

## 三、子 Agent 工作区

| Agent | 路径 |
|-------|------|
| planner | `~/.openclaw/agents/planner/` |
| moltbook | `~/.openclaw/agents/moltbook/` |
| minsu | `~/.openclaw/agents/minsu/` |
| shangfang | `~/.openclaw/agents/shangfang/` |
| zhaosheng | `~/.openclaw/agents/zhaosheng/` |
| yying | `~/.openclaw/agents/yying/` |

---

## 四、Graphify 工具（main 维护图谱构建）

| 工具 | 路径 |
|------|------|
| 建图脚本 | `~/.openclaw/scripts/graphify/rebuild-graph-if-needed.sh` |
| corpus 生成 | `~/.openclaw/scripts/graphify/prepare-openclaw-memory-corpus.sh` |
| 图谱构建器 | `~/.openclaw/scripts/graphify/build-local-graph.py` |
| 三合一 HTML | `~/.openclaw/data/graphify/openclaw-memory/graphify-out/graph-triple-in-one.html` |

---

## 五、Desktop 文档路径

| 文档集 | 路径 |
|--------|------|
| 龙虾教练文档 | `~/Desktop/文档/` |

---

## 六、浏览器自动化（shangfang 专用，main 路由参考）

> shangfang 由 minsu 内部触发；main 一般不直接调用浏览器，但需要知道环境约束。

### Chrome 启动命令（shangfang 自身使用）

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

## 七、内容生成工具（main 调用）

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

*本文件是 main 特定的"工具箱说明书"。账号、群、Graphify 脚本、浏览器、内容生成相关。*
*通用工具配置在 `shared/TOOLS.md`。系统升级或环境变更时，同步更新本文件。*
