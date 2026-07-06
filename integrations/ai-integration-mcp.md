---
description: >-
  Query your enrollment data in natural language — connect Claude or any MCP
  client to the Autopilot Monitor MCP server.
---

# AI Integration (MCP)

Autopilot Monitor exposes a **Model Context Protocol (MCP)** server that lets AI assistants query enrollment data conversationally: *"show me all failed enrollments from the last 24 hours"*, *"why did session X fail?"*, *"which devices are affected by CVE-2024-30078?"*. Connect Claude Desktop, VS Code with Claude, or any MCP client supporting Streamable HTTP.

## How it works

```
AI client  →  MCP server  →  Backend API  →  Your data
```

Your existing sign-in token is forwarded with every request — the MCP server stores no credentials, and all data access is scoped to your tenant exactly like in the portal.

## Prerequisites

1. **MCP access for your account** — enabled by an administrator (access is managed per user). Tenant admins can see usage under **Configuration → Reporting → MCP Usage**.
2. **An MCP-compatible client** — Claude Desktop, VS Code with the Claude extension, or anything speaking Streamable HTTP with OAuth.

## Client setup

**Server URL:**

```
https://mcp.autopilotmonitor.com/mcp
```

* **Claude Desktop:** Settings → MCP Servers → Add, enter the URL. OAuth authentication runs automatically in the browser.
* **VS Code (Claude extension):** add to `.vscode/mcp.json` or user settings:

```json
{
  "servers": {
    "autopilot-monitor": {
      "type": "http",
      "url": "https://mcp.autopilotmonitor.com/mcp"
    }
  }
}
```

**Verify:** ask your assistant *"List all available tools from Autopilot Monitor"* — you should see 20+ tools. If authentication fails, your MCP access probably isn't enabled yet.

## Available tools

| Category | Tools |
| --- | --- |
| **Search & Discovery** | `search_sessions` (by status, device properties, serial, model, OS, location…) · `search_sessions_by_event` · `search_sessions_by_cve` · `search_events` (hybrid keyword + semantic — finds "machine restarted unexpectedly" without literal word overlap) · `search_knowledge` (semantic search over your rules and IME patterns) |
| **Session Analysis** | `get_session_summary` (the best starting point: overview, key events, rule analysis, stats) · `get_session` · `get_session_events` |
| **Metrics & Observability** | `get_metrics` · `get_app_install_metrics` (incl. Delivery Optimization rollup) · `get_geographic_metrics` / `get_geographic_sessions` · `get_vulnerability_summary` · `get_rule_stats` · `get_ime_version_history` · `get_usage_metrics` |
| **Inventory & Audit** | `get_software_inventory` · `get_audit_logs` |
| **Raw Data** | `query_raw_events` · `query_raw_sessions` · `get_resource` (discovery catalogs) |

Two **discovery resources** help the assistant use the right vocabulary: `event_types` (every event type string, by category) and `device_properties` (dot-notation property keys like `tpm_status.specVersion` or `hardware_spec.ramTotalGB`).

## Example prompts

* *"Show me all failed enrollments from the last 24 hours"*
* *"Summarize session abc-123 and suggest fixes"*
* *"Which apps cause the most timeouts during enrollment?"*
* *"Compare enrollment performance between Germany and the US"*
* *"Find enrollments with BitLocker issues"*
* *"Which devices are affected by CVE-2024-30078?"*
* *"How has the failure rate changed this week?"*

The assistant picks the right tools and chains them — e.g. finding a session by device name first, then pulling its event timeline.

## Rate limits

60 requests per minute per user (sliding window). Exceeding it returns HTTP 429 with `retryAfterSeconds`; clients typically retry automatically.
