## RKE2 Multi-Cluster Observability Stack

This repository contains the configuration and deployment steps for a centralized observability pipeline. It enables Metrics and Distributed Tracing for RKE2 clusters using OpenTelemetry, Prometheus, and Jaeger.

### 🏗️ Architecture

The system consists of a centralized Backend VM and Local Agents running on each RKE2 cluster.

* **Backend VM (10.240.14.200)**: Runs Prometheus for metrics and Jaeger for traces via Docker Compose.

* **Cluster Agents**: OpenTelemetry Collector (DaemonSet) running on the RKE2 hostNetwork to scrape Kubelet stats and receive application data.
