# TOOLS.md - Local Notes

Skills define _how_ tools work. This file is for _your_ specifics — the stuff that's unique to your setup.

## 环境信息

- **OS**: Windows 10 (19045)，中文版，默认编码 GBK
- **Shell**: PowerShell（执行策略受限，需用 `-ExecutionPolicy Bypass` 或 `cmd /c`）
- **Node.js**: v22.16.0
- **Python**: 3.11.9

## Python 编码规则（Windows 必须）

所有 Python 文件操作必须显式指定 UTF-8：
```python
# ✅ 正确
path.read_text(encoding='utf-8')
path.write_text(content, encoding='utf-8')
with open(f, 'w', encoding='utf-8') as fh:
    json.dump(data, fh, ensure_ascii=False)

# ❌ 错误（Windows 上会用 GBK）
path.read_text()
path.write_text(content)
open(f, 'w')
```

## 跨平台兼容

Windows 上缺少的 Unix 工具及替代方案：
| Unix | Windows 替代 |
|------|-------------|
| `fcntl` | `msvcrt` |
| `pgrep` | `tasklist /FI "IMAGENAME eq ..."` |
| `&` (后台) | `Start-Process` 或 OpenClaw `background: true` |

## GitHub 国内镜像

直接 `git clone` GitHub 超时时用镜像站：
- `https://ghfast.top/https://github.com/...`
- `https://ghproxy.com/https://github.com/...`
- 或手动下载 ZIP

## 端口分配

- **18789** — OpenClaw Gateway
- **7891** — edict 三省六部看板

## 三省六部 (edict)

- 安装路径：`~/.openclaw/workspace/skills/edict-main`
- 数据目录：`~/.openclaw/workspace/skills/edict-main/data`
- 看板启动：`python dashboard/server.py`（需手动启动）
- 数据同步：需先跑 sync 再跑 refresh，已设置 cron 每 60 秒自动同步

## Windows Python 规则（2026-03-24 踩坑总结）

### subprocess 调用 openclaw
```python
# ✅ 正确 — Windows 下 openclaw 是 .cmd 文件，需要 shell 解析
import platform
IS_WINDOWS = platform.system() == "Windows"
subprocess.run(["openclaw", "agent", ...], shell=IS_WINDOWS)

# ❌ 错误 — [WinError 2] 系统找不到指定的文件
subprocess.run(["openclaw", "agent", ...])
```

### 文件写入立即可见
```python
# ✅ 正确 — flush + fsync 确保写入磁盘
with open(str(f), 'w', encoding='utf-8', newline='\n') as fh:
    fh.write(content)
    fh.flush()
    os.fsync(fh.fileno())

# ❌ 错误 — pathlib.write_text() 可能不立即刷盘
f.write_text(content, encoding='utf-8')
```

### asyncio 信号处理
```python
# ✅ 正确 — Windows 不支持 SIGTERM
if platform.system() != "Windows":
    for sig in (signal.SIGTERM, signal.SIGINT):
        loop.add_signal_handler(sig, handler)

# ❌ 错误 — NotImplementedError: add_signal_handler() not implemented on Windows
loop.add_signal_handler(signal.SIGTERM, handler)
```

### openclaw agent 多行消息
```python
# ✅ 正确 — 单行消息，服务端直接执行逻辑
msg = f'📜 请为以下旨意起草方案: {title[:50]} | 任务ID: {task_id}'

# ❌ 错误 — 多行消息在 Windows shell 传递中被截断
msg = f'步骤1...\n步骤2...\n步骤3...'
```

### 共享脚本路径
```python
# ✅ 正确 — 被复制到多处的脚本用固定路径
_EDICT_DATA = pathlib.Path.home() / '.openclaw' / 'workspace' / 'skills' / 'edict-main' / 'data'
TASKS_FILE = _EDICT_DATA / 'tasks_source.json'

# ❌ 错误 — __file__ 相对路径在每个副本中解析不同
_BASE = pathlib.Path(__file__).resolve().parent.parent
TASKS_FILE = _BASE / 'data' / 'tasks_source.json'
```

### 编辑源文件而非副本
> 如果 `sync_agent_config.py` 每 5 秒把源脚本复制到各 agent workspace，
> **永远编辑 `edict-main/scripts/` 下的源文件**，让 sync 自动传播。
> 编辑 workspace 副本会被覆盖。

---

Add whatever helps you do your job. This is your cheat sheet.
