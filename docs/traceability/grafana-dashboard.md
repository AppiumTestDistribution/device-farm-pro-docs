---
title: Grafana Dashboard
---

# Grafana Dashboard

## Importing the Dashboard

1. Open Grafana at [http://localhost:3001](http://localhost:3001)
2. Go to **Dashboards** → **Import**
3. Paste the JSON below or upload the file
4. Click **Import**

## Dashboard Features

The Device Farm Logs dashboard provides:

- **Log viewer** with timestamp, log level, service (HUB/NODE), and message
- **Filter by Session ID** - View all logs for a specific test session
- **Filter by Trace ID** - Correlate logs across services
- **Text filter** - Search for any text in logs
- **Log volume chart** - Visualize log activity over time by level (info, warn, error)

## Dashboard JSON

Create a file named `device-farm-logs.json` with the following content:

```json
{
  "annotations": {
    "list": [
      {
        "builtIn": 1,
        "datasource": {
          "type": "grafana",
          "uid": "-- Grafana --"
        },
        "enable": true,
        "hide": true,
        "iconColor": "rgba(0, 211, 255, 1)",
        "name": "Annotations & Alerts",
        "type": "dashboard"
      }
    ]
  },
  "editable": true,
  "fiscalYearStartMonth": 0,
  "graphTooltip": 0,
  "id": null,
  "links": [],
  "panels": [
    {
      "datasource": {
        "type": "loki",
        "uid": "loki"
      },
      "description": "Logs from hub and node services. Filter by Trace ID, Session ID, or any text.",
      "fieldConfig": {
        "defaults": {},
        "overrides": []
      },
      "gridPos": {
        "h": 20,
        "w": 24,
        "x": 0,
        "y": 0
      },
      "id": 1,
      "options": {
        "dedupStrategy": "none",
        "enableInfiniteScrolling": false,
        "enableLogDetails": true,
        "prettifyLogMessage": false,
        "showCommonLabels": false,
        "showControls": false,
        "showLabels": false,
        "showTime": true,
        "sortOrder": "Descending",
        "syntaxHighlighting": false,
        "wrapLogMessage": true
      },
      "pluginVersion": "12.3.1",
      "targets": [
        {
          "datasource": {
            "type": "loki",
            "uid": "loki"
          },
          "direction": "backward",
          "editorMode": "code",
          "expr": "{service=~\"hub|node\"} | traceId =~ `$traceId` | sessionId =~ `$sessionId` |= `$filter`",
          "queryType": "range",
          "refId": "A"
        }
      ],
      "title": "Device Farm Logs",
      "transparent": true,
      "type": "logs"
    },
    {
      "datasource": {
        "type": "loki",
        "uid": "loki"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisBorderShow": false,
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "barWidthFactor": 0.6,
            "drawStyle": "bars",
            "fillOpacity": 100,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "insertNulls": false,
            "lineInterpolation": "linear",
            "lineWidth": 1,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "never",
            "showValues": false,
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "normal"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": 0
              }
            ]
          },
          "unit": "short"
        },
        "overrides": [
          {
            "matcher": {
              "id": "byName",
              "options": "error"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "red",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "warn"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "yellow",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "info"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "blue",
                  "mode": "fixed"
                }
              }
            ]
          }
        ]
      },
      "gridPos": {
        "h": 6,
        "w": 24,
        "x": 0,
        "y": 20
      },
      "id": 2,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "hideZeros": false,
          "mode": "multi",
          "sort": "none"
        }
      },
      "pluginVersion": "12.3.1",
      "targets": [
        {
          "datasource": {
            "type": "loki",
            "uid": "loki"
          },
          "editorMode": "code",
          "expr": "sum by (log_level) (count_over_time({service=~\"hub|node\"}[$__interval]))",
          "legendFormat": "{{log_level}}",
          "queryType": "range",
          "refId": "A"
        }
      ],
      "title": "Log Volume by Level",
      "type": "timeseries"
    }
  ],
  "preload": false,
  "refresh": "15m",
  "schemaVersion": 42,
  "tags": [
    "logs",
    "device-farm"
  ],
  "templating": {
    "list": [
      {
        "current": {
          "text": "",
          "value": ""
        },
        "description": "Filter by Trace ID",
        "label": "Trace ID",
        "name": "traceId",
        "options": [
          {
            "selected": true,
            "text": "",
            "value": ""
          }
        ],
        "query": "",
        "type": "textbox"
      },
      {
        "current": {
          "text": "",
          "value": ""
        },
        "description": "Filter by Session ID",
        "label": "Session ID",
        "name": "sessionId",
        "options": [
          {
            "selected": true,
            "text": "",
            "value": ""
          }
        ],
        "query": "",
        "type": "textbox"
      },
      {
        "current": {
          "text": "",
          "value": ""
        },
        "description": "Filter by any text",
        "label": "Filter",
        "name": "filter",
        "options": [
          {
            "selected": true,
            "text": "",
            "value": ""
          }
        ],
        "query": "",
        "type": "textbox"
      }
    ]
  },
  "time": {
    "from": "now-1h",
    "to": "now"
  },
  "timepicker": {},
  "timezone": "browser",
  "title": "Device Farm Logs",
  "uid": "device-farm-logs",
  "version": 1
}
```
