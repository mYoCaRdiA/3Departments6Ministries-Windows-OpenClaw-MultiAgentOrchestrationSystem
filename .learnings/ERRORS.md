# Errors Log

## [ERR-20260324-001] subprocess.run openclaw not found on Windows

**Logged**: 2026-03-24T09:30:00+08:00
**Priority**: critical
**Status**: resolved
**Area**: infra

### Summary
`subprocess.run(["openclaw", ...])` fails on Windows with `[WinError 2] 系统找不到指定的文件`

### Error
```
[WinError 2] 系统找不到指定的文件
```

### Context
- Command: `subprocess.run(["openclaw", "agent", "--agent", agent_id, "-m", msg])`
- Environment: Windows 10, PowerShell, openclaw installed via npm as `openclaw.cmd`
- Affected files: `dashboard/server.py`, `edict/backend/app/workers/dispatch_worker.py`

### Resolution
- **Resolved**: 2026-03-24T10:00:00+08:00
- **Notes**: Add `shell=IS_WINDOWS` to all `subprocess.run` calls that invoke `openclaw`. Windows needs shell=True to resolve `.cmd` extensions.

### Metadata
- Reproducible: yes
- Related Files: dashboard/server.py, edict/backend/app/workers/dispatch_worker.py
- Tags: windows, subprocess, openclaw

---

## [ERR-20260324-002] kanban_update.py reads empty tasks_source.json

**Logged**: 2026-03-24T10:40:00+08:00
**Priority**: critical
**Status**: resolved
**Area**: infra

### Summary
Agent workspace copies of `kanban_update.py` write to wrong `tasks_source.json` (empty file in agent workspace, not edict data dir)

### Error
Agent reads `[]` from tasks_source.json despite 12 tasks existing in edict-main/data/

### Context
- `sync_agent_config.py` copies `kanban_update.py` to each agent workspace every 5 seconds
- The script uses `_BASE = pathlib.Path(__file__).resolve().parent.parent` to locate data
- When copied to `workspace-zhongshu/scripts/`, `_BASE` points to `workspace-zhongshu/` which has an empty `data/tasks_source.json`

### Resolution
- **Resolved**: 2026-03-24T14:00:00+08:00
- **Notes**: Changed `TASKS_FILE` and `REFRESH_SCRIPT` to use fixed paths (`_EDICT_DATA`, `_EDICT_SCRIPTS`) instead of `_BASE` relative paths. Source file in `edict-main/scripts/` is the single source of truth; sync propagates the fix.

### Metadata
- Reproducible: yes
- Related Files: scripts/kanban_update.py
- Tags: path, sync, agent-workspace, data-consistency

---

## [ERR-20260324-003] Scheduler auto-rollback overwrites agent state changes

**Logged**: 2026-03-24T11:30:00+08:00
**Priority**: critical
**Status**: resolved
**Area**: infra

### Summary
Agent successfully calls `kanban_update.py state X → Menxia` but scheduler auto-rollback reverts it to `Zhongshu` within seconds

### Error
State changes made by agents are reverted by the 60-second scheduler scan when `lastProgressAt` hasn't been updated

### Context
- Scheduler scans every ~60 seconds
- If `lastProgressAt` exceeds stall threshold, triggers auto-rollback to last known stable state
- `kanban_update.py cmd_state()` updated task state but didn't update `_scheduler.lastProgressAt`

### Resolution
- **Resolved**: 2026-03-24T14:30:00+08:00
- **Notes**: Two fixes: (1) `cmd_state()` now updates `_scheduler.lastProgressAt`, `stallSince`, `retryCount` on every state change; (2) `dispatch_for_state()` directly updates task state in-memory with `autoRollback=False` before spawning agent thread.

### Metadata
- Reproducible: yes
- Related Files: scripts/kanban_update.py, dashboard/server.py
- Tags: scheduler, rollback, race-condition, state-management

---

## [ERR-20260324-004] Dispatch multi-line message truncated on Windows

**Logged**: 2026-03-24T13:00:00+08:00
**Priority**: high
**Status**: resolved
**Area**: infra

### Summary
`openclaw agent -m "multi-line message"` delivers only the first line to the agent when using `--deliver --channel feishu`

### Error
Agent receives `📜 旨意已到中书省` but the step-by-step instructions in subsequent lines are lost

### Context
- Command: `openclaw agent --agent zhongshu -m "line1\nline2\nline3" --deliver --channel feishu --timeout 300`
- The `\n` separated lines are truncated during shell argument passing on Windows

### Resolution
- **Resolved**: 2026-03-24T14:30:00+08:00
- **Notes**: Changed dispatch strategy: (1) Server directly updates kanban state before spawning agent; (2) Notification message is single-line and concise; (3) Agent reads kanban autonomously using its skills.

### Metadata
- Reproducible: yes
- Related Files: dashboard/server.py
- Tags: dispatch, message-truncation, windows, shell

---

## [ERR-20260324-005] SIGTERM signal handler crashes on Windows

**Logged**: 2026-03-24T09:35:00+08:00
**Priority**: medium
**Status**: resolved
**Area**: infra

### Summary
`loop.add_signal_handler(signal.SIGTERM, ...)` raises error on Windows because SIGTERM is not supported

### Error
```
NotImplementedError: add_signal_handler() not implemented on Windows
```

### Context
- `edict/backend/app/workers/orchestrator_worker.py` and `dispatch_worker.py` register SIGTERM/SIGINT handlers
- Windows doesn't support SIGTERM; only SIGINT partially works

### Resolution
- **Resolved**: 2026-03-24T10:00:00+08:00
- **Notes**: Wrapped signal handler registration in `if not IS_WINDOWS:` guard.

### Metadata
- Reproducible: yes
- Related Files: edict/backend/app/workers/orchestrator_worker.py, edict/backend/app/workers/dispatch_worker.py
- Tags: windows, signal, asyncio

---

## [ERR-20260324-006] Agent kanban_update.py path fix overwritten by sync

**Logged**: 2026-03-24T11:50:00+08:00
**Priority**: high
**Status**: resolved
**Area**: infra

### Summary
Fixing `kanban_update.py` in each agent workspace had no effect because `sync_agent_config.py` overwrites them every 5 seconds from the source

### Error
Manually edited workspace copies reverted within seconds; edit tool reported success but changes didn't persist

### Context
- `sync_scripts_to_workspaces()` runs every 5 seconds via dashboard auto_sync thread
- Copies `edict-main/scripts/*` to each `workspace-{agent}/scripts/`
- My edits to workspace copies were immediately overwritten

### Resolution
- **Resolved**: 2026-03-24T13:50:00+08:00
- **Notes**: Fix the SOURCE file (`edict-main/scripts/kanban_update.py`) instead of copies. Changed `_BASE`-relative paths to fixed `_EDICT_DATA` paths. Sync thread then propagates the correct version to all workspaces.

### Metadata
- Reproducible: yes
- Related Files: scripts/kanban_update.py, scripts/sync_agent_config.py
- Tags: sync, overwrite, file-edit, race-condition

---

## [ERR-20260324-007] pathlib.Path.read_text() on Windows with non-ASCII

**Logged**: 2026-03-24T11:55:00+08:00
**Priority**: medium
**Status**: resolved
**Area**: infra

### Summary
Python file writes on Windows not persisting when using `pathlib.Path.write_text()` without explicit flush/fsync

### Error
`pathlib.Path.read_text(encoding='utf-8')` returns stale content immediately after `write_text()`

### Context
- Windows file system buffering
- `write_text()` doesn't guarantee immediate flush to disk
- Verified with `open()` + `flush()` + `os.fsync()` which works reliably

### Resolution
- **Resolved**: 2026-03-24T11:55:00+08:00
- **Notes**: Use `open(str(f), 'w', encoding='utf-8', newline='\n')` with explicit `fh.flush()` and `os.fsync(fh.fileno())` for reliable file writes on Windows.

### Metadata
- Reproducible: sometimes
- Related Files: scripts/kanban_update.py
- Tags: windows, filesystem, python, buffering
