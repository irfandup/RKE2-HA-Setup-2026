# RKE2 Multi-Cluster Observability Stack

This repository contains the configuration and deployment steps for a centralized observability pipeline. It enables Metrics and Distributed Tracing for RKE2 clusters using OpenTelemetry, Prometheus, and Jaeger.

## 🏗️ Architecture

The system consists of a centralized Backend VM (the monitoring machine) and Local Agents running on each RKE2 cluster.

* **Backend VM (10.240.14.200)**: Runs Prometheus for metrics and Jaeger for traces via Docker Compose. (Will need docker installed on it)

* **Cluster Agents**: OpenTelemetry Collector (DaemonSet) running on the RKE2 hostNetwork to scrape Kubelet stats and receive application data.

## 🛠️ Phase 1: Central Backend Setup (VM)
On your monitoring VM (10.240.14.200), create a directory and add the following files.

1. Create three directory for the monitoring stack installation

```bash
mkdir -p /opt/observability/prometheus-data
mkdir -p /opt/observaility/grafana-data
```
2. Go to `/opt/observability/`,create two files `prometheus.yml` and `docker-compose.yml`
```bash
cd /opt/observability/
vim prometheus.yml
vim docker-compose.yml
```
Create `prometheus.yml` file with the following content:
```bash
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

Create `docker-compose.yml` with the following content:
```bash
services:
  # Prometheus for Metrics
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./prometheus-data:/prometheus
    ports:
      - "9090:9090"
    command:
      - "--config.file=/etc/prometheus/prometheus.yml"
      - "--storage.tsdb.path=/prometheus"
      - "--web.enable-remote-write-receiver"
  # Jaeger for Tracing
  jaeger:
    image: jaegertracing/all-in-one:latest
    environment:
      - COLLECTOR_OTLP_ENABLED=true
    ports:
      - "16686:16686" # UI
      - "4317:4317"   # OTLP gRPC (Where OTel sends data)
      - "4318:4318"   # OTLP Http (Where OTel sends data)

  # Grafana for Visualization
  grafana:
    image: grafana/grafana:latest
    volumes:
      - ./grafana-data:/var/lib/grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=katalaluanG0
```
3. Start the services prometheus, jaeger and grafana

```bash
docker compose up -d
```
## 🚀 Phase 2: RKE2 Cluster Deployment

1. Create `otel-values.yaml`
This configuration handles the RKE2 networking quirks and identifies the cluster.

Create `otel-values.yaml` with the following content:
```bash
image:
  repository: "otel/opentelemetry-collector-contrib"

mode: daemonset # CRITICAL: This puts a collector on every worker node

service:
  enabled: true
  type: ClusterIP
  #extraEnvs:
  #  - name: K8S_NODE_IP
  #  valueFrom:
  #    fieldRef:
  #      fieldPath: status.hostIP
hostNetwork: true

config:
  receivers:
    otlp:
      protocols:
        grpc:
          endpoint: 0.0.0.0:4317
        http:
          endpoint: 0.0.0.0:4318
    # 1. Node Monitoring
    hostmetrics:
      collection_interval: 10s
      scrapers:
        cpu: {}
        memory: {}
        disk: {}
        network: {}
    # 2. Pod Monitoring
    kubeletstats:
      collection_interval: 20s
      auth_type: "serviceAccount"
      endpoint: "https://localhost:10250"
      insecure_skip_verify: true
      metric_groups:
        - node
        - pod
        - container
        - volume

  processors:
    batch: {}
    memory_limiter:
      check_interval: 1s
      limit_percentage: 70
      spike_limit_percentage: 30

    # Adds Pod Name, Namespace, etc. to metrics
    k8sattributes:
      extract:
        metadata:
          - k8s.pod.name
          - k8s.pod.uid
          - k8s.namespace.name
          - k8s.node.name
      pod_association:
        - sources:
            - from: resource_attribute
              name: k8s.pod.ip
    resourcedetection:
      detectors: [env, system]
      timeout: 2s

  exporters:
    prometheusremotewrite:
      endpoint: "http://10.240.14.200:9090/api/v1/write"
      external_labels:
        cluster: "rke2-development"
      resource_to_telemetry_conversion:
        enabled: true # Turns "host.name" into a Prometheus label

    otlp/jaeger:
      endpoint: "10.240.14.200:4317" # Pointing to your VM
      tls:
        insecure: true

  service:
    pipelines:
      metrics:
        receivers: [otlp, hostmetrics, kubeletstats]
        processors: [memory_limiter, k8sattributes, resourcedetection, batch]
        exporters: [prometheusremotewrite]

      traces:
        receivers: [otlp]
        processors: [memory_limiter, k8sattributes, resourcedetection, batch]
        exporters: [otlp/jaeger]

# Pass the Node Name from K8s to the Collector so it knows where it is
extraEnvs:
  - name: K8S_NODE_NAME
    valueFrom:
      fieldRef:
        fieldPath: spec.nodeName


clusterRole:
  create: true
  rules:
    - apiGroups: [""]
      resources: ["nodes", "nodes/stats", "nodes/proxy", "pods", "services", "endpoints"]
      verbs: ["get", "list", "watch"]
    - apiGroups: ["apps"]
      resources: ["replicasets"]
      verbs: ["get", "list", "watch"]

serviceAccount:
  create: true
```

2. Install open-telemetry
```bash
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
helm install otel-collector open-telemetry/opentelemetry-collector -f otel-values.yaml -n monitoring --create-namespace
```
3. Access the three monitoring stack and start to explore 

`grafana`:http://ipaddress:3000

`prometheus`:http://ipaddress:9090

`jaeger`:http://ipaddress:16686






