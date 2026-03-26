# Infrastructure & Prerequisites

The infrastructure is distributed across virtual environments at `vcenter.um.edu.my` and `lotus-cluster.um.edu.my`.

### Hardware Specifications

| Node Type | Qty | OS | CPU | RAM | Disk | IP Range |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **HAProxy** | 2 | Ubuntu 24.04 | 8 vCPU | 8 GB | 80 GB | 10.240.5.165 - 166 |
| **Master** | 3 | Rocky Linux 9.6 | 4 vCPU | 8 GB | 50 GB | 10.240.5.167 - 169 |
| **Worker** | 2 | Rocky Linux 9.6 | 16 vCPU | 32 GB | 50 GB | 10.240.5.171 - 172 |

**Virtual IP (Keepalived):** 10.240.5.164
