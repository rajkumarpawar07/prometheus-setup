# Prometheus Monitoring Setup with Docker Compose

A complete monitoring stack using Prometheus, Node Exporter, and Alertmanager, all containerized with Docker Compose.

## Overview

This setup provides a comprehensive monitoring solution that includes:
- **Prometheus**: Time-series database and monitoring server
- **Node Exporter**: System metrics collector for hardware and OS metrics
- **Alertmanager**: Handles alerts sent by Prometheus server

## Architecture

```
<img width="1184" height="864" alt="Image_cwfcvocwfcvocwfc" src="https://github.com/user-attachments/assets/7eb24138-dd7c-44da-84e5-51921a976a6d" />

```

## Components

### Prometheus Server
- **Image**: `prom/prometheus`
- **Port**: 9090
- **Function**: Scrapes metrics from targets and stores time-series data
- **Configuration**: `prometheus.yml`

### Node Exporter
- **Image**: `prom/node-exporter`
- **Port**: 9100
- **Function**: Exposes hardware and OS metrics from the host system

### Alertmanager
- **Image**: `prom/alertmanager`
- **Port**: 9093
- **Function**: Manages alerts, including silencing, inhibition, and notification routing
- **Configuration**: `alertmanager.yml`

## Quick Start

### Prerequisites
- Docker installed on your system
- Docker Compose installed

### Running the Stack

1. **Clone or navigate to the project directory**
   ```bash
   cd prometheus-setup
   ```

2. **Start all services**
   ```bash
   docker-compose up -d
   ```

3. **Verify services are running**
   ```bash
   docker-compose ps
   ```

### Accessing the Services

- **Prometheus Web UI**: http://localhost:9090
- **Node Exporter Metrics**: http://localhost:9100/metrics
- **Alertmanager Web UI**: http://localhost:9093

## Configuration Files

### prometheus.yml
Main Prometheus configuration file that defines:
- Global scrape and evaluation intervals (10s)
- Scrape targets (Prometheus itself and Node Exporter)
- Alert rule files
- Alertmanager configuration

### rules.yml
Defines alerting rules:
- **InstanceDown**: Triggers when any monitored instance is down for more than 1 minute

### alertmanager.yml
Configures alert routing and notification channels:
- SMTP configuration for email notifications
- Route configuration for alert grouping
- Receiver definitions for notification destinations

## Monitoring Targets

The setup automatically monitors:
- **Prometheus server**: Self-monitoring on port 9090
- **Node Exporter**: System metrics on port 9100

## Alerting

### Available Alerts
- **InstanceDown**: Fired when `up == 0` for more than 1 minute

### Email Notifications
Configure email settings in `alertmanager.yml`:
```yaml
global:
  smtp_smarthost: 'your-smtp-server:587'
  smtp_from: 'prometheus@yourcompany.com'

receivers:
 - name: example-email
   email_configs:
    - to: 'admin@yourcompany.com'
      subject: 'Prometheus Alert: {{ .GroupLabels.alertname }}'
```

## Useful Commands

### Docker Compose Operations
```bash
# Start services
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs -f

# Restart specific service
docker-compose restart prometheus

# Check service status
docker-compose ps
```

### Prometheus Queries (Examples)
Access Prometheus at http://localhost:9090 and try these queries:

```promql
# CPU usage
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100

# Disk usage
100 - (node_filesystem_avail_bytes / node_filesystem_size_bytes * 100)

# Network traffic
rate(node_network_receive_bytes_total[5m])
```

## Troubleshooting

### Common Issues

1. **Port conflicts**: Ensure ports 9090, 9100, and 9093 are not in use
2. **Docker network issues**: Check if the `localprom` network is created properly
3. **Configuration errors**: Validate YAML syntax in configuration files

### Checking Logs
```bash
# View all logs
docker-compose logs

# View specific service logs
docker-compose logs prometheus
docker-compose logs alertmanager
docker-compose logs node-exporter
```

### Configuration Validation
```bash
# Validate Prometheus config
docker run --rm -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus:latest promtool check config /etc/prometheus/prometheus.yml

# Validate alert rules
docker run --rm -v $(pwd)/rules.yml:/etc/prometheus/rules.yml prom/prometheus:latest promtool check rules /etc/prometheus/rules.yml
```

## Customization

### Adding More Targets
Edit `prometheus.yml` to add more scrape targets:
```yaml
scrape_configs:
 - job_name: 'my-application'
   static_configs:
    - targets: ['app-server:8080']
```

### Adding Custom Alerts
Edit `rules.yml` to add more alerting rules:
```yaml
groups:
 - name: custom-alerts
   rules:
   - alert: HighCPUUsage
     expr: 100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
     for: 5m
     labels:
       severity: warning
     annotations:
       summary: "High CPU usage detected"
```

## Network Configuration

The setup uses a custom Docker network named `localprom` with bridge driver, allowing all services to communicate using service names as hostnames.

## Data Persistence

Currently, data is not persisted between container restarts. To add persistence, mount volumes for Prometheus data:

```yaml
prometheus:
  image: prom/prometheus
  volumes:
    - "./prometheus.yml:/etc/prometheus/prometheus.yml"
    - "./rules.yml:/etc/prometheus/rules.yml"
    - "prometheus-data:/prometheus"  # Add this line

volumes:
  prometheus-data:  # Add this section
```

## References

- [Prometheus Documentation](https://prometheus.io/docs/)
- [Alertmanager Documentation](https://prometheus.io/docs/alerting/latest/alertmanager/)
- [Node Exporter Documentation](https://github.com/prometheus/node_exporter)
- [Original Setup Guide](https://mxulises.medium.com/simple-prometheus-setup-on-docker-compose-f702d5f98579)
