# Troubleshooting

## File Monitor

**No events flowing:**

1. Verify `$CLAUDE_HOME` is set correctly and points to the right directory for your OS.
2. Check that the file monitor discovered files in the Cribl Edge logs.
3. Verify the route is not disabled in the Cribl UI.
4. Confirm the output destination is reachable.

**EACCES errors in worker log (Linux):**
The `cribl` user cannot read the `.jsonl` files. Run the ACL mask fix commands from the [Linux permissions](setup-file-monitor.md#linux) section and restart the Cribl Edge worker:

```bash
sudo -u cribl /opt/cribl/bin/cribl restart
```

**Access denied errors (Windows):**
If running Edge under a custom service account (not `LocalSystem`), apply the NTFS permissions from the [Windows permissions](setup-file-monitor.md#windows) section and restart the Cribl Edge service:

```powershell
Restart-Service CriblEdge
```

**Path separator issues (Windows):**
Ensure the `CLAUDE_HOME` variable uses backslashes (`\`) and not forward slashes. The File Monitor source on Windows expects native Windows paths.

**Stale file tracking:**
Cribl Edge tracks file state in its kvstore. If you need to re-ingest files from the beginning, stop the worker, clear the relevant kvstore directories, and restart.

- Linux/macOS: `/opt/cribl/state/kvstore/default/file_claude-code-logs*/`
- Windows: `C:\ProgramData\Cribl\state\kvstore\default\file_claude-code-logs*\`

## OpenTelemetry

**No OTLP telemetry arriving:**

1. Confirm `CLAUDE_CODE_ENABLE_TELEMETRY=1` is set in the Claude Code shell environment.
2. Verify `OTEL_EXPORTER_OTLP_ENDPOINT` points to the correct Cribl Edge host and port (`http://localhost:4317`).
3. Check that port 4317 is not blocked by a firewall or already in use by another process.
4. Inspect the Cribl Edge worker logs for connection or parsing errors on the OTLP source.
5. Verify the `claude-code-otel` input is not disabled in the Cribl UI.
