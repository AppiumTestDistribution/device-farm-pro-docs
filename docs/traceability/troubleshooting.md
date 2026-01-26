---
title: Troubleshooting
---

# Troubleshooting

## Logs not appearing in Grafana

1. Check if services are running:
   ```bash
   ./scripts/status-observability.sh
   ```

2. Verify environment variables are set:
   ```bash
   echo $OTEL_EXPORTER_OTLP_LOGS_ENDPOINT
   # Should output: http://localhost:3100
   ```

3. Check Loki logs:
   ```bash
   tail -f ~/.local/share/device-farm-observability/loki.log
   ```

## Grafana shows "No data"

1. Ensure the Loki datasource is configured with UID `loki`
2. Check if Hub/Node are emitting logs (look for `OTLP logs flushed successfully` in console)
3. Verify the time range in Grafana covers when logs were generated

## Session ID filter not working

The Session ID filter uses structured metadata. Ensure you're using the exact session ID format (UUID).

## Docker Compose issues

### Services not starting

Check if ports are already in use:

```bash
lsof -i :3001  # Grafana
lsof -i :3100  # Loki
lsof -i :3200  # Tempo
lsof -i :4318  # OTLP HTTP
```

### Logs not reaching Loki

Verify the container is healthy:

```bash
docker-compose ps
docker-compose logs loki
```

## Native Scripts issues

### Binary not found

Re-run the setup script:

```bash
./scripts/setup-observability.sh
```

### Permission denied

Ensure scripts are executable:

```bash
chmod +x ./scripts/*.sh
```
