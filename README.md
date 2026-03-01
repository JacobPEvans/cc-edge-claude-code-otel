# Claude Code Pack For Edge

## Overview

This Cribl Edge pack collects Claude Code telemetry from 10 sources and forwards it to a Cribl Stream worker group for indexing, analysis, and search.

### Data Sources

| # | Input Name | Path (under `~/.claude/`) | Format | `datatype` Value | Interval |
|---|---|---|---|---|---|
| 1 | `claude-code-session` | `projects/**/*.jsonl` | JSONL | `claude-code-session` | 10s |
| 2 | `claude-code-history` | `history.jsonl` | JSONL | `claude-code-history` | 30s |
| 3 | `claude-code-stats` | `stats-cache.json` | JSON | `claude-code-stats` | 60s |
| 4 | `claude-code-logs` | `logs/**/*.jsonl` | JSONL | `claude-code-logs` | 10s |
| 5 | `claude-code-plans` | `plans/*.md` | Markdown | `claude-code-plans` | 60s |
| 6 | `claude-code-tasks` | `tasks/**/*.json` | JSON | `claude-code-tasks` | 30s |
| 7 | `claude-code-teams` | `teams/**/config.json` | JSON | `claude-code-teams` | 60s |
| 8 | `claude-code-plugins` | `plugins/installed_plugins.json` | JSON | `claude-code-plugins` | 120s |
| 9 | `claude-code-otel` | (OTLP gRPC :4317) | OTLP | `claude-code-otel` | N/A |

**Note:** The `claude-code-session` input collects all `.jsonl` files under `projects/`, including subagent transcripts in `subagents/` subdirectories. Downstream consumers can differentiate by checking if the source path contains `/subagents/`.

### Excluded Paths (Security)

The following paths are explicitly NOT collected due to sensitivity or low value:

- `.credentials.json` — OAuth/API credentials
- `settings.json`, `settings.local.json` — User preferences (may contain paths/tokens)
- `security_warnings_state_*.json` — Internal security state
- `debug/` — Debugging artifacts
- `telemetry/` — Raw telemetry (collected via OTLP instead)
- `paste-cache/` — Clipboard contents
- `file-history/` — File history snapshots
- `backups/` — Session backup files
- `cache/` — Transient cache

## Architecture

```
Claude Code (CLI) ──writes──▶ ~/.claude/projects/**/*.jsonl   (sessions + subagents)
       │                      ~/.claude/history.jsonl          (command history)
       │                      ~/.claude/stats-cache.json       (usage stats)
       │                      ~/.claude/logs/**/*.jsonl        (application logs)
       │                      ~/.claude/plans/*.md             (implementation plans)
       │                      ~/.claude/tasks/**/*.json        (task lists)
       │                      ~/.claude/teams/**/config.json   (team configs)
       │                      ~/.claude/plugins/installed_plugins.json
       │                                 │
       │                      Cribl Edge (9 file monitors)
       │                                 │
       └──OTLP/gRPC:4317──▶ Cribl Edge (OpenTelemetry source)
                                         │
                              Cribl HTTP ──▶ Cribl Stream Worker Group
```

## Metadata

Each input sets a `datatype` metadata field. This value is used downstream (e.g., in a Cribl Stream pipeline) to map to the appropriate Splunk `sourcetype`. The pack does NOT set `sourcetype` directly — that mapping is a deployment concern.

The naming convention is `claude-code-<type>`, which maps cleanly to `claude:code:<type>` via string replacement.

## Configuration

### Environment Variable

The pack uses `$CLAUDE_HOME` to locate the Claude Code data directory. All file inputs reference paths relative to `$CLAUDE_HOME/.claude/`.

Set this environment variable on your Cribl Edge instance:

```bash
export CLAUDE_HOME=/home/username   # Linux
export CLAUDE_HOME=/Users/username  # macOS
```

In Kubernetes, set it in the pod spec:

```yaml
env:
  - name: CLAUDE_HOME
    value: "/home/claude"
```

### Outputs

The pack routes all events to the `default` output. Configure your Edge outputs to forward to a Cribl Stream worker group or directly to your destination.

## Installation

### From GitHub Release

```bash
curl -sL "https://github.com/JacobPEvans/cc-edge-claude-code-otel/releases/latest/download/cc-edge-claude-code-otel.crbl" \
  -o /tmp/cc-edge-claude-code-otel.crbl
$CRIBL_HOME/bin/cribl pack install /tmp/cc-edge-claude-code-otel.crbl
```

### From Source

Clone the repository into your Cribl Edge packs directory:

```bash
git clone https://github.com/JacobPEvans/cc-edge-claude-code-otel.git \
  $CRIBL_HOME/data/edge/packs/cc-edge-claude-code
```

## Version History

- **v2.0.0** — Expanded from 2 inputs to 10. Added history, stats, logs, plans, tasks, teams, plugins. Per-input `datatype` metadata. Removed duplicate `Session-Logs-File-Monitor` input.
- **v1.2.0** — Initial VisiCore release. Session logs + OTLP inputs.
