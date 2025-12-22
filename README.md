# Datadog Claude Code Marketplace

Claude Code plugin for debugging and triaging with [Datadog](https://www.datadoghq.com/) logs, metrics, and dashboards.

## Quickstart

Open Claude Code, choose `/Plugin` to interactively add the marketplace `leonardocouy/cc-datadog`, or:

```bash
claude /plugin marketplace add leonardocouy/cc-datadog
claude /plugin install datadog@cc-datadog
```

For local development or fork:

```bash
# Change the path on 'add' to your path.
claude /plugin marketplace add ./
```

### Environment Variables (Required)

```bash
export DD_API_KEY="your-api-key"
export DD_APP_KEY="your-app-key"
```

Get keys from: https://app.datadoghq.com/organization-settings/api-keys

## Plugin: datadog

Use this plugin when you need to search Datadog logs, query metrics, tail logs in real-time, trace distributed requests, investigate errors, compare time periods, find log patterns, check service health, manage dashboards, or export observability data.

**Trigger phrases:** "search logs", "tail logs", "query metrics", "check Datadog", "find errors", "trace request", "compare errors", "what services exist", "log patterns", "CPU usage", "service health", "dashboard", "dashboards"

### Commands

| Command | Description |
|---------|-------------|
| `logs search` | Search and filter logs |
| `logs agg` | Aggregate logs by facet |
| `logs tail` | Stream logs in real-time |
| `logs trace` | Find logs for a trace ID |
| `logs context` | Get logs around a timestamp |
| `logs patterns` | Group similar log messages |
| `logs compare` | Compare current vs previous period |
| `logs multi` | Run multiple queries in parallel |
| `metrics query` | Query timeseries metrics |
| `dashboards list` | List dashboards |
| `dashboards get` | Get a dashboard definition |
| `dashboards create` | Create a dashboard |
| `dashboards update` | Update a dashboard |
| `dashboards delete` | Delete a dashboard |
| `dashboard-lists list` | List dashboard lists |
| `dashboard-lists get` | Get a dashboard list |
| `dashboard-lists create` | Create a dashboard list |
| `dashboard-lists update` | Update a dashboard list |
| `dashboard-lists delete` | Delete a dashboard list |
| `errors` | Quick error summary |
| `services` | List services with log activity |

### Example Workflows

**Incident Triage:**
```bash
datadog errors --from 1h --pretty                                    # Overview
datadog logs compare --query "status:error" --period 1h --pretty     # Is this new?
datadog logs patterns --query "status:error" --from 1h --pretty      # What patterns?
datadog logs search --query "status:error service:api" --from 1h --pretty  # Drill down
datadog logs trace --id "TRACE_ID" --pretty                          # Follow trace
```

**Real-time Monitoring:**
```bash
datadog logs tail --query "status:error" --pretty
```

**Export for Analysis:**
```bash
datadog logs search --query "status:error" --from 24h --limit 1000 --output errors.json
```
