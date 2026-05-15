---
tags:
  - monitoring
  - alerting
  - observability
---

# Monitoring SOP

**Version:** 1.0  
**Last Updated:** 2024-01-01  
**Owner:** DevOps Team

## Purpose

Define standards for monitoring infrastructure, applications, and alerting to ensure system health and reliability.

## Monitoring Stack

| Component | Tool | Purpose |
|-----------|------|---------|
| Metrics | Prometheus | Time-series data collection |
| Visualization | Grafana | Dashboards and charts |
| Logging | Loki / ELK | Centralized log aggregation |
| Tracing | Jaeger | Distributed request tracing |
| Alerting | Alertmanager | Alert routing and notification |
| Uptime | Uptime Robot | External health checks |

## Alert Thresholds

### Infrastructure

| Metric | Warning | Critical | Description |
|--------|---------|----------|-------------|
| CPU Usage | > 80% for 5m | > 95% for 2m | Node CPU utilization |
| Memory | > 85% | > 95% | Node memory utilization |
| Disk | > 80% | > 90% | Root disk usage |
| Service status | Unhealthy | Down | Service health check |

### Application

| Metric | Warning | Critical | Description |
|--------|---------|----------|-------------|
| 5xx errors | > 1% for 5m | > 5% for 2m | Server errors |
| Latency p99 | > 500ms | > 2s | Request latency |
| Health check | — | 3 consecutive fails | Service health |

## Adding a New Alert

1. Define metric and thresholds in `prometheus/rules/`
2. Add alert to `alertmanager/config.yml` with proper routing
3. Create Grafana dashboard panel
4. Test alert fires correctly
5. Update this documentation

## Dashboard Management

### Dashboard Naming Convention

- `App / <service>` — Application-level dashboards
- `Infra / <component>` — Infrastructure dashboards
- `Host / <server>` — Server-level dashboards

### Dashboard Checklist

- [ ] Title and description
- [ ] Relevant panels with units
- [ ] Time range selector
- [ ] Proper templating variables
- [ ] Team annotations (deployments, incidents)

## On-Call Rotation

| Week | Primary | Secondary |
|------|---------|-----------|
| Week 1 | Alice | Bob |
| Week 2 | Bob | Charlie |
| Week 3 | Charlie | Alice |

!!! info "Escalation"
    If primary does not acknowledge within 5 minutes, secondary is paged. If both miss, the incident escalates to the DevOps lead.

## Verification

- [ ] All services have at least one dashboard
- [ ] Critical alerts have proper routing
- [ ] On-call schedule is current
- [ ] Backup monitoring has alerts
- [ ] Dashboard content is up to date
