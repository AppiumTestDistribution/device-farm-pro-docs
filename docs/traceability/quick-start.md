---
title: Quick Start
---

# Traceability & Observability

Device Farm includes built-in observability features using OpenTelemetry for distributed tracing and centralized logging. This enables you to:

- **Trace requests** across Hub and Node services
- **Filter logs** by Session ID to see all logs for a specific test session
- **Correlate events** between Hub and Node using Trace IDs
- **Debug issues** by viewing the complete flow of a session

## Quick Start

You can run the observability stack using either **Docker Compose** (recommended) or **native scripts**.

=== "Docker Compose"

    ### 1. Start the Observability Stack

    The `docker-compose.yml` file is located in the root of the device-farm repository.

    ```bash
    docker-compose up -d
    ```

    ??? note "View docker-compose.yml"
        ```yaml
        version: '3.8'

        # Device Farm Observability Stack
        # Usage: docker-compose up -d
        #
        # Services:
        #   - Grafana: http://localhost:3001 (anonymous admin access)
        #   - Tempo:   http://localhost:3200 (traces via OTLP)
        #   - Loki:    http://localhost:3100 (logs via OTLP)
        #
        # Environment variables for Hub/Node:
        #   export OTEL_SDK_DISABLED=false
        #   export OTEL_TRACES_ENABLED=true
        #   export OTEL_LOGS_ENABLED=true
        #   export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318
        #   export OTEL_EXPORTER_OTLP_LOGS_ENDPOINT=http://localhost:3100

        services:
          tempo:
            image: grafana/tempo:2.3.0
            command: ["-config.file=/etc/tempo/tempo.yaml"]
            volumes:
              - ./tempo-config.yml:/etc/tempo/tempo.yaml
              - tempo-data:/var/tempo
            ports:
              - "3200:3200"   # Tempo HTTP API
              - "4317:4317"   # OTLP gRPC receiver
              - "4318:4318"   # OTLP HTTP receiver
            networks:
              - observability

          loki:
            image: grafana/loki:3.0.0
            ports:
              - "3100:3100"   # Loki HTTP API + OTLP logs receiver
            command: -config.file=/etc/loki/loki-config.yaml
            volumes:
              - ./loki-config.yml:/etc/loki/loki-config.yaml
              - loki-data:/loki
            networks:
              - observability

          # Note: Promtail is NOT needed - logs are sent directly via OTLP from Hub/Node

          grafana:
            image: grafana/grafana:10.0.0
            ports:
              - "3001:3000"   # Using 3001 to avoid conflict with Hub on 3000
            environment:
              - GF_AUTH_ANONYMOUS_ENABLED=true
              - GF_AUTH_ANONYMOUS_ORG_ROLE=Admin
              - GF_AUTH_DISABLE_LOGIN_FORM=false
            volumes:
              - grafana-data:/var/lib/grafana
              - ./grafana/provisioning:/etc/grafana/provisioning
            networks:
              - observability
            depends_on:
              - loki
              - tempo

        volumes:
          loki-data:
          grafana-data:
          tempo-data:

        networks:
          observability:
            driver: bridge
        ```

    This starts:

    - **Grafana** on port 3001
    - **Tempo** on port 3200 (traces)
    - **Loki** on port 3100 (logs)

    ### 2. Configure Environment

    Set the following environment variables before starting Hub/Node:

    ```bash
    export OTEL_SDK_DISABLED=false
    export OTEL_TRACES_ENABLED=true
    export OTEL_LOGS_ENABLED=true
    export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318
    export OTEL_EXPORTER_OTLP_LOGS_ENDPOINT=http://localhost:3100
    ```

    ### 3. Stop the Stack

    ```bash
    docker-compose down
    ```

=== "Native Scripts"

    ### 1. Setup Observability Stack

    Run the setup script to install and configure all components:

    ```bash
    ./scripts/setup-observability.sh
    ```

    This script will:

    - Install Grafana via Homebrew
    - Download Tempo and Loki binaries to `~/.local/bin`
    - Create configuration files in `~/.config/device-farm-observability`
    - Generate start/stop/status scripts

    ### 2. Start the Observability Stack

    ```bash
    ./scripts/start-observability.sh
    ```

    ### 3. Configure Environment

    Source the environment file before starting Hub/Node:

    ```bash
    source .env.observability
    ```

## Start Device Farm

```bash
# Start Hub (with observability enabled)
source .env.observability  # or set env vars manually for Docker Compose
farm-orchestrator

# Start Node (in another terminal)
source .env.observability  # or set env vars manually for Docker Compose
farm-runner
```

## View Logs in Grafana

Open [http://localhost:3001](http://localhost:3001) and navigate to the Device Farm Logs dashboard.

## Disabling Observability

To run Device Farm without observability:

```bash
export OTEL_SDK_DISABLED=true
```

Or simply don't source the `.env.observability` file before starting Hub/Node.
