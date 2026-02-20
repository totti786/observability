# Observability Stack

A comprehensive, production-ready observability platform built on Prometheus, Grafana, and Loki. Designed for infrastructure teams who need deep visibility into their systems with minimal operational overhead.

## Features

- **Metrics Collection**: Prometheus with automatic service discovery
- **Visualization**: Pre-built Grafana dashboards for common use cases
- **Log Aggregation**: Loki with Promtail for efficient log collection
- **Alerting**: AlertManager with multi-channel notifications
- **Service Discovery**: Automatic target discovery for Kubernetes, Docker, and static targets
- **High Availability**: Supports clustered deployments
- **Long-term Storage**: Integration with Thanos or Cortex

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          Observability Platform                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐                   │
│  │   Sources   │────▶│   Collect   │────▶│    Store    │                   │
│  └─────────────┘     └─────────────┘     └─────────────┘                   │
│       │                    │                    │                          │
│       ▼                    ▼                    ▼                          │
│  ┌──────────┐       ┌─────────────┐     ┌─────────────┐                   │
│  │ Nodes    │       │ Prometheus  │     │  TSDB       │                   │
│  │ Containers│      │ Promtail    │     │  Loki       │                   │
│  │ Apps     │       │ Exporters   │     │             │                   │
│  │ Services │       │             │     │             │                   │
│  └──────────┘       └─────────────┘     └─────────────┘                   │
│                                                │                           │
│                                                ▼                           │
│                                         ┌─────────────┐                    │
│                                         │  Visualize  │                    │
│                                         └─────────────┘                    │
│                                                │                           │
│                                                ▼                           │
│                                         ┌─────────────┐                    │
│                                         │   Grafana   │                    │
│                                         │  Dashboards │                    │
│                                         └─────────────┘                    │
│                                                │                           │
│                                                ▼                           │
│                                         ┌─────────────┐                    │
│                                         │   Alert     │                    │
│                                         └─────────────┘                    │
│                                                │                           │
│                                                ▼                           │
│                                    ┌─────────────────────┐                │
│                                    │   AlertManager      │                │
│                                    │   Slack/PagerDuty   │                │
│                                    │   Email/Webhook     │                │
│                                    └─────────────────────┘                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Quick Start

### Prerequisites

- Docker and Docker Compose
- 4GB RAM minimum (8GB recommended)
- Ports 9090, 3000, 3100, 9093 available

### 1. Deploy Stack

```bash
git clone https://github.com/Totti786/observability-stack.git
cd observability-stack

# Start all services
docker-compose up -d

# Check status
docker-compose ps
```

### 2. Access Dashboards

| Service | URL | Default Credentials |
|---------|-----|-------------------|
| Grafana | http://localhost:3000 | admin/admin |
| Prometheus | http://localhost:9090 | - |
| AlertManager | http://localhost:9093 | - |
| Loki | http://localhost:3100 | - |

### 3. Import Pre-built Dashboards

Pre-built dashboards are automatically provisioned:

- Infrastructure Overview
- Kubernetes Cluster
- Node Metrics
- Container Metrics
- Nginx Metrics
- PostgreSQL Metrics
- Redis Metrics
- Application Performance

## Project Structure

```
observability-stack/
├── prometheus/
│   ├── prometheus.yml      # Main configuration
│   ├── rules/              # Alert rules
│   └── targets/            # Static targets
├── grafana/
│   ├── dashboards/         # Dashboard JSON files
│   ├── provisioning/       # Auto-provisioning config
│   └── grafana.ini         # Grafana configuration
├── loki/
│   └── loki-config.yml     # Loki configuration
├── promtail/
│   └── promtail-config.yml # Log collection config
├── alertmanager/
│   └── alertmanager.yml    # Alert routing
├── docker-compose.yml
└── scripts/
    ├── health-check.sh
    └── backup.sh
```

## Alert Rules

### Critical Alerts

| Alert | Condition | Severity |
|-------|-----------|----------|
| InstanceDown | Node unreachable for 1m | critical |
| HighCPUUsage | CPU > 80% for 5m | warning |
| HighMemoryUsage | Memory > 85% for 5m | warning |
| DiskSpaceLow | Disk > 85% for 5m | warning |
| ContainerRestarting | Container restart > 5 in 10m | warning |

### Application Alerts

| Alert | Condition | Severity |
|-------|-----------|----------|
| HighErrorRate | Error rate > 5% for 5m | critical |
| HighLatency | P99 latency > 1s for 5m | warning |
| LowThroughput | RPS < 100 for 5m | info |

## Configuration

### Prometheus

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: 'production'
    region: 'us-east-1'

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

rule_files:
  - /etc/prometheus/rules/*.yml

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  # ... more jobs
```

### AlertManager Routes

```yaml
route:
  receiver: 'default'
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  routes:
    - match:
        severity: critical
      receiver: 'pagerduty'
    - match:
        severity: warning
      receiver: 'slack'
```

### Grafana Data Sources

Data sources are auto-provisioned:

```yaml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    url: http://prometheus:9090
    access: proxy
    isDefault: true
  - name: Loki
    type: loki
    url: http://loki:3100
    access: proxy
```

## Supported Exporters

### Infrastructure

| Exporter | Port | Purpose |
|----------|------|---------|
| node-exporter | 9100 | Node metrics |
| cadvisor | 8080 | Container metrics |
| blackbox-exporter | 9115 | Endpoint probing |

### Databases

| Exporter | Port | Purpose |
|----------|------|---------|
| postgres-exporter | 9187 | PostgreSQL metrics |
| redis-exporter | 9121 | Redis metrics |
| mysqld-exporter | 9104 | MySQL metrics |

### Web Servers

| Exporter | Port | Purpose |
|----------|------|---------|
| nginx-exporter | 9113 | Nginx metrics |
| apache-exporter | 9117 | Apache metrics |

### Application

| Exporter | Port | Purpose |
|----------|------|---------|
| jmx-exporter | 9404 | Java/JMX metrics |
| statsd-exporter | 9102 | StatsD metrics |

## Dashboards

### Infrastructure Dashboard

Key metrics displayed:
- CPU, Memory, Disk usage per node
- Network I/O
- Load average
- System processes

### Application Dashboard

Key metrics displayed:
- Request rate (RPS)
- Error rate (%)
- Latency (P50, P95, P99)
- Throughput
- Active connections

### Kubernetes Dashboard

Key metrics displayed:
- Pod status
- Container resource usage
- Deployment health
- Node capacity

## Alerting Channels

### Slack Integration

```yaml
receivers:
  - name: 'slack'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/...'
        channel: '#alerts'
        send_resolved: true
```

### PagerDuty Integration

```yaml
receivers:
  - name: 'pagerduty'
    pagerduty_configs:
      - service_key: '<your-service-key>'
        severity: critical
```

### Email Integration

```yaml
receivers:
  - name: 'email'
    email_configs:
      - to: 'ops@example.com'
        from: 'alertmanager@example.com'
        smarthost: 'smtp.example.com:587'
```

## Maintenance

### Backup Prometheus Data

```bash
./scripts/backup.sh prometheus
```

### Backup Grafana Dashboards

```bash
./scripts/backup.sh grafana
```

### Health Check

```bash
./scripts/health-check.sh
```

## Scaling

### High Availability

For production deployments, consider:

1. **Prometheus HA**: Run multiple replicas with identical config
2. **AlertManager Cluster**: Use mesh gossip protocol
3. **Loki SimpleScalable**: Deploy read/write path separately

### Long-term Storage

For metrics retention beyond 15 days:

1. **Thanos**: Query multiple Prometheus instances
2. **Cortex**: Multi-tenant, long-term storage
3. **VictoriaMetrics**: High-performance alternative

## Troubleshooting

### Prometheus Targets Down

```bash
# Check target status
curl http://localhost:9090/api/v1/targets | jq '.data.activeTargets[] | select(.health != "up")'

# Check network connectivity
kubectl exec -it prometheus-pod -- wget -qO- http://target:port/metrics
```

### Loki Not Receiving Logs

```bash
# Check Promtail status
docker logs promtail

# Verify Loki ingestion
curl -G 'http://localhost:3100/loki/api/v1/query' --data-urlencode 'query={job="varlogs"}'
```

### Grafana Dashboard Issues

```bash
# Check data source connectivity
curl http://localhost:3000/api/datasources/proxy/1/api/v1/query?query=up
```

## Metrics

### Key SLO Metrics

| Metric | Target |
|--------|--------|
| Availability | 99.9% |
| MTTR | < 15 minutes |
| MTTF | > 30 days |
| Alert Accuracy | > 95% |

### Resource Requirements

| Service | CPU | Memory | Storage |
|---------|-----|--------|---------|
| Prometheus | 1 core | 2GB | 50GB |
| Grafana | 0.5 core | 512MB | 1GB |
| Loki | 1 core | 2GB | 100GB |
| AlertManager | 0.25 core | 256MB | 1GB |

## Best Practices

1. **Label Consistently**: Use consistent labels across all metrics
2. **Set Retention Wisely**: Balance storage costs with query needs
3. **Alert Meaningfully**: Avoid alert fatigue with proper thresholds
4. **Document Dashboards**: Add descriptions to all dashboards
5. **Version Control**: Keep all configs in Git

## License

MIT License - see [LICENSE](LICENSE)

## Author

**Tarek Deshli**
- GitHub: [@Totti786](https://github.com/Totti786)
- LinkedIn: [Totti786](https://linkedin.com/in/Totti786)
