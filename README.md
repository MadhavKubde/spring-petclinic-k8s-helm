# Spring PetClinic: Cloud-Native Kubernetes, Helm & Observability Pipeline

An end-to-end cloud-native deployment pipeline for the **Spring PetClinic (Java 17 / Spring Boot 3)** microservice backed by **PostgreSQL**. The architecture is containerized with Docker, packaged with **Helm v3**, deployed on a local **Kind (Kubernetes in Docker)** cluster, autoscaled using **Horizontal Pod Autoscaler (HPA)**, and monitored in real-time with **Prometheus Operator** and **Grafana**.

---

## 🏗️ Architecture & Component Flow

```text
[ Browser / Client ] ──> http://localhost:8080
         │
         ▼
[ Kind HostPort 8080 ──> NodePort 30080 ]
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
