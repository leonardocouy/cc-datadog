# Dashboards Reference

## ⚠️ CRITICAL: Dashboard Update is DESTRUCTIVE

**The `dashboards update` command REPLACES the entire dashboard, not just the fields you specify.**

If you omit any of these fields during an update, they will be **permanently deleted**:
- `--template-variables` → Template variables will be removed
- `--description` → Description will be cleared
- `--notify-list` → Notify list will be cleared

**Example of DATA LOSS:**
```bash
# This will DELETE template variables and description!
npx @leoflores/datadog-cli dashboards update \
  --id "abc-def-ghi" \
  --title "My Dashboard" \
  --layout ordered \
  --widgets '[...]'  # Only widgets provided, other fields wiped!
```

## Safe Dashboard Update Workflow

**ALWAYS follow this 3-step process when updating dashboards:**

### Step 1: Backup the Current Dashboard
```bash
# Save the current dashboard state BEFORE any changes
npx @leoflores/datadog-cli dashboards get --id "abc-def-ghi" > dashboard-backup.json
```

### Step 2: Modify and Preserve All Fields
```bash
# Extract existing values and modify only what you need
DASHBOARD_JSON=$(npx @leoflores/datadog-cli dashboards get --id "abc-def-ghi")

# Get existing template variables (MUST be preserved!)
TEMPLATE_VARS=$(echo "$DASHBOARD_JSON" | jq -c '.dashboard.templateVariables // []')

# Get existing description (MUST be preserved!)
DESCRIPTION=$(echo "$DASHBOARD_JSON" | jq -r '.dashboard.description // ""')

# Modify widgets (example: change title of widget at index 1)
WIDGETS=$(echo "$DASHBOARD_JSON" | jq -c '.dashboard.widgets | .[1].definition.title = "New Title"')

# Update with ALL fields preserved
npx @leoflores/datadog-cli dashboards update \
  --id "abc-def-ghi" \
  --title "My Dashboard" \
  --layout ordered \
  --widgets "$WIDGETS" \
  --description "$DESCRIPTION" \
  --template-variables "$TEMPLATE_VARS" \
  --pretty
```

### Step 3: Verify the Update
```bash
# Confirm all fields are intact
npx @leoflores/datadog-cli dashboards get --id "abc-def-ghi" --pretty
```

### Recovery from Accidental Data Loss
```bash
# If you have a backup, restore from it
BACKUP=$(cat dashboard-backup.json)
WIDGETS=$(echo "$BACKUP" | jq -c '.dashboard.widgets')
TEMPLATE_VARS=$(echo "$BACKUP" | jq -c '.dashboard.templateVariables // []')
DESCRIPTION=$(echo "$BACKUP" | jq -r '.dashboard.description // ""')
TITLE=$(echo "$BACKUP" | jq -r '.dashboard.title')
LAYOUT=$(echo "$BACKUP" | jq -r '.dashboard.layoutType')

npx @leoflores/datadog-cli dashboards update \
  --id "abc-def-ghi" \
  --title "$TITLE" \
  --layout "$LAYOUT" \
  --widgets "$WIDGETS" \
  --description "$DESCRIPTION" \
  --template-variables "$TEMPLATE_VARS" \
  --pretty
```

## Commands Overview

| Command | Description |
|---------|-------------|
| `dashboards list` | List all dashboards |
| `dashboards get` | Get full dashboard definition |
| `dashboards create` | Create a new dashboard |
| `dashboards update` | ⚠️ **DESTRUCTIVE** - Replaces entire dashboard |
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
| `--description` | create, update | ⚠️ **Required on update to preserve** |
| `--template-variables` | create, update | ⚠️ **Required on update to preserve** |
| `--notify-list` | create, update | ⚠️ **Required on update to preserve** |
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
