---
name: investigate
description: >
  Investigate issues in staging or local environments using Grafana (Prometheus + Loki).
  Use when the user says "investigate", "check staging", "check logs", "service down",
  "why errors", or describes symptoms like errors, slowness, or outages.
argument-hint: "[optional: symptom or service, e.g. 'api 500 errors']"
allowed-tools: Read, Glob, Grep, Bash(git log*), ToolSearch, mcp__grafana-local__*, mcp__grafana-stage__*
---

# Investigate Skill

You are investigating issues in the project's infrastructure using Grafana's Prometheus (metrics)
and Loki (logs) via MCP tools. This is a **read-only diagnostic** session.

Read the project's CLAUDE.md to understand:
- Service names and labels used in Grafana
- Available environments (local, staging, production)
- How to access each environment (SSH tunnels, direct access, etc.)
- Alert rules and dashboards

**Communicate in the user's language.** Detect from their messages and respond accordingly.

## CLAUDE.md Requirements

Before starting, read CLAUDE.md and verify these are documented. If **required** info is
missing, use `AskUserQuestion` to ask the user to provide it before proceeding.

| Info | Required | Used for |
|------|----------|----------|
| Grafana MCP server | **Yes** | Querying metrics and logs (tool must be configured) |
| Service names/labels | Recommended | Filtering logs and metrics by service |
| Environment access | Recommended | Knowing which environment to target, SSH tunnel setup |
| Alert rules | Recommended | Understanding what alerts mean and their thresholds |

## Important Principles

1. **Read-only**: NEVER make any changes — no restarts, no config edits, no fixes. Only observe
   and report.
2. **Evidence-based**: Every finding MUST include concrete evidence (metric value, log line,
   timestamp). Never make claims without data.
3. **Environment awareness**: Default to the primary environment the user mentions. If unclear,
   ask which environment to investigate. Use appropriate MCP tools for the target environment.
4. **Time window**: Always state the time window of your investigation in the report.

## Reference

Read `queries-template.md` in this skill's directory for PromQL/LogQL reference queries.
Adapt the label names and values to match the project's actual Grafana setup (check CLAUDE.md
for project-specific labels).

## Process

### Step 1: Load Grafana MCP Tools

Use ToolSearch to load the Grafana MCP tools for the target environment:

```
ToolSearch: "+grafana"
```

If the tools are not available, inform the user clearly what's needed (e.g., SSH tunnel,
Docker monitoring stack) and stop.

### Step 2: Triage — Check Active Alerts

Use `list_alert_groups` to check for any currently firing alerts.

If alerts are firing, they guide the investigation. If no alerts are firing, proceed with
the user's described symptoms.

### Step 3: Service Health

Check if all services are up using the `up` metric:

```promql
up
```

This shows all scrape targets and their status (1 = up, 0 = down).

### Step 4: Error Logs

Query Loki for recent errors (last 1 hour by default, adjust if user specifies a time range).

Use the project's Loki labels (check CLAUDE.md or use `list_loki_label_names` to discover them):

```logql
{<project_label>} | json | level="error"
```

If the user mentioned a specific service, filter accordingly.

### Step 5: Key Metrics (based on symptoms)

Depending on the symptoms or alerts, check relevant metrics. See `queries-template.md` for
common queries. Adapt to the project's metric names.

| Symptom | Metrics to Check |
|---------|-----------------|
| Slow responses | HTTP P95 latency, DB connection pool |
| 500 errors | HTTP error rate, exception logs |
| Service down | `up` metric, container restarts, OOM events |
| High resource usage | Memory usage per container, CPU usage |
| DB issues | Database connections, query duration |
| Disk alerts | Filesystem available bytes |

### Step 6: Deep Dive

If initial checks reveal issues, dig deeper:

- **Specific endpoints**: Filter HTTP metrics by route/method
- **Container restarts**: Check start time changes
- **OOM kills**: Check OOM event counters
- **Dashboard details**: Use `search_dashboards` and `get_dashboard_by_uid` for pre-built views
- **Panel images**: Use `get_panel_image` to capture visual evidence from dashboards

### Step 7: Correlation with Deployments (optional)

If issues seem time-correlated, check recent deployments:

```bash
git log --oneline --since="2 hours ago" --all
```

This can reveal if a recent deployment introduced the issue.

### Step 8: Present Report

Format the output as a structured investigation report:

```
## Investigation Report: {Environment} — {Date/Time}

**Time window:** {start} — {end}
**Trigger:** {What prompted the investigation}

### Active Alerts
{List of firing alerts, or "None"}

### Findings

#### 1. {Finding title} — Severity: {Critical|Warning|Info}
**Evidence:** {Concrete metric value, log line, or screenshot}
**Impact:** {What this means for users/system}

#### 2. {Finding title} — Severity: {Critical|Warning|Info}
**Evidence:** ...
**Impact:** ...

### Root Cause Analysis
{Best assessment of what's causing the issues, based on evidence}

### Recommendations
1. {Actionable recommendation with priority}
2. {Next recommendation}

### What's Healthy
{Brief note on what's working fine — gives confidence in scope of the issue}
```

If no issues are found, say so clearly and confirm that the system appears healthy.

## Severity Guidelines

| Severity | Criteria |
|----------|----------|
| **Critical** | Service down, data loss risk, OOM kills, disk full |
| **Warning** | High error rate, elevated latency, resource usage >85%, approaching limits |
| **Info** | Minor anomalies, slightly elevated metrics, non-impacting patterns |

## Rules

- NEVER modify any files, configs, containers, or services.
- NEVER run `docker restart`, `docker compose`, or any write operations.
- NEVER guess at metrics or logs — always query the actual data.
- ALWAYS load MCP tools via ToolSearch before attempting to use them.
- ALWAYS include the investigation time window in the report.
- If MCP tools are unreachable, inform the user and stop — do not fabricate data.
