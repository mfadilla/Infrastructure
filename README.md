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
        Worker1[Worker 1]
        Worker2[Worker 2]
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
    
    NE1 & NE2 & NE3 -->|Pull Metrics every 30s| Prom
    Prom -->|Query Data via PromQL| Graf
```
## 🛠️ Tech Stack
| Component | Purpose |
| :--- | :--- |
| **Proxmox VE** | Hypervisor for VM provisioning |
| **K3s / Docker** | Container orchestration and runtime |
| **Prometheus** | Time-series database for metrics scraping |
| **Node Exporter** | Hardware and OS metrics exporter |
| **Grafana** | Data visualization and dashboarding |
| **Teleport** | 1 Way Gate Remote Server |
| **LVM** | Logical Volume Manager for dynamic storage allocation |

## 🚨 Challenge & Troubleshooting: The SIGBUS Error
During the initial deployment, the Prometheus server unexpectedly crashed with a `SIGBUS` error and refused to restart.

**Root Cause:** 
Upon checking with `df -h`, I found that the root partition (`/`) was at 100% capacity. Prometheus relies on memory-mapped files, and the lack of disk space caused the crash.

**The Fix:**
To prevent this from happening again, I decoupled the Prometheus data directory from the root filesystem using LVM:
1. Provisioned a dedicated disk and created a new Logical Volume (LV).
2. Formatted and mounted it to `/mnt/prometheus_data`.
3. Updated the Prometheus service configuration to point to the new storage path.
4. Implemented a data retention policy to automatically prune old metrics.

**Result:** Prometheus now runs with stable storage, and the root partition utilization remains healthy.

## 🔒 Security Hardening
- **Least Privilege:** Configured services to run as dedicated, non-root users.
- **Network Isolation:** Prometheus is bound to the internal network. Only Grafana is exposed for querying.
- **Firewall:** Configured `ufw` to strictly allow only necessary ports for internal scraping.

## 📸 Visual Proof
<img width="1354" height="632" alt="image" src="https://github.com/user-attachments/assets/0de8a93b-e635-44e3-8fc3-daec1bb4d132" />
<img width="1346" height="509" alt="image" src="https://github.com/user-attachments/assets/b9dfe01b-2f90-4faa-8a28-58148f8cc011" />
<img width="1362" height="462" alt="image" src="https://github.com/user-attachments/assets/31f523fd-c311-4053-a126-71a81455656e" />

```
