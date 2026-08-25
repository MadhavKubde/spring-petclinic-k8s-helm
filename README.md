# Spring PetClinic: Cloud-Native Kubernetes, Helm & Observability Pipeline

An end-to-end cloud-native deployment pipeline for the **Spring PetClinic (Java 17 / Spring Boot 3)** microservice backed by **PostgreSQL**. The architecture is containerized with Docker, packaged with **Helm v3**, deployed on a local **Kind (Kubernetes in Docker)** cluster, autoscaled using **Horizontal Pod Autoscaler (HPA)**, and monitored in real-time with **Prometheus Operator** and **Grafana**.

---


Markdown
# Spring PetClinic: Cloud-Native Kubernetes, Helm & Observability Pipeline

An end-to-end cloud-native deployment pipeline for the **Spring PetClinic (Java 17 / Spring Boot 3)** microservice backed by **PostgreSQL**. The application is containerized with Docker, packaged using **Helm v3**, orchestrated on **Kind (Kubernetes in Docker)**, autoscaled using **Horizontal Pod Autoscaler (HPA)**, and monitored in real time with **Prometheus Operator** and **Grafana**.

---

## 🏗️ Architecture & Component Flow

[ Browser / Client ] ──> http://localhost:8080
│
▼
[ Kind HostPort: 8080 ──> NodePort: 30080 ]
│
▼
[ Service: petclinic ]
├──> Pod: petclinic-1 (JVM / Spring Boot) ───┐
└──> Pod: petclinic-2 (JVM / Spring Boot) ───┼──> [ Service: postgres (5432) ]
│              │
(HPA scales dynamically: 2 to 5 Pods)        │              ▼
└──> [ Pod: postgres:15-alpine ]

[ Prometheus Operator ] ──(Scrapes /actuator/prometheus)──> [ ServiceMonitor ]
│
▼
[ Grafana (Dashboard 4701) ] ──> http://localhost:3000


---

## 🛠️ Technology Stack

| Layer | Technology | Purpose |
| :--- | :--- | :--- |
| **Application** | Spring Boot 3 / Java 17 | Core web service with Spring Data JPA & Actuator |
| **Database** | PostgreSQL 15 (`postgres:15-alpine`) | Relational persistence backend |
| **Container Engine** | Docker Desktop | Multi-stage image build |
| **Container Registry** | Docker Hub | Public image hosting (`madhavkubde/spring-petclinic:v1.0`) |
| **Orchestration** | Kubernetes v1.30 via Kind | Container scheduling, networking, and cluster management |
| **Package Management** | Helm v3 | Declarative release lifecycle management and templating |
| **Autoscaling** | Metrics Server + HPA | Dynamic horizontal pod scaling based on CPU utilization |
| **Observability** | Prometheus Operator & Grafana | Real-time JVM heap, GC, thread, and request metrics |

---

## 🚀 Setup & Execution Guide

### 1. Provision Kind Kubernetes Cluster
```bash
kind create cluster --name petclinic-cluster --config kind-config.yaml
2. Deploy Metrics Server (Required for HPA)
Bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl patch deployment metrics-server -n kube-system --type="strategic" -p '{"spec":{"template":{"spec":{"containers":[{"name":"metrics-server","args":["--cert-dir=/tmp","--secure-port=4443","--kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname","--kubelet-use-node-status-port","--metric-resolution=15s","--kubelet-insecure-tls"]}]}}}}'
3. Deploy Observability Stack (Prometheus & Grafana)
Bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring --create-namespace
4. Deploy Spring PetClinic via Helm
Bash
helm install petclinic-release ./petclinic-chart
🔍 Validation & Testing
Access the Services
PetClinic Web Application: http://localhost:8080 (or run kubectl port-forward svc/petclinic 8080:8080)

Grafana Dashboard: http://localhost:3000 (Port-forward: kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80)

Default Username: admin

Dashboard ID: Import 4701 (JVM Micrometer)

Trigger Autoscaling (HPA Load Test)
Run an automated request loop in PowerShell:

PowerShell
while($true) { Invoke-RestMethod -Uri "http://localhost:8080/owners?lastName=" -TimeoutSec 1 }
Monitor pod scaling live:

Bash
kubectl get hpa petclinic-hpa -w

---



