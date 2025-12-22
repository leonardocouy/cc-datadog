# Dashboards Reference

Complete reference for managing Datadog dashboards via CLI.

## Commands Overview

| Command | Description |
|---------|-------------|
| `dashboards list` | List all dashboards (summaries) |
| `dashboards get` | Get full dashboard definition |
| `dashboards create` | Create a new dashboard |
| `dashboards update` | Update an existing dashboard |
| `dashboards delete` | Delete a dashboard |

## All Available Flags

### Dashboard Operations Flags

| Flag | Type | Commands | Description |
|------|------|----------|-------------|
| `--id` | string | get, update, delete | Dashboard ID (e.g., `abc-def-ghi`) |
| `--title` | string | create, update | Dashboard title |
| `--layout` | string | create, update | Layout type: `ordered` or `free` |
| `--widgets` | JSON | create, update | Widgets JSON array (or pipe via stdin) |
| `--description` | string | create, update | Dashboard description text |
| `--template-variables` | JSON | create, update | Template variables for dynamic filtering |
| `--notify-list` | JSON | create, update | User handles to notify on changes |
| `--read-only` | boolean | create, update | Make dashboard read-only |

### List Flags

| Flag | Type | Description |
|------|------|-------------|
| `--count` | number | Number of dashboards per page |
| `--start` | number | Pagination offset |
| `--shared` | boolean | Only show shared dashboards |
| `--deleted` | boolean | Only show deleted dashboards |

## Listing Dashboards

```bash
# List first 25 dashboards
datadog dashboards list --count 25 --pretty

# Paginate through results
datadog dashboards list --count 50 --start 0 --pretty
datadog dashboards list --count 50 --start 50 --pretty

# Filter by shared/deleted
datadog dashboards list --shared --pretty
datadog dashboards list --deleted --pretty
```

## Getting a Dashboard

```bash
# Get full dashboard definition (includes widgets, variables, etc.)
datadog dashboards get --id "abc-def-ghi" --pretty

# Export to file for backup/modification
datadog dashboards get --id "abc-def-ghi" --output dashboard-backup.json
```

## Creating Dashboards

### Basic Dashboard

```bash
datadog dashboards create \
  --title "My Dashboard" \
  --layout ordered \
  --widgets '[{"definition":{"type":"timeseries","requests":[{"q":"avg:system.cpu.user{*}"}]}}]' \
  --pretty
```

### With Description

```bash
datadog dashboards create \
  --title "API Performance" \
  --layout ordered \
  --description "Monitors API latency, error rates, and throughput for the production environment" \
  --widgets '[{"definition":{"type":"timeseries","requests":[{"q":"avg:trace.http.request.duration{service:api}"}]}}]' \
  --pretty
```

### With Template Variables

Template variables allow dynamic filtering across all widgets.

```bash
datadog dashboards create \
  --title "Service Dashboard" \
  --layout ordered \
  --description "Multi-service monitoring dashboard" \
  --template-variables '[
    {"name": "env", "prefix": "env", "default": "production"},
    {"name": "service", "prefix": "service", "default": "*"},
    {"name": "host", "prefix": "host", "default": "*"}
  ]' \
  --widgets '[{"definition":{"type":"timeseries","requests":[{"q":"avg:system.cpu.user{$env,$service,$host}"}]}}]' \
  --pretty
```

### With Notify List

```bash
datadog dashboards create \
  --title "Critical Alerts Dashboard" \
  --layout ordered \
  --notify-list '["user@company.com", "oncall@company.com"]' \
  --widgets '[{"definition":{"type":"alert_value","alert_id":"12345","precision":2}}]' \
  --pretty
```

### Full Featured Dashboard

```bash
datadog dashboards create \
  --title "Production Overview" \
  --layout ordered \
  --description "Comprehensive production monitoring with environment and service filtering" \
  --template-variables '[
    {"name": "env", "prefix": "env", "default": "prod"},
    {"name": "service", "prefix": "service", "default": "*"}
  ]' \
  --notify-list '["team@company.com"]' \
  --read-only \
  --widgets '[
    {"definition":{"type":"timeseries","title":"CPU Usage","requests":[{"q":"avg:system.cpu.user{$env,$service}"}]}},
    {"definition":{"type":"timeseries","title":"Memory Usage","requests":[{"q":"avg:system.mem.used{$env,$service}"}]}},
    {"definition":{"type":"timeseries","title":"Error Rate","requests":[{"q":"sum:trace.http.request.errors{$env,$service}.as_count()"}]}}
  ]' \
  --pretty
```

### Using Stdin for Complex Widgets

For complex widget configurations, pipe JSON via stdin:

```bash
cat << 'EOF' | datadog dashboards create --title "Complex Dashboard" --layout ordered --description "Dashboard with complex widgets"
[
  {
    "definition": {
      "type": "timeseries",
      "title": "Request Latency P99",
      "requests": [
        {"q": "p99:trace.http.request.duration{service:api}"}
      ]
    }
  },
  {
    "definition": {
      "type": "toplist",
      "title": "Top Errors by Service",
      "requests": [
        {"q": "top(sum:trace.http.request.errors{*} by {service}.as_count(), 10, 'sum', 'desc')"}
      ]
    }
  }
]
EOF
```

## Updating Dashboards

### Update Title and Description

```bash
datadog dashboards update \
  --id "abc-def-ghi" \
  --title "Updated Dashboard Title" \
  --layout ordered \
  --description "New description for the dashboard" \
  --widgets '[{"definition":{"type":"timeseries","requests":[{"q":"avg:system.cpu.user{*}"}]}}]' \
  --pretty
```

### Add Template Variables to Existing Dashboard

```bash
datadog dashboards update \
  --id "abc-def-ghi" \
  --title "Service Dashboard" \
  --layout ordered \
  --description "Now with environment filtering!" \
  --template-variables '[
    {"name": "env", "prefix": "env", "default": "prod"},
    {"name": "service", "prefix": "service", "default": "*"}
  ]' \
  --widgets '[{"definition":{"type":"timeseries","requests":[{"q":"avg:system.cpu.user{$env,$service}"}]}}]' \
  --pretty
```

### Update Workflow (Get -> Modify -> Update)

```bash
# 1. Get current dashboard and save to file
npx @leoflores/datadog-cli dashboards get --id "abc-def-ghi" > /tmp/dashboard.json

# 2. Extract widgets array
cat /tmp/dashboard.json | jq '.dashboard.widgets' > /tmp/widgets.json

# 3. Modify the widgets JSON as needed (edit file or use jq)
# Example: Make a query dynamic by grouping
# Before: "query": "sum:metric{app:emback}.as_count()"
# After:  "query": "sum:metric{$app} by {app}.as_count()"

# 4. Update dashboard by piping modified widgets
cat /tmp/widgets.json | npx @leoflores/datadog-cli dashboards update \
  --id "abc-def-ghi" \
  --title "Dashboard Title" \
  --layout ordered \
  --template-variables '[{"name":"app","prefix":"app","default":"*"}]' \
  --pretty
```

## Deleting Dashboards

```bash
datadog dashboards delete --id "abc-def-ghi" --pretty
```

## Template Variables Reference

### Structure

```json
{
  "name": "variable_name",      // Variable name (used as $variable_name in queries)
  "prefix": "tag_prefix",       // Tag prefix to filter on
  "default": "default_value",   // Default value (* for all)
  "available_values": []        // Optional: restrict available values
}
```

### Common Template Variables

```json
[
  {"name": "env", "prefix": "env", "default": "prod"},
  {"name": "service", "prefix": "service", "default": "*"},
  {"name": "host", "prefix": "host", "default": "*"},
  {"name": "region", "prefix": "region", "default": "*"},
  {"name": "availability_zone", "prefix": "availability-zone", "default": "*"}
]
```

### Using Variables in Queries

```
avg:system.cpu.user{$env,$service,$host}
sum:trace.http.request.errors{env:$env,service:$service}.as_count()
```

## Widget Types Reference

### Timeseries

```json
{
  "definition": {
    "type": "timeseries",
    "title": "CPU Usage",
    "requests": [
      {"q": "avg:system.cpu.user{*}"}
    ]
  }
}
```

### Query Value

```json
{
  "definition": {
    "type": "query_value",
    "title": "Current Error Count",
    "requests": [
      {"q": "sum:trace.http.request.errors{*}.as_count()"}
    ],
    "precision": 0
  }
}
```

### Top List

```json
{
  "definition": {
    "type": "toplist",
    "title": "Top Services by Errors",
    "requests": [
      {"q": "top(sum:trace.http.request.errors{*} by {service}.as_count(), 10, 'sum', 'desc')"}
    ]
  }
}
```

### Alert Value

```json
{
  "definition": {
    "type": "alert_value",
    "alert_id": "12345",
    "precision": 2,
    "title": "Alert Status"
  }
}
```

### Note (Markdown)

```json
{
  "definition": {
    "type": "note",
    "content": "## Dashboard Notes\n\nThis dashboard monitors production services.",
    "background_color": "white",
    "font_size": "14",
    "text_align": "left"
  }
}
```

## Layout Types

### Ordered Layout

Widgets are arranged automatically in a responsive grid.

```bash
datadog dashboards create --title "Auto Layout" --layout ordered --widgets '[...]'
```

### Free Layout

Widgets have explicit x, y, width, height positions.

```json
{
  "definition": {...},
  "layout": {
    "x": 0,
    "y": 0,
    "width": 6,
    "height": 3
  }
}
```

```bash
datadog dashboards create --title "Custom Layout" --layout free --widgets '[{"definition":{...},"layout":{"x":0,"y":0,"width":6,"height":3}}]'
```

## Common Workflows

### Clone a Dashboard

```bash
# Get original
datadog dashboards get --id "original-id" --output original.json

# Create new (modify title in the command)
cat original.json | jq '.dashboard.widgets' | datadog dashboards create \
  --title "Cloned Dashboard" \
  --layout ordered \
  --pretty
```

### Backup All Dashboards

```bash
# List all dashboard IDs
datadog dashboards list --count 1000 | jq -r '.dashboards[].id' > dashboard-ids.txt

# Backup each one
while read id; do
  datadog dashboards get --id "$id" --output "backups/${id}.json"
done < dashboard-ids.txt
```

### Create Dashboard from Template

```bash
# Define your template
TEMPLATE='[{"definition":{"type":"timeseries","requests":[{"q":"avg:system.cpu.user{service:SERVICE_NAME}"}]}}]'

# Replace placeholder and create
echo "$TEMPLATE" | sed 's/SERVICE_NAME/my-api/g' | \
  datadog dashboards create --title "my-api Dashboard" --layout ordered
```

## Troubleshooting

### "Missing required flags" Error

The `create` and `update` commands require `--title`, `--layout`, and widgets (via `--widgets` or stdin).

```bash
# Wrong - missing widgets
datadog dashboards create --title "Test" --layout ordered

# Correct - with empty widgets array
datadog dashboards create --title "Test" --layout ordered --widgets '[]'
```

### Template Variables Not Working

1. Ensure variable names match the prefix in your tags
2. Use `$variable_name` syntax in queries
3. Check that the tag exists in your metrics/logs

```bash
# Verify tags exist
datadog logs agg --query "*" --facet env --from 1h --pretty
```

### Widgets Not Rendering

1. Verify the widget definition structure
2. Check that metrics/queries are valid
3. Test queries independently first

```bash
# Test the query first
datadog metrics query --query "avg:system.cpu.user{*}" --from 1h --pretty
```
