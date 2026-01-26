---
title: Scripts Reference
---

# Scripts Reference

## setup-observability.sh

Initial setup script that installs all dependencies and creates configuration files.

```bash
./scripts/setup-observability.sh
```

**What it does:**

- Checks for Homebrew
- Installs Grafana via Homebrew
- Downloads Tempo and Loki binaries
- Creates configuration files for each component
- Generates start/stop/status scripts
- Creates `.env.observability` file

**Created directories:**

- `~/.config/device-farm-observability/` - Configuration files
- `~/.local/share/device-farm-observability/` - Data storage
- `~/.local/bin/` - Binary files (tempo, loki)

## start-observability.sh

Starts all observability services in the background.

```bash
./scripts/start-observability.sh
```

**Services started:**

| Service | Port | Log File |
|---------|------|----------|
| Tempo | 3200 | `~/.local/share/device-farm-observability/tempo.log` |
| Loki | 3100 | `~/.local/share/device-farm-observability/loki.log` |
| Grafana | 3001 | `~/.local/share/device-farm-observability/grafana.log` |

??? note "View start-observability.sh source code"
    ```bash
    #!/bin/bash

    # Start Device Farm Observability Stack

    set -e

    CONFIG_DIR="$HOME/.config/device-farm-observability"
    DATA_DIR="$HOME/.local/share/device-farm-observability"
    PID_DIR="$DATA_DIR/pids"

    mkdir -p "$PID_DIR"

    RED='\033[0;31m'
    GREEN='\033[0;32m'
    YELLOW='\033[1;33m'
    NC='\033[0m'

    log_info() { echo -e "${GREEN}[INFO]${NC} $1"; }
    log_warn() { echo -e "${YELLOW}[WARN]${NC} $1"; }
    log_error() { echo -e "${RED}[ERROR]${NC} $1"; }

    check_port() {
        if lsof -Pi :$1 -sTCP:LISTEN -t >/dev/null 2>&1; then
            return 0
        fi
        return 1
    }

    find_binary() {
        local name=$1
        if command -v "$name" &> /dev/null; then
            command -v "$name"
        elif [ -f "$HOME/.local/bin/$name" ]; then
            echo "$HOME/.local/bin/$name"
        else
            echo ""
        fi
    }

    start_tempo() {
        if check_port 3200; then
            log_warn "Tempo already running on port 3200"
            return
        fi

        local tempo_bin=$(find_binary tempo)
        if [ -z "$tempo_bin" ]; then
            log_error "Tempo binary not found. Run setup-observability.sh first"
            return 1
        fi

        log_info "Starting Tempo..."
        "$tempo_bin" -config.file="$CONFIG_DIR/tempo.yml" > "$DATA_DIR/tempo.log" 2>&1 &
        echo $! > "$PID_DIR/tempo.pid"
        sleep 2

        if check_port 3200; then
            log_info "Tempo started successfully (PID: $(cat $PID_DIR/tempo.pid))"
        else
            log_error "Failed to start Tempo. Check $DATA_DIR/tempo.log"
        fi
    }

    start_loki() {
        if check_port 3100; then
            log_warn "Loki already running on port 3100"
            return
        fi

        local loki_bin=$(find_binary loki)
        if [ -z "$loki_bin" ]; then
            log_error "Loki binary not found. Run setup-observability.sh first"
            return 1
        fi

        log_info "Starting Loki..."
        "$loki_bin" -config.file="$CONFIG_DIR/loki.yml" > "$DATA_DIR/loki.log" 2>&1 &
        echo $! > "$PID_DIR/loki.pid"
        sleep 2

        if check_port 3100; then
            log_info "Loki started successfully (PID: $(cat $PID_DIR/loki.pid))"
        else
            log_error "Failed to start Loki. Check $DATA_DIR/loki.log"
        fi
    }

    # Note: Promtail is NOT needed - logs are sent directly via OTLP from hub/node

    start_grafana() {
        if check_port 3001; then
            log_warn "Grafana already running on port 3001"
            return
        fi

        log_info "Starting Grafana on port 3001..."

        # Find Grafana home path
        local grafana_home
        if [ -d "/opt/homebrew/share/grafana" ]; then
            grafana_home="/opt/homebrew/share/grafana"
        elif [ -d "/usr/local/share/grafana" ]; then
            grafana_home="/usr/local/share/grafana"
        else
            log_error "Could not find Grafana installation"
            return 1
        fi

        grafana server \
            --homepath "$grafana_home" \
            --config "$grafana_home/conf/defaults.ini" \
            cfg:server.http_port=3001 \
            cfg:paths.data="$DATA_DIR/grafana" \
            cfg:paths.logs="$DATA_DIR/grafana/logs" \
            cfg:auth.anonymous.enabled=true \
            cfg:auth.anonymous.org_role=Admin \
            > "$DATA_DIR/grafana.log" 2>&1 &
        echo $! > "$PID_DIR/grafana.pid"
        sleep 3

        if check_port 3001; then
            log_info "Grafana started successfully (PID: $(cat $PID_DIR/grafana.pid))"
        else
            log_error "Failed to start Grafana. Check $DATA_DIR/grafana.log"
        fi
    }

    echo ""
    echo "╔═══════════════════════════════════════════════════════════╗"
    echo "║       Starting Device Farm Observability Stack            ║"
    echo "╚═══════════════════════════════════════════════════════════╝"
    echo ""

    start_tempo
    start_loki
    start_grafana

    echo ""
    echo "═══════════════════════════════════════════════════════════"
    echo ""
    log_info "Observability stack started!"
    echo ""
    echo "  Grafana:   http://localhost:3001 (anonymous admin access)"
    echo "  Tempo:     http://localhost:3200 (traces via OTLP)"
    echo "  Loki:      http://localhost:3100 (logs via OTLP)"
    echo ""
    echo "  Logs are sent directly via OTLP (no file writing)"
    echo ""
    echo "  Before running Device Farm, source the env file:"
    echo "    source .env.observability"
    echo ""
    echo "  To stop services: ./scripts/stop-observability.sh"
    echo ""
    ```

## stop-observability.sh

Stops all observability services.

```bash
./scripts/stop-observability.sh
```

??? note "View stop-observability.sh source code"
    ```bash
    #!/bin/bash

    # Stop Device Farm Observability Stack

    PID_DIR="$HOME/.local/share/device-farm-observability/pids"

    RED='\033[0;31m'
    GREEN='\033[0;32m'
    YELLOW='\033[1;33m'
    NC='\033[0m'

    log_info() { echo -e "${GREEN}[INFO]${NC} $1"; }
    log_warn() { echo -e "${YELLOW}[WARN]${NC} $1"; }

    stop_service() {
        local name=$1
        local pid_file="$PID_DIR/$name.pid"

        if [ -f "$pid_file" ]; then
            local pid=$(cat "$pid_file")
            if kill -0 "$pid" 2>/dev/null; then
                log_info "Stopping $name (PID: $pid)..."
                kill "$pid" 2>/dev/null || true
                rm -f "$pid_file"
            else
                log_warn "$name not running (stale PID file)"
                rm -f "$pid_file"
            fi
        else
            # Try to find and kill by process name
            pkill -f "$name" 2>/dev/null && log_info "Stopped $name" || log_warn "$name not running"
        fi
    }

    echo ""
    echo "Stopping Device Farm Observability Stack..."
    echo ""

    stop_service "grafana"
    stop_service "promtail"
    stop_service "loki"
    stop_service "tempo"

    echo ""
    log_info "All services stopped"
    echo ""
    ```

## status-observability.sh

Checks the status of all services.

```bash
./scripts/status-observability.sh
```

Example output:

```
Device Farm Observability Stack Status
══════════════════════════════════════

  Grafana : running (port 3001)
  Tempo   : running (port 3200)
  Loki    : running (port 3100)
```

??? note "View status-observability.sh source code"
    ```bash
    #!/bin/bash

    # Check status of Device Farm Observability Stack

    GREEN='\033[0;32m'
    RED='\033[0;31m'
    NC='\033[0m'

    check_service() {
        local name=$1
        local port=$2

        if lsof -Pi :$port -sTCP:LISTEN -t >/dev/null 2>&1; then
            echo -e "  $name: ${GREEN}running${NC} (port $port)"
        else
            echo -e "  $name: ${RED}stopped${NC}"
        fi
    }

    echo ""
    echo "Device Farm Observability Stack Status"
    echo "══════════════════════════════════════"
    echo ""

    check_service "Grafana " 3001
    check_service "Tempo   " 3200
    check_service "Loki    " 3100

    echo ""
    echo "Note: Logs are sent directly via OTLP (no Promtail needed)"
    echo ""
    ```
