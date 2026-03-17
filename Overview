# System Overview & Architecture

## Concept
Kubernetes (K8s) is an advanced orchestration tool designed to manage container scaling, self-healing, rolling updates, and intelligent scheduling. It builds upon the foundation of containerization, packaging applications and dependencies into isolated units.

## Core Components
* **Control Plane (Master Node):** Manages cluster state and scheduling decisions.
* **Worker Nodes:** Run application containers and handle networking.
* **API Server:** RESTful interface for cluster management.
* **etcd:** Distributed key-value store for cluster data.
* **Scheduler:** Assigns pods to nodes based on resource requirements.
* **Controller Manager:** Maintains the desired state of the cluster.
* **kubelet:** Node agent that manages pod lifecycle.
* **kube-proxy:** Handles network rules and load balancing.
* **Pod:** The smallest deployable unit in Kubernetes.

## Network Design
The infrastructure uses **RKE2** (Rancher Kubernetes Engine 2) for security and stability. Two **HAProxy** nodes sit in front of the cluster to provide a stable access point for the Control Plane and load-balance incoming traffic across application nodes via an Ingress Controller.
