# Setup: OpenTelemetry (OTLP)

The OTLP input is preconfigured in the pack and listens on port 4317 out of the box — no Cribl-side setup is needed. You just need to configure Claude Code to send telemetry to it.

Set the following environment variables in the shell where you run Claude Code:

```bash
export CLAUDE_CODE_ENABLE_TELEMETRY=1
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
export OTEL_EXPORTER_OTLP_PROTOCOL=grpc
export OTEL_METRICS_EXPORTER=otlp
export OTEL_LOGS_EXPORTER=otlp
export OTEL_METRIC_EXPORT_INTERVAL=60000
export OTEL_LOGS_EXPORT_INTERVAL=5000
export OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE=delta
```

```
Variable                                              Description
─────────────────────────────────────────────────────────────────────────────────────────
CLAUDE_CODE_ENABLE_TELEMETRY                          Enables Claude Code's built-in OTel instrumentation
OTEL_EXPORTER_OTLP_ENDPOINT                           OTLP collector endpoint (Cribl Edge OTLP source)
OTEL_EXPORTER_OTLP_PROTOCOL                           Transport protocol (grpc)
OTEL_METRICS_EXPORTER                                 Metrics exporter type (otlp)
OTEL_LOGS_EXPORTER                                    Logs exporter type (otlp)
OTEL_METRIC_EXPORT_INTERVAL                           Metrics flush interval in ms (default: 60000)
OTEL_LOGS_EXPORT_INTERVAL                             Logs flush interval in ms (default: 5000)
OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE     Metrics temporality (delta)
```

> **Tip:** Add these exports to your shell profile (`.bashrc`, `.zshrc`, etc.) so they persist across sessions. If Cribl Edge is running on a remote host, replace `localhost` with the appropriate hostname or IP.
