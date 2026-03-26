# claude-statusline

A lightweight, single-line statusline for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — pure bash, zero config required.

```
~/.my-project · ⎥ feat/INF-60-auth · opus · 154k/200k · $1.23 · 5h:34% 7d:13% · ⚡42% · 12m
```

## What it shows

| Segment | Description | Color logic |
|---------|-------------|-------------|
| **Task / Agent** | tmux pane title or active subagent name | Dim text |
| **Directory** | Current working directory (`~` shortened) | Yellow |
| **Branch** | Git branch with worktree badge `[wt]` | Purple |
| **Issue ID** | Extracted from branch name (e.g. `INF-60`, `PROJ-123`) | Purple |
| **Model** | Current model name (`opus`, `sonnet`, `haiku`) | Blue |
| **Context** | Token usage `Xk/Yk` or `N% ctx` | Green → Yellow (70%) → Red (90%) |
| **Cost** | Session cost in USD | Green → Yellow ($0.50) → Red ($2.00) |
| **Rate limits** | 5-hour and 7-day usage percentage | Green → Yellow (60%) → Red (85%) |
| **Cache** | Cache hit ratio (⚡) | Green (≥50%) → Yellow (≥20%) → Red |
| **Duration** | Session time (`Xs`, `Nm`, `NhMm`) | Dim text |

All segments are adaptive — they only appear when data is available.

## Install

### Clone (recommended)

```bash
git clone https://github.com/cheunjm/claude-statusline.git
cd claude-statusline
bash install.sh
```

### Manual

1. Copy `statusline.sh` to `~/.claude/statusline-command.sh`
2. Make it executable: `chmod +x ~/.claude/statusline-command.sh`
3. Add to `~/.claude/settings.json`:

```json
{
  "statusLine": {
    "type": "command",
    "command": "sh ~/.claude/statusline-command.sh"
  }
}
```

4. Restart Claude Code

## Requirements

- `jq` — JSON processor (the only dependency)
- `git` — for branch/worktree detection (optional)
- `tmux` — for task summary display (optional)

## Configuration

Copy `statusline.conf.example` to `~/.claude/statusline.conf` and edit. All settings are optional — the defaults work out of the box.

```sh
# Feature toggles
SHOW_GIT=true
SHOW_RATE_LIMITS=true
SHOW_CACHE=true
SHOW_DURATION=true
SHOW_ISSUE_ID=true
SHOW_TMUX_TASK=true

# Color thresholds
CONTEXT_WARN_PCT=70     # Context: yellow at 70%, red at 90%
CONTEXT_CRIT_PCT=90
COST_WARN_CENTS=50      # Cost: yellow at $0.50, red at $2.00
COST_CRIT_CENTS=200
RATE_WARN_PCT=60        # Rate limits: yellow at 60%, red at 85%
RATE_CRIT_PCT=85

# Performance
GIT_CACHE_TTL=5         # Cache git branch for N seconds
```

## How it works

Claude Code sends a JSON object to your statusline script's stdin on every update. This script:

1. Extracts all fields in a **single `jq` call** (no repeated parsing)
2. Caches git info to avoid subprocess overhead
3. Uses **pure shell math** for cost formatting (no Python/awk)
4. Outputs ANSI-colored text to stdout

Typical execution: **~200ms** cold, **~100ms** cached.

## vs. other statusline tools

| | claude-statusline | [claude-statusline (JungHoonGhae)](https://github.com/JungHoonGhae/claude-statusline) |
|---|---|---|
| Format | Single-line, compact | Multi-line dashboard |
| Dependencies | `jq` only | `jq` + optional `ccusage` |
| Config | Optional KEY=value file | Required config file |
| Issue ID extraction | Yes (Linear, Jira, etc.) | No |
| Git worktree detection | Yes | No |
| tmux task summary | Yes | No |
| Historical cost tracking | No | Yes (via ccusage) |
| Rate limit API calls | No (uses stdin data) | Yes (OAuth + cache) |
| Budget alerts | No | Yes |

**TL;DR**: This one is minimal and workflow-aware. The other is a full dashboard with historical data.

## License

MIT
