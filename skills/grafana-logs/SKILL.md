---
name: grafana-logs
description: >
  Query Grafana/Loki logs for Fuse Finance services. Use this skill when the user asks about
  production errors, log investigation, debugging service issues, checking error rates,
  or reviewing application logs. Trigger when the user mentions "grafana", "logs", "errors in production",
  "check logs for", "what errors", "investigate errors", or asks about specific service/client error patterns.
  Requires the Claude in Chrome MCP extension to be connected.
---

# Grafana Logs Investigation Skill

You have access to Grafana at `https://grafana.monitoring.fusefinance.com` via the Claude in Chrome browser extension.

## Prerequisites

The Claude in Chrome extension MUST be connected. Verify with:
```
mcp__Claude_in_Chrome__tabs_context_mcp(createIfEmpty: true)
```

If you get "No Chrome extension connected", tell the user to connect the extension first.

## Architecture Reference

- **Grafana URL**: `https://grafana.monitoring.fusefinance.com`
- **Loki datasource UID**: `loki`
- **Prometheus datasource UID**: `prometheus`
- **Tempo datasource UID**: `tempo`

### Loki Label Schema

Logs are indexed with these labels (NOT `namespace` or `pod`):

| Label | Description | Examples |
|-------|-------------|----------|
| `client` | Client identifier | `culs`, `acme`, `demo`, `navigant` |
| `env` | Environment (usually `{client}-production`) | `culs-production`, `dev-sandbox` |
| `service` | Service name | `data-builder-api`, `los-core-api` |
| `service_name` | Same as service | `data-builder-api` |
| `job` | Log collection job | `kubernetes` |
| `type` | Log category | `error`, `info` |
| `detected_level` | Auto-detected log level | `error`, `warn`, `info` |

## How to Investigate Logs

### Step 1: Determine the query parameters

From the user's request, identify:
- **client**: The customer/tenant identifier (e.g., `culs`, `navigant`)
- **service**: The microservice name (e.g., `data-builder-api`)
- **environment**: Usually `{client}-production` or `{client}-sandbox`
- **time range**: How far back to look (default: last 2 hours)
- **filter**: Error type, keyword, HTTP status, etc.

### Step 2: Use the Loki API to query full log content

**IMPORTANT**: Always use the Loki API directly for full log content. The Grafana dashboard UI truncates log messages.

#### API endpoint pattern:
```
https://grafana.monitoring.fusefinance.com/api/datasources/proxy/uid/loki/loki/api/v1/query_range?query={LOGQL_QUERY}&limit=N&since=TIME
```

#### Query construction:

**Basic error query:**
```
{client="CLIENT",service="SERVICE",env="ENVIRONMENT"}|="ERROR"
```

**Filter by keyword:**
```
{client="CLIENT",service="SERVICE",env="ENVIRONMENT"}|="KEYWORD"
```

**Filter by multiple keywords (OR):**
```
{client="CLIENT",service="SERVICE",env="ENVIRONMENT"}|~"keyword1|keyword2"
```

**Filter by HTTP status:**
```
{client="CLIENT",service="SERVICE",env="ENVIRONMENT"}|~"status.*400|status.*500"
```

**Combine line filter + regex:**
```
{client="CLIENT",service="SERVICE",env="ENVIRONMENT"}|="report-builder"|~"400|GROUP BY|ungrouped"
```

#### Time range options:
- Use `&since=2h` for relative time ranges (e.g., `1h`, `3h`, `24h`, `7d`)
- Use `&start=NANOSECONDS&end=NANOSECONDS` for absolute time ranges
- To convert dates to nanoseconds, use python3:
  ```bash
  python3 -c "import datetime; print(int(datetime.datetime(2026, 2, 27, 0, 0, 0).timestamp() * 1e9))"
  ```

#### Limit:
- Use `&limit=5` to `&limit=50` depending on how many entries you need
- Start small (5-10) and increase if needed

### Step 3: Navigate and read the response

1. Use `mcp__Claude_in_Chrome__navigate` to load the API URL
2. Wait 3-4 seconds for the response: `mcp__Claude_in_Chrome__computer(action: "wait", duration: 3)`
3. Read the response with `mcp__Claude_in_Chrome__get_page_text`
4. Parse the JSON response — logs are in `data.result[].values[]`

### Step 4: Analyze the logs

The log entries in the JSON response contain:
- **Timestamp** (nanoseconds since epoch) as the first element
- **Log message** (JSON string) as the second element

Each log message typically includes:
- `status`: HTTP status code
- `path`: API endpoint that was hit
- `method`: HTTP method
- `error.message`: The error description
- `error.stack`: Stack trace
- `request.body`: The full request payload (very useful for debugging report configs, etc.)
- `labels.controller`: Which controller handled the request
- `labels.method`: Which method in the controller

### Step 5: Present findings

Organize your findings by:
1. **Error categories** — Group similar errors together
2. **Frequency** — Which errors happen most
3. **Timeline** — When did errors start/spike
4. **Request context** — What was the user trying to do (from request body)
5. **Root cause** — If identifiable from the error message/stack trace

### Step 6: Offer the Explore UI link

After presenting your analysis, **always offer the user a link to explore the logs themselves**
in the Grafana Loki Explore UI. Build the URL using this pattern:

```
https://grafana.monitoring.fusefinance.com/explore?schemaVersion=1&panes=%7B%22one%22:%7B%22datasource%22:%22loki%22,%22queries%22:%5B%7B%22refId%22:%22A%22,%22expr%22:%22URL_ENCODED_LOGQL%22,%22queryType%22:%22range%22,%22datasource%22:%7B%22type%22:%22loki%22,%22uid%22:%22loki%22%7D%7D%5D,%22range%22:%7B%22from%22:%22FROM%22,%22to%22:%22TO%22%7D%7D%7D&orgId=1
```

Where:
- `URL_ENCODED_LOGQL` is the LogQL query with special characters URL-encoded
  - `{` → `%7B`, `}` → `%7D`, `"` → `%5C%22`, `=` → `%3D`, `|` → `%7C`, `,` → `%2C`, space → `+`
- `FROM` and `TO` are relative times like `now-2h` and `now`

**Example** — For the query `{client="culs",service="data-builder-api",env="culs-production"}|="report-builder"|~"400|GROUP BY|ungrouped"`:
```
https://grafana.monitoring.fusefinance.com/explore?schemaVersion=1&panes=%7B%22one%22:%7B%22datasource%22:%22loki%22,%22queries%22:%5B%7B%22refId%22:%22A%22,%22expr%22:%22%7Bclient%3D%5C%22culs%5C%22%2Cservice%3D%5C%22data-builder-api%5C%22%2Cenv%3D%5C%22culs-production%5C%22%7D%7C%3D%5C%22report-builder%5C%22%7C~%5C%22400%7CGROUP+BY%7Cungrouped%5C%22%22,%22queryType%22:%22range%22,%22datasource%22:%7B%22type%22:%22loki%22,%22uid%22:%22loki%22%7D%7D%5D,%22range%22:%7B%22from%22:%22now-3h%22,%22to%22:%22now%22%7D%7D%7D&orgId=1
```

Present it like:
> "Would you like to explore these logs in the Grafana Loki UI? Here's the direct link: [URL]"

Also provide the raw Loki API URL for programmatic access:
> "Or the raw Loki API query: [API URL]"

## Quick Reference: Common Queries

### All errors for a service (last 2h):
```
{client="CLIENT",service="SERVICE",env="CLIENT-production"}|="ERROR"&limit=20&since=2h
```

### Specific HTTP status errors:
```
{client="CLIENT",service="SERVICE",env="CLIENT-production"}|~"status.:403|status.:401"&limit=20&since=2h
```

### Search by endpoint path:
```
{client="CLIENT",service="SERVICE",env="CLIENT-production"}|="report-builder"&limit=10&since=2h
```

### Count errors (use Loki instant query):
```
https://grafana.monitoring.fusefinance.com/api/datasources/proxy/uid/loki/loki/api/v1/query?query=count_over_time({client="CLIENT",service="SERVICE",env="CLIENT-production"}|="ERROR"[2h])
```

## Using the Grafana Dashboard UI

For a visual overview, you can also navigate to dashboards:

### Logs dashboard:
```
https://grafana.monitoring.fusefinance.com/d/DASHBOARD_ID/logs?orgId=1&from=now-2h&to=now&var-Client=CLIENT&var-Service=SERVICE&var-LogFilter=ERROR&var-env=CLIENT-production
```

**Note**: Dashboard UI truncates log messages. Always follow up with the Loki API for full log details.

## Available Loki Labels (for discovery)

To list available values for any label:
```
https://grafana.monitoring.fusefinance.com/api/datasources/proxy/uid/loki/loki/api/v1/label/LABEL_NAME/values
```

For example:
- List all clients: `.../label/client/values`
- List all environments: `.../label/env/values`
- List all services: `.../label/service/values`
