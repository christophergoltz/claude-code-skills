# Grafana Query Reference — Template

Adapt these queries to your project by replacing placeholder labels with your actual
Grafana label names and values. Check CLAUDE.md for project-specific label mappings.

## Discovering Labels

Before using queries, discover available labels in your environment:

```
# Loki labels
list_loki_label_names(datasourceUid)

# Prometheus labels
list_prometheus_label_names(datasourceUid)
```

Common label patterns:
- `project`, `app`, `namespace` — top-level grouping
- `service`, `job`, `container` — individual service
- `level`, `severity` — log levels

## LogQL Queries

### Error Logs (all services)
```logql
{<project_label>="<project_value>"} | json | level="error"
```

### Errors for a specific service
```logql
{<project_label>="<project_value>", <service_label>="<service_name>"} | json | level="error"
```

### Fatal + Error combined
```logql
{<project_label>="<project_value>"} | json | level=~"error|fatal"
```

### Filter by message content
```logql
{<project_label>="<project_value>"} | json | level="error" |~ "NullReference|timeout|deadlock"
```

### Exceptions with stack traces
```logql
{<project_label>="<project_value>"} | json | level="error" |~ "Exception"
```

### Filter by TraceId (for request tracing)
```logql
{<project_label>="<project_value>"} | json | TraceId="<traceId>"
```

### Log volume over time (errors per minute)
```logql
sum(count_over_time({<project_label>="<project_value>"} | json | level="error" [1m]))
```

### Log volume by service
```logql
sum by (<service_label>) (count_over_time({<project_label>="<project_value>"} | json | level="error" [5m]))
```

## PromQL Queries

### Service Health

```promql
# All scrape targets — 1 = up, 0 = down
up

# Specific job
up{job="<service_name>"}
```

### HTTP Latency

```promql
# P95 latency per job (last 5 min)
histogram_quantile(0.95, sum(rate(http_server_requests_seconds_bucket[5m])) by (le, job))

# P95 latency per route
histogram_quantile(0.95, sum(rate(http_server_requests_seconds_bucket[5m])) by (le, route))

# P99 latency
histogram_quantile(0.99, sum(rate(http_server_requests_seconds_bucket[5m])) by (le, job))
```

Note: The metric name may vary (`http_server_request_duration_seconds_bucket`,
`http_request_duration_seconds_bucket`, etc.). Use `list_prometheus_metric_names`
to find the correct one.

### HTTP Error Rate

```promql
# Error rate (5xx) per job as percentage
sum(rate(http_server_requests_seconds_count{status=~"5.."}[5m])) by (job)
/
sum(rate(http_server_requests_seconds_count[5m])) by (job)
* 100

# Request rate by status code
sum(rate(http_server_requests_seconds_count[5m])) by (job, status)
```

### Memory

```promql
# Container memory usage (bytes)
container_memory_usage_bytes{name=~"<container_pattern>"}

# Container memory as % of limit
(container_memory_usage_bytes{name=~"<container_pattern>"} / container_spec_memory_limit_bytes{name=~"<container_pattern>"}) * 100

# Host memory usage %
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100
```

### CPU

```promql
# Host CPU usage %
100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Container CPU usage
rate(container_cpu_usage_seconds_total{name=~"<container_pattern>"}[5m])
```

### Database (PostgreSQL example)

```promql
# Active connections
pg_stat_activity_count

# Connections vs max
pg_stat_activity_count / pg_settings_max_connections * 100
```

Note: Database metrics depend on the exporter used. Use `list_prometheus_metric_names`
with a regex like `pg_|mysql_|mongo_` to find available database metrics.

### Disk

```promql
# Free disk space %
(node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100

# Disk space prediction (hours until full)
predict_linear(node_filesystem_avail_bytes{mountpoint="/"}[6h], 24*60*60)
```

### Container Health

```promql
# Container restarts (increase in last 5 min)
increase(container_start_time_seconds{name=~"<container_pattern>"}[5m])

# OOM kills (increase in last 5 min)
increase(container_oom_events_total{name=~"<container_pattern>"}[5m])

# Container uptime (seconds since start)
time() - container_start_time_seconds{name=~"<container_pattern>"}
```

## Common Alert Rule Patterns

These are typical alert rules. Check your actual alert configuration for specifics.

| Alert | Typical Expression | Severity |
|-------|-----------|----------|
| DiskSpaceWarning | Free disk < 30% | warning |
| DiskSpaceCritical | Free disk < 15% | critical |
| HighMemoryUsage | Memory > 85% | warning |
| HighCPUUsage | CPU > 90% for 5m | warning |
| ServiceDown | up == 0 for 1m | critical |
| HighErrorRate | 5xx rate > 5% for 5m | warning |
| DatabaseConnectionsHigh | Connections > 80% max | warning |
| ContainerOOMKilled | OOM events > 0 | critical |
