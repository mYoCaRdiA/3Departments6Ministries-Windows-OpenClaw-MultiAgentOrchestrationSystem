# Learnings Log

## [LRN-20260324-001] best_practice

**Logged**: 2026-03-24T10:00:00+08:00
**Priority**: high
**Status**: promoted
**Area**: infra

### Summary
Windows Python subprocess must use `shell=True` to resolve `.cmd`/`.bat` extensions

### Details
On Windows, executables installed by npm are `.cmd` files (e.g., `openclaw.cmd`). `subprocess.run(["openclaw", ...])` without `shell=True` cannot find the file. Always detect platform and set `shell=IS_WINDOWS`.

### Suggested Action
Promoted to TOOLS.md as a Windows Python rule.

### Metadata
- Source: conversation
- Related Files: dashboard/server.py
- Tags: windows, python, subprocess
- Pattern-Key: windows.subprocess.cmd_resolution
- First-Seen: 2026-03-24

---

## [LRN-20260324-002] best_practice

**Logged**: 2026-03-24T14:00:00+08:00
**Priority**: high
**Status**: promoted
**Area**: infra

### Summary
Shared scripts should use absolute/fixed paths, not `__file__`-relative paths when copied across workspaces

### Details
When a script using `_BASE = Path(__file__).parent.parent` is copied to multiple locations, each copy resolves `_BASE` differently. If the script must always access the same data directory, use a fixed path (e.g., `pathlib.Path.home() / '.openclaw' / 'workspace' / 'skills' / 'edict-main' / 'data'`) instead of relative paths.

### Suggested Action
Promoted to TOOLS.md as a path resolution rule.

### Metadata
- Source: conversation
- Related Files: scripts/kanban_update.py
- Tags: path, workspace, sync, copy
- Pattern-Key: shared_scripts.absolute_paths
- First-Seen: 2026-03-24

---

## [LRN-20260324-003] best_practice

**Logged**: 2026-03-24T14:30:00+08:00
**Priority**: high
**Status**: promoted
**Area**: infra

### Summary
State changes + scheduler timestamps must be updated atomically to prevent rollback

### Details
When a system has both "agent makes state changes" and "scheduler auto-rolls back stale states", the state change must atomically update `lastProgressAt` to prevent the scheduler from reverting it. Race conditions occur when state is updated in one transaction and scheduler timestamp in another.

### Suggested Action
Promoted to TOOLS.md as an edict architecture rule.

### Metadata
- Source: conversation
- Related Files: scripts/kanban_update.py, dashboard/server.py
- Tags: scheduler, state, atomic, race-condition
- Pattern-Key: state_change.atomic_scheduler_update
- First-Seen: 2026-03-24

---

## [LRN-20260324-004] best_practice

**Logged**: 2026-03-24T13:00:00+08:00
**Priority**: medium
**Status**: promoted
**Area**: infra

### Summary
Windows file writes need explicit flush + fsync to be immediately visible

### Details
On Windows, `pathlib.Path.write_text()` may not flush to disk immediately. Subsequent reads may return stale content. Use `open()` with explicit `fh.flush()` and `os.fsync(fh.fileno())` for reliable writes, especially when another process reads the file immediately after.

### Suggested Action
Promoted to TOOLS.md as a Windows Python rule.

### Metadata
- Source: conversation
- Related Files: scripts/kanban_update.py
- Tags: windows, python, filesystem, flush
- Pattern-Key: windows.file_write.flush_fsync
- First-Seen: 2026-03-24

---

## [LRN-20260324-005] knowledge_gap

**Logged**: 2026-03-24T09:35:00+08:00
**Priority**: medium
**Status**: promoted
**Area**: infra

### Summary
Windows doesn't support SIGTERM; only SIGINT partially works in asyncio signal handlers

### Details
`asyncio.get_event_loop().add_signal_handler(signal.SIGTERM, ...)` raises `NotImplementedError` on Windows. Must guard with `if platform.system() != 'Windows':` before registering signal handlers.

### Suggested Action
Promoted to TOOLS.md as a Windows asyncio rule.

### Metadata
- Source: conversation
- Related Files: edict/backend/app/workers/orchestrator_worker.py
- Tags: windows, asyncio, signal
- First-Seen: 2026-03-24

---

## [LRN-20260324-006] best_practice

**Logged**: 2026-03-24T11:50:00+08:00
**Priority**: high
**Status**: promoted
**Area**: infra

### Summary
Fix the SOURCE file, not the copies — sync processes will overwrite edits

### Details
When `sync_agent_config.py` periodically copies scripts from source to agent workspaces, editing the copies is futile. Always edit the source file in `edict-main/scripts/` and let the sync propagate the fix. This is a general pattern for any system with automated file synchronization.

### Suggested Action
Promoted to TOOLS.md as a general system design rule.

### Metadata
- Source: conversation
- Related Files: scripts/kanban_update.py, scripts/sync_agent_config.py
- Tags: sync, source-of-truth, file-management
- Pattern-Key: sync_systems.edit_source_not_copies
- First-Seen: 2026-03-24

---

## [LRN-20260324-007] best_practice

**Logged**: 2026-03-24T13:30:00+08:00
**Priority**: medium
**Status**: promoted
**Area**: infra

### Summary
OpenClaw agent `-m` messages on Windows truncate multi-line content — use single-line notifications

### Details
`openclaw agent --agent X -m "line1\nline2\nline3"` on Windows via PowerShell only delivers the first line. Instead of embedding complex instructions in the dispatch message, make the server do the work directly and send a simple single-line notification to the agent.

### Suggested Action
Promoted to TOOLS.md as an OpenClaw + Windows rule.

### Metadata
- Source: conversation
- Related Files: dashboard/server.py
- Tags: openclaw, dispatch, windows, message
- First-Seen: 2026-03-24

---

## [LRN-20260324-008] best_practice

**Logged**: 2026-03-24T10:45:00+08:00
**Priority**: medium
**Status**: pending
**Area**: infra

### Summary
Edict architecture: dispatch_for_state should update kanban state before spawning agent, not after

### Details
The original design had the agent update kanban state via `kanban_update.py`. This failed on Windows due to path issues and message truncation. Better architecture: server directly updates state (authoritative), then spawns agent to do the actual work. Agent updates progress/completion independently.

### Suggested Action
Consider making this the standard dispatch pattern for edict v2.

### Metadata
- Source: conversation
- Related Files: dashboard/server.py
- Tags: architecture, dispatch, kanban
- Pattern-Key: dispatch.server_side_state_update
- First-Seen: 2026-03-24
