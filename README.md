# 🧠 StrokeGuard AI

**An end-to-end, clinically-framed machine learning web app that estimates stroke risk from routine patient data.**

*Built by [HayMedics Academy](https://github.com/HayMedics) — Data | Research | Innovation*

![Python](https://img.shields.io/badge/Python-3.11-1B2A6B)
![scikit-learn](https://img.shields.io/badge/scikit--learn-ML-F5A623)
![Streamlit](https://img.shields.io/badge/Streamlit-Deployed-1B2A6B)
![License](https://img.shields.io/badge/License-MIT-F5A623)

🔗 **Live app:** https://strokeguard-ai-haymedics.streamlit.app
📂 **Source:** https://github.com/HayMedics/strokeguard-ai

---

## 📋 Overview

StrokeGuard AI is a stroke-risk screening tool that takes routinely-collected clinical
inputs — age, hypertension and heart-disease history, glucose, BMI, smoking status and
more — and returns a calibrated, explainable estimate of stroke risk. It was built to
demonstrate **responsible, transparent machine learning in a healthcare setting**,
following the spirit of the **TRIPOD-AI** reporting standard for clinical prediction models.

The project covers the full lifecycle: data exploration, leak-free modelling, honest
validation, explainability, and a polished, branded web application deployed to the public.

---

## ✨ Key features

| Feature | What it does |
|---|---|
| 🩺 **Risk assessment** | Calibrated stroke-risk score with a custom radial gauge and clear elevated/lower flag |
| 🔍 **Explainability** | SHAP-style (perturbation-based) feature contributions showing *why* a score was given |
| 📈 **Confidence intervals** | Monte-Carlo simulation of measurement variation gives a 95% range, not just a point estimate |
| 🎛️ **What-If simulator** | Adjust factors and watch the risk recalculate live, with the change vs baseline |
| 📊 **Population comparison** | Plots the patient against the 5,110-patient reference distribution |
| 📁 **Batch screening** | Upload a CSV of patients, score them all at once, download results |
| 🕘 **Session history** | Tracks the last 10 screenings in the session |
| 🧪 **Model validation** | Live sensitivity / specificity / PPV / NPV and an internal-vs-external AUC comparison |
| 📄 **PDF reports** | One-click branded clinical report (ReportLab) |

---

## 🏥 Clinical framing

Stroke screening has an **asymmetric cost**: missing a real stroke patient (a false
negative) is far more dangerous than a false alarm (a false positive). StrokeGuard AI
makes this explicit — its decision threshold is chosen by a **cost-based rule that
weights a false negative 10× a false positive**, prioritising sensitivity, as a screening
tool should.

---

## 📊 Dataset

- **Source:** [Kaggle Stroke Prediction Dataset](https://www.kaggle.com/datasets/fedesoriano/stroke-prediction-dataset) (fedesoriano)
- **Size:** 5,110 patients, 11 predictors + binary `stroke` outcome
- **Key challenge — severe class imbalance:** only **~4.9%** of patients had a stroke
- **Data-quality notes:** `bmi` has ~4% missing values (median-imputed); `smoking_status`
  has ~30% "Unknown" entries, kept as a valid category rather than discarded

---

## 🧬 Model & methodology

- **Pipeline:** a single, leak-free scikit-learn `Pipeline` → feature engineering →
  median/most-frequent imputation → `RobustScaler` → one-hot encoding → classifier.
  All preprocessing learns from training data only, so the test set can never leak in.
- **Feature engineering:** clinical interaction terms (`age × hypertension`,
  `glucose × bmi`), flags (`is_diabetic`, `is_obese`, `is_elderly`) and a 0–5
  `vascular_risk_score`.
- **Imbalance handling:** `class_weight="balanced"` so the rare stroke cases are not ignored.
- **Calibration:** `CalibratedClassifierCV` so a "20% risk" really means roughly 20-in-100.
- **Threshold:** cost-based (false negative weighted 10×), giving an operating threshold of ~0.09.
- **Reporting:** documented in the spirit of **TRIPOD-AI** — transparent methods, honest
  validation, and explicit limitations.

### Performance (held-out test set)

| Metric | Value |
|---|---|
| ROC-AUC | **0.84** |
| Sensitivity / Recall | **0.72** |
| Specificity | 0.83 |
| NPV | 0.98 |
| PPV / Precision | 0.18 |

> These are honest, held-out numbers. The high NPV means a "lower-risk" result is fairly
> trustworthy; the low PPV is the expected trade-off for catching more true cases in a
> rare-outcome screening setting.

---

## 🛠️ Tech stack

`Python` · `scikit-learn` · `pandas` · `numpy` · `matplotlib` · `ReportLab` · `Streamlit`

**Architecture note:** the app **trains its model on startup** from the dataset rather than
loading a saved `.pkl`. This deliberately avoids the version-mismatch crashes that happen
when a model pickled with one library version is loaded under another — a common
deployment pitfall. Training on 5,110 rows takes about a second on a cold start.

---

## 📁 Project structure

```
strokeguard-ai/
├── app.py                  # the full Streamlit application
├── stroke_data.csv         # the dataset the app trains on
├── requirements.txt        # Python dependencies
├── runtime.txt             # pins Python 3.11
├── .streamlit/
│   └── config.toml         # navy/gold theme
└── README.md
```

---

## ▶️ Run it locally

```bash
# 1. Clone the repo
git clone https://github.com/HayMedics/strokeguard-ai.git
cd strokeguard-ai

# 2. Create and activate an environment (conda example)
conda create -n strokeguard python=3.11 -y
conda activate strokeguard

# 3. Install dependencies
pip install -r requirements.txt

# 4. Launch
streamlit run app.py
```

The app opens automatically in your browser at `http://localhost:8501`.

---

## ☁️ Deploy on Streamlit Community Cloud

1. Push these files to a public GitHub repo (top level): `app.py`, `requirements.txt`,
   `runtime.txt`, `stroke_data.csv`, and `.streamlit/config.toml`.
2. Go to [share.streamlit.io](https://share.streamlit.io), sign in with GitHub.
3. **Create app** → select the repo, branch `main`, main file `app.py` → **Deploy**.

The first load shows *"Preparing the model…"* for a few seconds while it trains — this only
happens on a cold start.

---

## ⚠️ Limitations & disclaimer

- **This is an educational screening tool — not a diagnosis and not a medical device.**
- It lacks the strongest real-world stroke predictors (brain imaging, atrial fibrillation,
  medication history, clotting factors), so accuracy has a natural ceiling.
- Trained on a single dataset with no ethnicity data, so fairness across populations cannot
  be guaranteed and it must not be used for real clinical decisions.
- Final judgement always rests with the treating clinician.

---

## 📚 References

- World Health Organization — Stroke fact sheets
- American Heart Association / American Stroke Association — Guideline for the Prevention of Stroke
- Feigin V.L. et al. (2022). *World Stroke Organization (WSO): Global Stroke Fact Sheet.*
- Collins G.S. et al. (2024). *TRIPOD+AI statement: reporting guideline for clinical prediction models.*
- Kaggle Stroke Prediction Dataset (fedesoriano)

---

## 📄 License

Released under the **MIT License**.

---

<p align="center"><b>HayMedics Academy</b> · Data | Research | Innovation</p>
