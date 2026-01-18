# ML-Based Drift Detection on DNS Telemetry

## Project Summary
This project studies how concept drift manifests in an ML-based DNS detection system and how that drift impacts security outcomes in a SOC workflow. Using daily DNS telemetry, I trained a classifier on early data, monitored performance over time, identified drift, and evaluated retraining behavior after the environment changed.

The emphasis is on operational signals: when drift becomes visible, how it affects attack detection, and what retraining does and does not recover.

## Data and Setup
The data used in this project is derived from https://data.mendeley.com/datasets/c4n7fckkz3/3, DOI:10.17632/c4n7fckkz3.3.

The dataset consists of daily DNS traffic files (day1 through day10), where each row represents a DNS request labeled as benign or malicious. Features describe structural properties of the domain name, including:
- entropy
- digits ratio
- uppercase ratio
- subdomain count

The model is trained on early days assumed to represent a stable environment and then evaluated sequentially on later days, mirroring how a production detector would be monitored.

## Baseline Behavior
When trained on pre-drift data (days 1–7), the classifier performs near perfectly:
- overall accuracy remains in the high nineties,
- recall on malicious DNS traffic stays consistently high,
- daily accuracy values fall within the Wilson confidence interval derived from training data.

This establishes a stable baseline for comparison.

## Drift Detection
Drift becomes apparent **between day 7 and day 8**.

On day 8:
- overall accuracy drops sharply from the high nineties to around **0.9**,
- recall on malicious traffic collapses to **nearly zero**,
- accuracy values for days 8–10 fall well below the lower bound of the Wilson confidence interval.

This combination indicates a distributional shift rather than random noise. The model is still classifying many benign samples correctly, but it is no longer recognizing attacks.

## Suspected Poisoning Event
Day 8 is the most likely candidate for a backdoor or poisoning event.

Evidence from the notebook includes:
- a sudden divergence in feature distributions starting on day 8,
- attack samples clustering differently from earlier days,
- sustained impact across days 9 and 10.

Notably, the failure mode is asymmetric: accuracy remains relatively high while attack recall collapses. This pattern is consistent with a targeted integrity failure rather than general model degradation.

## Retraining and Recovery
After retraining on day 8 data:
- overall accuracy does not return to pre-drift levels,
- however, recall on malicious traffic recovers strongly on days 9 and 10.

This is an important operational result. The model adapts to the new environment and regains its ability to detect attacks, even though the data distribution is noisier and less separable than before.

The lower post-retraining accuracy reflects a harder problem space, not a broken model.

## Security Interpretation
This experiment highlights several realities of ML in security:
- accuracy alone is a weak metric for detectors,
- drift can silently disable attack detection before obvious failures appear,
- retraining restores capability but does not rewind the environment.

In a SOC setting, the collapse of recall on day 8 is the signal that matters most, not the absolute accuracy number.

## Drift Without Labels
If labels were unavailable, drift could still be detected by monitoring feature distributions over time. In this dataset, features such as entropy, digits ratio, and subdomain count remain stable during days 1–7 and shift noticeably starting on day 8.

Tracking statistical distance from baseline feature distributions provides an early warning that the model’s assumptions no longer hold.

## MLOps Security Controls
Based on the observed behavior, additional safeguards are needed beyond the model itself:
- data provenance checks during ingestion,
- model integrity verification during packaging and release,
- dependency freezing and supply chain monitoring in the runtime environment.

These controls reduce the risk of silent failures caused by poisoned data, tampered models, or compromised libraries.

## How to Run
1. Place daily DNS CSV files in the `data/` directory.
2. Open the notebook and execute cells sequentially.
3. The notebook plots daily accuracy, recall, and confidence bounds, and performs retraining when drift is detected.

## Notes
This repository is intended for defensive ML security research. The implementation is simplified to make drift behavior, retraining effects, and security tradeoffs easy to inspect.
