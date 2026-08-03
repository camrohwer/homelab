# Observability

## Stack

Monitoring consists of:

- Prometheus: metrics collection and storage
- Grafana: visualization
- Node Exporter: host metrics
- kube-state-metrics: Kubernetes object metrics

## Data Flow

Nodes
    |
    +-- node-exporter
    |
    +-- kube-state-metrics
    |
    v
Prometheus
    |
    v
Grafana

## Grafana Dashboards

Provisioned through Helm values

Managed Declaratively through GitOps

