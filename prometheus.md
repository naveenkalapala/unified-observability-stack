## 📌 What is Prometheus?

Prometheus is an open-source monitoring and alerting toolkit designed for reliability and scalability. It collects metrics, stores them as time-series data, and allows for powerful queries and visualizations.

---

## 🧱 Architecture

+-----------------+         +----------------+         +------------------+
| Exporters/App   | <--pull | Prometheus     | -->push | Alertmanager     |
| (Expose metrics)|         | (scrapes data) |         | (sends alerts)   |
+-----------------+         +----------------+         +------------------+
                                 |
                                 v
                         PromQL + Grafana
                       (visualize/query metrics)


````

### Components:
- **Prometheus Server**: Scrapes and stores metrics.
- **Exporters**: Expose metrics to be scraped (e.g., `node_exporter`, `kube-state-metrics`).
- **Pushgateway**: For short-lived jobs that push metrics.
- **Alertmanager**: Manages and routes alerts.
- **Grafana**: Visualization frontend (commonly used with Prometheus).

---

## 📂 Installation (Basic)

### Docker
```bash
docker run -p 9090:9090 prom/prometheus
````

### Kubernetes (Helm)

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/prometheus
```

## 📈 Data Model

* **Metric** = `metric_name{label1="value1", label2="value2"}`
* **Types**:

  * `Counter` – Always increasing (e.g., total requests)
  * `Gauge` – Can go up or down (e.g., memory usage)
  * `Histogram` – Observations grouped into buckets
  * `Summary` – Similar to histogram, with quantiles

---

## 🔍 PromQL Basics

### Example Queries:

```promql
# Total HTTP requests per second
rate(http_requests_total[1m])

# CPU usage per pod
sum(rate(container_cpu_usage_seconds_total{image!=""}[5m])) by (pod)

# Free memory
node_memory_MemAvailable_bytes

# Alert if CPU usage > 90%
100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100)
```

---

## ⚙️ Configuration (prometheus.yml)

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
```

---

## 🛠️ Exporters

| Exporter           | Description                   | Port |
| ------------------ | ----------------------------- | ---- |
| node\_exporter     | System metrics (CPU, memory)  | 9100 |
| blackbox\_exporter | Probes endpoints (ping, HTTP) | 9115 |
| kube-state-metrics | K8s object states             | 8080 |
| cadvisor           | Container metrics             | 8080 |

---

## 📦 Prometheus in Kubernetes

### Service Annotations

```yaml
metadata:
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "8080"
```

### Using Prometheus Operator

* Simplifies deployment and management
* CRDs: `ServiceMonitor`, `PrometheusRule`, etc.

---

## 🚨 Alerts

### Example Alert Rule

```yaml
groups:
  - name: cpu_alerts
    rules:
      - alert: HighCPUUsage
        expr: node_cpu_seconds_total{mode="user"} > 80
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage detected"
```

---

## 📚 Resources

* [Official Docs](https://prometheus.io/docs/)
* [Awesome Prometheus](https://github.com/roaldnefs/awesome-prometheus)
* [Prometheus Operator](https://github.com/prometheus-operator/prometheus-operator)

---

## ✅ Best Practices

* Avoid high-cardinality labels.
* Use dashboards in Grafana for visibility.
* Record rules for expensive PromQL queries.
* Tune scrape intervals for optimal performance.
* Group alerts logically with meaningful labels.

---

```
## 2. ⚙️ Setting Up Prometheus in EKS

### Prerequisites:
- AWS Account with sufficient IAM permissions.
- AWS CLI installed and configured.
- `kubectl` installed.
- `eksctl` (for easier EKS provisioning).
- `Helm` installed.

### Step-by-Step Guide

#### 🛠️ Step 1: Create the EKS Cluster
```bash
# Create EKS cluster with eksctl
eksctl create cluster \
  --name prometheus-demo \
  --region us-west-2 \
  --nodes 2 \
  --node-type t3.medium \
  --with-oidc \
  --ssh-access \
  --ssh-public-key your-key-name
```

#### 🛠️ Step 2: Verify Access to Cluster
```bash
# Update kubeconfig
aws eks --region us-west-2 update-kubeconfig --name prometheus-demo

# Test connectivity
kubectl get nodes
```

---

## 3. 📦 Installing `kube-prometheus-stack` with Helm

### What is Helm?
Helm is a package manager for Kubernetes that helps you define, install, and upgrade complex Kubernetes applications.

### Add Prometheus Community Helm Repository:
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

### Install the Stack:
```bash
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace
```

### Validate Installation:
```bash
kubectl get pods -n monitoring
```
You should see pods like `prometheus-kube-prometheus`, `grafana`, `alertmanager`, etc.

### Port Forward Grafana and Prometheus:
```bash
# Grafana UI (default login: admin/prom-operator)
kubectl port-forward svc/prometheus-grafana 3000:80 -n monitoring

# Prometheus UI
kubectl port-forward svc/prometheus-kube-prometheus-prometheus 9090 -n monitoring
```

---

## 4. 📊 Using Prometheus and Grafana

### Access Prometheus UI:
- Open http://localhost:9090
- Use the "Targets" tab to see metrics endpoints.

### Basic PromQL Queries:
```promql
# CPU usage of all pods
sum(rate(container_cpu_usage_seconds_total[5m])) by (pod)

# Memory usage
sum(container_memory_usage_bytes) by (pod)

# Uptime of Prometheus
up
```

### Access Grafana Dashboard:
- Open http://localhost:3000
- Default credentials: `admin` / `prom-operator`

### Add Prometheus as Data Source:
1. Go to *Settings → Data Sources* → Add Prometheus.
2. URL: `http://prometheus-kube-prometheus-prometheus.monitoring.svc:9090`

### Import Dashboards:
- Use Grafana dashboard ID from [Grafana.com](https://grafana.com/grafana/dashboards/)
- Examples: Kubernetes cluster monitoring (ID: `315`), Node Exporter (ID: `1860`)

---

## 💡 Pro Tips & Troubleshooting
- **Pod CrashLoopBackOff?**
  - Check logs: `kubectl logs <pod> -n monitoring`
  - Check resource limits in Helm values.

- **Missing Grafana UI?**
  - Ensure service type is `ClusterIP` (use port forwarding or change to `LoadBalancer`).

- **Query returns empty?**
  - Try a longer time range.
  - Use the "Targets" tab to verify if Prometheus scrapes the metrics endpoints.

---

> 🔄 Feel free to fork this repo and expand on your Prometheus setup!
