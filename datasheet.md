# Datasheet — ToD-K8s-OOD

This datasheet follows the *"Datasheets for Datasets"* framework (Gebru et al.,
2021) to document the motivation, composition, collection, and recommended use
of the **ToD-K8s-OOD** dataset.

---

## Motivation

- **Why was the dataset created?**
  To provide a labeled benchmark for evaluating machine-learning anomaly
  detectors under **out-of-distribution (OOD)** conditions in **tele-operated
  driving (ToD) / cellular-IoT** microservice deployments on Kubernetes. Most
  anomaly-detection datasets assume i.i.d. conditions; this one captures
  distribution shift between nominal operation and fault-injection scenarios.

- **Who created it and for whom?**
  H. Siddiqui and F. Khendek. Collected from a research C-V2X testbed and
  released for the research and practitioner community.

- **Funding / context.** This work has been partially supported by the Natural Sciences and Engineering Research Council (NSERC) of Canada.

---

## Composition

- **What do instances represent?**
  Each row is a per-pod telemetry snapshot at a 5-second sampling step, with six
  engineered resource-usage features and a binary label.

- **Files and counts:**

  | File | Rows | Columns | Normal (`0`) | Anomaly (`1`) |
  |------|------|---------|--------------|----------------|
  | `data/D_nominal.csv`  | 12,969 | 11 | 11,528 | 1,441 |
  | `data/D_stressed.csv` | 12,969 | 11 | 9,557  | 3,412 |

- **Schema (11 columns):** `timestamp`, `datetime`, `node`, `pod`,
  `cpu_percent`, `memory_percent`, `network_io_bytes_per_sec`,
  `disk_io_bytes_per_sec`, `cpu_growth_rate`, `memory_growth_rate`,
  `true_label`. The six metric columns are model features; `timestamp`,
  `datetime`, `node`, `pod` are metadata; `true_label` is the target
  (`0` = normal, `1` = anomaly).

- **Sources captured:**
  - Nodes (`node`): `edge` (MEC hub), `obu` (on-board unit, Raspberry Pi 4),
    `rds` (remote driving station).
  - Pods (`pod`): `broker` (`eclipse-mosquitto`), `ms-speed`, `ms-direction`,
    `ms-cruise` (`ms-tod-app:v1`).

- **Is any data missing?**
  Some feature values may be `NaN` (e.g., growth-rate columns at the start of a
  window). Drop rows with missing feature values before training/evaluation
  (see the README Quick Start).

- **Does it contain confidential / personal data?**
  No personal data. Identifiers are ephemeral Kubernetes pod-name suffixes
  (e.g., `broker-7f57c8d79b-f2m5m`) and internal node role names; no end-user
  PII is present.

---

## Collection Process

- **How was the data acquired?**
  System/resource metrics were collected via **Prometheus** from the testbed
  pods and resampled to a **5-second** step. Features were engineered per pod
  (utilization plus short-window growth rates).

- **Testbed.** Three independent single-node **MicroK8s** clusters (`edge`,
  `obu`, `rds`), each running the `tod` namespace with an MQTT broker and the
  ToD microservices. See the companion `tod` and `k8s-failure-injection`
  repositories.

- **Sampling step:** 5 seconds.

---

## Labeling

- **How were labels (`true_label`) assigned?**
  Rows collected during nominal operation are labeled `0`; rows collected during
  active fault-injection windows are labeled `1`. More details can be found at [1] (See README *Citation* section).

- **Fault types.** Synthetic stressors injected with `stress-ng` via the
  [`k8s-failure-injection`](https://github.com/hassaanas/k8s-failure-injection) toolkit: memory exhaustion (OOM) and CPU stress targeting the broker and the `ms-*`
  microservices.

- **Fault parameters per session.** Details can be found at [1] (See README *Citation* section).

---

## Preprocessing / Cleaning

- Raw Prometheus telemetry was resampled to a 5-second step and reduced to six
  engineered features plus metadata and label.
- The preprocessing/feature-engineering script is **TBD**

---

## Uses

- **Intended uses.**
  - OOD anomaly detection: train/calibrate on the nominal distribution, evaluate
    generalization on the fault-injection distribution.
  - Baselines (e.g., IsolationForest, supervised classifiers) and
    distribution-shift analysis.

- **Recommended evaluation.**
  The classes are **imbalanced**; prefer **PR-AUC**, **F1**, precision/recall
  over raw accuracy.

- **Uses to avoid / cautions.**
  - Single-testbed data: generalization to other clusters/hardware is untested.
  - Labels are heuristic (window-based).
  - Not a safety certification artifact; for research/benchmarking only.

---

## Distribution

- **How is it distributed?** Public GitHub repository.
- **License.** Data: **CC BY 4.0** (`LICENSE-DATA`); code: **MIT** (`LICENSE`).
- **DOI.** TBD
- **Integrity.** Verify with `sha256sum -c SHA256SUMS`.

---

## Maintenance

- **Maintainer.** Hassaan Siddiqui.
- **Contact.** hassaan.a.s@gmail.com
- **Updates / versioning.** Tracked via Git tags / GitHub Releases.
- **Contributions.** Via GitHub issues and pull requests.

---

## Citation

See `CITATION.cff` and the README *Citation* section.
