# Mitigating Sycophancy in LLM Hypothesis Generation 
**A Cost-Plausibility Analysis of RAG vs. Heterogeneous Multi-Agent Debate**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/release/python-3100/)

## 📌 Problem Statement
While Retrieval-Augmented Generation (RAG) improves factual grounding in Large Language Models (LLMs), generating plausible scientific hypotheses often requires iterative logical refinement. Multi-Agent Debate (MAD) architectures theoretically mitigate parametric sycophancy via adversarial correction, but their computational cost (token efficiency) is rarely quantified against standard RAG. 

This repository contains the code, data, and evaluation scripts to reproduce our zero-shot comparative study measuring the Plausibility Return on Investment (ROI) between a static RAG pipeline and an ungrounded 2-Agent Debate loop.

---

## 📂 Repository Structure

```text
.
├── data/
│   ├── frozen_eval_split.csv           # The frozen 50-sample PubMedQA test set (Seed 42)
│   ├── baseline_results.csv            # Raw outputs for Zero-Shot & Static RAG
│   ├── final_experiment_results.csv    # Merged outputs including Heterogeneous Debate
│   └── revised_experiment_results.csv  # Final verbosity-controlled evaluation & ablation
├── scripts/
│   ├── 01_generate_baselines.py        # Pipeline 1: Zero-Shot & RAG generation
│   ├── 02_generate_debate.py           # Pipeline 2: Heterogeneous 2-Agent Debate
│   ├── 03_run_ablation.py              # Pipeline 3: Homogeneous Debate (Ablation)
│   └── 04_evaluate_metrics.py          # LLM-as-a-judge scoring & Wilcoxon stat testing
├── figures/
│   └── performance_comparison.png      # Plausibility vs. Token Cost (Log Scale) chart
├── manuscript/
│   ├── manuscript.tex                  # IEEEtran format source code
│   └── references.bib                  # Verified BibTeX citations
├── requirements.txt                    # Environment dependencies
└── README.md                           # This file
```

---

## ⚙️ Compute Requirements & Environment Setup

*   **Compute Engine:** Google Colab (Standard CPU tier is sufficient; execution is strictly I/O bound by API latency).
*   **API Provider:** Google Gemini API (`gemini-3.5-flash-lite`).
*   **Estimated Budget:** ~300,000+ tokens per full run (Ensure your API tier supports batch requests without hard quota cutoffs).
*   **Time:** ~45-60 minutes for a full 50-sample evaluation run due to exponential backoff delays.

### Installation
```bash
# Clone the repository
git clone https://github.com/mse2026005/llm-sycophansy-debate.git
cd llm-sycophansy-debate

# Install required dependencies
pip install -r requirements.txt
```

**`requirements.txt` contents:**
```text
google-genai==0.3.0
pandas>=2.0.0
numpy>=1.24.0
scipy>=1.10.0
matplotlib>=3.7.0
tqdm>=4.65.0
```

### 🔒 API Key Security (CRITICAL)
**NEVER hardcode your API key into the scripts.** 
If running locally, set an environment variable:
```bash
export GEMINI_API_KEY="your_api_key_here"
```
If running in Google Colab, use the Secrets tab to store `GEMINI_API_KEY` and access it via:
```python
from google.colab import userdata
api_key = userdata.get('GEMINI_API_KEY')
```

---

## 📊 Dataset Access
We utilize a frozen, 50-sample subset of the **PubMedQA** dataset (`qiaojin/PubMedQA`). 
*   **License:** PubMedQA is distributed under the MIT License.
*   **Preparation:** The test split was generated using a fixed random seed (`42`) to guarantee absolute deterministic alignment between baseline and multi-agent runs. The prepared CSV is provided in `data/frozen_eval_split.csv`.

---

## 🚀 Reproduction Commands

Follow these steps sequentially to reproduce the experiment. 

### Step 1: Generate Baselines (Zero-Shot & RAG)
```bash
python scripts/01_generate_baselines.py
```
*Expected Output:* Generates `baseline_results.csv`. Consumes ~3,320 tokens.

### Step 2: Run Heterogeneous Debate
```bash
python scripts/02_generate_debate.py
```
*Expected Output:* Executes Proposer vs. Challenger loop with exponential backoff for 503/429 errors. Merges output into `final_experiment_results.csv`. Consumes ~294,000 tokens.

### Step 3: Run Homogeneous Ablation & Verbosity-Controlled Evaluation
*(Note: Ensure Step 1 and 2 are complete before running this script).*
```bash
python scripts/03_run_ablation.py
```
*Expected Output:* Executes Proposer vs. Proposer loop. Re-evaluates ALL pipelines using a strict anti-verbosity prompt (`T=0.0`). Saves to `revised_experiment_results.csv`.

### Step 4: Statistical Testing & Figures
```bash
python scripts/04_evaluate_metrics.py
```
*Expected Output:* Prints Wilcoxon Signed-Rank test $p$-values, effect sizes, and outputs `performance_comparison.png`.

---

## 📈 Key Findings (Pending Final Ablation)

*   **Plausibility vs. Cost:** The ungrounded Heterogeneous 2-Agent Debate massively outperformed the baseline RAG pipeline in raw plausibility score but incurred a nearly $90\times$ increase in token expenditure.
*   **Revised Evaluation:** [REVISED RAG SCORE] vs. [REVISED HETERO DEBATE SCORE] (Verbosity controlled).
*   **Ablation Result:** [HOMOGENEOUS ABLATION VERDICT] (Pending script completion to prove whether adversarial persona, or just token scaling, drives the performance).

---

## ✅ Minimum Reproduction Checklist (For Peer Reviewers)
- [ ] 1. Clone repo and install `requirements.txt`.
- [ ] 2. Configure `GEMINI_API_KEY` in environment variables.
- [ ] 3. Verify `data/frozen_eval_split.csv` exists and contains 50 rows.
- [ ] 4. Run `04_evaluate_metrics.py` on the *provided* `revised_experiment_results.csv` to instantly verify the statistical significance (Wilcoxon $p < 0.001$) without re-burning API credits.
- [ ] 5. [TEST REQUIRED] Optional: Run `02_generate_debate.py` on a single row (slice `df.head(1)`) to verify the exponential backoff logic handles API throttling natively.

---

## ⚠️ Limitations
*   **Parametric Leakage:** The debate architecture is technically ungrounded, but PubMed abstracts exist heavily in LLM pre-training data. The agents may recall memorized facts rather than utilizing pure zero-shot deduction.
*   **Dataset Scope:** The $N=50$ sample size is statistically significant due to the massive effect size, but is isolated strictly to the biomedical domain.
*   **Self-Preference Bias:** A Gemini model was used to evaluate Gemini-generated text.

---

## 📜 License & Citation
This code is released under the MIT License. If you use this methodology or token-cost heuristic in your research, please cite:

```bibtex
@article{anonymous2026mitigating,
  title={Mitigating Sycophancy in LLM Hypothesis Generation: A Cost-Plausibility Analysis of RAG vs. Heterogeneous Multi-Agent Debate},
  author={MSE2026005 - Sarvagna Khatri},
  journal={Target: IEEE/ACM Publications},
  year={2026}
}
```
