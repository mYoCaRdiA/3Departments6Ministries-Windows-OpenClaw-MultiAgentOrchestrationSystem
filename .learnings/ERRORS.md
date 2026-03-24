# ERRORS.md — 错误日志

## [ERR-20260323-001] PowerShell 执行策略阻止 npm/脚本运行

**Logged**: 2026-03-23T15:54:00+08:00
**Priority**: high
**Status**: resolved
**Area**: infra

### Summary
PowerShell 默认执行策略阻止了 `openclaw`、`npm` 等命令的运行

### Error
```
无法加载文件 C:\Program Files\nodejs\npm.ps1，因为在此系统上禁止运行脚本
```

### Context
- 命令：`openclaw models set ...`、`npm install`
- 环境：Windows 10，PowerShell 默认执行策略 Restricted

### Suggested Fix
使用 `powershell -ExecutionPolicy Bypass -Command "..."` 或改用 `cmd /c` 执行

### Metadata
- Reproducible: yes
- Related Files: 所有需要调用外部命令的场景
- See Also: ERR-20260323-002

---

## [ERR-20260323-002] Windows 默认 GBK 编码导致中文乱码

**Logged**: 2026-03-23T16:37:00+08:00
**Priority**: critical
**Status**: resolved
**Area**: backend

### Summary
Python 脚本在 Windows 中文系统上默认使用 GBK 编码写文件，导致中文字符损坏

### Error
```
UnicodeEncodeError: 'gbk' codec can't encode character '\U0001f680' in position 678
```

### Context
- 命令：Python 脚本通过 `json.dump()`、`pathlib.write_text()` 写入含中文/emoji 的 JSON
- 环境：Windows 10 中文版，默认编码 GBK

### Suggested Fix
所有 Python 文件操作必须显式指定 `encoding='utf-8'`：
```python
# 错误
path.write_text(content)
json.dump(data, f, ensure_ascii=False)

# 正确
path.write_text(content, encoding='utf-8')
json.dump(data, f, ensure_ascii=False, indent=2)  # fdopen 也要加 encoding
```

### Metadata
- Reproducible: yes
- Related Files: scripts/file_lock.py, scripts/utils.py, scripts/skill_manager.py
- See Also: ERR-20260323-003

---

## [ERR-20260323-003] fcntl 模块在 Windows 上不可用

**Logged**: 2026-03-23T17:55:00+08:00
**Priority**: critical
**Status**: resolved
**Area**: backend

### Summary
edict 项目的 `file_lock.py` 使用了 Unix 专用的 `fcntl` 模块，Windows 上无法运行

### Error
```
ModuleNotFoundError: No module named 'fcntl'
```

### Context
- 命令：`python dashboard/server.py`
- 环境：Windows 10

### Suggested Fix
添加跨平台兼容：
```python
try:
    import fcntl
    HAS_FCNTL = True
except ImportError:
    HAS_FCNTL = False
    import msvcrt
```

### Metadata
- Reproducible: yes
- Related Files: scripts/file_lock.py

---

## [ERR-20260323-004] pgrep 命令在 Windows 上不可用

**Logged**: 2026-03-23T18:20:00+08:00
**Priority**: high
**Status**: resolved
**Area**: backend

### Summary
edict 看板的网关检测使用了 Linux 的 `pgrep` 命令，Windows 上 Gateway 显示离线

### Error
```
gateway_alive = False (实际 Gateway 正在运行)
```

### Context
- 函数：`_check_gateway_alive()` in server.py
- 原因：`subprocess.run(['pgrep', '-f', 'openclaw-gateway'])` 在 Windows 上不存在

### Suggested Fix
```python
import platform
if platform.system() == 'Windows':
    result = subprocess.run(['tasklist', '/FI', 'IMAGENAME eq node.exe', '/NH'], ...)
else:
    result = subprocess.run(['pgrep', '-f', 'openclaw-gateway'], ...)
```

### Metadata
- Reproducible: yes
- Related Files: dashboard/server.py

---

## [ERR-20260323-005] GitHub 直接访问超时

**Logged**: 2026-03-23T15:54:00+08:00
**Priority**: medium
**Status**: resolved
**Area**: infra

### Summary
国内网络无法直接访问 GitHub，git clone 超时

### Error
```
fatal: unable to access 'https://github.com/...': Failed to connect to github.com port 443
```

### Context
- 命令：`git clone https://github.com/cft0808/edict.git`
- 环境：国内网络，无代理

### Suggested Fix
使用镜像站：`git clone https://ghfast.top/https://github.com/...`
或让用户手动下载 ZIP

### Metadata
- Reproducible: yes
- See Also: ERR-20260323-006

---

## [ERR-20260323-006] ZIP 解压后文件编码损坏

**Logged**: 2026-03-23T17:42:00+08:00
**Priority**: high
**Status**: resolved
**Area**: infra

### Summary
通过浏览器下载的 GitHub ZIP 文件，解压后中文字符变成乱码

### Context
- 用户通过浏览器下载 ZIP 到桌面
- 使用 PowerShell `Expand-Archive` 解压
- 源文件本身 UTF-8 编码正确，但 ZIP 内的文件名/内容被错误编码

### Suggested Fix
优先使用 `git clone`（通过镜像站），而非下载 ZIP

### Metadata
- Reproducible: yes
- See Also: ERR-20260323-002

---

## [ERR-20260323-007] read_json 未指定 UTF-8 编码导致返回空

**Logged**: 2026-03-23T16:24:00+08:00
**Priority**: high
**Status**: resolved
**Area**: backend

### Summary
`scripts/utils.py` 的 `read_json()` 使用 `path.read_text()` 未指定编码，Windows 上读取含中文的 JSON 返回空对象

### Error
```python
# 问题代码
return json.loads(pathlib.Path(path).read_text())  # 默认 GBK，解码失败返回 {}
```

### Suggested Fix
```python
return json.loads(pathlib.Path(path).read_text(encoding='utf-8'))
```

### Metadata
- Reproducible: yes
- Related Files: scripts/utils.py

---
