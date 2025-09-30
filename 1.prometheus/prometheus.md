# Observability Stack: Prometheus & Grafana

## Overview

This repository covers the modern observability stack using **Prometheus** for monitoring and metrics collection, and **Grafana** for visualization and alerting.

## Prometheus

**Prometheus** is an open-source monitoring and alerting toolkit that collects and stores metrics as time series data.

### Key Features

- **Multi-dimensional data model** with labels
- **PromQL** - Powerful query language
- **Pull-based monitoring** over HTTP
- **Service discovery** support
- **Built-in alerting**

### Metric Types

- **Counter**: Only increases (e.g., requests served)
- **Gauge**: Can go up/down (e.g., memory usage)
- **Histogram**: Samples in buckets
- **Summary**: Calculates quantiles

## Grafana

**Grafana** is an open-source visualization platform that creates dashboards and alerts from multiple data sources.

### Key Features

- **Rich visualizations**: Charts, graphs, tables, heatmaps
- **Multiple data sources**: Prometheus, InfluxDB, Elasticsearch
- **Alerting**: Integrated alert rules and notifications
- **Templating**: Dynamic dashboards with variables
- **User management**: Role-based access control

## Architecture

```
Applications/Services → Exporters → Prometheus → Grafana
                                      ↓
                                 Alert Manager
```

### Data Flow

1. **Metrics Exposure**: Applications expose metrics via HTTP endpoints
2. **Scraping**: Prometheus pulls metrics at regular intervals
3. **Storage**: Metrics stored in time-series database
4. **Querying**: Grafana queries Prometheus using PromQL
5. **Visualization**: Dashboards display metrics as charts/graphs
6. **Alerting**: Rules trigger notifications when thresholds are breached

## Quick Setup

### Start Prometheus

```bash
docker run -p 9090:9090 prom/prometheus
```

### Start Grafana

```bash
docker run -p 3000:3000 grafana/grafana
```

### Access

- **Prometheus**: http://localhost:9090
- **Grafana**: http://localhost:3000 (admin/admin)

## Basic Configuration

### Prometheus Config (prometheus.yml)

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
```

### Grafana Data Source

Connect Grafana to Prometheus at `http://prometheus:9090`

## Benefits

- **Prometheus**: Pull-based model, powerful querying, cloud-native
- **Grafana**: Rich visualizations, multi-source support, easy sharing
- **Together**: Complete observability solution for modern infrastructure

## Resources

- [Prometheus Docs](https://prometheus.io/docs/)
- [Grafana Docs](https://grafana.com/docs/)
- [PromQL Guide](https://prometheus.io/docs/prometheus/latest/querying/basics/)
