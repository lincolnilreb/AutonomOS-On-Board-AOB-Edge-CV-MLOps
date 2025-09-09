# AutonomOS-On-Board-AOB-Edge-CV-MLOps (Ongoing)

> ⚠️ **Disclaimer (NDA):** Due to NDA restrictions, this repository provides only a high-level overview of the project. Proprietary data, code, and sensitive details cannot be shared.


## Project Overview

This project delivers a production-oriented **CVOps (MLOps)** pipeline around **RF-DETR** (a modern DETR-family object detector). It focuses on reproducible training on **cloud computing platform** (In this project, I do training on AWS EC2), **quality-gated promotion**, and **portable packaging** (PyTorch → ONNX → TensorRT) for **real-time edge inference** (e.g., NVIDIA Jetson). The approach enables continuous improvement via telemetry-driven **active learning**, while maintaining governance, observability, and rollback safety.

> Scope summary: Train and deploy RF-DETR object detection models with MLOps pipelines, enabling reproducible model development and real-time inference on edge devices.

---

## Planned CVOps (MLOps) Workflow

Roboflow Dataset Version] ──▶ pull (COCO) ──▶ train (EC2 GPU) ──▶ eval (COCO mAP)
                                          │                      │
                                          └────── package (ONNX/TensorRT) ───▶ deploy
                                                                                 │
                                              monitor (latency/quality/drift) ───┘
                                                              │
                                                     active learning ──▶ relabel/version

Stages:
1. **DataOps & Versioning** — Curate/label in Roboflow, freeze a **Version** (train/val/test, preprocessing, augs); run schema/label QA.
2. **Training (EC2 GPU)** — Deterministic configs (AdamW, warmup+cosine, AMP, early-stopping on mAP@50–95); experiment tracking.
3. **Evaluation & Quality Gates** — COCO mAP@50–95 (global + per-class floors), no regression vs current prod, latency budget verified.
4. **Packaging** — Export **ONNX** and optimize to **TensorRT** (FP16/INT8), capture pre/post-proc signature + class map.
5. **Deployment** — Cloud API (FastAPI + ONNX Runtime/TensorRT, k8s/Helm) and/or Edge (Jetson launcher, watchdog, OTA updates).
6. **Observability** — Prometheus/Grafana (p50/p95 latency, throughput, error rate, GPU/CPU util) + model signals (confidence/entropy, agreement vs baseline, drift alerts).
7. **Active Learning** — Mine low-confidence/disagreement frames → relabel → new dataset version → retrain.



## Key Highlights

- **Reproducibility by design**: dataset **Version ID**, config hash, code SHA, and seeds tracked per run.
- **Quality-gated promotion**: hard thresholds on **COCO mAP@50–95** and per-class floors; device-level latency budgets.
- **Edge-ready packaging**: PyTorch → **ONNX → TensorRT** (FP16/INT8) for high FPS on Jetson/RTX.
- **Safe rollouts**: canary/shadow deployments, health checks, instant rollback.
- **Observability**: `/metrics` endpoint for Prometheus; dashboards for latency, throughput, confidence drift, and agreement with baseline.
- **Continuous improvement**: active learning loop integrates production feedback directly into labeling and retraining.
- **Governance & security**: scoped secrets, audit trail for model promotions, image scanning/SBOM, documented data retention/PII handling.


## Tech Stack (Expected Tools & Choices)
Modeling & Training: PyTorch, RF-DETR, COCO format, AMP (mixed precision), AdamW + cosine LR  
DataOps: Roboflow (labeling/versioning), Great Expectations (schema/label QA)  
Experiment Tracking & Registry: Weights & Biases, MLflow (runs, metrics, artifacts, model registry)  
Orchestration / CI: GitHub Actions, Airflow, Prefect, Terraform (infra)  
Serving (Cloud): FastAPI, ONNX Runtime, TensorRT, Docker, Helm, Kubernetes  
Edge: NVIDIA TensorRT (FP16/INT8), Jetson launcher, watchdog, OTA updates  
Observability: Prometheus (metrics), Grafana (dashboards), structured JSON logs  
Cloud Infra: AWS EC2 (GPU) for training, ECR/EKS optional for serving  
Future Enhancements: Triton Inference Server, INT8 calibration pipeline, EvidentlyAI drift monitoring, model cards & registry hardening


## Impact

- **Engineering Efficiency** — One-command, reproducible training and evaluation on EC2; automated gates prevent low-quality releases; portable artifacts across environments.
- **Operational Reliability** — Real-time edge inference with predictable latency; safe canary/shadow rollouts and rapid rollback; clear runbooks and dashboards.
- **Business Value** — Faster iteration cycles and measurable quality improvements via active learning; SLOs (mAP & latency) align model performance with product goals.

