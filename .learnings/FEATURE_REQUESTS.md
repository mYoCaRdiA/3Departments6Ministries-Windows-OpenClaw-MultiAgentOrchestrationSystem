# Feature Requests Log

## [FEAT-20260324-001] Cross-platform dispatch mechanism

**Logged**: 2026-03-24T13:30:00+08:00
**Priority**: high
**Status**: pending
**Area**: backend

### Requested Capability
Dispatch messages to agents should work reliably on both Windows and macOS/Linux without platform-specific workarounds

### User Context
5 separate bugs were caused by Windows incompatibility in the dispatch pipeline (subprocess, message truncation, path resolution, signal handling, file sync). A unified cross-platform dispatch API would prevent this class of issues.

### Complexity Estimate
complex

### Suggested Implementation
- Abstract subprocess calls behind a platform-aware wrapper
- Use OpenClaw's internal API for dispatch instead of shelling out to `openclaw agent`
- Or use `sessions_spawn` / `sessions_send` for agent communication

### Metadata
- Frequency: recurring
- Related Features: dispatch_for_state, wake_agent

---

## [FEAT-20260324-002] Agent workspace path resolution

**Logged**: 2026-03-24T10:45:00+08:00
**Priority**: high
**Status**: pending
**Area**: backend

### Requested Capability
Scripts copied to agent workspaces should automatically resolve shared data paths correctly

### User Context
`kanban_update.py` uses `__file__` to find data, but when copied to 11 agent workspaces, each copy points to its own empty data directory. Need a way to declare "this script always reads from X" regardless of where it's installed.

### Complexity Estimate
medium

### Suggested Implementation
- Environment variable `EDICT_DATA_DIR` that scripts check
- Or a config file in each workspace that points to the shared data
- Or `sync_agent_config.py` patches paths during copy

### Metadata
- Frequency: recurring
- Related Features: kanban_update.py, sync_agent_config.py
