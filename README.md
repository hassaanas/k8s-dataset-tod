# ToD-K8s-OOD: A Robustness Benchmark for C-V2X Anomaly Detection

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![Kubernetes](https://img.shields.io/badge/Orchestrator-MicroK8s-blue.svg)](https://microk8s.io/)

## 📌 Overview

This repository contains the **ToD-K8s-OOD Dataset**, a telemetry collection for
evaluating Machine Learning anomaly detectors against **Out-of-Distribution
(OOD)** shifts in **Tele-operated Driving (ToD)** / cellular-IoT environments.

The data is collected from a multi-node **C-V2X testbed** where each site is an
independent single-node **MicroK8s** cluster running an MQTT broker and a set of
Python microservices in the `tod` namespace. Faults (memory exhaustion / OOM,
CPU stress) are injected into these pods to produce the anomalous samples.

## 🖥️ Testbed

Telemetry is gathered from three nodes (the `node` column in the data):

| Node (`node`) | Tier | Role |
|---------------|------|------|
| `edge` | MEC (Multi-access Edge Computing) | Central MQTT hub; bridges OBU and RDS |
| `obu`  | On-Board Unit | Vehicle-side unit (Raspberry Pi 4) |
| `rds`  | Remote Driving Station | Remote-driving / cloud-side stack |

Each node runs the same per-site stack:

- **Broker:** `eclipse-mosquitto` (pod name contains `broker`).
- **Microservices:** `ms-speed`, `ms-direction`, `ms-cruise` (image
  `ms-tod-app:v1`, Debian / `python:3.11-slim`).

More details on testbed can be found at [`ToD Testbed`](https://github.com/hassaanas/tod).

## 📊 Dataset Specifications

- **Monitoring:** Prometheus telemetry, resampled to a **5-second** step.
- **Telemetry sources:** `broker`, `ms-speed`, `ms-direction`, `ms-cruise` pods
  across the `edge`, `obu`, and `rds` nodes.
- **Features:** 6 engineered per-pod metrics (CPU, memory, network I/O, disk I/O,
  plus CPU and memory growth rates).
- **Faults:** Synthetic stressors (memory/OOM and CPU) injected with
  `stress-ng` via the companion
  [`k8s-failure-injection`](https://github.com/hassaanas/k8s-dataset-tod) toolkit — e.g.
  broker-stress and microservice (`ms-apps`) stress sessions.
- **Labels:** Per-row `true_label` (`0` = normal, `1` = anomaly).

## 📁 Repository Structure

```text
├── data/
│   ├── D_nominal.csv          # Telemetry from the nominal-operation collection run
│   └── D_stressed.csv         # Telemetry from the fault-injection collection runs
├── datasheet.md               # Datasheet for the dataset (collection, labeling, uses)
├── CITATION.cff               # Machine-readable citation metadata
├── requirements.txt           # Python dependencies for the Quick Start / examples
├── SHA256SUMS                 # Integrity checksums for the data files
├── LICENSE                    # MIT License (code)
├── LICENSE-DATA               # CC BY 4.0 License (data)
└── README.md
```

## 🧬 Data Schema

Both CSVs share the same 11-column layout:

| Column | Type | Description |
|--------|------|-------------|
| `timestamp` | int | Unix epoch (seconds) |
| `datetime` | str | Human-readable timestamp (`DD-MM-YY H:MM`) |
| `node` | str | Source node / tier: `edge`, `obu`, `rds` |
| `pod` | str | Source pod: `broker`, `ms-speed`, `ms-direction`, `ms-cruise` |
| `cpu_percent` | float | CPU utilization (%) |
| `memory_percent` | float | Memory residency (%) |
| `network_io_bytes_per_sec` | float | Network I/O throughput (B/s) |
| `disk_io_bytes_per_sec` | float | Disk I/O throughput (B/s) |
| `cpu_growth_rate` | float | Short-window CPU growth rate |
| `memory_growth_rate` | float | Short-window memory growth rate |
| `true_label` | int | `0` = normal, `1` = anomaly |

The six feature columns (`cpu_percent` … `memory_growth_rate`) are the model
inputs; `timestamp`, `datetime`, `node`, and `pod` are metadata.

### Composition

| File | Rows | Normal (`0`) | Anomaly (`1`) |
|------|------|--------------|----------------|
| `data/D_nominal.csv`  | 12,969 | 11,528 | 1,441 |
| `data/D_stressed.csv` | 12,969 | 9,557  | 3,412 |

> Both files carry a per-row `true_label`. `D_nominal.csv` is dominated by normal
> samples; `D_stressed.csv` contains a higher proportion of injected anomalies.

## 🚀 Quick Start

```python
import pandas as pd

FEATURES = [
    "cpu_percent",
    "memory_percent",
    "network_io_bytes_per_sec",
    "disk_io_bytes_per_sec",
    "cpu_growth_rate",
    "memory_growth_rate",
]

nominal = pd.read_csv("data/D_nominal.csv")
stressed = pd.read_csv("data/D_stressed.csv")

# Drop rows with missing feature values before training/evaluation
nominal = nominal.dropna(subset=FEATURES)
stressed = stressed.dropna(subset=FEATURES)

X_train = nominal[FEATURES]            # baseline distribution
X_test, y_test = stressed[FEATURES], stressed["true_label"]
```

## 📚 Citation

If you use this dataset, please cite:

> [1] H. Siddiqui and F. Khendek, *Robustness of ML-Based Anomaly Detection under Distribution Shift in Microservices Based Safety-Critical CIoT*, 
> 17th International Conference on Network of the Future (NoF 2026), Italy
> (Submitted)

> [2] H. Siddiqui and F. Khendek, *Microservices for Reliable Safety-Critical
> Cellular IoT Systems — A Case Study*, GLOBECOM 2024 - 2024 IEEE Global
> Communications Conference, Dec. 2024, pp. 1455–1460.
> doi: [10.1109/GLOBECOM52923.2024.10901455](https://doi.org/10.1109/GLOBECOM52923.2024.10901455)

> [3] H. Siddiqui and F. Khendek, *Memory failures in microservices based
> Cellular IoT systems - An experimental evaluation of service availability*,
> 2025 IEEE 102nd Vehicular Technology Conference (VTC2025-Fall), Chengdu, China,
> Oct. 2025.
> doi: [10.1109/VTC2025-Fall65116.2025.11309934](https://doi.org/10.1109/VTC2025-Fall65116.2025.11309934)

## 📄 License

This repository is dual-licensed:

- **Data** (`data/` — `D_nominal.csv`, `D_stressed.csv`): [Creative Commons
  Attribution 4.0 International (CC BY 4.0)](LICENSE-DATA).
- **Code** (scripts/examples): [MIT License](LICENSE).

Please attribute the dataset using the references in the
[Citation](#-citation) section or the `CITATION.cff` file.
