# Reference

## Pack Components

These are preconfigured in the pack. This section is for reference — no action is needed here unless you want to customize the defaults.

### File Monitor Inputs

All file monitors resolve paths via `$CLAUDE_HOME/.claude/<subdir>`. Each sets a `datatype` metadata field for downstream routing.

| Input | Path (under `$CLAUDE_HOME/.claude/`) | Filter | Recursive | Interval |
|---|---|---|---|---|
| `claude-code-session` | `projects` | `*.jsonl` | Yes | 10s |
| `claude-code-history` | `.` | `history.jsonl` | No | 30s |
| `claude-code-stats` | `.` | `stats-cache.json` | No | 60s |
| `claude-code-logs` | `logs` | `*.jsonl` | Yes | 10s |
| `claude-code-plans` | `plans` | `*.md` | No | 60s |
| `claude-code-tasks` | `tasks` | `*.json` | Yes | 30s |
| `claude-code-teams` | `teams` | `config.json` | Yes | 60s |
| `claude-code-plugins` | `plugins` | `installed_plugins.json` | No | 120s |

### Input: `claude-code-otel`

- **Type:** OpenTelemetry (`open_telemetry`)
- **Protocol:** gRPC
- **Host:** `0.0.0.0`
- **Port:** `4317`
- **TLS:** Disabled
- **Auth:** None
- **OTLP Version:** 1.3.1
- **Max active connections:** 1000

Receives OTLP telemetry (metrics and logs) directly from Claude Code over gRPC.

### Route: Default

- **Pipeline:** `main`
- **Output:** `defaultWG` (Cribl HTTP)

All inputs route through the default pipeline.

### Output: `defaultWG`

- **Type:** Cribl HTTP
- **Compression:** gzip
- **Concurrency:** 5

Forwards events to the Cribl Stream worker group for further routing and delivery to your destination of choice (Splunk, S3, Elastic, etc.).

---

## Session Log Event Types

Each line in a `.jsonl` session log file is a standalone JSON object. Most event types share a common envelope with fields like `sessionId`, `uuid`, `timestamp`, `cwd`, and `type`. The `assistant` and `user` events additionally carry an API-shaped `message` object with `content` blocks and `usage` telemetry.

### Top-Level Event Types

Identified by the `type` field on each JSON object:

```
Type                     Description
─────────────────────────────────────────────────────────
assistant                Claude's response turns (text, tool invocations, token usage)
user                     User messages and tool results (stdout/stderr) returned to Claude
summary                  Conversation summary metadata (model, token counts, cost, turns, duration)
file-history-snapshot    File state checkpoints for session undo/restore
system                   Internal system events (hook summaries, configuration signals)
progress                 Wrapper for real-time progress updates (see subtypes below)
queue-operation          Message queue lifecycle events (enqueue/dequeue)
```

### Progress Subtypes

`progress` events contain a `data.type` field that identifies the specific kind of progress update:

```
Subtype                  Description
─────────────────────────────────────────────────────────
agent_progress           Subagent (Task tool) execution progress and streamed responses
tool_result              Completed tool execution results returned to the model
hook_progress            User-configured hook lifecycle events (start, stop, output)
text                     Streamed text fragments during assistant response generation
bash_progress            Real-time output from long-running Bash tool commands
query_update             Web search query sent to a search provider
search_results_received  Web search results returned from a search provider
create                   File or resource creation events from tool execution
update                   File or resource modification events from tool execution
mcp_progress             MCP (Model Context Protocol) server tool execution progress
```

### Example Event (summary)

```json
{
  "type": "summary",
  "summary": {
    "model": "claude-opus-4-6",
    "inputTokens": 42531,
    "outputTokens": 1247,
    "totalCostUSD": 0.285,
    "numTurns": 8,
    "durationMs": 134520
  }
}
```

---

## OpenTelemetry Metrics and Events

Claude Code emits 13 telemetry signals over OTLP — 8 metrics and 5 events.

### Metrics

Exported via `OTEL_METRICS_EXPORTER`. These are numerical measurements collected at the interval defined by `OTEL_METRIC_EXPORT_INTERVAL`.

```
Metric                                Description
─────────────────────────────────────────────────────────────────────────────
claude_code.session.count              Number of Claude Code sessions started
claude_code.cost.usage                 Cost in USD incurred by API calls
claude_code.token.usage                Token consumption (input and output)
claude_code.active_time.total          Total active time spent in sessions
claude_code.lines_of_code.count        Lines of code written or modified
claude_code.code_edit_tool.decision    Code edit tool invocation decisions (accept/reject)
claude_code.pull_request.count         Pull requests created
claude_code.commit.count               Git commits created
```

### Events

Exported via `OTEL_LOGS_EXPORTER`. These are discrete log entries emitted when specific actions occur during a session.

```
Event                                  Description
─────────────────────────────────────────────────────────────────────────────
claude_code.user_prompt                User prompt submitted to Claude
claude_code.tool_result                Result returned from a tool execution
claude_code.api_request                API request sent to the Claude API
claude_code.api_error                  Error response received from the Claude API
claude_code.tool_decision              Model's decision to invoke a specific tool
```
