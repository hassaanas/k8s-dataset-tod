# ToD-K8s-OOD: A Robustness Benchmark for C-V2X Anomaly Detection

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![Kubernetes](https://img.shields.io/badge/Orchestrator-MicroK8s-blue.svg)](https://microk8s.io/)

## 📌 Overview
This repository contains the **ToD-K8s-OOD Dataset**, a high-granularity telemetry collection designed to evaluate the robustness of Machine Learning models against **Out-of-Distribution (OOD)** shifts in Tele-operated Driving (ToD) environments.

While most anomaly detection datasets focus on static, *i.i.d.* conditions, this dataset provides a "stress test" across a three-tier vehicle-to-cloud continuum: **On-Board Unit (OBU)**, **Multi-access Edge Computing (MEC)**, and **Cloud**.

## 📊 Dataset Specifications
* **Infrastructure:** 3-tier C-V2X testbed (Raspberry Pi 4 OBUs, VM-based MEC, Cloud Cluster).
* **Monitoring:** Prometheus with a **1-second** raw scrape interval.
* **Duration:** 4 hours total (2 hours Nominal, 2 hours Stressed/Fault Injection).
* **Features:** 38 system metrics covering CPU, Memory residency, Network I/O, and Disk Pressure.
* **Faults:** Synthetic stressors injected via Chaos Mesh (CPU hogs, memory leaks, network latency).

## 📁 Repository Structure
```text
├── data/
│   ├── D_nominal.csv          # Standard operational telemetry
│   ├── D_stressed.csv         # Telemetry during fault injection
│   └── tier_metadata.json     # Hardware specs for OBU, MEC, Cloud
├── notebooks/
│   ├── 01_Preprocessing.ipynb # 5-second resampling and feature engineering
│   ├── 02_P0_Baseline.ipynb   # i.i.d. Training and Evaluation
│   └── 03_P2_OOD_Test.ipynb   # Cross-tier generalization experiments
├── src/
│   └── feature_engine.py      # Script for window-based feature extraction
├── LICENSE
└── README.md
