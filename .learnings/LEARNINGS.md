# LEARNINGS.md — 经验教训

## [LRN-20260323-001] best_practice

**Logged**: 2026-03-23T18:00:00+08:00
**Priority**: high
**Status**: pending
**Area**: config

### Summary
在 Windows 上部署 Python 项目时，必须全面检查文件编码兼容性

### Details
edict 项目在 Linux 上开发，迁移到 Windows 时遇到一连串编码问题：
1. `open()` / `write_text()` 未指定 `encoding='utf-8'`
2. `json.dump()` 通过 `os.fdopen()` 写入时未指定编码
3. `read_text()` 同样未指定编码
4. PowerShell 终端默认 GBK 导致显示乱码（与实际文件编码无关）

这些问题在 Linux 上不会出现（默认 UTF-8），但在 Windows 中文版上全部暴露。

### Suggested Action
安装任何 Python 项目到 Windows 时，第一步就检查并修复所有文件操作的编码参数。

### Metadata
- Source: error
- Related Files: scripts/file_lock.py, scripts/utils.py, dashboard/server.py
- Tags: windows, encoding, python, deployment

---

## [LRN-20260323-002] best_practice

**Logged**: 2026-03-23T18:00:00+08:00
**Priority**: high
**Status**: pending
**Area**: infra

### Summary
在 Windows 上部署 Unix 项目时，需要系统性替换 Linux 专用调用

### Details
edict 项目多处使用了 Unix 特有的 API：
1. `fcntl` 模块（文件锁）→ 需要 `msvcrt` 替代
2. `pgrep` 命令（进程检测）→ 需要 `tasklist` 替代
3. `&` 后台运行 → PowerShell 不支持，需用 `Start-Process` 或 OpenClaw 的 background exec

### Suggested Action
在 Windows 上部署任何项目前，全局搜索以下关键词并替换：
- `import fcntl`
- `pgrep`
- `subprocess.run([...])` 中的 Linux 命令

### Metadata
- Source: error
- Tags: windows, unix-compat, deployment

---

## [LRN-20260323-003] best_practice

**Logged**: 2026-03-23T18:46:00+08:00
**Priority**: medium
**Status**: pending
**Area**: config

### Summary
edict 看板的数据同步是两步流程：sync → refresh

### Details
edict 的数据流：
1. `sync_from_openclaw_runtime.py` → 写入 `tasks_source.json`
2. `refresh_live_data.py` → 读取 `tasks_source.json`，生成 `live_status.json`
3. 服务器读取 `live_status.json` 返回给前端

两个脚本都需要运行，只跑 sync 不跑 refresh 看板不会有数据。
另外，edict 本身没有自动同步机制，需要外部 cron 定时触发。

### Suggested Action
部署 edict 时同步设置 cron 定时任务，每 60 秒运行一次完整的同步流程。

### Metadata
- Source: error
- Related Files: scripts/sync_from_openclaw_runtime.py, scripts/refresh_live_data.py
- Tags: edict, data-sync, cron

---

## [LRN-20260323-004] correction

**Logged**: 2026-03-23T16:24:00+08:00
**Priority**: medium
**Status**: pending
**Area**: backend

### Summary
看板服务器默认不会自动启动，需要手动运行

### Details
用户安装 edict 后访问 http://127.0.0.1:7891 发现打不开。原因：
- OpenClaw Gateway 运行在 18789 端口
- edict 看板服务器运行在 7891 端口，需要单独启动
- 用户可能混淆了两个端口

### Suggested Action
安装完成后明确告知用户需要手动启动看板服务器，或创建开机启动脚本。

### Metadata
- Source: user_feedback
- Tags: edict, server, port-confusion

---

## [LRN-20260323-005] knowledge_gap

**Logged**: 2026-03-23T15:25:00+08:00
**Priority**: low
**Status**: pending
**Area**: config

### Summary
OpenRouter 的并发限制取决于账户余额，非固定值

### Details
用户询问 openrouter/xiaomi/mimo-v2-pro 的并发数。OpenRouter 不公布每个模型的具体并发限制，而是根据账户余额动态调整。余额越高，并发限制越宽松。具体数值需要查看 OpenRouter 控制台或 API 响应头。

### Suggested Action
遇到类似问题时直接引导用户查看 OpenRouter 控制台的 Settings → Limits 页面。

### Metadata
- Source: user_feedback
- Tags: openrouter, rate-limit

---
