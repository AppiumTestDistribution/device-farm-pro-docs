---
title: Distributed Setup
---

# Distributed Setup

When running Hub and Node on different machines, the Node needs to send logs/traces to the observability stack running on the Hub machine.

## On the Hub Machine

1. Run the observability stack as described in [Quick Start](quick-start.md)
2. Ensure ports are accessible from Node machines:
   - **3100** - Loki (logs)
   - **4318** - Tempo (traces)

## On Each Node Machine

Create a `.env.observability` file with the Hub machine's IP address:

```bash
# Enable OpenTelemetry
export OTEL_SDK_DISABLED=false
export OTEL_TRACES_ENABLED=true
export OTEL_LOGS_ENABLED=true

# Point to Hub machine's observability stack
export OTEL_EXPORTER_OTLP_ENDPOINT=http://<HUB_IP>:4318
export OTEL_EXPORTER_OTLP_LOGS_ENDPOINT=http://<HUB_IP>:3100
```

Replace `<HUB_IP>` with the actual IP address of your Hub machine (e.g., `192.168.1.100`).

Then start the Node:

```bash
source .env.observability
farm-runner
```

Now logs from all Nodes will appear in the centralized Grafana dashboard on the Hub machine.
