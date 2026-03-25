---
name: edict-installer
description: "三省六部多Agent编排系统一键安装器。Clone仓库后执行此skill即可自动部署12个Agent、看板系统、技能配置。支持Windows/macOS/Linux。"
---

# ⚔️ 三省六部 · Edict 安装器

将此仓库部署到 OpenClaw，自动创建 12 个 Agent 组成三省六部协作系统。

## 安装步骤

**按顺序执行以下步骤，每步确认成功后再进行下一步。**

### Step 1 — Clone 仓库

```bash
git clone https://github.com/mYoCaRdiA/3Departments6Ministries-Windows-OpenClaw-MultiAgentOrchestrationSystem.git ~/.openclaw/workspace/skills/edict-main
```

如果 GitHub 超时（中国大陆），使用镜像：

```bash
git clone https://ghfast.top/https://github.com/mYoCaRdiA/3Departments6Ministries-Windows-OpenClaw-MultiAgentOrchestrationSystem.git ~/.openclaw/workspace/skills/edict-main
```

### Step 2 — 检查 Python 依赖

```bash
python --version
```

需要 Python 3.9+。如果失败，检查 Python 安装。

### Step 3 — 创建 Agent 工作空间

以下 11 个 Agent 需要创建工作空间目录：

Agent 列表：`taizi`, `zhongshu`, `menxia`, `shangshu`, `libu`, `hubu`, `bingbu`, `xingbu`, `gongbu`, `libu_hr`, `zaochao`

对每个 Agent 执行：

**Windows (PowerShell):**
```powershell
$agents = @('taizi','zhongshu','menxia','shangshu','libu','hubu','bingbu','xingbu','gongbu','libu_hr','zaochao')
foreach ($a in $agents) {
    $ws = "$env:USERPROFILE\.openclaw\workspace-$a"
    New-Item -ItemType Directory -Path "$ws\skills\kanban_update" -Force | Out-Null
    New-Item -ItemType Directory -Path "$ws\scripts" -Force | Out-Null
    New-Item -ItemType Directory -Path "$ws\data" -Force | Out-Null
    Write-Host "$a : workspace created"
}
```

**macOS / Linux:**
```bash
for a in taizi zhongshu menxia shangshu libu hubu bingbu xingbu gongbu libu_hr zaochao; do
    mkdir -p ~/.openclaw/workspace-$a/{skills/kanban_update,scripts,data}
    echo "$a : workspace created"
done
```

### Step 4 — 复制看板脚本到各 Agent

将 `kanban_update.py` 和依赖脚本复制到每个 Agent 的 workspace。

**Windows (PowerShell):**
```powershell
$src = "$env:USERPROFILE\.openclaw\workspace\skills\edict-main"
$agents = @('taizi','zhongshu','menxia','shangshu','libu','hubu','bingbu','xingbu','gongbu','libu_hr','zaochao')
foreach ($a in $agents) {
    $dst = "$env:USERPROFILE\.openclaw\workspace-$a"
    Copy-Item "$src\scripts\kanban_update.py" "$dst\scripts\" -Force
    Copy-Item "$src\scripts\file_lock.py" "$dst\scripts\" -Force
    Copy-Item "$src\scripts\utils.py" "$dst\scripts\" -Force
    Copy-Item "$src\skills\kanban_update\SKILL.md" "$dst\skills\kanban_update\" -Force
    # 复制 AGENTS.md
    $agentMd = "$src\agents\$a\AGENTS.md"
    if (Test-Path $agentMd) { Copy-Item $agentMd "$dst\" -Force }
}
Write-Host "All scripts deployed"
```

**macOS / Linux:**
```bash
SRC=~/.openclaw/workspace/skills/edict-main
for a in taizi zhongshu menxia shangshu libu hubu bingbu xingbu gongbu libu_hr zaochao; do
    DST=~/.openclaw/workspace-$a
    cp $SRC/scripts/kanban_update.py $DST/scripts/
    cp $SRC/scripts/file_lock.py $DST/scripts/
    cp $SRC/scripts/utils.py $DST/scripts/
    cp $SRC/skills/kanban_update/SKILL.md $DST/skills/kanban_update/
    [ -f "$SRC/agents/$a/AGENTS.md" ] && cp "$SRC/agents/$a/AGENTS.md" "$DST/"
done
echo "All scripts deployed"
```

### Step 5 — 初始化数据目录

```bash
# 确保 data 目录存在且 tasks_source.json 为空数组
mkdir -p ~/.openclaw/workspace/skills/edict-main/data
echo '[]' > ~/.openclaw/workspace/skills/edict-main/data/tasks_source.json
```

### Step 6 — 注册 Agent 到 OpenClaw

运行同步脚本将 Agent 配置写入 OpenClaw：

```bash
python ~/.openclaw/workspace/skills/edict-main/scripts/sync_agent_config.py
```

### Step 7 — 重启 Gateway

```bash
openclaw gateway restart
```

### Step 8 — 启动看板

```bash
# 新终端窗口
python ~/.openclaw/workspace/skills/edict-main/dashboard/server.py
```

打开浏览器访问：http://127.0.0.1:7891

## 验证安装

执行以下检查确认安装成功：

```bash
# 检查 Agent 注册
openclaw agents list

# 检查看板 API
curl http://127.0.0.1:7891/healthz

# 检查 Agent workspace
ls ~/.openclaw/workspace-zhongshu/scripts/kanban_update.py
ls ~/.openclaw/workspace-zhongshu/skills/kanban_update/SKILL.md
```

## 快速安装（推荐）

如果不想手动执行上面的步骤，使用安装脚本：

**Windows:**
```powershell
cd ~/.openclaw/workspace/skills/edict-main
.\setup.ps1
```

**macOS / Linux:**
```bash
cd ~/.openclaw/workspace/skills/edict-main
chmod +x install.sh && ./install.sh
```

## 使用方法

### 下旨

通过 Feishu / Telegram / Signal 给任意 Agent 发消息，或直接在看板创建任务。

### 看板

访问 http://127.0.0.1:7891 查看：
- 📋 旨意看板 — 任务状态和流转
- 🔭 省部调度 — Agent 健康监控
- 📜 奏折阁 — 已完成任务归档
- ⚙️ 模型配置 — 切换每个 Agent 的 LLM

### 任务流程

```
你(皇上) → 太子分拣 → 中书省规划 → 门下省审议 → 尚书省派发 → 六部执行 → 完成
```

## 故障排查

### Agent 不响应
```bash
# 检查 Gateway
openclaw gateway status

# 检查 Agent workspace 文件
ls ~/.openclaw/workspace-zhongshu/AGENTS.md
ls ~/.openclaw/workspace-zhongshu/scripts/kanban_update.py
```

### 看板无法访问
```bash
# 检查端口
netstat -an | grep 7891

# 重启看板
python ~/.openclaw/workspace/skills/edict-main/dashboard/server.py
```

### Windows subprocess 报错
如果遇到 `[WinError 2] 系统找不到指定的文件`，确保 `dashboard/server.py` 中的 `subprocess.run` 调用包含 `shell=IS_WINDOWS`。详细修复见 README.md 的"Windows 兼容性适配"章节。
