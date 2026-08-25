# 🐱 Spring PetClinic: Cloud-Native Kubernetes, Helm & Observability

<div align="center">

[![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.30-blue?logo=kubernetes)](https://kubernetes.io/)
[![Helm](https://img.shields.io/badge/Helm-v3-blue?logo=helm)](https://helm.sh/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3-green?logo=spring)](https://spring.io/projects/spring-boot)
[![Java](https://img.shields.io/badge/Java-17-orange?logo=java)](https://www.java.com/)
[![Docker](https://img.shields.io/badge/Docker-Containerized-blue?logo=docker)](https://www.docker.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-blue?logo=postgresql)](https://www.postgresql.org/)

An end-to-end **cloud-native deployment pipeline** for the Spring PetClinic microservice (Java 17 / Spring Boot 3) backed by PostgreSQL. Fully containerized with Docker, orchestrated by Kubernetes (Kind), packaged with Helm, and monitored with Prometheus & Grafana.

[Quick Start](#-quick-start) • [Architecture](#-architecture--component-flow) • [Tech Stack](#-technology-stack) • [Verification](#-verification--testing) • [Troubleshooting](#-troubleshooting)

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture--component-flow)
- [Technology Stack](#-technology-stack)
- [Prerequisites](#-prerequisites)
- [Quick Start](#-quick-start)
- [Detailed Setup](#-detailed-setup-guide)
- [Verification & Testing](#-verification--testing)
- [Accessing Services](#-accessing-services)
- [Troubleshooting](#-troubleshooting)
- [Project Structure](#-project-structure)

---

## 📖 Overview

This project demonstrates a **production-ready, cloud-native architecture** with:

- ✅ **Multi-pod Spring Boot application** with dynamic horizontal pod autoscaling (HPA: 2-5 pods)
- ✅ **PostgreSQL 15 backend** running in-cluster with persistent storage
- ✅ **Real-time observability** via Prometheus Operator and Grafana dashboards
- ✅ **Helm-based templating** for declarative, repeatable deployments
- ✅ **Local Kubernetes cluster** using Kind (Docker-in-Docker)
- ✅ **Production-grade monitoring** with JVM metrics, GC stats, and custom application metrics

---

## 🏗️ Architecture & Component Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                         Browser / Client                        │
│                    http://localhost:8080                        │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│              Kind Cluster (Local Kubernetes)                     │
│  HostPort: 8080 ──> NodePort Service: 30080                     │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Service: petclinic (ClusterIP)                         │   │
│  │  ├─ Pod: petclinic-1 (JVM / Spring Boot 3)  ◄──┐       │   │
│  │  ├─ Pod: petclinic-2 (JVM / Spring Boot 3)  ◄──┤       │   │
│  │  └─ Pod: petclinic-N (HPA: 2-5 replicas)     ◄──┤       │   │
│  └─────────────────────────────────────────────────┼───────┘   │
│                                                     │            │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Service: postgres (ClusterIP: 5432)                     │  │
│  │  └─ Pod: postgres:15-alpine (StatefulSet)               │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Observability Stack (monitoring namespace)              │  │
│  │  ├─ Prometheus Operator                                 │  │
│  │  ├─ Grafana (http://localhost:3000)                     │  │
│  │  ├─ Metrics Server (HPA data source)                    │  │
│  │  └─ ServiceMonitor (scrapes /actuator/prometheus)       │  │
│  └──────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Technology Stack

| Layer | Technology | Purpose |
|:------|:-----------|:--------|
| **Application** | Spring Boot 3 / Java 17 | Core web service with Spring Data JPA, Spring Web & Spring Boot Actuator |
| **Database** | PostgreSQL 15 (alpine) | Relational persistence backend with high availability |
| **Containerization** | Docker Desktop | Multi-stage image builds for optimized container images |
| **Container Registry** | Docker Hub | Public image hosting (`madhavkubde/spring-petclinic:v1.0`) |
| **Orchestration** | Kubernetes v1.30 via Kind | Container scheduling, service discovery, networking, and cluster management |
| **Package Manager** | Helm v3 | Declarative release lifecycle management and templating |
| **Autoscaling** | Metrics Server + HPA | Dynamic horizontal pod scaling based on CPU utilization (2-5 replicas) |
| **Monitoring** | Prometheus Operator | Time-series database for metrics collection and alerting |
| **Visualization** | Grafana | Real-time dashboards for JVM heap, GC, threads, request rates, and latency |

---

## ✅ Prerequisites

Ensure you have the following installed and running:

| Tool | Version | Download |
|:-----|:--------|:---------|
| **Docker Desktop** | Latest | [Download](https://www.docker.com/products/docker-desktop) |
| **kubectl** | v1.30+ | [Install](https://kubernetes.io/docs/tasks/tools/) |
| **Helm** | v3+ | [Install](https://helm.sh/docs/intro/install/) |
| **Kind** | v0.20+ | [Install](https://kind.sigs.k8s.io/docs/user/quick-start/#installation) |

**System Requirements:**
- CPU: 4+ cores
- RAM: 8GB+ (16GB recommended)
- Disk: 20GB+ free space
- Docker Desktop: 2GB memory allocation (increase if needed)

**Verify installations:**
```bash
docker --version      # Docker 24.0.0+
kubectl version --client  # v1.30.0+
helm version          # v3.10.0+
kind version          # v0.20.0+
```

---

## 🚀 Quick Start

### One-Command Deployment (Optional)

For first-time users, here's a complete end-to-end setup:

```bash
# Clone repository
git clone https://github.com/MadhavKubde/spring-petclinic-k8s-helm.git
cd spring-petclinic-k8s-helm

# Run complete setup (all steps below in one script)
bash scripts/deploy.sh
```

Or follow the **Detailed Setup Guide** below for step-by-step instructions.

---

## 📋 Detailed Setup Guide

### 1. Provision Kind Kubernetes Cluster

Create a local Kubernetes cluster optimized for this project:

```bash
kind create cluster --name petclinic-cluster --config kind-config.yaml
```

**Verify cluster is running:**
```bash
kubectl cluster-info
kubectl get nodes
```

**Expected Output:**
```
KIND             STATUS   ROLES    
petclinic-cluster-control-plane   Ready    control-plane,master
```

---

### 2. Deploy Metrics Server (Required for HPA)

The Metrics Server provides CPU/memory metrics for horizontal pod autoscaling:

```bash
# Apply the official metrics-server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Patch for Kind (disable secure port checks)
kubectl patch deployment metrics-server -n kube-system --type=json -p='[
  {
    "op": "add",
    "path": "/spec/template/spec/containers/0/args/-",
    "value": "--kubelet-insecure-tls"
  },
  {
    "op": "add",
    "path": "/spec/template/spec/containers/0/args/-",
    "value": "--kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname"
  }
]'

# Verify metrics-server is running
kubectl wait --for=condition=available --timeout=300s deployment/metrics-server -n kube-system
kubectl get deployment -n kube-system | grep metrics-server
```

---

### 3. Deploy Observability Stack (Prometheus & Grafana)

Install the Prometheus Operator stack for monitoring:

```bash
# Add Prometheus Helm repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install Prometheus stack in monitoring namespace
helm install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring \
  --create-namespace \
  --values monitoring-values.yaml

# Verify deployment
kubectl get all -n monitoring
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=kube-prometheus-stack -n monitoring --timeout=300s
```

---

### 4. Deploy Spring PetClinic Application via Helm

Deploy the Spring PetClinic application and PostgreSQL using Helm charts:

```bash
# Install the petclinic Helm chart
helm install petclinic-release ./petclinic-chart \
  --namespace petclinic \
  --create-namespace \
  --values petclinic-chart/values.yaml

# Verify deployment (wait for pods to be ready)
kubectl get all -n petclinic
kubectl wait --for=condition=ready pod -l app=petclinic -n petclinic --timeout=300s
kubectl wait --for=condition=ready pod -l app=postgres -n petclinic --timeout=300s
```

**Expected Output:**
```
NAME                                   READY   STATUS    RESTARTS   AGE
pod/petclinic-deployment-xxx          3/3     Running   0          2m
pod/postgres-statefulset-0            1/1     Running   0          2m
```

---

## 🔍 Verification & Testing

### Service Access Endpoints

| Service | URL | Port-Forward Command | Credentials |
|---------|-----|----------------------|-------------|
| **Spring PetClinic** | http://localhost:8080 | `kubectl port-forward svc/petclinic 8080:8080 -n petclinic` | N/A |
| **Grafana Dashboards** | http://localhost:3000 | `kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80` | `admin` / `prom-operator` |
| **Prometheus UI** | http://localhost:9090 | `kubectl port-forward -n monitoring svc/monitoring-prometheus 9090:9090` | N/A |

### Access Services Locally

**Terminal 1: Spring PetClinic**
```bash
kubectl port-forward svc/petclinic 8080:8080 -n petclinic
# Visit http://localhost:8080
```

**Terminal 2: Grafana Dashboards**
```bash
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
# Visit http://localhost:3000
# Login: admin / prom-operator
# Dashboard ID: 4701 (Spring Boot / JVM Metrics)
```

**Terminal 3: Prometheus**
```bash
kubectl port-forward -n monitoring svc/monitoring-prometheus 9090:9090
# Visit http://localhost:9090
```

---

### Autoscaling Verification (HPA Load Test)

#### Generate HTTP Load (Bash)

```bash
# Continuous HTTP requests to trigger HPA
kubectl run -it --rm load-generator --image=busybox:1.28 --restart=Never -n petclinic -- \
  sh -c "while true; do wget -q -O- http://petclinic:8080; done"
```

#### Generate HTTP Load (PowerShell - Windows)

```powershell
# Run this in PowerShell to simulate load
$url = "http://localhost:8080"
while ($true) {
    try { Invoke-WebRequest -Uri $url -UseBasicParsing | Out-Null }
    catch { }
    Start-Sleep -Milliseconds 100
}
```

#### Watch Pod Autoscaling in Real-Time

```bash
# In a new terminal, watch HPA status
kubectl get hpa petclinic-hpa -n petclinic -w

# Watch pods scale up/down
kubectl get pods -n petclinic -l app=petclinic -w

# View HPA events
kubectl describe hpa petclinic-hpa -n petclinic
```

**Expected Behavior:**
- Initial: 2 pods running
- Under Load: Scales to 3, 4, 5 pods
- Load Stops: Scales back down to 2 pods (after cooldown period)

---

## 🌐 Accessing Services

### 1. Spring PetClinic Application

```bash
kubectl port-forward svc/petclinic 8080:8080 -n petclinic
# Visit: http://localhost:8080
```

**Features:**
- Home page with welcome message
- PetClinic owner & pet management
- Spring Boot Actuator endpoints: http://localhost:8080/actuator
- Prometheus metrics: http://localhost:8080/actuator/prometheus

---

### 2. Grafana Dashboards

```bash
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
# Visit: http://localhost:3000
```

**Login Credentials:**
- Username: `admin`
- Password: `prom-operator`

**Recommended Dashboards:**
- **Dashboard ID 4701**: Spring Boot / JVM Metrics (GC, memory, threads)
- **Dashboard ID 1860**: Node Exporter (cluster resource usage)
- **Kubernetes Cluster Monitoring**: Pod CPU/memory usage

---

### 3. Prometheus Queries

```bash
kubectl port-forward -n monitoring svc/monitoring-prometheus 9090:9090
# Visit: http://localhost:9090
```

**Example Queries:**
```promql
# JVM Heap Memory Usage
jvm_memory_used_bytes{area="heap"}

# CPU Usage
container_cpu_usage_seconds_total

# HTTP Requests Per Second
rate(http_requests_total[1m])

# Pod Memory Usage
container_memory_usage_bytes
```

---

## 🐛 Troubleshooting

### Cluster Issues

**Problem: Kind cluster fails to start**
```bash
# Solution: Check Docker daemon
docker ps
docker system prune -a --volumes

# Recreate cluster
kind delete cluster --name petclinic-cluster
kind create cluster --name petclinic-cluster --config kind-config.yaml
```

**Problem: Nodes are NotReady**
```bash
# Check node logs
kubectl describe nodes
kubectl get events -A

# Restart Docker Desktop and retry
```

---

### Metrics Server Issues

**Problem: HPA shows "unknown" for metrics**
```bash
# Check metrics-server logs
kubectl logs -n kube-system -l app.kubernetes.io/name=metrics-server

# Verify metrics are available
kubectl get --raw /apis/metrics.k8s.io/v1beta1/nodes
```

---

### Postgres Connection Issues

**Problem: Pods cannot connect to PostgreSQL**
```bash
# Check PostgreSQL pod status
kubectl get pods -n petclinic -l app=postgres
kubectl logs -n petclinic pod/postgres-statefulset-0

# Test connection from petclinic pod
kubectl exec -it -n petclinic pod/petclinic-deployment-xxx -- \
  sh -c "psql -h postgres -U petclinic -d petclinic -c 'SELECT 1'"
```

**Solution: Update connection string in values.yaml**
```yaml
spring:
  datasource:
    url: jdbc:postgresql://postgres:5432/petclinic
    username: petclinic
    password: petclinic
```

---

### Grafana Dashboard Missing

**Problem: Dashboard 4701 not available**
```bash
# Manually import dashboard
# 1. Go to http://localhost:3000/grafana
# 2. Click "+" → Import
# 3. Enter Dashboard ID: 4701
# 4. Select Prometheus as data source
```

---

### Pod Restart Loop

**Problem: Petclinic pods keep restarting**
```bash
# Check pod logs
kubectl logs -n petclinic -f pod/petclinic-deployment-xxx

# Check pod events
kubectl describe pod -n petclinic pod/petclinic-deployment-xxx

# Common issues:
# - Database not ready: Wait for postgres pod to be ready
# - Memory issues: Increase pod requests/limits in values.yaml
# - Invalid configuration: Check environment variables
```

---

### Helm Deployment Issues

**Problem: Helm install fails**
```bash
# Debug Helm chart
helm lint ./petclinic-chart
helm template petclinic-release ./petclinic-chart

# Check values
helm get values petclinic-release -n petclinic

# Rollback failed release
helm rollback petclinic-release -n petclinic
```

---

### Port-Forward Connection Refused

**Problem: Cannot connect via localhost:8080**
```bash
# Verify port-forward is running and listening
kubectl port-forward svc/petclinic 8080:8080 -n petclinic --address=127.0.0.1

# Try alternative (binds to all interfaces)
kubectl port-forward svc/petclinic 8080:8080 -n petclinic --address=0.0.0.0

# Check if port is already in use
lsof -i :8080  # macOS/Linux
netstat -ano | findstr :8080  # Windows
```

---

## 📁 Project Structure

```
spring-petclinic-k8s-helm/
├── README.md                          # This file
├── kind-config.yaml                   # Kind cluster configuration
├── monitoring-values.yaml             # Prometheus/Grafana helm values
│
├── petclinic-chart/                   # Helm chart for Spring PetClinic
│   ├── Chart.yaml                     # Chart metadata
│   ├── values.yaml                    # Default chart values
│   ├── templates/
│   │   ├── deployment.yaml            # Petclinic deployment
│   │   ├── service.yaml               # Kubernetes service
│   │   ├── hpa.yaml                   # Horizontal pod autoscaler
│   │   ├── configmap.yaml             # Application configuration
│   │   ├── ingress.yaml               # Ingress rules
│   │   └── ...
│   └── charts/                        # Dependency charts
│
├── postgres-chart/                    # Helm chart for PostgreSQL
│   ├── Chart.yaml
│   ├── values.yaml
│   ├── templates/
│   │   ├── statefulset.yaml           # PostgreSQL StatefulSet
│   │   ├── service.yaml
│   │   ├── pvc.yaml                   # Persistent volume claims
│   │   └── ...
│   └── ...
│
└── scripts/                           # Deployment & utility scripts
    ├── deploy.sh                      # Complete setup script
    ├── cleanup.sh                     # Remove all resources
    ├── verify.sh                      # Health checks
    └── load-test.sh                   # Generate HTTP load
```

---

## 📚 Key Concepts

### Helm Chart Variables
- `values.yaml`: Default configuration values
- `Chart.yaml`: Chart metadata and dependencies
- `templates/`: Kubernetes manifest templates

### Kubernetes Components
- **Deployment**: Manages petclinic application pods
- **StatefulSet**: Manages PostgreSQL with stable identity
- **Service**: Exposes pods within cluster (ClusterIP) or externally (NodePort)
- **HPA**: Automatically scales pods based on metrics
- **ConfigMap**: Stores application configuration
- **PersistentVolumeClaim**: Storage for PostgreSQL data

### Observability
- **ServiceMonitor**: Tells Prometheus where to scrape metrics
- **Grafana Dashboards**: Visualize metrics from Prometheus
- **Actuator Endpoints**: Spring Boot exposes `/actuator/prometheus`

---

## 🔗 Useful Resources

- [Spring PetClinic Original Project](https://github.com/spring-projects/spring-petclinic)
- [Kind Documentation](https://kind.sigs.k8s.io/)
- [Kubernetes Official Docs](https://kubernetes.io/docs/)
- [Helm Best Practices](https://helm.sh/docs/chart_best_practices/)
- [Prometheus Operator](https://prometheus-operator.dev/)
- [Grafana Dashboards](https://grafana.com/grafana/dashboards/)

---

## 📝 License

This project is licensed under the [Apache License 2.0](LICENSE).

---

<div align="center">

**⭐ If you found this helpful, please consider starring the repository!**

[Back to Top](#-spring-petclinic-cloud-native-kubernetes-helm--observability)

</div>
