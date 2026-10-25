# Bio-Inspired Temporal Network Intrusion Detection System (AIS-NIDS)

This repository contains the complete empirical experiment notebooks for the research manuscript:
> **"Bio-Inspired Temporal Network Intrusion Detection with Transformer–LSTM Learning, Negative Selection, and Concept-Drift Adaptation"**

The proposed framework combines temporal deep learning (Multi-Head Attention + Bi-LSTM) with biological immune principles (Negative Selection Algorithm) and streaming statistical drift detection (ADWIN) for robust, zero-day-resilient intrusion detection.

---

## 🏛 System Architecture Overview

```
                        [ Ingress Flow Stream ]
                                   │
                     ┌─────────────▼─────────────┐
                     │  Temporal Windowing (T=5) │
                     │  MinMax Normalized (D=42) │
                     └─────────────┬─────────────┘
                                   │
              ┌────────────────────┴────────────────────┐
              ▼                                         ▼
   ┌───────────────────────┐                 ┌───────────────────────┐
   │ Multi-Head Attention  │                 │  Bidirectional LSTM   │
   │  (4 Heads, key_dim=64)│                 │   (128 -> 64 Units)   │
   └──────────┬────────────┘                 └──────────┬────────────┘
              │ (128-dim)                               │ (64-dim)
              └────────────────────┬────────────────────┘
                                   ▼
                       ┌───────────────────────┐
                       │ Feature Fusion (192-d)│
                       │ Dense(64, L2=1e-4)    │
                       └───────────┬───────────┘
                                   │ p(y=1|x)
                                   ▼
                       ┌───────────────────────┐
                       │  Two-Tier Decision    │
                       └───────────┬───────────┘
                                   │
             ┌─────────────────────┴─────────────────────┐
             │ p >= 0.40                                 │ 0.35 <= p < 0.40
             ▼                                           ▼
      [ Flagged Attack ]                    ┌─────────────────────────┐
                                            │ Negative Selection (NSA)│
                                            │  Mature Detectors Check │
                                            └────────────┬────────────┘
                                                         │
                                  ┌──────────────────────┴──────────────────────┐
                                  │ dist <= r (0.12)                            │ dist > r
                                  ▼                                             ▼
                         [ NSA Safety Override ]                               [ Benign ]
                                  │                                             │
                                  └──────────────────────┬──────────────────────┘
                                                         │
                                            ┌────────────▼────────────┐
                                            │ ADWIN Streaming Drift   │
                                            │ Tracking (delta=0.002)  │
                                            └─────────────────────────┘
```

---

## 📓 Experiment Notebooks

All experimental pipelines are organized into self-contained, sequentially numbered Jupyter Notebooks with full documentation, code, and pre-rendered outputs:

| Notebook | Focus Area | Key Metrics / Artifacts |
|---|---|---|
| [`01_Data_Preprocessing_and_Temporal_Windowing.ipynb`](01_Data_Preprocessing_and_Temporal_Windowing.ipynb) | UNSW-NB15 cleaning, categorical encoding, zero-leakage MinMax scaling, and temporal sequence generation ($T=5$, $D=42$). | Feature heatmaps, correlation matrix, class distribution, balanced weights ($w_0=0.8936, w_1=1.1352$). |
| [`02_Transformer_LSTM_Hybrid_Architecture_and_Training.ipynb`](02_Transformer_LSTM_Hybrid_Architecture_and_Training.ipynb) | Dual-branch neural network construction, baseline implementations (Dual-LSTM, Trans-LSTM), Adam optimization, and 30-epoch training. | Model convergence, learning rate schedule trajectory, baseline performance comparisons. |
| [`03_Bio_Inspired_Negative_Selection_Algorithm.ipynb`](03_Bio_Inspired_Negative_Selection_Algorithm.ipynb) | Self-space extraction, random candidate screening, detector maturation ($r=0.12$, $N=1,500$), and two-tier safety-net override. | 2D PCA projection of self vs mature detectors, dual confusion matrices, radius sensitivity sweep ($r \in [0.05, 0.20]$). |
| [`04_Streaming_Concept_Drift_Adaptation_ADWIN.ipynb`](04_Streaming_Concept_Drift_Adaptation_ADWIN.ipynb) | Streaming simulation, online prediction error tracking with River ADWIN ($\delta=0.002$), and dual-tier adaptation. | Confirmed 63 concept drift events, rolling accuracy timeline ($W=200$), associative memory recall (0.85 ms) vs retraining (450 ms). |
| [`05_Zero_Day_LOAFO_Evaluation.ipynb`](05_Zero_Day_LOAFO_Evaluation.ipynb) | Leave-One-Attack-Family-Out (LOAFO) benchmark across all 9 attack categories in UNSW-NB15. | **32.39% zero-day macro-F1** (+6.68% recovery over neural baseline), recall vs attack support volume analysis. |
| [`06_Ablation_Study_and_Threshold_Sensitivity.ipynb`](06_Ablation_Study_and_Threshold_Sensitivity.ipynb) | 7-stage ablation progression (C1 $\to$ C7), decision threshold sweep ($\theta \in [0.20, 0.80]$), and 5-seed confidence intervals. | Dual-axis waterfall (F1 vs FN reduction), ROC curve (AUC=0.9803), 5-axis operational radar, 95% CI validation. |
| [`07_CICIDS2017_Cross_Dataset_Evaluation.ipynb`](07_CICIDS2017_Cross_Dataset_Evaluation.ipynb) | External cross-dataset benchmark with 4-point data integrity audit (deduplication, identifier exclusion, session split). | **99.71% Accuracy**, **99.79% F1**, **0.51% FPR** ($TN=29,847, FP=153, FN=165, TP=77,848$), attention spot-checks. |
| [`08_Inference_Latency_and_Throughput_Benchmark.ipynb`](08_Inference_Latency_and_Throughput_Benchmark.ipynb) | Commodity CPU profiling (Intel Core i7 @ 1.80 GHz) measuring execution latency and batch throughput scaling. | **0.235 ms total latency** (ADWIN: 12 μs, NSA: 41 μs, Model: 182 μs), **18,450 flows/s** batched throughput (3.7x 1 Gbps line-rate demand). |

---

## 📊 Summary of Headline Benchmark Results

### UNSW-NB15 Closed-World & Zero-Day Benchmark
- **Accuracy**: 92.50%
- **Precision**: 95.90%
- **Recall (Detection Rate)**: 90.24%
- **F1-Score**: 92.98%
- **False Positive Rate (FPR)**: 4.73%
- **ROC-AUC**: 0.9803
- **LOAFO Zero-Day Macro-F1**: 32.39% (+6.68% recovery over baseline neural models)

### CICIDS2017 Audited Cross-Dataset Benchmark
- **Accuracy**: 99.71%
- **Precision**: 99.80%
- **Recall**: 99.79%
- **F1-Score**: 99.79%
- **False Positive Rate (FPR)**: 0.51%
- **ROC-AUC**: 0.9998
- **Confusion Matrix**: $TN = 29,847$, $FP = 153$, $FN = 165$, $TP = 77,848$

### Commodity Hardware Profiling (CPU-Only)
- **Single-Flow Execution Latency**: 0.235 ms (235.0 μs)
- **Streaming Batch Throughput ($B=64$)**: 18,450 flows/second
- **Saturated Peak Throughput ($B=128$)**: 19,200 flows/second
- **Associative Immune Memory Recall**: 0.85 ms
- **Adaptive Full Retraining**: 450 ms

---

## ⚙️ Installation & Environment Setup

### 1. Clone the Repository
```bash
git clone https://github.com/Nideesh-CG/AIS.git
cd AIS
```

### 2. Create and Activate Virtual Environment
```bash
# Using Python venv
python -m venv venv
# On Windows:
venv\Scripts\activate
# On Linux/macOS:
source venv/bin/activate
```

### 3. Install Dependencies
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

---

## 🚀 Running the Notebooks

Launch Jupyter Lab or Jupyter Notebook:
```bash
jupyter lab
```
Run the notebooks in numerical order (`01` through `08`) to follow the complete experimental pipeline from raw data preprocessing to commodity hardware latency profiling.

---

## 🔒 Data Integrity & Reproducibility Protocols
1. **Zero-Leakage Guarantee**: MinMax scaling parameters and label encoders are fitted exclusively on training sets.
2. **Deterministic Seed Control**: Random seeds (`seed=42`, `tf.random.set_seed(42)`) are enforced across all stochastic operations.
3. **Identifier Sanitization**: Network identifiers (`Flow ID`, `Source IP`, `Destination IP`, `Source Port`, `Destination Port`, `Timestamp`) are strictly excluded prior to modeling to prevent shortcut memorization.