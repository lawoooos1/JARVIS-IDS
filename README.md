# Network Event Threat Triage

A Python-based network security analysis pipeline that transforms raw network events into behavioral risk scores, heuristic attack classifications, explanations, and machine-learning features.

The project explores how simple flow-like network data can be transformed into information that is more useful for a defender: **What looks suspicious? What attack pattern might it represent? And why?**

This is an **active prototype**, not a finished intrusion detection system. The current implementation focuses on building and testing the detection pipeline and establishing a baseline for future machine-learning development.

---

## Problem

Raw network connection logs are difficult to analyze manually. A single dataset can contain normal traffic alongside port scans, network scans, brute-force attempts, beacon-like communication, and large outbound transfers.

The goal of this project is to build a pipeline that can:

* Detect unusual source-host behavior
* Aggregate network activity by source IP
* Generate behavioral features from network events
* Assign a working attack classification
* Explain why an event or host was considered suspicious
* Use the resulting features as input for machine-learning models

---

## Current Pipeline

The project currently processes `network_events.csv`, containing approximately 1,000 network events with the following fields:

* `src_ip`
* `dst_ip`
* `port`
* `bytes`

The current dataset does **not** contain timestamps, protocol information, payload data, or analyst-verified ground-truth labels. These limitations are important when interpreting the current detection results.

The pipeline consists of three main stages.

### 1. Behavioral Suspicion Scoring

Network events are analyzed using behavioral indicators derived from source and destination activity.

Examples include:

* A source contacting many different ports
* A source contacting many different destination IPs
* High total traffic volume from a source
* A source/port pair contacting many destinations
* A destination receiving connections from many different sources

These indicators are converted into boolean suspicion flags and combined into a `suspicious_score`.

The score is then mapped to a working risk level:

`normal → low → medium → high → critical`

This layer is intended as an initial triage mechanism rather than a definitive security verdict.

### 2. Behavioral Features and Attack Classification

Network events are aggregated by `src_ip` to create behavioral features such as:

* Event frequency
* Total bytes
* Average bytes per event
* Maximum bytes per event
* Number of unique destination IPs
* Number of unique ports

These features are then used by heuristic rules to assign a working `attack_type`.

| Attack Type         | Detection Concept                                                                  |
| ------------------- | ---------------------------------------------------------------------------------- |
| `port_scan`         | Many ports contacted across relatively few destinations with small traffic volumes |
| `network_scan`      | Many destination IPs contacted with relatively few ports and repeated activity     |
| `brute_force`       | Repeated connections toward the same destination and port                          |
| `beaconing`         | Repeated small connections toward the same destination and port                    |
| `data_exfiltration` | Large transfers with high total traffic and limited destination spread             |
| `normal`            | Activity that does not satisfy the current detection rules                         |

The system also generates a `reasons` field containing human-readable explanations for non-normal classifications.

For example:

> Many unique ports contacted by the same source IP

This makes the output more interpretable than simply returning an attack label.

### 3. XGBoost Baseline

The engineered behavioral features are used as input to an `XGBClassifier`.

The current experiment uses:

* 80/20 stratified train/test split
* Per-source behavioral features
* Classification report
* Confusion matrix
* Feature importance analysis

The machine-learning stage is currently **exploratory**.

The present labels are generated from heuristic rules that are based on the same behavioral features provided to the model. As a result, a high classification score does **not** demonstrate that the model can independently detect previously unseen attacks.

In addition, multiple events from the same source IP can appear across both training and testing data, meaning the current split does not represent a strict separation between known and unseen hosts.

The ML results should therefore be interpreted as a **baseline for future experimentation**, rather than as evidence of real-world detection performance.

---

## Current Results

On the current dataset, most events remain classified as `normal`.

The existing rules primarily identify:

* Data-exfiltration-like behavior
* Beaconing-like behavior
* Brute-force-like behavior
* A smaller number of port-scan events

The `network_scan` logic has been implemented but does not currently match the available dataset consistently. This is an area for further rule development and dataset expansion.

---

## Important Limitations

The current dataset limits what can be inferred from the network events.

### No timestamps

Without timestamps, the system cannot establish whether connections occur at regular intervals. Therefore, the current `beaconing` classification is only a behavioral approximation based on repeated small connections to the same destination and port.

### No ground-truth labels

The current attack labels are heuristic labels rather than analyst-verified ground truth.

### Small and synthetic dataset

The dataset is small, synthetic, and simplified compared with real network telemetry. The current results should therefore not be interpreted as production-level IDS performance.

### Potential data leakage

Because features and heuristic labels are generated from the same source behavior, the current ML experiment can learn the rules that generated the labels. Evaluation must eventually use independent labels and source-level train/test separation.

---

## Future Development

Planned improvements include:

* Add timestamps for temporal and session-based analysis
* Improve beaconing detection using connection intervals
* Tune and validate the `network_scan` detection logic
* Tighten and validate suspicion thresholds
* Separate heuristic labels from independently generated ML labels
* Split training and testing data by source host
* Evaluate against a larger and more realistic dataset
* Add additional network features
* Compare multiple machine-learning models
* Move stable detection logic from the notebook into reusable Python modules
* Build a dashboard for interactive threat triage and visualization

---

## Project Structure

```text
Network-Event-Threat-Triage/
├── current.ipynb
├── network_events.csv
├── requirements.txt
└── README.md
```

The primary artifact is currently `current.ipynb`.

---

## How to Run

Install the required dependencies:

```bash
python -m pip install -r requirements.txt
```

Start Jupyter:

```bash
jupyter notebook current.ipynb
```

Run the notebook from the beginning.

The notebook expects `network_events.csv` to be located in the same directory.

---

## Technologies

* Python
* Pandas
* Scikit-learn
* XGBoost
* Jupyter Notebook
* Matplotlib

---

## Project Status

**Status: Active development**

The current version establishes the initial behavioral detection pipeline and an XGBoost baseline. The next stage is improving the dataset, validation methodology, temporal analysis, and separation between heuristic detection and machine-learning evaluation.

The project focuses on **behavior-based network threat detection, explainable security analytics, and establishing a reliable foundation for future machine-learning-based intrusion detection**.
