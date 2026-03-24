# FEATURE_REQUESTS.md — 功能需求

## [FEAT-20260323-001] edict 自动数据同步

**Logged**: 2026-03-23T18:46:00+08:00
**Priority**: high
**Status**: resolved
**Area**: backend

### Requested Capability
edict 看板数据需要自动同步，不能手动触发

### User Context
用户发了新旨意但看板上看不到，因为 sync 和 refresh 脚本没有被自动调用

### Complexity Estimate
simple

### Suggested Implementation
通过 OpenClaw cron 设置每 60 秒运行一次同步脚本

### Resolution
- **Resolved**: 2026-03-23T18:47:00+08:00
- **Notes**: 创建了 cron job `edict-data-sync`，每 60 秒自动同步

### Metadata
- Frequency: first_time
- Related Features: edict dashboard

---

## [FEAT-20260323-002] GitHub 镜像站克隆支持

**Logged**: 2026-03-23T15:54:00+08:00
**Priority**: medium
**Status**: pending
**Area**: infra

### Requested Capability
自动检测国内网络环境，优先使用镜像站克隆 GitHub 仓库

### User Context
用户在国内，直接 git clone GitHub 仓库超时。需要记住国内用 ghfast.top 等镜像。

### Complexity Estimate
simple

### Suggested Implementation
在 TOOLS.md 记录国内 GitHub 镜像站列表：
- `https://ghfast.top/https://github.com/...`
- `https://ghproxy.com/https://github.com/...`

### Metadata
- Frequency: recurring
- Related Features: skill installation

---
