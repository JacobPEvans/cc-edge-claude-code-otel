# Claude Code Pack For Edge

## Overview

This Cribl Edge pack collects Claude Code telemetry from 10 sources (9 file monitors + 1 OTLP receiver) and forwards it to a Cribl Stream worker group for indexing, analysis, and search:

1. **Session logs** — Conversation transcripts (`.jsonl`) from `~/.claude/projects/`
2. **Command history** — `~/.claude/history.jsonl`
3. **Usage stats** — `~/.claude/stats-cache.json`
4. **Application logs** — `~/.claude/logs/**/*.jsonl`
5. **Plans** — `~/.claude/plans/*.md`
6. **Tasks** — `~/.claude/tasks/**/*.json`
7. **Teams** — `~/.claude/teams/**/config.json`
8. **Plugins** — `~/.claude/plugins/installed_plugins.json`
9. **OpenTelemetry (OTLP)** — Metrics and logs via gRPC on port 4317

## Architecture

```
Claude Code (CLI) ──writes──▶ ~/.claude/projects/**/*.jsonl   (sessions)
       │                      ~/.claude/history.jsonl          (command history)
       │                      ~/.claude/stats-cache.json       (usage stats)
       │                      ~/.claude/logs/**/*.jsonl        (application logs)
       │                      ~/.claude/plans/*.md             (plans)
       │                      ~/.claude/tasks/**/*.json        (tasks)
       │                      ~/.claude/teams/**/config.json   (teams)
       │                      ~/.claude/plugins/installed_plugins.json
       │                                 │
       │                      Cribl Edge (9 file monitors)
       │                                 │
       └──OTLP/gRPC:4317──▶ Cribl Edge (OpenTelemetry source)
                                         │
                              Cribl HTTP ──▶ Cribl Stream Worker Group
```

## Data Sources

This pack collects two distinct categories of data from Claude Code. You can enable either or both depending on your needs.

### Session Logs (File Monitor)

The file monitor inputs read files that Claude Code writes to disk. The primary input monitors `.jsonl` transcript files containing the **full content** of each session — prompts, responses, tool calls, and results. Seven additional monitors collect history, stats, logs, plans, tasks, teams, and plugin data.

Each input sets a `datatype` metadata field (e.g., `claude-code-session`, `claude-code-history`) used downstream to map to the appropriate sourcetype.

**Excluded paths (security):** `.credentials.json`, `settings.json`, `settings.local.json`, `security_warnings_state_*.json`, `debug/`, `telemetry/`, `paste-cache/`, `file-history/`, `backups/`, `cache/`

### OpenTelemetry (OTLP)

The OTLP input receives **operational telemetry** pushed directly from Claude Code's built-in OpenTelemetry instrumentation over gRPC. This is structured metrics and logs — not conversation content. Use this for dashboards, alerting, and fleet-wide visibility.

### Choosing Between Them

```
Session Logs (File Monitor)
───────────────────────────────────────────────────
What it captures         Full conversation content (prompts, responses, tool calls)
Data format              JSONL (one JSON object per line)
Collection method        Cribl Edge reads files from disk
Best for                 Audit, compliance, session replay, content search
Permissions needed       Filesystem read access to ~/.claude/
Contains PII             Yes

OpenTelemetry (OTLP)
───────────────────────────────────────────────────
What it captures         Operational metrics and instrumented logs
Data format              OTLP (standard OpenTelemetry protocol)
Collection method        Claude Code pushes telemetry over gRPC
Best for                 Dashboards, alerting, performance monitoring
Permissions needed       Network access to port 4317
Contains PII             No
```

---

## Requirements

- **Cribl Edge** 4.13.0+
- **Claude Code** installed for the local user
- **Supported platforms:** Linux, macOS (Sonoma 14+, Sequoia 15+, Tahoe 26), Windows (Server 2016/2019/2022, Windows 10/11)
- **Filesystem permissions:** The Cribl Edge process must have read access to `~/.claude/` and its subdirectories (if using the file monitor inputs)
- **Port 4317** available for the OTLP gRPC listener (if using the OpenTelemetry input)

---

## Setup

### File Monitor

Set `CLAUDE_HOME` to the home directory of the user running Claude Code, then grant the Cribl Edge process read access to `~/.claude/`.

```bash
# Linux
export CLAUDE_HOME=/home/<user>

# macOS
export CLAUDE_HOME=/Users/<user>
```

For detailed platform-specific instructions (Windows, filesystem permissions, ACLs, cron jobs): see [docs/setup-file-monitor.md](docs/setup-file-monitor.md).

### OpenTelemetry (OTLP)

The OTLP input listens on port 4317 out of the box. Configure Claude Code to send telemetry:

```bash
export CLAUDE_CODE_ENABLE_TELEMETRY=1
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
export OTEL_EXPORTER_OTLP_PROTOCOL=grpc
export OTEL_METRICS_EXPORTER=otlp
export OTEL_LOGS_EXPORTER=otlp
```

For the full environment variable reference: see [docs/setup-otel.md](docs/setup-otel.md).

---

## Documentation

- [File Monitor Setup](docs/setup-file-monitor.md) — `CLAUDE_HOME` configuration, platform-specific filesystem permissions
- [OpenTelemetry Setup](docs/setup-otel.md) — OTLP environment variables
- [Reference](docs/reference.md) — Pack components, session log event types, OTLP metrics and events
- [Troubleshooting](docs/troubleshooting.md) — Common issues and fixes

---

## Release Notes

- **1.2.1** — 2026-03-05
  - Expanded from 2 inputs to 10 (added history, stats, logs, plans, tasks, teams, plugins)
  - Per-input `datatype` metadata for downstream sourcetype mapping
  - `CLAUDE_HOME` now points to user home directory (paths use `$CLAUDE_HOME/.claude/<subdir>`)
  - Removed duplicate `Session-Logs-File-Monitor` input
- **1.2.0** — 2026-02-17
  - Added OpenTelemetry (OTLP) gRPC source on port 4317 for Claude Code native telemetry
  - Client-side OTEL environment variable configuration guide
  - Full session log event type documentation (top-level types and progress subtypes)
  - OTLP troubleshooting section
- **1.1.0** — 2026-02-17
  - Added cross-platform support for macOS and Windows
  - Platform-specific permission guidance (POSIX ACLs, macOS ACLs, NTFS)
  - Platform-specific environment variable configuration
  - Windows service account and `LocalSystem` documentation
  - Expanded troubleshooting for all platforms
- **1.0.0** — 2026-02-16
  - Initial release
  - File monitor input for Claude Code `.jsonl` session logs
  - ACL-based permission model for non-root Cribl Edge access
  - Cribl HTTP output to Stream worker group

## Authors

* Andrew Hendrix - <Andrewh@VisiCoreTech.com>
* Jacob Evans - <jevans@VisiCoreTech.com>

To contact us, please email <CriblPacks@VisiCoreTech.com>.

## Contributing to the Pack

To contribute to this Pack, or to report any issues or enhancement requests, please connect with **VisiCore Tech** on [Cribl Community Slack](https://cribl-community.slack.com) or email us at: <CriblPacks@visicoretech.com>.

## License
---
This Pack uses the following license: [`Apache 2.0`](https://github.com/criblio/appscope/blob/master/LICENSE)
