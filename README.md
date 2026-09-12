# JARVIS-IDS

## Intelligent Network Intrusion Detection System

JARVIS-IDS is a Python-based network intrusion detection project that analyzes network traffic, extracts behavioral features, identifies suspicious activity, classifies potential attack patterns, and provides human-readable explanations for detected threats.

The project explores how network events can be transformed into meaningful security information:

**What looks suspicious? What type of attack might it represent? And why was it flagged?**

JARVIS-IDS is currently an **active prototype** focused on developing and evaluating a behavior-based intrusion detection pipeline and establishing a foundation for future machine-learning development.

---

## Problem

Raw network traffic contains large amounts of activity that can be difficult to analyze manually. Normal connections can exist alongside behaviors associated with port scanning, network scanning, brute-force attempts, beaconing, and data exfiltration.

JARVIS-IDS aims to automate part of this analysis by:

* Detecting unusual network behavior
* Aggregating activity by source IP
* Extracting behavioral network features
* Identifying potential attack patterns
* Assigning working threat classifications
* Explaining why activity was classified as suspicious
* Using engineered features for machine-learning experiments

---

## Detection Pipeline

The current implementation processes `network_events.csv`, which contains approximately 1,000 network events with the following fields:

* `src_ip`
* `dst_ip`
* `port`
* `bytes`

The current dataset does not contain timestamps, protocol information, payload data, or independently verified ground-truth labels. These limitations are important when interpreting the current results.

### 1. Suspicious Behavior Detection

The system analyzes network activity using behavioral indicators such as:

* Number of unique ports contacted by a source IP
* Number of unique destination IPs contacted
* Total bytes sent by a source
* Number of destinations contacted through the same port

These indicators are converted into suspicion flags and combined into a behavioral suspicion score.

The resulting score is mapped to a working risk level:

`normal → low → medium → high → critical`

This layer is designed as a **threat-triage mechanism**, rather than a definitive security verdict.

### 2. Behavioral Feature Engineering

JARVIS-IDS aggregates network activity by `src_ip` and generates behavioral features including:

* Event frequency
* Total bytes
* Average bytes per event
* Maximum bytes per event
* Destination diversity
* Port diversity

These features describe the behavior of each source and provide structured input for subsequent classification.

### 3. Attack Classification

Heuristic detection rules are used to assign a working `attack_type` to network activity.

The current classifications include:

| Attack Type         | Detection Concept                                              |
| ------------------- | -------------------------------------------------------------- |
| `port_scan`         | Many unique ports contacted by the same source                 |
| `network_scan`      | Many unique destination IPs contacted                          |
| `brute_force`       | Repeated attempts toward a destination                         |
| `beaconing`         | Repeated small connections to the same destination and port    |
| `data_exfiltration` | Extremely large data transfers and high total outbound traffic |
| `normal`            | Activity that does not satisfy the current attack rules        |

The system also generates human-readable detection reasons.

For example:

> Many unique ports contacted by the same source IP

This provides context behind a classification instead of returning only an attack label.

---

## Machine Learning

The engineered behavioral features are used in an **XGBoost classification experiment**.

The current experiment includes:

* `XGBClassifier`
* 80/20 stratified train/test split
* Classification report
* Confusion matrix
* Feature importance analysis

The machine-learning component is currently **experimental**.

The current attack labels are generated using heuristic rules based on behavioral characteristics. Because those same behavioral characteristics are also provided to the machine-learning model, a high classification score does not demonstrate independent real-world intrusion detection capability.

Additionally, multiple events from the same source IP may appear in both the training and testing sets. This means the current evaluation does not provide strict separation between previously observed and unseen hosts.

The current ML results should therefore be interpreted as a **baseline experiment**, not as production-level IDS performance.

---

## Current Results

The current dataset is predominantly classified as `normal`.

The implemented detection logic identifies behavior associated with:

* Data-exfiltration-like activity
* Beaconing-like activity
* Brute-force-like activity
* Port scanning

The `network_scan` detection logic is implemented but does not consistently match the current dataset. Improving this detection logic and expanding the dataset are planned areas of development.

---

## Explainability

One of the goals of JARVIS-IDS is to make detections easier to understand.

Instead of returning only:

`attack_type = port_scan`

the system can provide an explanation such as:

`Many unique ports contacted by the same source IP`

This approach makes the detection pipeline more useful for **security analysis and threat triage**, where understanding the reason behind an alert is important.

---

## Limitations

### No timestamps

The current dataset does not contain timestamps. As a result, the system cannot determine whether connections occur at regular time intervals.

The current `beaconing` classification is therefore a behavioral approximation based on repeated small connections to the same destination and port.

### No independently verified ground truth

The current attack classifications are generated by heuristic rules rather than analyst-verified labels.

### Synthetic dataset

The dataset is small and synthetic compared with real-world network telemetry.

The current results should therefore not be interpreted as production-level intrusion detection performance.

### Potential data leakage

The current ML experiment uses features derived from the same behavioral information used to generate the heuristic labels.

This means the model can effectively learn the rules that generated the labels.

Future evaluation should use independently generated labels and source-level separation between training and testing data.

---

## Future Development

Planned improvements include:

* Add timestamps for temporal and session-based analysis
* Improve beaconing detection using connection intervals
* Improve and validate network-scan detection
* Tune and validate detection thresholds
* Expand the network feature set
* Introduce independently generated or verified labels
* Split training and testing data by source host
* Evaluate against larger and more realistic network datasets
* Compare multiple machine-learning models
* Move stable detection logic into reusable Python modules
* Develop an interactive dashboard for threat triage and visualization

---

## Project Structure

```text
JARVIS-IDS/
├── JARVIS-IDS.ipynb
├── network_events.csv
├── requirements.txt
└── README.md
```

The primary artifact is currently `JARVIS-IDS.ipynb`.

---

## How to Run

Install the required dependencies:

```bash
python -m pip install -r requirements.txt
```

Launch the notebook:

```bash
jupyter notebook JARVIS-IDS.ipynb
```

Run the notebook from the beginning.

The notebook expects `network_events.csv` to be located in the same directory.

---

## Technologies

* Python
* Pandas
* Scikit-learn
* XGBoost
* Matplotlib
* Jupyter Notebook

---

## Project Status

**Status: Active Development**

JARVIS-IDS currently establishes a behavior-based network threat detection pipeline, explainable attack classification, engineered behavioral features, and an XGBoost baseline.

The next stage focuses on improving the dataset, validation methodology, temporal analysis, and separation between heuristic detection and machine-learning evaluation.

The long-term goal is to develop JARVIS-IDS into a more robust **behavior-based network intrusion detection and threat-triage system** capable of combining explainable detection logic with machine-learning techniques.
