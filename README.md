# Infrastructure
My Homelab Trial n Error
# 📊 Enterprise Monitoring Stack with LVM Storage Optimization

![Prometheus](https://img.shields.io/badge/Prometheus-v3.12.0-orange?logo=prometheus)
![Kubernetes](https://img.shields.io/badge/K3s-Cluster-blue?logo=kubernetes)
![Proxmox](https://img.shields.io/badge/Proxmox-VE-white?logo=proxmox)
![Linux](https://img.shields.io/badge/OS-Ubuntu_22.04-orange?logo=ubuntu)

## 📖 Executive Summary
This project documents the architecture, deployment, and troubleshooting of a centralized monitoring stack for a 3-node Kubernetes (K3s) cluster. The primary objective was to achieve **full infrastructure observability** while ensuring **high availability** and preventing storage bottlenecks through dedicated Logical Volume Manager (LVM) partitioning.

## 🏗️ Architecture & Data Flow
The monitoring stack follows a pull-based architecture to ensure secure and scalable metrics collection.

```mermaid
graph TD
    subgraph Proxmox_VE_Cluster
        Master[Docker Master Node]
        Worker1[ Worker 1]
        Worker2[ Worker 2]
    end

    subgraph Monitoring_Stack
        NE1[Node Exporter :9100]
        NE2[Node Exporter :9100]
        NE3[Node Exporter :9100]
        Prom[Prometheus Server :9090]
        Graf[Grafana Dashboard :3000]
    end

    Master --- NE1
    Worker1 --- NE2
    Worker2 --- NE3
    
    NE1 & NE2 & NE3 -->|Pull Metrics every 15s| Prom
    Prom -->|Query Data via PromQL| Graf

## 🛠️ Tech Stack
| Component | Purpose |
| :--- | :--- |
| **Proxmox VE** | Hypervisor for VM provisioning |
| **K3s / Docker** | Container orchestration and runtime |
| **Prometheus** | Time-series database for metrics scraping |
| **Node Exporter** | Hardware and OS metrics exporter |
| **Grafana** | Data visualization and dashboarding |
| **LVM** | Logical Volume Manager for dynamic storage allocation |
