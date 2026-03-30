# grafana-logs

> Queries Grafana/Loki logs for Fuse Finance services and presents categorized error analysis with direct explore links.

## What it does

Connects to the Fuse Finance Grafana instance at `grafana.monitoring.fusefinance.com` via the Claude in Chrome browser extension and queries Loki directly through its API — bypassing the dashboard UI, which truncates log messages. Given a client, service, environment, time range, and optional filter keywords, it builds a LogQL query, fetches results, and parses the JSON response to extract timestamps, HTTP status codes, endpoint paths, error messages, stack traces, and request bodies. Findings are organized by error category, frequency, timeline, and root cause. Every analysis concludes with a pre-built Grafana Loki Explore UI link and a raw Loki API URL so the user can drill further themselves.

## How to use it

Auto-triggers when the user mentions "grafana", "logs", "errors in production", "check logs for", "what errors", or asks about specific service/client error patterns.

Example prompts:
- "Check logs for the culs data-builder-api in the last 2 hours"
- "What 500 errors are hitting los-core-api in production?"
- "Investigate the report-builder errors for navigant"

Requires the Claude in Chrome MCP extension to be connected before use. The skill supports filtering by keyword, HTTP status code, endpoint path, and combined conditions using LogQL syntax.

## Why it's useful

Investigating production errors across a multi-tenant, multi-service architecture normally means navigating multiple Grafana dashboards, constructing LogQL by hand, and dealing with truncated UI output. This skill collapses that workflow into a single conversational query and returns a structured, readable summary with root cause context — shaving minutes off every debugging session.
