# Remote Control Templates

给牛总的 OpenClaw 远程控制模板。

目标：少废话、直接执行、适合飞书私聊触发。

## 已确认的环境前提

- OpenClaw 当前为高权限模式：`agents.defaults.sandbox.mode = off`
- Exec 当前为少确认模式：`tools.exec.ask = off`
- 主飞书账号私聊仅允许牛总本人触发
- 主群仅 `oc_4a6c58a8c0a89f36a8e9e3d5b7f2c1d` 允许免 @ 触发
- `planner` / `moltbook` 不监听群消息

## 推荐触发方式

推荐你以后直接这样说：

- `打开 Safari`
- `重启 openclaw`
- `检查 openclaw 状态`
- `看最近 200 行日志，告诉我有没有报错`
- `把桌面 xxx 文件移动到 Downloads`
- `执行 /Users/zaihuilou/scripts/xxx.sh`
- `帮我检查某服务挂没挂，挂了就重启`

不要故意写得很像 shell。直接说中文目标就行。

## 模板 1：打开 / 关闭应用

### 打开应用

用户示例：

- `打开 Safari`
- `打开飞书和微信`
- `打开访达`

执行思路：

```bash
open -a "Safari"
open -a "WeChat"
open -a "Finder"
```

### 关闭应用

用户示例：

- `退出 Safari`
- `关闭微信`

执行思路：

```bash
osascript -e 'tell application "Safari" to quit'
osascript -e 'tell application "WeChat" to quit'
```

## 模板 2：OpenClaw 运维

### 检查状态

用户示例：

- `检查 openclaw 状态`
- `看看 gateway 正不正常`

执行思路：

```bash
openclaw status --deep
```

### 重启 OpenClaw

用户示例：

- `重启 openclaw`
- `重启网关`

执行思路：

```bash
openclaw gateway restart
```

### 查看日志

用户示例：

- `看最近 200 行 openclaw 日志`
- `检查最近有没有报错`

执行思路：

```bash
tail -n 200 /Users/zaihuilou/.openclaw/logs/openclaw.log
```

## 模板 3：系统控制

### 锁屏

用户示例：

- `锁屏`

执行思路：

```bash
/System/Library/CoreServices/Menu\ Extras/User.menu/Contents/Resources/CGSession -suspend
```

### 发通知

用户示例：

- `弹个通知，内容是任务完成`

执行思路：

```bash
osascript -e 'display notification "任务完成" with title "OpenClaw"'
```

### 音量控制

用户示例：

- `把音量调到 30%`
- `静音`

执行思路：

```bash
osascript -e 'set volume output volume 30'
osascript -e 'set volume with output muted'
```

## 模板 4：文件操作

### 移动文件

用户示例：

- `把桌面的 test.txt 移到 Downloads`

执行思路：

```bash
mv ~/Desktop/test.txt ~/Downloads/
```

### 打包文件夹

用户示例：

- `把 ~/Desktop/demo 打包成 zip`

执行思路：

```bash
cd ~/Desktop && zip -r demo.zip demo
```

### 查最近修改文件

用户示例：

- `查 Downloads 最近 20 个文件`

执行思路：

```bash
find ~/Downloads -type f -print0 | xargs -0 ls -lt | head -20
```

## 模板 5：脚本 / 工作流执行

### 直接执行本地脚本

用户示例：

- `执行 /Users/zaihuilou/scripts/backup.sh`
- `运行 xxx 脚本`

执行思路：

```bash
bash /Users/zaihuilou/scripts/backup.sh
```

### 带日志执行

用户示例：

- `执行 xxx 脚本，并把输出发给我`

执行思路：

```bash
bash /path/to/script.sh 2>&1 | tail -n 200
```

## 模板 6：开发/运维排障

### Git 仓库检查

用户示例：

- `看看这个项目有没有未提交改动`
- `帮我看 git 状态`

执行思路：

```bash
git -C /path/to/repo status --short
```

### 测试/构建

用户示例：

- `跑这个项目测试`
- `帮我 build 一下并看报错`

执行思路：

```bash
cd /path/to/repo && npm test
cd /path/to/repo && npm run build
```

### 端口/进程检查

用户示例：

- `看 3000 端口是不是占用了`
- `查某个进程在不在`

执行思路：

```bash
lsof -i :3000
ps aux | grep keyword | grep -v grep
```

## 模板 7：建议做成固定自动化的高频动作

这些最值得以后继续做成 cron 或专用脚本：

1. `重启 openclaw 并回报状态`
2. `检查日志中的 ERROR / failed / timeout`
3. `检查某服务是否在线，不在线则自动重启`
4. `执行固定备份脚本`
5. `打开一组常用应用`
6. `汇总某目录的新增文件`

## 执行原则

- 默认优先执行明确、可逆、低歧义动作
- 会修改文件、删除文件、覆盖内容时，若用户表达不清，要先补一句确认
- 对外发送消息、发帖、邮件，继续默认先确认
- 本机打开应用、查状态、查日志、运行已知脚本，可以直接执行

## 后续可扩展

后面可以再加两类：

- `SHORTCUTS.md`：固定口令 -> 固定脚本
- `REMOTE_JOBS.md`：定时巡检/自动恢复任务清单
