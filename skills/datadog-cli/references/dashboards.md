# Dashboards Reference

## Commands Overview

| Command | Description |
|---------|-------------|
| `dashboards list` | List all dashboards |
| `dashboards get` | Get full dashboard definition |
| `dashboards create` | Create a new dashboard |
| `dashboards update` | Update an existing dashboard |
| `dashboards delete` | Delete a dashboard |
| `dashboard-lists list` | List all dashboard lists |
| `dashboard-lists get` | Get dashboard list details |
| `dashboard-lists create` | Create a new list |
| `dashboard-lists update` | Update list name |
| `dashboard-lists delete` | Delete a list |
| `dashboard-lists items` | List dashboards in a list |
| `dashboard-lists add-items` | Add dashboards to list |
| `dashboard-lists delete-items` | Remove dashboards from list |

## Dashboard Flags

| Flag | Commands | Description |
|------|----------|-------------|
| `--id` | get, update, delete | Dashboard ID |
| `--title` | create, update | Dashboard title |
| `--layout` | create, update | `ordered` or `free` |
| `--widgets` | create, update | Widgets JSON (or stdin) |
| `--description` | create, update | Dashboard description |
| `--template-variables` | create, update | Template variables JSON |
| `--read-only` | create, update | Make read-only |

## Examples

### List & Get
```bash
npx @leoflores/datadog-cli dashboards list --pretty
npx @leoflores/datadog-cli dashboards get --id "abc-def-ghi" --pretty
```

### Create Dashboard
```bash
npx @leoflores/datadog-cli dashboards create \
  --title "API Monitoring" \
  --layout ordered \
  --widgets '[{"definition":{"type":"timeseries","requests":[{"q":"avg:system.cpu.user{*}"}]}}]' \
  --pretty
```

### With Template Variables
```bash
npx @leoflores/datadog-cli dashboards create \
  --title "Service Dashboard" \
  --layout ordered \
  --template-variables '[{"name":"env","prefix":"env","default":"prod"},{"name":"service","prefix":"service","default":"*"}]' \
  --widgets '[{"definition":{"type":"timeseries","requests":[{"q":"avg:system.cpu.user{$env,$service}"}]}}]' \
  --pretty
```

### Using Stdin
```bash
cat widgets.json | npx @leoflores/datadog-cli dashboards create --title "My Dashboard" --layout ordered --pretty
```

## Template Variables

```json
{"name": "env", "prefix": "env", "default": "prod"}
```

Use in queries: `avg:system.cpu.user{$env,$service}`

## Widget Types

### Timeseries
```json
{"definition": {"type": "timeseries", "title": "CPU", "requests": [{"q": "avg:system.cpu.user{*}"}]}}
```

### Query Value
```json
{"definition": {"type": "query_value", "title": "Errors", "requests": [{"q": "sum:errors{*}.as_count()"}]}}
```

### Top List
```json
{"definition": {"type": "toplist", "title": "Top Services", "requests": [{"q": "top(sum:errors{*} by {service}, 10, 'sum', 'desc')"}]}}
```

## Layout Types

- **ordered**: Auto-arranged responsive grid
- **free**: Manual positioning with x, y, width, height

## Dashboard Lists

```bash
# List all
npx @leoflores/datadog-cli dashboard-lists list --pretty

# Add dashboards to list
npx @leoflores/datadog-cli dashboard-lists add-items --id 123 \
  --dashboards '[{"type":"custom_timeboard","id":"abc-def-ghi"}]' --pretty
```
